# Patrones de diseño para automatización de pruebas

Cuando una suite de automatización crece (de 10 tests a 500), el código sin estructura se vuelve imposible de mantener: un cambio de UI rompe 40 tests a la vez, hay código duplicado por todos lados, y nadie quiere tocar la suite por miedo a romperla más. Estos patrones existen para resolver ese problema. Son **agnósticos de herramienta**: aplican igual en Selenium, Cypress, Playwright, Appium, etc. — cambia la sintaxis, no la idea.

---

## 1. El problema que resuelven

Sin ningún patrón, un test típico se ve así:

```javascript
test('login', async () => {
  await page.fill('#username', 'usuario');
  await page.fill('#password', 'clave123');
  await page.click('#login-btn');
  expect(await page.textContent('.welcome-msg')).toContain('Bienvenido');
});
```

**Analogía cotidiana**: es como si cada vez que quisieras encender la tele, tuvieras que explicar paso a paso "metete detrás del mueble, buscá el cable negro, enchufalo en el tomacorriente de la izquierda, después apretá el botón rojo que está a 3cm del borde". Funciona, pero si un día mueven el mueble, tenés que reescribir esa explicación en **cada lugar** donde la usaste.

El problema: si el diseñador cambia el `id` de `#login-btn` a `#btn-login`, tenés que salir a buscar y corregir esa línea en **todos los tests** que hacen login (que probablemente sean decenas).

---

## 2. Page Object Model (POM)

### Qué es

POM propone crear una **clase por cada página** (o componente importante) de la aplicación. Esa clase encapsula:
- Los **localizadores** (selectores) de los elementos de esa página.
- Los **métodos** que representan acciones que un usuario podría hacer en esa página (`login()`, `agregarAlCarrito()`, `buscarProducto()`).

Los tests ya no conocen los selectores ni los detalles de la UI — solo llaman a métodos con nombres claros.

**Analogía cotidiana**: en vez de explicar cada vez "metete detrás del mueble, buscá el cable negro...", instalás un control remoto universal. El test (el usuario) solo aprieta el botón "encender". Si mañana cambian la tele, solo hay que reprogramar el control remoto una vez — nadie más necesita saber cómo funciona por dentro.

### Ejemplo (Playwright)

```javascript
// pages/LoginPage.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameInput = page.locator('#username');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('#login-btn');
    this.welcomeMsg = page.locator('.welcome-msg');
  }

  async login(username, password) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getWelcomeMessage() {
    return this.welcomeMsg.textContent();
  }
}
module.exports = LoginPage;
```

```javascript
// tests/login.test.js
const LoginPage = require('../pages/LoginPage');

test('login exitoso', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.login('usuario', 'clave123');
  expect(await loginPage.getWelcomeMessage()).toContain('Bienvenido');
});
```

Ahora, si el `id` del botón cambia, solo se corrige **una línea** en `LoginPage.js`. Todos los tests que usan `login()` siguen funcionando sin tocarlos.

### Ventajas
- Reduce muchísimo la duplicación de código.
- El mantenimiento se centraliza: un cambio de UI = un solo lugar para corregir.
- Los tests se leen casi como lenguaje natural (`loginPage.login(...)`), lo cual los hace más entendibles para no-técnicos.

### Desventajas / límites
- Con aplicaciones muy grandes, las clases de página pueden volverse enormes y difíciles de navegar.
- No captura bien el "por qué" de una acción (la intención del usuario), solo el "cómo". Para eso está el Screenplay Pattern.
- Puede generar herencia compleja si no se diseña con cuidado (páginas que heredan de páginas).

---

## 3. Page Factory

### Qué es

Page Factory **no es un patrón nuevo**, es una implementación específica (originalmente de Selenium, en Java) que facilita la creación de objetos Page Object usando anotaciones y **inicialización perezosa (lazy)** de los elementos: los localizadores se declaran como propiedades anotadas, y el framework los resuelve automáticamente cuando se usan, en vez de buscarlos todos al crear el objeto.

**Analogía cotidiana**: siguiendo con el control remoto — en vez de que el control salga de fábrica ya "buscando" todos los botones físicos al encenderlo (lo cual sería lento e innecesario si solo vas a usar uno), cada botón se conecta al circuito real recién en el momento en que lo apretás.

### Ejemplo (Selenium + Java, el caso clásico de Page Factory)

```java
public class LoginPage {
    @FindBy(id = "username")
    private WebElement usernameInput;

    @FindBy(id = "password")
    private WebElement passwordInput;

    @FindBy(id = "login-btn")
    private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        PageFactory.initElements(driver, this);
    }

    public void login(String username, String password) {
        usernameInput.sendKeys(username);
        passwordInput.sendKeys(password);
        loginButton.click();
    }
}
```

La diferencia con un POM "a mano" es la anotación `@FindBy` y la línea `PageFactory.initElements(...)`, que hacen que Selenium resuelva los elementos automáticamente en vez de escribir `driver.findElement(By.id("username"))` en cada línea.

### Dato importante
Page Factory es básicamente **"POM con azúcar sintáctica"** en el ecosistema Java/Selenium. En herramientas como Cypress o Playwright no existe un "Page Factory" formal porque esos frameworks ya resuelven los elementos de forma perezosa por diseño — por eso vas a ver Page Factory mencionado casi exclusivamente en contextos de Selenium + Java o C#.

---

## 4. Screenplay Pattern

### Qué es

Es un patrón más moderno y más orientado a **actores e intenciones** que a páginas. En vez de pensar "¿qué hay en esta página?", pensás "¿qué quiere lograr este actor (usuario)?". Se apoya en varios conceptos:

- **Actor**: quien realiza las acciones (representa al usuario).
- **Task** (tarea): una acción de alto nivel con sentido de negocio (ej. "iniciar sesión", "comprar un producto").
- **Interaction** (interacción): la acción de bajo nivel real (click, escribir texto).
- **Question**: una consulta sobre el estado del sistema (¿está visible el mensaje de bienvenida?).
- **Ability**: una capacidad que el actor tiene (ej. "navegar por el browser", "hacer llamadas a la API").

**Analogía cotidiana**: en POM, el control remoto sabe apretar botones de una tele específica. En Screenplay, en cambio, tenés un **mayordomo** (el actor) al que le decís "quiero ver una película" (la tarea), y él decide qué pasos seguir para lograrlo — prender la tele, bajar las persianas, poner el volumen. Si mañana cambiás de tele o hasta de sistema de streaming, el mayordomo se adapta, porque lo que le pediste fue el objetivo, no los botones.

### Ejemplo (concepto, con Serenity/JS estilo Screenplay)

```javascript
const actor = Actor.named('María').whoCan(BrowseTheWeb.using(page));

const IniciarSesion = (usuario, clave) =>
  Task.where('Iniciar sesión',
    Enter.theValue(usuario).into(LoginPage.usernameInput),
    Enter.theValue(clave).into(LoginPage.passwordInput),
    Click.on(LoginPage.loginButton),
  );

await actor.attemptsTo(IniciarSesion('usuario', 'clave123'));

const mensaje = await actor.asks(Text.of(LoginPage.welcomeMsg));
expect(mensaje).toContain('Bienvenido');
```

Nota que `IniciarSesion` es reutilizable como una **tarea de negocio**, no como "una secuencia de clicks en una página específica". Eso permite, por ejemplo, reutilizar la misma tarea si mañana el login se hace por API en vez de por UI, cambiando solo la implementación interna de la tarea.

### Ventajas
- Separa mejor la **intención** (qué quiere el actor) de la **implementación** (cómo se logra).
- Escala mejor en proyectos grandes con múltiples actores/roles (ej. "admin", "cliente", "invitado").
- Facilita reutilizar tareas de negocio entre distintos tipos de test (UI, API) sin duplicar lógica.

### Desventajas
- Curva de aprendizaje más alta que POM — más conceptos, más código de infraestructura al principio.
- Para proyectos chicos o equipos que recién arrancan con automatización, puede ser sobre-ingeniería.
- Requiere librerías específicas (Serenity/JS, Serenity BDD para Java) que no todos los equipos quieren sumar como dependencia.

---

## 5. Comparación directa

| Aspecto | Page Object Model | Page Factory | Screenplay Pattern |
|---|---|---|---|
| Unidad de organización | Una clase por página | Una clase por página (con anotaciones) | Actores, tareas, interacciones |
| Foco | Elementos y acciones de una página | Igual que POM, con inicialización automática | Intención del usuario/actor |
| Complejidad inicial | Baja-media | Baja-media | Alta |
| Curva de aprendizaje | Suave | Suave (si ya sabés POM) | Empinada |
| Ideal para | La mayoría de los proyectos | Selenium + Java/C# específicamente | Proyectos grandes, múltiples roles, equipos maduros en automatización |
| Reutilización entre UI/API | Limitada | Limitada | Alta (una tarea puede implementarse por UI o API) |
| Disponible "nativo" en | Cualquier herramienta (patrón, no librería) | Selenium (WebDriver) | Requiere librería (Serenity/JS, Serenity BDD) |

---

## 6. ¿Cuál elegir?

- **Empezando con automatización o proyecto chico/mediano** → Page Object Model. Es el estándar de facto, lo entiende cualquier persona que se sume al equipo, y resuelve el 90% de los problemas de mantenibilidad.
- **Usando Selenium con Java o C#** → Page Factory es simplemente la forma "idiomática" de implementar POM en ese ecosistema. No es una alternativa a POM, es POM con su propia sintaxis.
- **Proyecto grande, con múltiples roles de usuario, y equipo con experiencia en automatización** → Screenplay Pattern, si el equipo está dispuesto a invertir en la curva de aprendizaje y valora la separación entre intención e implementación a largo plazo.

Una regla práctica: **empezar siempre con POM**. Si con el tiempo la suite crece tanto que el mantenimiento se vuelve doloroso incluso con POM bien implementado (muchos roles, mucha lógica de negocio repetida, necesidad de correr las mismas tareas contra UI y API), ahí recién vale la pena evaluar migrar a Screenplay.
