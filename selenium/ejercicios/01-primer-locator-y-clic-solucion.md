# Solución — Ejercicio 01: Tu primer locator y clic

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. La solución no es única — si tu enfoque funciona y tiene sentido, es tan válida como esta.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class Ejercicio01 {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        try {
            driver.get("https://the-internet.herokuapp.com");

            // Espera explícita antes de interactuar (el reto extra)
            var enlaceLogin = wait.until(
                ExpectedConditions.elementToBeClickable(By.linkText("Form Authentication"))
            );
            enlaceLogin.click();

            wait.until(ExpectedConditions.urlContains("login"));

            String urlActual = driver.getCurrentUrl();
            assertTrue(urlActual.contains("login"), "La URL debería contener 'login'");

            System.out.println("✅ Test pasó — URL actual: " + urlActual);
        } finally {
            driver.quit();
        }
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Usaste `By.linkText(...)` o algún otro locator? Ambos son válidos si el texto es único en la página.
- ¿Cerraste el driver al final (`driver.quit()`), incluso si algo falla? Por eso aquí está en un `finally`.
- Si hiciste el reto extra: ¿tu espera explícita usa una condición específica (`elementToBeClickable`) en vez de solo `presenceOfElementLocated`?

## Errores comunes al hacer este ejercicio

- Olvidar `driver.quit()` — deja procesos de Chrome abiertos acumulándose.
- Usar `Thread.sleep()` en vez de una espera explícita — funciona, pero es justo el antipatrón que ya viste en la nota de esperas.
