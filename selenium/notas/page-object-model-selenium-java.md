# Page Object Model (POM) en Selenium — con ejemplo completo funcional

El POM es el patrón de diseño más importante para que tus tests no se conviertan en un desastre imposible de mantener.

> **Analogía general:** imagina que tienes una casa (tu aplicación web) y varios técnicos (tus tests) que necesitan hacer cosas en ella — revisar el medidor de luz, abrir la puerta del garaje, prender el calentador. Sin POM, cada técnico memoriza de memoria "el interruptor está a la izquierda de la puerta, subiendo dos escalones" y lo repite en cada visita. Si un día mueves el interruptor de lugar, **todos los técnicos** quedan perdidos al mismo tiempo. Con POM, contratas a **un solo electricista especializado por habitación** (una clase "Page Object" por página) que conoce exactamente dónde está cada cosa en su zona. Los técnicos (tests) no tocan los interruptores directamente — le piden al electricista de esa habitación "enciende la luz", y él sabe cómo hacerlo. Si el interruptor se mueve, **solo actualizas al electricista de esa habitación**, no a todos los técnicos.

---

## 1. El problema sin POM

```java
// Test sin POM — todo mezclado
driver.findElement(By.id("username")).sendKeys("usuario123");
driver.findElement(By.id("password")).sendKeys("clave123");
driver.findElement(By.cssSelector("button[type='submit']")).click();

WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("welcome-message")));
String texto = driver.findElement(By.id("welcome-message")).getText();
assertTrue(texto.contains("Bienvenido"));
```

Si mañana el campo `username` cambia de `id` a `name`, tienes que buscar y reemplazar en **cada test** que haga login. Con 50 tests que inician sesión, son 50 lugares que arreglar.

---

## 2. La estructura del POM

Cada página (o componente reutilizable, como un header o un modal) se convierte en una **clase**:
- Los locators (`By.id`, `By.cssSelector`, etc.) son **atributos privados**.
- Las acciones que un usuario puede hacer en esa página son **métodos públicos**.
- El test nunca usa `driver.findElement` directamente — solo llama métodos del Page Object.

> **Analogía:** los locators son como el plano interno de la habitación que solo el electricista necesita ver. Los métodos públicos son como el "menú de servicios" que el electricista ofrece: "puedo encender la luz", "puedo revisar el medidor" — el técnico (test) solo pide el servicio, sin saber ni importarle el cableado interno.

---

## 3. Ejemplo completo: Login + Dashboard

Vamos a modelar un flujo real: login → verificar bienvenida → cerrar sesión.

### 3.1 Clase base (código compartido entre todas las páginas)

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.support.ui.WebDriverWait;
import java.time.Duration;

public abstract class BasePage {

    protected WebDriver driver;
    protected WebDriverWait wait;

    public BasePage(WebDriver driver) {
        this.driver = driver;
        this.wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    }

    protected WebElement esperarVisible(By locator) {
        return wait.until(ExpectedConditions.visibilityOfElementLocated(locator));
    }

    protected WebElement esperarClickeable(By locator) {
        return wait.until(ExpectedConditions.elementToBeClickable(locator));
    }

    protected void click(By locator) {
        esperarClickeable(locator).click();
    }

    protected void escribir(By locator, String texto) {
        WebElement campo = esperarVisible(locator);
        campo.clear();
        campo.sendKeys(texto);
    }

    protected String obtenerTexto(By locator) {
        return esperarVisible(locator).getText();
    }
}
```
> **Analogía:** esta clase es como el "kit de herramientas estándar" que todo electricista de la empresa lleva consigo (linterna, destornillador, multímetro). Todas las habitaciones (páginas) heredan este kit básico, así no hay que reinventarlo en cada una.

### 3.2 Page Object de Login

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class LoginPage extends BasePage {

    // Locators — el "plano" que solo esta clase conoce
    private final By campoUsuario = By.id("username");
    private final By campoClave = By.id("password");
    private final By botonIngresar = By.cssSelector("button[type='submit']");
    private final By mensajeError = By.id("error-message");

    public LoginPage(WebDriver driver) {
        super(driver);
    }

    // Métodos públicos = "servicios" que ofrece esta página
    public void ingresarUsuario(String usuario) {
        escribir(campoUsuario, usuario);
    }

    public void ingresarClave(String clave) {
        escribir(campoClave, clave);
    }

    public DashboardPage clickIngresar() {
        click(botonIngresar);
        return new DashboardPage(driver); // navegamos a la siguiente "habitación"
    }

    public String obtenerMensajeError() {
        return obtenerTexto(mensajeError);
    }

    // Método de conveniencia que combina varios pasos
    public DashboardPage loginExitoso(String usuario, String clave) {
        ingresarUsuario(usuario);
        ingresarClave(clave);
        return clickIngresar();
    }
}
```
> **Analogía:** `LoginPage` es el electricista especializado en la puerta principal de la casa. Sabe exactamente dónde está el timbre (`campoUsuario`), la cerradura (`campoClave`) y la manija (`botonIngresar`). El método `loginExitoso` es como decirle "abre la puerta principal completa" en una sola instrucción, sin que tú tengas que saber los tres pasos internos que eso implica.

Nota algo clave: `clickIngresar()` **retorna un `DashboardPage`**. Esto modela el hecho de que, tras el login, el usuario "se mueve" a una nueva página — y el test puede encadenar acciones directamente sobre esa nueva página.

### 3.3 Page Object del Dashboard

```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class DashboardPage extends BasePage {

    private final By mensajeBienvenida = By.id("welcome-message");
    private final By botonCerrarSesion = By.id("logout-btn");
    private final By nombreUsuarioMostrado = By.className("user-name");

    public DashboardPage(WebDriver driver) {
        super(driver);
    }

    public String obtenerMensajeBienvenida() {
        return obtenerTexto(mensajeBienvenida);
    }

    public String obtenerNombreUsuarioMostrado() {
        return obtenerTexto(nombreUsuarioMostrado);
    }

    public LoginPage cerrarSesion() {
        click(botonCerrarSesion);
        return new LoginPage(driver); // volvemos a la "habitación" de login
    }
}
```
> **Analogía:** `DashboardPage` es el electricista de la sala principal de la casa, una vez ya entraste. Sabe leer el "letrero de bienvenida" en la pared y sabe dónde está el botón para "cerrar la casa" (cerrar sesión), que te devuelve de vuelta a la puerta principal.

### 3.4 El test (limpio, legible, sin un solo `findElement`)

```java
package tests;

import org.junit.jupiter.api.*;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import pages.DashboardPage;
import pages.LoginPage;

import static org.junit.jupiter.api.Assertions.*;

public class LoginTest {

    private WebDriver driver;
    private LoginPage loginPage;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.get("https://app-ejemplo.com/login");
        loginPage = new LoginPage(driver);
    }

    @Test
    void loginExitosoMuestraMensajeDeBienvenida() {
        DashboardPage dashboard = loginPage.loginExitoso("usuario123", "clave123");

        String mensaje = dashboard.obtenerMensajeBienvenida();
        assertTrue(mensaje.contains("Bienvenido"));

        assertEquals("usuario123", dashboard.obtenerNombreUsuarioMostrado());
    }

    @Test
    void loginConClaveIncorrectaMuestraError() {
        loginPage.ingresarUsuario("usuario123");
        loginPage.ingresarClave("clave-incorrecta");
        loginPage.clickIngresar();

        String error = loginPage.obtenerMensajeError();
        assertEquals("Usuario o contraseña incorrectos", error);
    }

    @Test
    void cerrarSesionRegresaALogin() {
        DashboardPage dashboard = loginPage.loginExitoso("usuario123", "clave123");
        LoginPage loginDeNuevo = dashboard.cerrarSesion();

        assertNotNull(loginDeNuevo);
        assertTrue(driver.getCurrentUrl().contains("/login"));
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```
> **Analogía:** el test es como el **cliente de la empresa de electricistas** — nunca toca cables, nunca sabe qué destornillador se usa. Solo dice "haz login con estas credenciales" y "dime si veo el mensaje de bienvenida". Todo el conocimiento técnico vive escondido dentro de `LoginPage` y `DashboardPage`.

Fíjate qué tan legible es el test: **se lee casi como una historia en español**, no como una lista de comandos técnicos de Selenium. Ese es el objetivo real del POM.

---

## 4. Diagrama del flujo

![Diagrama de Page Object Model](pom_diagrama.svg)

*(Diagrama ilustrativo: el test solo conoce los "Page Objects", nunca los locators directamente. `LoginPage.clickIngresar()` retorna un `DashboardPage`, modelando la navegación real entre páginas.)*

---

## 5. Por qué esto evita dolores de cabeza reales

| Problema sin POM | Con POM |
|---|---|
| El `id` de un botón cambia → hay que arreglar 30 tests | Se arregla **una sola línea** en el Page Object correspondiente |
| Un test hace login copiando y pegando 5 líneas | Un test hace login llamando `loginPage.loginExitoso(user, pass)` |
| Difícil saber qué hace un test con solo leerlo | El test se lee casi como texto natural |
| Locators repetidos y duplicados por todo el código | Locators centralizados en un solo lugar por página |
| Cambiar de "clic normal" a "clic con JS" implica tocar cada test | Se cambia una sola vez dentro del método `click()` de `BasePage` |

---

## 6. Variante moderna: `PageFactory` y anotaciones `@FindBy`

Selenium también ofrece una forma "declarativa" de definir locators usando anotaciones, en vez de `By` explícitos dentro del constructor:

```java
package pages;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPageConAnotaciones {

    private WebDriver driver;

    @FindBy(id = "username")
    private WebElement campoUsuario;

    @FindBy(id = "password")
    private WebElement campoClave;

    @FindBy(css = "button[type='submit']")
    private WebElement botonIngresar;

    public LoginPageConAnotaciones(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this); // "inyecta" los elementos automáticamente
    }

    public void login(String usuario, String clave) {
        campoUsuario.sendKeys(usuario);
        campoClave.sendKeys(clave);
        botonIngresar.click();
    }
}
```
> **Analogía:** es como si en vez de que el electricista busque cada interruptor manualmente cada vez que lo necesita, tuviera **etiquetas pre-pegadas** en cada uno desde el principio ("este es el interruptor de la sala", "este es el del pasillo"). `PageFactory.initElements` es el momento en que el electricista "lee todas las etiquetas de golpe" al entrar a la habitación.

**Advertencia práctica:** con `@FindBy`, los `WebElement` se resuelven de forma *perezosa* (lazy) y pueden generar `StaleElementReferenceException` con más facilidad en páginas muy dinámicas (React/Vue), porque el elemento se re-busca en cada uso, no se cachea de forma explícita como con locators `By` + esperas manuales. Muchos equipos prefieren el enfoque explícito con `By` + `WebDriverWait` (como en la sección 3) precisamente por ese control más fino.

---

## 7. Buenas prácticas adicionales

1. **Un Page Object no debería tener asserts.** Las validaciones (`assertEquals`, `assertTrue`) van en el test, no en la página. El Page Object solo *expone información* (ej. `obtenerMensajeError()`); el test decide si eso es correcto o no.
2. **Métodos que naveguen a otra página deben retornar esa página.** Como vimos con `clickIngresar()` devolviendo `DashboardPage`.
3. **Componentes reutilizables (headers, modales, menús) también son Page Objects** — no hace falta que sean una "página completa"; un `NavbarComponent` compartido por varias páginas es totalmente válido.
4. **Nombra los métodos por la intención del usuario, no por la acción técnica**: `loginExitoso()` es mejor que `clickBotonYEsperarRedireccion()`.
