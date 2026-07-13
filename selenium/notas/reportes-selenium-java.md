# Generación de reportes — Allure Report y HTML reports (Java)

Un test que falla en CI sin un reporte claro es casi tan inútil como un test que no corrió. Los reportes son la forma en que **cualquier persona del equipo** (no solo quien escribió el test) entiende qué pasó, cuándo, y por qué — sin tener que leer logs de consola línea por línea.

> **Analogía general:** correr tests sin un buen reporte es como hacer una auditoría completa de una empresa y al final solo decir "algunas cosas estaban bien, otras mal" de palabra, sin entregar ningún documento. Un buen reporte es el **informe de auditoría formal**: qué se revisó, qué pasó, qué falló, con evidencia (capturas, tiempos, pasos) — algo que puedes archivar, compartir con el jefe, y consultar meses después.

---

## 1. El reporte básico que ya viene "gratis" (Surefire / Maven)

Cuando corres tests con Maven, automáticamente se genera un reporte XML/TXT en `target/surefire-reports/`.

```
target/surefire-reports/tests.LoginTest.txt
```

```
Tests run: 3, Failures: 1, Errors: 0, Skipped: 0

loginConCredencialesValidas: PASSED
loginConClaveIncorrecta: PASSED
loginConUsuarioBloqueado: FAILED
  Expected: "Cuenta bloqueada"
  Actual: "Usuario o contraseña incorrectos"
```

> **Analogía:** esto es como el **recibo simple de una máquina expendedora** — te dice si tu compra fue exitosa o no, y poco más. Sirve para un vistazo rápido, pero si quieres entender **por qué** falló algo (¿qué se veía en pantalla en ese momento? ¿cuánto tardó cada paso?), este recibo no te alcanza.

Para convertir esto en HTML navegable, se agrega el plugin Surefire Report:

```xml
<!-- pom.xml -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-report-plugin</artifactId>
    <version>3.2.5</version>
</plugin>
```

```bash
mvn surefire-report:report
# genera target/site/surefire-report.html
```

> **Analogía:** esto es como llevar ese recibo simple a una **oficina que te lo convierte en un documento con tablas, colores y formato legible** — mismo contenido, pero ahora presentable para mostrarle a alguien más.

---

## 2. Allure Report — el estándar de facto para reportes ricos

Allure agrega pasos (`@Step`), adjuntos (screenshots, HTML, logs), categorías de fallo, historial de ejecuciones, y una interfaz web interactiva. Funciona tanto con JUnit 5 como con TestNG.

### 2.1 Configuración en Maven

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-junit5</artifactId>
        <version>2.29.0</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <argLine>
                    -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/1.9.22/aspectjweaver-1.9.22.jar"
                </argLine>
            </configuration>
        </plugin>
    </plugins>
</build>
```

> **Analogía:** esta configuración es como **instalar las cámaras de seguridad y el sistema de grabación** antes de que la tienda abra — es trabajo previo de infraestructura, pero una vez instalado, todo se graba automáticamente sin que tengas que pensarlo en cada test.

### 2.2 `@Step` — dividir el test en pasos legibles

```java
import io.qameta.allure.Step;
import io.qameta.allure.Epic;
import io.qameta.allure.Feature;
import io.qameta.allure.Severity;
import io.qameta.allure.SeverityLevel;

@Epic("Autenticación")
@Feature("Login")
public class LoginPage {

    @Step("Ingresar usuario: {usuario}")
    public void ingresarUsuario(String usuario) {
        driver.findElement(By.id("username")).sendKeys(usuario);
    }

    @Step("Ingresar contraseña")
    public void ingresarClave(String clave) {
        driver.findElement(By.id("password")).sendKeys(clave);
    }

    @Step("Hacer clic en el botón de ingresar")
    public void clickIngresar() {
        driver.findElement(By.cssSelector("button[type='submit']")).click();
    }
}
```

```java
@Severity(SeverityLevel.CRITICAL)
@Test
void loginConCredencialesValidas() {
    loginPage.ingresarUsuario("usuario123");
    loginPage.ingresarClave("clave123");
    loginPage.clickIngresar();
    // ...
}
```

> **Analogía:** `@Step` es como si, en vez de solo anotar "se hizo la auditoría", el auditor **numerara cada paso de su revisión** con una descripción clara: "Paso 1: revisé la caja registradora", "Paso 2: conté el inventario". Si algo falla, el reporte te dice exactamente **en qué paso específico** ocurrió el problema, no solo "el test falló en algún punto".

### 2.3 Adjuntar screenshots automáticamente (conectando con el tema anterior)

```java
import io.qameta.allure.Allure;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import java.io.ByteArrayInputStream;

public class ScreenshotOnFailureExtension implements TestWatcher {

    private WebDriver driver;

    @Override
    public void testFailed(ExtensionContext context, Throwable cause) {
        byte[] captura = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
        Allure.addAttachment("Captura al fallar", new ByteArrayInputStream(captura));

        Allure.addAttachment("HTML de la página", driver.getPageSource());
    }
}
```

> **Analogía:** esto es exactamente lo que vimos en el tema de screenshots, pero ahora la foto **queda pegada directamente dentro del informe de auditoría**, junto al paso exacto donde ocurrió el incidente — no como un archivo suelto que hay que ir a buscar aparte.

### 2.4 Categorizar fallos (para distinguir "bug real" de "problema de infraestructura")

```json
// categories.json (en src/test/resources/)
[
  {
    "name": "Fallos de red/infraestructura",
    "matchedStatuses": ["broken"],
    "messageRegex": ".*TimeoutException.*"
  },
  {
    "name": "Bugs de producto confirmados",
    "matchedStatuses": ["failed"],
    "messageRegex": ".*AssertionError.*"
  }
]
```

> **Analogía:** es como separar en la auditoría los hallazgos en dos carpetas distintas: **"problemas reales de la empresa"** (bugs confirmados en el producto) vs. **"no pude revisar esta área porque el edificio no tenía luz ese día"** (fallos de infraestructura, red, timeouts) — mezclar ambas cosas en un solo montón hace que la gente ignore el reporte completo por desconfianza.

### 2.5 Generar y ver el reporte

```bash
mvn test
allure serve target/allure-results
```

Esto abre un servidor local con el reporte interactivo: gráfico de tendencias, duración por test, pasos expandibles, capturas incrustadas, y filtros por severidad/feature.

> **Analogía:** `allure serve` es como abrir la **sala de juntas donde se presenta el informe de auditoría completo**, con gráficas, anexos y todo organizado — en vez de solo entregar una hoja de papel con un resumen de dos líneas.

---

## 3. Diagrama comparativo

![Diagrama comparativo de reportes](reportes_diagrama.svg)

*(Diagrama ilustrativo: del recibo simple de Surefire, al reporte HTML navegable, hasta el reporte interactivo de Allure con pasos, capturas y tendencias históricas.)*

---

## 4. Extent Reports — alternativa a Allure

Si el equipo prefiere algo más simple de configurar (sin el javaagent de AspectJ que requiere Allure), Extent Reports es una alternativa popular:

```java
import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;

public class ExtentManager {
    private static ExtentReports extent;

    public static ExtentReports getInstance() {
        if (extent == null) {
            ExtentSparkReporter reporter = new ExtentSparkReporter("target/extent-report.html");
            extent = new ExtentReports();
            extent.attachReporter(reporter);
        }
        return extent;
    }
}
```

```java
ExtentTest test = ExtentManager.getInstance().createTest("Login con credenciales válidas");
test.pass("Usuario ingresó correctamente");
test.fail("El mensaje de bienvenida no apareció", MediaEntityBuilder.createScreenCaptureFromPath("captura.png").build());
```

> **Analogía:** si Allure es como contratar una auditora externa completa con su propia infraestructura de cámaras, Extent Reports es como **armar tú mismo el informe con una plantilla ya lista** — más simple de instalar, con menos "magia" automática detrás, pero igual de presentable al final.

---

## 5. Tabla comparativa: Surefire vs Allure vs Extent Reports

| Característica | Surefire (básico) | Allure | Extent Reports |
|---|---|---|---|
| Configuración inicial | Ya viene de fábrica con Maven | Requiere javaagent + dependencias | Requiere dependencia + código de setup |
| Pasos detallados (`@Step`) | No | Sí | Manual (`test.info(...)`) |
| Capturas incrustadas | No | Sí, nativo | Sí, nativo |
| Historial de tendencias entre ejecuciones | No | Sí | Limitado |
| Categorías de fallo (bug vs infraestructura) | No | Sí | Manual |
| Curva de aprendizaje | Ninguna | Media | Baja-media |
| Uso típico | Chequeo rápido local | Estándar en equipos QA serios | Alternativa liviana |

---

## 6. Buenas prácticas para reportes útiles

1. **Usa `@Step` de forma consistente** — un reporte con pasos vagos ("hacer login") es casi tan inútil como uno sin pasos.
2. **Adjunta evidencia SOLO en fallos** (como vimos en el tema anterior) — adjuntar en cada paso exitoso satura el reporte y lo hace lento de cargar.
3. **Usa `@Severity`/prioridades** para que el equipo sepa qué fallos atender primero — no todos los fallos son igual de urgentes.
4. **Integra la generación del reporte al pipeline de CI**, no solo localmente — de nada sirve un reporte hermoso que solo tú puedes ver en tu máquina.
5. **Revisa el reporte, no solo el "PASS/FAIL" final** — la mayor parte del valor de estas herramientas está en poder **investigar** un fallo sin tener que reproducirlo manualmente desde cero.
