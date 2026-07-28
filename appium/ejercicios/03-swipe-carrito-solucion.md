# Solución — Ejercicio 03: Swipe para eliminar del carrito

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. Las coordenadas exactas dependen de la resolución de tu emulador — ajústalas según lo que veas en el Inspector.

```java
import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.AppiumBy;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.interactions.PointerInput;
import org.openqa.selenium.interactions.Sequence;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;

import java.time.Duration;
import java.util.Collections;

import static org.junit.jupiter.api.Assertions.assertTrue;

public class Ejercicio03Appium {

    public static void ejecutar(AndroidDriver driver) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        // Ir al carrito (asumiendo que ya se agregó un producto antes, como en el ejercicio 02)
        var iconoCarrito = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.accessibilityId("cart") // confirmar con Inspector
        ));
        iconoCarrito.click();

        var productoEnCarrito = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.id("com.saucelabs.mydemoapp.rn:id/cartListItemContainer") // confirmar con Inspector
        ));

        var rect = productoEnCarrito.getRect();
        int startX = rect.getX() + rect.getWidth() - 20;
        int startY = rect.getY() + (rect.getHeight() / 2);
        int endX = rect.getX() + 20;

        PointerInput finger = new PointerInput(PointerInput.Kind.TOUCH, "finger");
        Sequence swipe = new Sequence(finger, 0);
        swipe.addAction(finger.createPointerMove(Duration.ofMillis(0), PointerInput.Origin.viewport(), startX, startY));
        swipe.addAction(finger.createPointerDown(PointerInput.MouseButton.LEFT.asArg()));
        swipe.addAction(finger.createPointerMove(Duration.ofMillis(600), PointerInput.Origin.viewport(), endX, startY));
        swipe.addAction(finger.createPointerUp(PointerInput.MouseButton.LEFT.asArg()));

        driver.perform(Collections.singletonList(swipe));

        // Confirmar que el carrito quedó vacío
        var mensajeCarritoVacio = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.accessibilityId("Cart is empty") // confirmar con Inspector
        ));

        assertTrue(mensajeCarritoVacio.isDisplayed(), "El carrito debería mostrarse vacío tras el swipe");
        System.out.println("✅ Producto eliminado con swipe, carrito vacío confirmado");
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Calculaste las coordenadas de inicio/fin a partir del `getRect()` del elemento real, en vez de hardcodear números fijos que solo funcionan en una resolución específica?
- ¿Tu swipe tiene una duración razonable (no `0ms`), para que la app lo reconozca como deslizamiento y no como tap?
- ¿Tu assert final verifica un estado concreto ("carrito vacío"), no solo que el swipe "se ejecutó sin error"?

## Errores comunes al hacer este ejercicio

- Hardcodear coordenadas de píxeles fijas copiadas del Inspector una sola vez — si el emulador cambia de resolución o el layout se ajusta, esas coordenadas dejan de ser válidas. Calcularlas a partir de `getRect()` del elemento es mucho más robusto.
- Un swipe demasiado corto en distancia (por ejemplo, moverse solo 10px) — la app puede no interpretarlo como un gesto de swipe completo.
