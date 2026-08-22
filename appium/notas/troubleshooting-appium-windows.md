# Troubleshooting: configurar Appium en Windows desde cero

## El problema, en una frase

Configurar Appium en Windows por primera vez casi nunca falla por un solo motivo: es una cadena de 4-5 problemas independientes que aparecen uno detrás de otro (PATH, SDK, variables de entorno, IDE, y hasta OneDrive), y cada uno se disfraza de "otro error nuevo" cuando en realidad son piezas sueltas del mismo rompecabezas de configuración.

Esta nota documenta, en orden, los errores más comunes al levantar un entorno de automatización móvil con **Appium + UiAutomator2 + Serenity/Cucumber + IntelliJ** en Windows, con su causa real y su solución.

---

## 1. `"appium" no se reconoce como un comando`

### Síntoma

```
C:\Users\HP>appium
"appium" no se reconoce como un comando interno o externo,
programa o archivo por lotes ejecutable.
```

### Causa

No es un problema de PATH en la mayoría de los casos: **Appium directamente no está instalado**. Es fácil asumirlo al revés porque el mensaje de error de Windows es el mismo tanto si falta el PATH como si falta el paquete.

### Solución

Verifica primero que Node.js y npm existan:

```bash
node -v
npm -v
```

Si responden con una versión, instala Appium de forma **global**:

```bash
npm install -g appium
```

El flag `-g` es la parte importante: sin él, Appium se instala solo en la carpeta actual y nunca queda disponible como comando.

Verifica:

```bash
appium -v
```

Si después de instalarlo sigue sin reconocerse, ahí sí es un tema de PATH: revisa que la carpeta que devuelve `npm config get prefix` esté agregada a la variable de entorno `Path` de Windows.

---

## 2. Falta el driver (`uiautomator2` o `xcuitest`)

Appium se instala "vacío": el servidor no trae ningún driver por defecto. Sin un driver, el comando `appium` arranca, pero no puede controlar ningún dispositivo.

```bash
appium driver install uiautomator2   # Android
appium driver install xcuitest       # iOS (requiere Mac)
```

Verifica lo instalado:

```bash
appium driver list --installed
```

📌 Si ves `Error: A driver named "uiautomator2" is already installed`, no es un error real — solo significa que ya lo tenías. Confírmalo con el comando de arriba.

---

## 3. IntelliJ no muestra el botón de ejecución (▶) ni en el `.feature` ni en el Runner

### Síntoma

El plugin de Cucumber está instalado y habilitado, las dependencias de Gradle están bien, el archivo `.feature` está en la ruta correcta... y aun así, no aparece ningún ícono de "Run" en ningún lado del proyecto. Ni en la clase Runner (`@RunWith(CucumberWithSerenity.class)`), ni en el escenario Gherkin.

### Causa real

**Power Save Mode** de IntelliJ. Este modo apaga el análisis de código en segundo plano — inspecciones, resaltado de errores y, junto con eso, los íconos de ejecución en el margen izquierdo. No es un problema de Cucumber, de Gradle ni de dependencias: es que IntelliJ, literalmente, dejó de analizar el código.

### Solución

```
File → Power Save Mode → desactivar (quitar el check)
```

Dale unos segundos al IDE para re-indexar. Los íconos verdes deberían aparecer tanto en el Runner como en el `.feature`.

### Por qué es tan fácil pasarlo por alto

Es una casilla que se activa con un solo clic (a veces sin querer), no da ningún aviso visible de que está afectando la ejecución de tests, y el síntoma que produce ("no aparece el botón") parece apuntar directamente a un problema de configuración de Cucumber o Gradle — cuando en realidad el IDE entero está en modo de bajo consumo.

---

## 4. `Error: Could not find 'aapt2.exe'`

### Síntoma

```
Encountered internal error running command: Error: Could not find 'aapt2.exe' in [...]
Do you have Android Build Tools installed at 'C:\Android\Sdk'?
```

### Causa real: dos SDKs de Android conviviendo en la misma máquina

Este es el error más engañoso de todos, porque tiene **dos capas** de causa:

1. **Build-Tools no estaba realmente instalado** (aunque la casilla en el SDK Manager pareciera marcada), y
2. Aunque se instalara, Android Studio y las variables de entorno de Windows pueden **apuntar a rutas distintas del SDK** sin que nadie lo note. Por ejemplo:
   - Android Studio instala en: `C:\Users\HP\AppData\Local\Android\Sdk`
   - Pero `ANDROID_SDK_ROOT` apuntaba a: `C:\Android\Sdk` (una carpeta vacía o desactualizada)

Appium busca `aapt2.exe` únicamente en la ruta que le indican esas variables de entorno — si apuntan al SDK equivocado, no importa que el archivo sí exista en otro lado del disco: Appium jamás lo va a encontrar.

### Cómo diagnosticarlo

1. Verifica dónde dice Android Studio que instala el SDK:
   ```
   Android Studio → More Actions → SDK Manager → "Android SDK Location" (arriba)
   ```
2. Compáralo con tus variables de entorno actuales:
   ```bash
   echo %ANDROID_HOME%
   echo %ANDROID_SDK_ROOT%
   ```
3. Si son rutas **distintas**, ahí está el problema.

### Solución

1. En el SDK Manager, pestaña **SDK Tools** → marca **"Show Package Details"** → instala una versión específica de **Android SDK Build-Tools**.
2. Corrige las variables de entorno para que apunten a la ruta **real** donde Android Studio instala (normalmente `...\AppData\Local\Android\Sdk`, no `C:\Android\Sdk`):
   ```
   ANDROID_HOME = C:\Users\<usuario>\AppData\Local\Android\Sdk
   ANDROID_SDK_ROOT = C:\Users\<usuario>\AppData\Local\Android\Sdk
   ```
3. Agrega al `Path`:
   ```
   C:\Users\<usuario>\AppData\Local\Android\Sdk\platform-tools
   C:\Users\<usuario>\AppData\Local\Android\Sdk\build-tools\<version>
   ```
4. Cierra **todas** las terminales, IntelliJ incluido, y ábrelos de nuevo.
5. Verifica:
   ```bash
   where adb
   where aapt2
   ```

---

## 5. `Android SDK root folder does not exist` (aunque la ruta se ve bien)

### Síntoma

```
Error: The Android SDK root folder '  C:\Users\HP\AppData\Local\Android\Sdk' does not exist on the local file system.
```

### Causa

Fíjate con atención en el mensaje: hay **espacios en blanco antes de la ruta** (`'  C:\Users\...'`). La ruta en sí está bien escrita, pero la variable de entorno tiene espacios sobrantes al inicio o al final del valor — algo que pasa fácilmente al copiar y pegar en el campo de "Editar variable de entorno" de Windows. Ese espacio hace que el sistema busque literalmente una carpeta con espacios en el nombre, que no existe.

### Solución

1. Abre la variable (`ANDROID_HOME` o `ANDROID_SDK_ROOT`) en las Variables de Entorno.
2. **Borra el campo completo** (Ctrl+A dentro del campo → Suprimir), no edites sobre el texto existente.
3. Escribe la ruta de nuevo, limpia, sin espacios al inicio ni al final.
4. Cierra y abre una terminal nueva, y confirma con `echo %ANDROID_HOME%` que no haya espacios antes de la ruta.

---

## 6. `Unable to delete directory '...\build\test-results\test\binary'`

### Síntoma

Gradle falla al correr los tests con un error de que no puede borrar una carpeta dentro de `build/`, incluso después de borrarla manualmente y volver a intentar.

### Causa real: el proyecto vive dentro de una carpeta sincronizada por OneDrive

Cuando el proyecto está en una ruta como:

```
C:\Users\HP\OneDrive - EMPRESA\...\MiProyecto
```

OneDrive puede bloquear archivos momentáneamente mientras los sincroniza — justo cuando Gradle necesita borrarlos o sobreescribirlos durante el build. Pausar la sincronización ayuda a veces, pero no siempre es suficiente: basta con que el proceso de OneDrive siga corriendo en segundo plano para que el lock persista.

### Solución (la definitiva, no el parche)

Mover el proyecto **fuera** de cualquier carpeta sincronizada (OneDrive, Google Drive, Dropbox, etc.), a una ruta 100% local:

```
C:\Users\HP\Documents\MiCarpetaDeProyectos\MiProyecto
```

Pasos:

1. Cierra IntelliJ y cualquier terminal abierta en esa carpeta.
2. Copia el proyecto completo a la nueva ruta:
   ```bash
   xcopy "C:\Users\HP\OneDrive - EMPRESA\...\MiProyecto" "C:\Users\HP\Documents\MiCarpetaDeProyectos\MiProyecto" /E /H /C /I
   ```
3. Borra la carpeta `build` en la copia nueva (por si trajo residuos):
   ```bash
   rmdir /s /q "C:\Users\HP\Documents\MiCarpetaDeProyectos\MiProyecto\build"
   ```
4. Abre el proyecto desde la nueva ubicación en IntelliJ (`File → Open`) y deja que Gradle sincronice.
5. Ya puedes reactivar OneDrive sin riesgo, porque el proyecto ya no vive dentro de su carpeta sincronizada.

📌 Nota: `Documents` normalmente no está sincronizado por OneDrive personal, pero algunas políticas corporativas fuerzan el respaldo de "Escritorio, Documentos e Imágenes". Verifica si tu carpeta `Documents` tiene el ícono de nube de OneDrive antes de asumir que ya estás a salvo.

---

## Analogía: la mudanza con direcciones cruzadas

Imagina que pediste que te instalaran internet en tu casa, pero diste dos direcciones distintas en dos formularios diferentes: en uno pusiste "Calle 10" (donde el técnico realmente fue e instaló el router), y en otro pusiste "Calle 20" (la dirección que le diste a la empresa de facturación para que revisen el servicio).

Cuando la empresa de soporte va a "Calle 20" a verificar por qué no tienes señal, no encuentra nada — porque ahí nunca se instaló nada. El servicio sí existe, solo que está en la dirección equivocada según sus registros.

Eso es exactamente lo que pasa con `ANDROID_HOME`/`ANDROID_SDK_ROOT` apuntando a `C:\Android\Sdk` mientras Android Studio instala todo en `...\AppData\Local\Android\Sdk`: Appium (el "soporte técnico") va a la dirección que le diste en la variable de entorno, no a donde realmente vive el SDK.

---

## Resumen rápido: síntoma → causa real

| Síntoma | Causa real | Solución |
|---|---|---|
| `"appium" no se reconoce como un comando` | Appium no está instalado (no un problema de PATH) | `npm install -g appium` |
| No aparece el botón ▶ en `.feature` ni en el Runner | Power Save Mode activado en IntelliJ | Desactivarlo en `File → Power Save Mode` |
| `Could not find 'aapt2.exe'` | Build-Tools no instalado o dos SDKs distintos en la máquina | Instalar Build-Tools y unificar `ANDROID_HOME`/`ANDROID_SDK_ROOT` a la ruta real |
| `SDK root folder does not exist` (con ruta aparentemente correcta) | Espacios en blanco en el valor de la variable de entorno | Borrar el campo completo y reescribir la ruta limpia |
| `Unable to delete directory` en `build/` | Proyecto ubicado dentro de una carpeta sincronizada por OneDrive | Mover el proyecto a una ruta 100% local |

---

## Checklist para armar un entorno nuevo sin repetir esta cadena de errores

- [ ] `node -v` y `npm -v` responden con una versión
- [ ] `npm install -g appium` ejecutado, `appium -v` responde
- [ ] Driver instalado (`appium driver list --installed` muestra `uiautomator2` o `xcuitest`)
- [ ] `Android SDK Location` en Android Studio coincide exactamente con `ANDROID_HOME`/`ANDROID_SDK_ROOT`
- [ ] `Android SDK Build-Tools` instalado con "Show Package Details" activado (para confirmar la versión real)
- [ ] Variables de entorno revisadas **sin espacios extra** al inicio/final del valor
- [ ] `Path` incluye `platform-tools` y `build-tools\<version>` de la ruta correcta
- [ ] Proyecto ubicado en una carpeta **local**, fuera de OneDrive/Google Drive/Dropbox
- [ ] Power Save Mode de IntelliJ desactivado
- [ ] Plugin "Cucumber for Java" instalado y habilitado
