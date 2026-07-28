# Solución — Ejercicio 01: Capabilities y tu primer tap

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. Los locators exactos de este archivo son **ilustrativos** — reemplázalos por los que confirmaste con Appium Inspector, ya que pueden variar entre versiones de la app.

```java
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import io.appium.java_client.AppiumBy;
import java.net.URL;
import java.time.Duration;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class Ejercicio01Appium {
    public static void main(String[] args) throws Exception {
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platformName", "Android");
        caps.setCapability("appium:automationName", "UiAutomator2");
        caps.setCapability("appium:deviceName", "emulator-5554");
        caps.setCapability("appium:app", "/ruta/a/mda.apk");

        AndroidDriver driver = new AndroidDriver(new URL("http://127.0.0.1:4723"), caps);
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        try {
            // Reto extra: espera explícita antes de interactuar
            var primerProducto = wait.until(
                ExpectedConditions.presenceOfElementLocated(
                    AppiumBy.accessibilityId("Sauce Labs Backpack") // confirmar con Inspector
                )
            );
            primerProducto.click();

            var botonAgregar = wait.until(
                ExpectedConditions.presenceOfElementLocated(
                    AppiumBy.id("com.saucelabs.mydemoapp.rn:id/btnAddToCart") // confirmar con Inspector
                )
            );

            assertTrue(botonAgregar.isDisplayed(), "Debería estar en la pantalla de detalle del producto");
            System.out.println("✅ Test pasó — se abrió la pantalla de detalle");
        } finally {
            driver.quit();
        }
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Confirmaste los locators reales con Appium Inspector antes de escribir el código, en vez de adivinar?
- ¿Tu assert verifica algo específico de la pantalla de detalle, no solo "no hubo excepción"?
- Si hiciste el reto extra: ¿tu espera usa una condición concreta (`presenceOfElementLocated` o similar), no un `Thread.sleep()`?

## Errores comunes al hacer este ejercicio

- Copiar los `resource-id` de este documento sin verificarlos — están puestos como ejemplo ilustrativo del patrón, no como valores garantizados para tu versión exacta de la app.
- Olvidar que el `deviceName` debe coincidir con el emulador realmente corriendo (`adb devices` para confirmarlo).
