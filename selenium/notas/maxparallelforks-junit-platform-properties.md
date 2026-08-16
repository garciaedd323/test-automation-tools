# maxParallelForks y junit-platform.properties: dos paralelismos distintos que se confunden

## La analogía general

Imaginar una empresa de mensajería que tiene que entregar 11 paquetes grandes en la ciudad (los 11 runners de una suite Serenity + Screenplay + Cucumber). La empresa tiene **camionetas** — cada camioneta es un proceso Java independiente, con su propia memoria y su propio motor arrancado desde cero (una JVM, lo que Gradle llama "fork") — y, dentro de cada camioneta, puede ir **uno o varios mensajeros** repartiendo los paquetes que le tocaron a esa camioneta (los hilos de ejecución dentro de esa misma JVM).

Hoy la empresa solo saca **una camioneta** a la calle, y en ella va **un solo mensajero** que entrega los 11 paquetes uno detrás del otro, volviendo a la base entre cada uno. Por eso la ronda completa tarda ~50 minutos.

- **`maxParallelForks`** decide **cuántas camionetas salen a la calle al mismo tiempo**. Subirlo a 2 no cambia nada dentro de cada camioneta — sigue yendo un solo mensajero por camioneta, entregando de a un paquete — pero ahora hay dos camionetas circulando en paralelo, cada una con su propio tanque de gasolina y (en la analogía del proyecto real) su propio Chrome levantado por Serenity.
- **`junit-platform.properties`** decide algo distinto: si **dentro de una misma camioneta** puede ir más de un mensajero repartiendo paquetes al mismo tiempo, en vez de uno solo haciendo todo el recorrido de esa camioneta en particular.

Son dos preguntas distintas — "¿cuántos vehículos salen?" y "¿cuánta gente trabaja dentro de cada vehículo?" — y, como se ve más abajo, en un proyecto que corre con `@RunWith(CucumberWithSerenity.class)` (como es el caso típico de Serenity clásico), solo la primera pregunta tiene efecto real hoy.

---

## 1. `maxParallelForks`: cuántas camionetas salen a la calle

### Qué es un "fork" aquí

Cuando Gradle ejecuta la tarea `test`, no corre los tests dentro del mismo proceso Java que está corriendo Gradle. Levanta procesos Java **separados** — llamados *test workers* — cada uno con su propia JVM, su propia memoria, su propio classloader. A esos procesos hijos es a lo que Gradle llama **forks**.

`maxParallelForks` le dice a Gradle **cuántos de esos procesos puede tener corriendo al mismo tiempo**. Cada clase de test (en este tipo de proyecto, cada runner — `LoginV2RunnersTest`, `MapasPV2RunnersTest`, etc.) es una unidad de trabajo que Gradle reparte entre los forks disponibles.

### Sintaxis en `build.gradle`

Groovy DSL (lo más común en proyectos Serenity/Cucumber clásicos):

```groovy
test {
    // ... el resto de la configuración existente (finalizedBy, etc.) se mantiene igual

    // Cuántos procesos JVM puede tener Gradle corriendo tests en paralelo.
    // Cada fork = una JVM independiente = un Chrome independiente levantado por Serenity.
    maxParallelForks = 2
}
```

Kotlin DSL (`build.gradle.kts`), mismo efecto, otra sintaxis:

```kotlin
tasks.test {
    maxParallelForks = 2
}
```

Un valor dinámico basado en los núcleos disponibles de la máquina, en vez de un número fijo:

```groovy
test {
    maxParallelForks = (Runtime.runtime.availableProcessors() / 2).intValue() ?: 1
}
```

> **Analogía:** dividir entre 2 en vez de usar todos los núcleos es como no sacar TODAS las camionetas de la flota al mismo tiempo — hay que dejar motor (CPU) disponible para que el propio sistema operativo, el navegador Chrome de cada fork, y el resto de procesos de la máquina no se atropellen entre sí.

### Qué valor tiene sentido usar

No hay un número "correcto" universal — depende de cuánta RAM/CPU tiene la máquina que corre la suite y de cuántos Chrome simultáneos soporta sin quedarse sin memoria. Como cada fork levanta su propio Chrome completo (no es liviano), es más realista pensar en núcleos disponibles ÷ 2, o incluso empezar con un número fijo bajo (2) y subirlo de a poco mientras se observa el uso de memoria, en vez de saltar directo al máximo teórico de núcleos.

### El efecto real: paraleliza por runner, no por escenario

Este es el punto que más se presta a confusión: `maxParallelForks` reparte **clases de test completas** entre los forks disponibles. Si un runner (una clase) tiene 15 escenarios definidos vía `Esquema del escenario` + `Ejemplos`, esos 15 escenarios siguen corriendo **uno detrás de otro, dentro del mismo fork, en el mismo hilo** — subir `maxParallelForks` no toca eso. Lo único que cambia es que, mientras el runner A corre sus 15 escenarios en el fork 1, el runner B puede estar corriendo sus propios escenarios al mismo tiempo en el fork 2.

> **Analogía (retomando la de arriba):** subir `maxParallelForks` de 1 a 2 es sacar una segunda camioneta a la calle. Cada camioneta sigue llevando un solo mensajero que reparte sus paquetes de a uno — no se volvió más rápida la entrega *dentro* de una camioneta, simplemente ahora hay dos camionetas trabajando a la vez en vez de una sola.

---

## 2. `junit-platform.properties`: paralelismo dentro de una misma JVM

### Dónde vive el archivo

`src/test/resources/junit-platform.properties`. La plataforma JUnit 5 lo detecta automáticamente si está en el classpath de test — no hace falta declararlo en ningún lado del `build.gradle`.

### Propiedades principales

```properties
# Habilita el motor de paralelismo del engine Jupiter (JUnit 5). Apagado por defecto.
junit.jupiter.execution.parallel.enabled = true

# Modo por defecto para métodos de test: "concurrent" permite que corran en
# distintos hilos al mismo tiempo. El valor por defecto real es "same_thread"
# (secuencial), incluso con enabled=true, si no se declara esta línea.
junit.jupiter.execution.parallel.mode.default = concurrent

# Lo mismo, pero aplicado a nivel de clase de test (varias clases corriendo
# a la vez, no solo varios métodos de una misma clase).
junit.jupiter.execution.parallel.mode.classes.default = concurrent

# Estrategia para decidir CUÁNTOS hilos usar.
# "dynamic"  = calcula el número de hilos según los núcleos disponibles.
# "fixed"    = usa siempre el número exacto que se indique.
junit.jupiter.execution.parallel.config.strategy = dynamic

# Solo aplica si la estrategia es "dynamic". Factor multiplicador sobre los
# núcleos disponibles: 1 = un hilo por núcleo.
junit.jupiter.execution.parallel.config.dynamic.factor = 1

# Alternativa si se usa strategy = fixed: número fijo de hilos.
junit.jupiter.execution.parallel.config.fixed.parallelism = 4
```

> **Analogía:** esto es contratar más mensajeros para que vayan **dentro de la misma camioneta**. `enabled` es la decisión de si se permite ir acompañado o no; `mode.default` decide si esos mensajeros trabajan al mismo tiempo o se turnan uno a la vez aunque vayan juntos; `strategy`/`fixed.parallelism` decide cuántos mensajeros caben realmente en esa camioneta.

### La diferencia clave con `maxParallelForks`

`maxParallelForks` es paralelismo **entre procesos** (varias JVMs, cada una aislada, coordinadas por Gradle). `junit-platform.properties` es paralelismo **entre hilos dentro de un mismo proceso** (una sola JVM, coordinada por el motor de JUnit 5). Son capas distintas de la misma torta, y se pueden combinar — pero solo tienen efecto sobre lo que cada motor realmente ejecuta.

### Por qué en un proyecto con runners clásicos de Cucumber esto no cambia nada (todavía)

Acá está el detalle que más vale la pena dejar por escrito, porque es contraintuitivo: activar `junit-platform.properties` **no** hace que los escenarios de Cucumber corran en paralelo si el proyecto sigue usando el estilo clásico:

```java
@RunWith(CucumberWithSerenity.class)
@CucumberOptions(features = "src/test/resources/features/...", glue = {"..."}, tags = "@AlgunTag")
public class LoginV2RunnersTest {
}
```

`@RunWith` es una anotación de **JUnit 4**. Cuando JUnit 5 (la plataforma) encuentra una clase con `@RunWith`, la ejecuta a través del **motor Vintage** (`junit-vintage-engine`), que existe justamente para dar compatibilidad hacia atrás con JUnit 4. El motor Vintage trata **toda la clase completa como un solo descriptor de test opaco** frente a la plataforma — no le informa a JUnit 5 que "adentro" hay 15 escenarios individuales paralelizables. Desde el punto de vista de `junit-platform.properties`, ese runner es una caja cerrada: una unidad, no 15.

Por eso, el paralelismo real que hoy mueve la aguja en un proyecto así viene **exclusivamente de `maxParallelForks`** (Gradle repartiendo runners completos entre forks), no de `junit-platform.properties` (que no tiene nada granular para paralelizar dentro de un runner Vintage).

Para que `junit-platform.properties` empiece a paralelizar escenarios de Cucumber individuales (no solo runners completos), habría que migrar del estilo clásico `@RunWith(CucumberWithSerenity.class)` al motor nativo de Cucumber sobre JUnit Platform (`cucumber-junit-platform-engine`), que sí declara cada Scenario/fila de Examples como un descriptor de test independiente ante JUnit 5 — recién ahí `mode.default = concurrent` tendría escenarios reales que repartir entre hilos.

Mientras tanto, `junit-platform.properties` **sí** tiene efecto inmediato sobre cualquier test JUnit 5 puro que se agregue al proyecto — por ejemplo, los tests unitarios para parsers de Excel/PDF o para `DatabaseConnection` que suelen quedar pendientes en este tipo de proyectos (todo con `@Test` de `org.junit.jupiter.api`, sin Cucumber de por medio). Esos sí son descriptores nativos de Jupiter y sí se benefician de este archivo desde el día uno.

---

## 3. Comparación directa

| | `maxParallelForks` | `junit-platform.properties` |
|---|---|---|
| Nivel de paralelismo | Entre procesos (JVMs / forks de Gradle) | Entre hilos, dentro de una misma JVM |
| Motor que lo ejecuta | Gradle (`Test` task worker) | JUnit Platform, motor Jupiter |
| Dónde se configura | `build.gradle` / `build.gradle.kts` | `src/test/resources/junit-platform.properties` |
| Qué paraleliza hoy en un proyecto con `@RunWith(CucumberWithSerenity.class)` | Runners completos (clases enteras) | Nada relacionado a Cucumber — el runner es un descriptor opaco para Vintage |
| Qué paralelizaría si se migra a `cucumber-junit-platform-engine` | Sigue igual (procesos) | Escenarios/Examples individuales |
| Aislamiento entre unidades paralelas | Total — cada fork tiene su propia memoria, su propio Chrome | Parcial — los hilos comparten memoria de la misma JVM, hay que garantizar thread-safety real |
| Costo de arranque | Alto (levantar una JVM nueva completa por fork) | Bajo (son hilos dentro de un proceso que ya está arriba) |

---

## 4. Riesgos compartidos: por qué no basta con subir un número

Subir cualquiera de los dos paralelismos expone dos riesgos típicos, más allá de la configuración en sí:

**WebDriver/Chrome no es thread-safe por instancia.** Cada actor de Screenplay necesita su propio WebDriver — no se puede compartir una misma instancia de Chrome entre dos ejecuciones simultáneas. Screenplay resuelve esto con `OnStage`/`Cast`, que asigna un actor (y su navegador) por hilo mediante `ThreadLocal` — pero eso solo protege de verdad cuando el paralelismo ocurre **dentro de una misma JVM** (el caso de `junit-platform.properties`). Con `maxParallelForks`, el aislamiento viene gratis por otra vía: cada fork ya es un proceso completo con su propia memoria, así que no hay ningún `ThreadLocal` que cuidar entre runners — cada uno vive en su propio mundo.

**Datos de prueba compartidos.** Esto sí es independiente del mecanismo de paralelismo: si dos runners (o dos hilos) usan el mismo usuario o el mismo registro mutable de backend, pueden pisarse aunque cada uno tenga su propio Chrome. El caso concreto de un proyecto real: 11 runners iniciando sesión con el mismo usuario compartido — la duda era si el backend invalida la sesión de uno al detectar un segundo login concurrente. Se probó en vivo subiendo `maxParallelForks` a 2 temporalmente, corriendo dos runners al mismo tiempo contra el ambiente real: ambos Chrome arrancaron con un par de segundos de diferencia (confirmando concurrencia real) y ambos completaron sin conflicto de sesión. Quedó sin verificar, en cambio, si dos runners que compartan el mismo patrón de "tomar el primer registro no asignado" sobre una misma tabla podrían competir por el mismo registro al correr en paralelo — ese es un riesgo de datos, no de WebDriver, y no lo resuelve ninguna de las dos configuraciones de este documento.

> **Analogía:** el Chrome de cada camioneta nunca se comparte entre camionetas — eso ya viene resuelto por diseño. El riesgo real es que dos camionetas distintas intenten entregar el mismo paquete a la misma persona al mismo tiempo (dos runners tocando el mismo registro de backend) — ahí no importa cuántas camionetas o mensajeros haya, el problema es de logística de paquetes, no de vehículos.

---

## 5. Comparación con Selenium Grid (ya cubierto)

Vale la pena distinguir esto de lo que resuelve [Selenium Grid](selenium-grid.md), porque los tres suenan a "paralelismo" pero operan en capas distintas:

| | Selenium Grid | `maxParallelForks` + `junit-platform.properties` |
|---|---|---|
| Qué reparte | Sesiones de navegador entre **máquinas/nodos** distintos (incluso distintos SO/navegador) | Trabajo entre **procesos e hilos dentro de una misma máquina** |
| Resuelve | Escalar horizontalmente (más máquinas) y cobertura multi-navegador/SO | Aprovechar los núcleos de UNA máquina que ya está corriendo la suite |
| Se pueden combinar | Sí — los forks/hilos locales pueden apuntar sus `RemoteWebDriver` a un Grid en vez de lanzar Chrome local | — |

En otras palabras: Grid decide "¿en qué máquina y navegador corre esta sesión?"; `maxParallelForks`/`junit-platform.properties` deciden "¿cuántas de esas sesiones lanzo al mismo tiempo desde esta máquina?". No son alternativas — son capas que se apilan.

---

## Por qué esto importa

Antes de subir el número de `maxParallelForks` en un proyecto real, conviene tener clara la diferencia entre "correr más runners a la vez" (lo que sí funciona hoy con Cucumber clásico) y "correr más escenarios a la vez dentro de un mismo runner" (lo que requeriría migrar a `cucumber-junit-platform-engine`, un cambio de arquitectura mucho más grande). Confundir ambos lleva a configurar `junit-platform.properties` esperando una mejora de velocidad que, con runners `@RunWith`, simplemente no va a aparecer.

Esto también es la base técnica para decidir cuándo vale la pena migrar a `cucumber-junit-platform-engine`: si la mayoría del tiempo de una suite se va en pocos runners con muchísimos escenarios cada uno (en vez de muchos runners con pocos escenarios), `maxParallelForks` por sí solo tiene un techo bajo — ahí es donde el paralelismo a nivel de escenario empieza a justificar el esfuerzo de migración.
