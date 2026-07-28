# Solución — Ejercicio 04: Carrito de compras con Page Object Model

> ⚠️ Este es el ejercicio más largo — intenta resolverlo por tu cuenta, aunque te tome varios intentos, antes de leer esto. Tu estructura de carpetas/clases puede ser distinta y seguir siendo correcta.

## Estructura de archivos

```
src/
├── pages/
│   ├── BasePage.java       (ya la tienes de la nota de POM)
│   ├── LoginPage.java
│   ├── InventoryPage.java
│   └── CartPage.java
├── utils/
│   └── ScreenshotOnFailureExtension.java   (ya la tienes de la nota de screenshots)
└── tests/
    └── CarritoTest.java
```

## `LoginPage.java`

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage extends BasePage {
    private final By campoUsuario = By.id("user-name");
    private final By campoClave = By.id("password");
    private final By botonIngresar = By.id("login-button");

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    public InventoryPage login(String usuario, String clave) {
        escribir(campoUsuario, usuario);
        escribir(campoClave, clave);
        click(botonIngresar);
        return new InventoryPage(driver);
    }
}
```

## `InventoryPage.java`

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class InventoryPage extends BasePage {
    private final By contadorCarrito = By.className("shopping_cart_badge");
    private final By iconoCarrito = By.className("shopping_cart_link");

    public InventoryPage(WebDriver driver) {
        super(driver);
    }

    public void agregarProducto(String nombreProducto) {
        By botonAgregar = By.id("add-to-cart-" + nombreProducto);
        click(botonAgregar);
    }

    public String obtenerContadorCarrito() {
        return obtenerTexto(contadorCarrito);
    }

    public CartPage irAlCarrito() {
        click(iconoCarrito);
        return new CartPage(driver);
    }
}
```

## `CartPage.java`

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import java.util.List;

public class CartPage extends BasePage {
    private final By nombresProductos = By.className("inventory_item_name");

    public CartPage(WebDriver driver) {
        super(driver);
    }

    public List<String> obtenerProductosEnCarrito() {
        return driver.findElements(nombresProductos)
            .stream()
            .map(el -> el.getText())
            .toList();
    }
}
```

## `CarritoTest.java`

```java
package tests;

import org.junit.jupiter.api.*;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import pages.*;
import utils.ScreenshotOnFailureExtension;

import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

public class CarritoTest {

    private WebDriver driver;
    private InventoryPage inventoryPage;

    @RegisterExtension
    ScreenshotOnFailureExtension screenshotExtension;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        screenshotExtension = new ScreenshotOnFailureExtension(driver);

        driver.get("https://www.saucedemo.com");
        LoginPage loginPage = new LoginPage(driver);
        inventoryPage = loginPage.login("standard_user", "secret_sauce");
    }

    @Test
    void agregarDosProductosAlCarrito() {
        inventoryPage.agregarProducto("sauce-labs-backpack");
        inventoryPage.agregarProducto("sauce-labs-bike-light");

        assertEquals("2", inventoryPage.obtenerContadorCarrito());

        CartPage cartPage = inventoryPage.irAlCarrito();
        List<String> productos = cartPage.obtenerProductosEnCarrito();

        assertTrue(productos.contains("Sauce Labs Backpack"));
        assertTrue(productos.contains("Sauce Labs Bike Light"));
    }

    @AfterEach
    void tearDown() {
        if (driver != null) driver.quit();
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Tu `CarritoTest` tiene **cero** `driver.findElement()` directos? Todo debería pasar por los Page Objects.
- ¿`agregarProducto` en `InventoryPage` construye el locator dinámicamente a partir del nombre, en vez de tener un método distinto por cada producto?
- ¿Reutilizaste el `ScreenshotOnFailureExtension` ya construido, en vez de escribir lógica de captura nueva dentro del test?
- ¿`login()` en `LoginPage` retorna un `InventoryPage`, modelando la navegación real (como ya viste en la nota de POM)?

## Errores comunes al hacer este ejercicio

- Poner los `assert` dentro de los Page Objects en vez de en el test — recuerda la regla ya vista: un Page Object solo *expone información*, el test decide si es correcta.
- Hardcodear el nombre del producto directamente en el `id` del locator sin parametrizar, obligando a duplicar el método `agregarProducto` por cada producto distinto.
