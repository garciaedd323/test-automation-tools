# Solución — Ejercicio 02: Checkboxes y dropdown

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.Select;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class Ejercicio02 {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver();

        try {
            // --- Parte 1: Checkboxes ---
            driver.get("https://the-internet.herokuapp.com/checkboxes");
            List<WebElement> checkboxes = driver.findElements(By.cssSelector("#checkboxes input"));

            WebElement primerCheckbox = checkboxes.get(0);
            if (!primerCheckbox.isSelected()) {
                primerCheckbox.click();
            }
            assertTrue(primerCheckbox.isSelected(), "El primer checkbox debería estar marcado");

            // Reto extra: marcar ambos
            WebElement segundoCheckbox = checkboxes.get(1);
            if (!segundoCheckbox.isSelected()) {
                segundoCheckbox.click();
            }
            assertTrue(segundoCheckbox.isSelected(), "El segundo checkbox también debería estar marcado");

            // --- Parte 2: Dropdown ---
            driver.get("https://the-internet.herokuapp.com/dropdown");
            WebElement elementoSelect = driver.findElement(By.id("dropdown"));
            Select dropdown = new Select(elementoSelect);

            dropdown.selectByVisibleText("Option 2");

            String opcionElegida = dropdown.getFirstSelectedOption().getText();
            assertEquals("Option 2", opcionElegida, "Debería haberse seleccionado 'Option 2'");

            System.out.println("✅ Ambas partes del test pasaron correctamente");
        } finally {
            driver.quit();
        }
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Verificaste `isSelected()` antes de hacer clic en cada checkbox, en vez de hacer clic a ciegas?
- ¿Usaste la clase `Select` para el dropdown, en vez de intentar hacer clicks manuales sobre las opciones?
- ¿Tu assert del dropdown compara contra el texto visible correcto ("Option 2", no "option2" ni el value interno)?

## Errores comunes al hacer este ejercicio

- Hacer clic directo en los checkboxes sin revisar su estado — si el test se corre dos veces sin reiniciar el navegador, el segundo intento desmarcaría lo que ya estaba marcado.
- Intentar usar `Select` sobre un dropdown que en realidad es un `<div>` custom (no aplica en este ejercicio puntual, pero es un error común en apps reales — repasa la sección 3.1 de la nota de interacciones).
