# Solución — Ejercicio 02: Localizar por texto con UiSelector y agregar al carrito

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. Confirma todos los locators con Appium Inspector.

```java
import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.AppiumBy;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import java.time.Duration;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class Ejercicio02Appium {

    public static void ejecutar(AndroidDriver driver) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

        // Reto extra: UiScrollable + UiSelector combinados, por si el producto no está visible
        var producto = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.androidUIAutomator(
                "new UiScrollable(new UiSelector().scrollable(true))" +
                ".scrollIntoView(new UiSelector().textContains(\"Sauce Labs Onesie\"))"
            )
        ));
        producto.click();

        var botonAgregar = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.id("com.saucelabs.mydemoapp.rn:id/btnAddToCart") // confirmar con Inspector
        ));
        botonAgregar.click();

        var contadorCarrito = wait.until(ExpectedConditions.presenceOfElementLocated(
            AppiumBy.id("com.saucelabs.mydemoapp.rn:id/cartQuantityBadgeView") // confirmar con Inspector
        ));

        assertEquals("1", contadorCarrito.getText(), "El contador del carrito debería mostrar 1");
        System.out.println("✅ Producto agregado y contador confirmado");
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Usaste `textContains()` o `text()` según corresponda (texto parcial vs exacto), y confirmaste cuál aplicaba con el Inspector?
- Si el producto no era visible sin scroll: ¿usaste `UiScrollable` en vez de calcular coordenadas de swipe manualmente?
- ¿Tu assert compara contra el texto exacto que muestra el contador ("1"), no solo que "algo cambió"?

## Errores comunes al hacer este ejercicio

- Usar `text()` (coincidencia exacta) cuando el texto real tiene mayúsculas/espacios distintos a lo esperado — `textContains()` es más tolerante para este tipo de ejercicio.
- Olvidar que el contador del carrito puede no existir en el DOM hasta que se agrega el primer producto — si tu test falla buscándolo antes de agregar algo, revisa el orden de tus pasos.
