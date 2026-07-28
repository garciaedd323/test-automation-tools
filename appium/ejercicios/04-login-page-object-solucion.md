# Solución — Ejercicio 04: Login con Page Object Model

> ⚠️ Este es el ejercicio más largo — intenta resolverlo por tu cuenta, aunque te tome varios intentos, antes de leer esto. Confirma todos los locators con Appium Inspector; los de aquí son ilustrativos.

## `pages/MenuPage.java`

```java
package pages;

import io.appium.java_client.AppiumBy;
import io.appium.java_client.android.AndroidDriver;

public class MenuPage extends BasePage {
    private final org.openqa.selenium.By iconoMenu = AppiumBy.accessibilityId("Open Menu");
    private final org.openqa.selenium.By opcionLogin = AppiumBy.accessibilityId("Log In");

    public MenuPage(AndroidDriver driver) {
        super(driver);
    }

    public void abrirMenu() {
        click(iconoMenu);
    }

    public LoginPage irALogin() {
        abrirMenu();
        click(opcionLogin);
        return new LoginPage(driver);
    }
}
```

## `pages/LoginPage.java`

```java
package pages;

import io.appium.java_client.AppiumBy;
import io.appium.java_client.android.AndroidDriver;
import org.openqa.selenium.By;

public class LoginPage extends BasePage {
    private final By campoUsuario = AppiumBy.accessibilityId("username-input");
    private final By campoClave = AppiumBy.accessibilityId("password-input");
    private final By botonIngresar = AppiumBy.accessibilityId("login-button");
    private final By mensajeError = AppiumBy.accessibilityId("error-message");

    public LoginPage(AndroidDriver driver) {
        super(driver);
    }

    public void loginExitoso(String usuario, String clave) {
        escribir(campoUsuario, usuario);
        escribir(campoClave, clave);
        click(botonIngresar);
    }

    public String obtenerMensajeError() {
        return obtenerTexto(mensajeError);
    }
}
```

## `tests/LoginTest.java`

```java
package tests;

import io.appium.java_client.android.AndroidDriver;
import org.junit.jupiter.api.*;
import org.openqa.selenium.remote.DesiredCapabilities;
import pages.LoginPage;
import pages.MenuPage;

import java.net.URL;

import static org.junit.jupiter.api.Assertions.*;

public class LoginTest {

    private AndroidDriver driver;
    private MenuPage menuPage;

    @BeforeEach
    void setUp() throws Exception {
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setCapability("platformName", "Android");
        caps.setCapability("appium:automationName", "UiAutomator2");
        caps.setCapability("appium:deviceName", "emulator-5554");
        caps.setCapability("appium:app", "/ruta/a/mda.apk");

        driver = new AndroidDriver(new URL("http://127.0.0.1:4723"), caps);
        menuPage = new MenuPage(driver);
    }

    @Test
    void loginExitosoMuestraBienvenida() {
        LoginPage loginPage = menuPage.irALogin();
        loginPage.loginExitoso("standard_user", "secret_sauce"); // confirmar credenciales reales en pantalla

        // Reemplazar por el elemento real que confirma el login exitoso en esta app
        var confirmacion = driver.findElement(
            io.appium.java_client.AppiumBy.accessibilityId("Welcome")
        );
        assertTrue(confirmacion.isDisplayed(), "Debería mostrarse el estado de sesión iniciada");
    }

    @Test
    void loginConClaveIncorrectaMuestraError() {
        LoginPage loginPage = menuPage.irALogin();
        loginPage.loginExitoso("standard_user", "clave-incorrecta");

        String error = loginPage.obtenerMensajeError();
        assertFalse(error.isEmpty(), "Debería mostrarse un mensaje de error");
    }

    @AfterEach
    void tearDown() {
        if (driver != null) driver.quit();
    }
}
```

## Puntos clave a revisar en tu solución

- ¿Tu `LoginTest` tiene **cero** `driver.findElement()` directos fuera de los Page Objects? (el ejemplo de arriba tiene uno en el primer test como placeholder — en tu solución real, debería vivir dentro de un Page Object también).
- ¿`irALogin()` en `MenuPage` retorna un `LoginPage`, modelando la navegación real entre pantallas, igual que en Selenium/Cypress/Playwright?
- Si hiciste el reto extra: ¿reutilizaste el mismo `LoginPage` para el caso de error, en vez de duplicar locators en una clase nueva?

## Errores comunes al hacer este ejercicio

- Asumir que las credenciales de ejemplo son siempre `standard_user`/`secret_sauce` (las de `saucedemo.com`) — esta es una app **distinta**, confirma las credenciales reales que la propia app muestra en su pantalla de login.
- Poner los `assert` dentro de los Page Objects en vez del test — la misma regla ya vista en las otras herramientas: el Page Object expone información, el test decide si es correcta.
