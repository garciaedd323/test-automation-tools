# Git worktree para correr Serenity en paralelo sin choques

## La analogía general

Imaginar una biblioteca universitaria con un único catálogo maestro (todo el historial de commits) pero, por defecto, un solo escritorio físico de trabajo (un solo checkout en disco) donde cualquiera que quiera consultar o modificar algo tiene que sentarse. Si dos personas intentan usar ese mismo escritorio a la vez — una archivando papeles en un cajón mientras la otra intenta vaciar ese mismo cajón para poner los suyos — se estorban entre sí, aunque cada una esté trabajando en "su propio libro".

Un **git worktree** es pedirle a la biblioteca una **sala de lectura satélite por persona**: mismo catálogo maestro, mismo libro (mismo commit), pero cada quien con su propio escritorio físico y su propio cajón de notas. Ya no hay forma de que se estorben, porque ni siquiera comparten mueble.

Esta nota retoma justo donde quedó [maxParallelForks y junit-platform.properties](maxparallelforks-junit-platform-properties.md): ese documento resuelve el paralelismo **dentro de un mismo checkout** (varios forks o hilos compartiendo la misma carpeta de proyecto). Esta nota cubre qué pasa cuando eso no alcanza, y por qué la solución real terminó siendo separar el escritorio físico, no repartir mejor el trabajo sobre el mismo escritorio.

---

## 1. Por qué aislar `maxParallelForks` y `serenity.outputDirectory` no fue suficiente

El primer intento fue correr dos invocaciones separadas de `gradlew` (una por módulo) contra el **mismo checkout**, cada una con `-Dserenity.outputDirectory=target/site/serenity-<modulo>` distinto, para que cada módulo escribiera su propio reporte. Sobre el papel, cada invocación es un proceso independiente — no debería haber problema. En la práctica, aparecieron tres choques sucesivos, cada uno destapando el siguiente apenas se resolvía el anterior.

### 1.1 Colisión 1: la carpeta binaria de resultados de Gradle

Gradle guarda los resultados de la tarea `test` en una ruta fija por proyecto: `build/test-results/test/binary/output.bin`. Esa ruta **no** se ve afectada por `-Dserenity.outputDirectory` — es un mecanismo interno de Gradle, no de Serenity. Cuando el segundo módulo arrancó su propia tarea `:test`, intentó limpiar esa carpeta antes de escribir — pero el primer módulo la tenía abierta, escribiendo sus propios resultados en ese mismo instante:

```
java.io.IOException: Unable to delete directory 'build\test-results\test\binary'
Failed to delete some children. This might happen because a process has files open
```

Se corrigió agregando al `test {}` de `build.gradle` tres propiedades opcionales, con el mismo valor de siempre como respaldo si no se pasan:

```groovy
test {
    // ...

    binaryResultsDirectory = file(System.getProperty("test.binaryResultsDir", "$buildDir/test-results/test/binary"))
    reports {
        junitXml.outputLocation = file(System.getProperty("test.junitXmlDir", "$buildDir/test-results/test"))
        html.outputLocation = file(System.getProperty("test.htmlReportDir", "$buildDir/reports/tests/test"))
    }
}
```

> **Analogía:** es como dos secretarias compartiendo el mismo cajón físico de formularios — una está archivando el suyo justo cuando la otra intenta vaciar el cajón para poner los suyos. No importa que cada una tenga su propio escritorio de trabajo si el cajón de archivo final sigue siendo uno solo.

### 1.2 Colisión 2: el `aggregate` de Serenity no respeta el `outputDirectory` por invocación

Con la colisión 1 resuelta, apareció una más sutil. Los archivos JSON **por escenario** sí se escribieron en la carpeta correcta durante la tarea `:test` (se confirmó revisando los archivos reales, uno por uno) — pero la tarea `:aggregate` (la que arma el `index.html` final) de uno de los dos módulos escribió su reporte en la carpeta genérica `target/site/serenity/`, **ignorando** por completo el `-Dserenity.outputDirectory` que sí había respetado la tarea `:test` segundos antes.

> **Analogía:** es como un bibliotecario que sí toma el pedido correcto de cada persona durante el día ("archívame esto en tu carpeta"), pero al final de la jornada arma el resumen del día completo siempre en la misma carpeta central, sin importar quién pidió qué.

### 1.3 Colisión 3: Gradle trató `:test` como si no hubiera nada que correr

En esa misma corrida, el otro módulo terminó con `BUILD SUCCESSFUL` — pero su `summary.txt` mostraba `Test Cases: 0`. Gradle, al ver dos invocaciones compitiendo por el mismo directorio de proyecto, aparentemente trató esa tarea como ya satisfecha (sin ejecutar ningún escenario real) en vez de correrla de verdad.

> **Analogía:** es como un empleado que, al ver una carpeta que a simple vista no cambió desde la última revisión, simplemente sella "todo en orden" sin abrirla — sin saber que otra persona la estaba usando en ese preciso momento.

### La conclusión, tras las tres

Cada capa de estado compartido que se aislaba destapaba otra: la carpeta binaria de Gradle, después la carpeta de agregación de Serenity, después la caché de "up-to-date" del propio Gradle. El patrón quedó claro: **Gradle y el plugin de Serenity no están pensados para que dos invocaciones corran al mismo tiempo contra el mismo directorio de proyecto**. Parchar una ruta a la vez no era un camino que fuera a terminar — hacía falta separar el escritorio físico completo, no repartir mejor el trabajo sobre el mismo escritorio.

---

## 2. Qué es un git worktree

Un worktree es una **segunda carpeta física**, en otro lugar del disco, con los archivos de un commit ya puestos ahí — pero sigue siendo el mismo repositorio por dentro, no uno nuevo. Se crea así, desde la raíz del repositorio real:

```bash
git worktree add --detach ../ProyectoWebQA-autoventa HEAD
```

`--detach` significa "no ates esta copia a ninguna rama, deja el commit suelto ahí" (lo que Git llama *detached HEAD*). Es necesario porque Git no permite tener la **misma rama** activa en dos worktrees a la vez — para evitar el problema de "¿cuál de las dos copias es la versión real de la rama si hago un commit desde ahí?". Como estos worktrees son solo para **ejecutar** tests, nunca para comitear, el detached HEAD evita esa restricción sin ningún costo real.

Para ver los que existen, o quitar uno cuando ya no hace falta:

```bash
git worktree list
git worktree remove ../ProyectoWebQA-autoventa
```

> **Analogía:** seguir con la sala de lectura satélite — `git worktree add` es pedirle a la biblioteca "ábreme una sala nueva, con una fotocopia de este libro exacto, en otro edificio". `--detach` es aclarar "esta fotocopia no es la copia oficial de préstamo de nadie, es solo para consulta" — así la biblioteca no se confunde pensando que hay dos personas reclamando el mismo préstamo oficial.

---

## 3. Qué se comparte y qué no — y por qué no se actualizan solos

Lo único que comparten los worktrees es el **historial de Git** (`.git`: commits, ramas, objetos — referenciado, no duplicado). Lo que **no** comparten es el contenido de los archivos de trabajo en disco: cada carpeta tiene su propia copia física, congelada en el commit con el que se creó.

Esto tiene una consecuencia importante, fácil de asumir mal: **los worktrees no se actualizan solos.**

- Un cambio **sin commitear** en la carpeta original nunca llega a ningún worktree, bajo ninguna circunstancia.
- Un **commit nuevo** en la carpeta original sí queda guardado en el historial compartido — pero cada worktree solo actualiza sus archivos en disco cuando se le pide explícitamente:

```bash
git -C ../ProyectoWebQA-autoventa checkout HEAD -- .
```

Hay que correr ese comando en cada worktree, uno por uno, cada vez que se quiera traer cambios nuevos — nada pasa automáticamente.

En cuanto a costo: **no** se duplica el historial de Git completo (mucho más liviano que un `git clone`), ni la caché de dependencias de Gradle (`~/.gradle` es global del usuario, no por proyecto) — solo se duplican los archivos de trabajo y lo que cada build compila.

> **Analogía:** el catálogo central se actualiza en cuanto se agrega un libro nuevo (un commit) — pero la fotocopia que ya está sobre el escritorio de la sala satélite sigue siendo la versión vieja hasta que alguien va y saca una fotocopia nueva a propósito.

---

## 4. Archivos ignorados por git que hay que copiar a mano

Como un worktree solo trae lo que está **commiteado**, cualquier archivo en `.gitignore` que el proyecto necesite en tiempo de ejecución hay que copiarlo a mano a cada worktree nuevo. En este proyecto fueron dos:

- `chromedriver.exe` (vive en `src/test/resources/chromedriver.exe`)
- `.env` (vive en la **raíz** del proyecto — no en `src/test/resources` como decía la documentación vieja del propio proyecto; hubo que ubicarlo con `find` porque esa nota estaba desactualizada)

Sin el `.env` copiado, el primer intento de correr dio un error real y bastante confuso a primera vista:

```
io.github.cdimascio.dotenv.DotEnvException: Could not find /.env on the classpath
Could not find .\.env on the file system (working directory: ...)
```

La clase que carga las credenciales (`HooksColgasWebV2.java`) las lee vía `dotenv` dentro de un bloque estático (`static {}`). En Java, si la inicialización estática de una clase falla una vez, **queda rota para el resto de esa misma JVM** — cualquier otro código que dependa de esa clase después vuelve a fallar en cascada (`ExceptionInInitializerError`), aunque el problema real ya haya pasado.

> **Analogía:** es como una fotocopia que trajo el libro pero no la llave del cajón donde el bibliotecario guarda las fichas de préstamo — sin esa llave especial (que nunca se archiva en el catálogo general, por seguridad), la sala satélite no puede completar el proceso, y una vez que la primera consulta del día falla por eso, todas las demás fallan igual mientras nadie traiga la llave.

---

## 5. Orquestar varios worktrees

**Manual, sin script** — dos terminales, cada una en su propia carpeta:

```bash
# Terminal 1
cd ../ProyectoWebQA-autoventa
./gradlew.bat test --tests "com.celuweb.co.runners.ColgasWebV2.ColgasAutoventaV2.AutoventaV2RunnersTest"
```

```bash
# Terminal 2, al mismo tiempo
cd ../ProyectoWebQA-preventa
./gradlew.bat test --tests "com.celuweb.co.runners.ColgasWebV2.ColgasPreventaV2.PreventaV2RunnersTest"
```

**Con script (PowerShell)** — automatiza exactamente eso, para cualquier número de módulos:

```powershell
$modulos = @(
    @{ Nombre = "Autoventa"; Clase = "com.celuweb.co.runners.ColgasWebV2.ColgasAutoventaV2.AutoventaV2RunnersTest"; Worktree = "../ProyectoWebQA-autoventa" },
    @{ Nombre = "Preventa";  Clase = "com.celuweb.co.runners.ColgasWebV2.ColgasPreventaV2.PreventaV2RunnersTest";   Worktree = "../ProyectoWebQA-preventa" }
)

$procesos = foreach ($m in $modulos) {
    $gradlew = Join-Path $m.Worktree "gradlew.bat"
    Start-Process -FilePath $gradlew -WorkingDirectory $m.Worktree `
        -ArgumentList "test", "--tests", $m.Clase -PassThru -NoNewWindow
}

$procesos | Wait-Process
```

Nótese que ya **no** hace falta ningún `-Dserenity.outputDirectory` ni `-Dtest.binaryResultsDir` — las tres colisiones de la sección 1 desaparecen solas, porque cada worktree tiene su propio `build/` y `target/` físicamente separados. El script no hace nada mágico: solo automatiza abrir las terminales y esperar a que ambas terminen (`Wait-Process`) antes de avisar dónde quedó cada reporte, en `<worktree>/target/site/serenity/index.html`.

Se corre así, desde la raíz del proyecto original:

```powershell
.\correr-modulos-en-paralelo.ps1
```

---

## 6. Correr un escenario suelto: desde el proyecto, no desde el worktree

Para depurar **un solo escenario**, el patrón correcto sigue siendo el de siempre (documentado ya en el propio `CLAUDE.md` del proyecto), corrido desde el **proyecto original** — no hace falta ningún worktree para esto:

```bash
./gradlew test --tests "com.celuweb.co.runners.<paquete>.<RunnerClass>" -Dcucumber.filter.tags="@TagUnicoDelEscenario"
```

Aquí hubo un error real cometido en la sesión, que vale la pena dejar anotado: un escenario de Preventa había fallado dentro de la corrida completa **en su worktree** (23 de 24 pasaron). Para depurarlo se corrió suelto — pero en el **proyecto original**, no en `ProyectoWebQA-preventa`. El escenario pasó solo (1/1) — pero el reporte oficial del worktree de Preventa, el que de verdad se le muestra al cliente, siguió mostrando la falla vieja, porque nunca se tocó ese worktree.

La lección no termina ahí. Como el escenario pasó solo pero falló dentro de la suite completa, lo más probable es un problema de **orden o estado compartido entre escenarios** (por ejemplo, un dato duplicado que dejó un escenario anterior) — una falla intermitente real. Si simplemente se hubiera corrido ese mismo escenario suelto **dentro** del worktree para "arreglar" el reporte, el JSON se habría sobrescrito con un resultado en verde, maquillando una falla real sin confirmar si de verdad se resolvió. Lo correcto es volver a correr la **suite completa** del módulo, dentro de su propio worktree, para que el reporte refleje una corrida real de punta a punta — no una mezcla de corridas parciales.

---

## Tabla resumen

| Se quiere... | Dónde correr | Comando |
|---|---|---|
| Depurar un escenario suelto | Proyecto original | `gradlew test --tests "<Runner>" -Dcucumber.filter.tags="@Tag"` |
| Correr un módulo completo, solo | Proyecto original o su worktree | `gradlew test --tests "<Runner>"` |
| Correr 2+ módulos al mismo tiempo | Un worktree por módulo | `.\correr-modulos-en-paralelo.ps1` |
| Traer cambios nuevos a un worktree | Ese worktree específico | `git -C <worktree> checkout HEAD -- .` |
| Crear un worktree nuevo | Proyecto original | `git worktree add --detach <ruta> HEAD` |
| Quitar un worktree | Proyecto original | `git worktree remove <ruta>` |

---

## Comparación con lo ya cubierto

| | [maxParallelForks / junit-platform.properties](maxparallelforks-junit-platform-properties.md) | Este documento (git worktree) | [Selenium Grid](selenium-grid.md) |
|---|---|---|---|
| Qué separa | Procesos (forks) o hilos, **dentro** de un mismo checkout | Copias de trabajo completas, **entre** checkouts, en la misma máquina | Sesiones de navegador, entre **máquinas/nodos** distintos |
| Por qué se necesitó | Punto de partida — el nivel más simple de paralelismo | Las 3 colisiones de la sección 1 probaron que un solo checkout no alcanza para módulos completos corriendo a la vez | Escalar horizontalmente o cubrir varios navegadores/SO |
| Se pueden combinar | — | Sí, con Grid: cada worktree podría apuntar su `RemoteWebDriver` a un nodo de Grid en vez de un Chrome local | — |

---

## Por qué esto importa

Aislar `maxParallelForks` o `serenity.outputDirectory` parece, sobre el papel, suficiente para correr módulos en paralelo — y de hecho es el primer paso razonable a intentar. Pero cuando el mismo checkout sigue siendo el punto de choque, cada parche resuelto destapa el siguiente, porque el problema de fondo (dos procesos escribiendo sobre el mismo directorio de proyecto) no se resuelve repartiendo mejor las rutas, sino separando el directorio en sí. Git worktree es la herramienta que hace eso sin duplicar el historial completo del repositorio — la pieza que faltaba para que el paralelismo entre módulos fuera confiable de verdad, no solo "la mayoría de las veces".
