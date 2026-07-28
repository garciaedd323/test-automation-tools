# Solución — Ejercicio 03: Login con esperas explícitas y assert

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class Ejercicio03 {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        try {
            driver.get("https://the-internet.herokuapp.com/login");

            // --- Intento fallido ---
            login(driver, wait, "usuario_invalido", "clave_invalida");

            WebElement mensaje = wait.until(
                ExpectedConditions.visibilityOfElementLocated(By.id("flash"))
            );
            assertTrue(mensaje.getText().contains("invalid"), "Debería mostrar mensaje de error");

            // --- Intento exitoso ---
            driver.findElement(By.id("username")).clear();
            driver.findElement(By.id("password")).clear();
            login(driver, wait, "tomsmith", "SuperSecretPassword!");

            mensaje = wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("flash")));
            assertTrue(mensaje.getText().contains("logged into a secure area"),
                "Debería mostrar mensaje de éxito");

            // --- Logout ---
            WebElement botonLogout = wait.until(
                ExpectedConditions.elementToBeClickable(By.cssSelector("a[href='/logout']"))
            );
            botonLogout.click();

            wait.until(ExpectedConditions.urlContains("login"));
            assertTrue(driver.getCurrentUrl().contains("login"), "Debería volver al login");

            System.out.println("✅ Los tres escenarios pasaron correctamente");
        } finally {
            driver.quit();
        }
    }

    private static void login(WebDriver driver, WebDriverWait wait, String usuario, String clave) {
        driver.findElement(By.id("username")).sendKeys(usuario);
        driver.findElement(By.id("password")).sendKeys(clave);
        driver.findElement(By.cssSelector("button[type='submit']")).click();
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Esperaste explícitamente el mensaje (`#flash`) antes de leer su texto, en vez de leerlo inmediatamente después del clic?
- ¿Limpiaste los campos (`clear()`) antes de escribir las credenciales correctas, para no concatenar texto sobre el intento anterior?
- ¿Tu assert de logout confirma la URL, no solo que "no hubo error"?

## Errores comunes al hacer este ejercicio

- Leer el texto de `#flash` sin esperar — como el mensaje puede tardar unos milisegundos en actualizarse, a veces se lee el mensaje anterior por error.
- Crear un nuevo `driver` para el segundo intento de login, en vez de reutilizar el mismo — esto reinicia la sesión innecesariamente y hace el test más lento.
