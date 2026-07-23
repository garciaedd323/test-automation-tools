# Anatomía de un test en Cypress

## La analogía general

Con Selenium, el código y la "organización del test" eran cosas separadas: Selenium solo sabía manejar el navegador, y había que traer JUnit o TestNG desde afuera para tener `@Test`, fixtures y asserts. Es como comprar un auto sin radio y tener que instalar un equipo de sonido aparte.

Cypress viene con **el equipo de sonido ya instalado de fábrica** — trae integrado su propio framework de estructura y aserciones (basado en Mocha y Chai, dos librerías estándar de JavaScript). No se elige entre opciones como se hacía con JUnit vs TestNG; ya viene decidido, y todo vive en un solo archivo `.cy.js` con una sintaxis particular que hay que aprender a leer.

---

## 1. La estructura básica: `describe` e `it`

```javascript
describe("Login de usuario", () => {
  it("permite iniciar sesión con credenciales válidas", () => {
    cy.visit("/login");
    cy.get("[data-cy=username]").type("usuario123");
    cy.get("[data-cy=password]").type("clave123");
    cy.get("[data-cy=btn-login]").click();
    cy.get("[data-cy=welcome-message]").should("contain", "Bienvenido");
  });

  it("muestra un error con clave incorrecta", () => {
    cy.visit("/login");
    cy.get("[data-cy=username]").type("usuario123");
    cy.get("[data-cy=password]").type("clave-mala");
    cy.get("[data-cy=btn-login]").click();
    cy.get("[data-cy=error-message]").should("be.visible");
  });
});
```

> **Analogía:** `describe` es como el **título de un capítulo de un libro** ("Login de usuario") — agrupa todo lo relacionado con ese tema. Cada `it()` es **una escena específica dentro de ese capítulo** ("permite iniciar sesión con credenciales válidas") — una historia corta y autocontenida, con su propio inicio y final. Igual que un buen libro no mezcla capítulos distintos en la misma página, no se debería mezclar la lógica de "login" con la de "checkout" dentro del mismo `describe`.

**Equivalencia con lo ya conocido:** `describe` es conceptualmente similar a una clase de test en JUnit (`LoginTest`), e `it` es equivalente a cada método anotado con `@Test`.

---

## 2. Fixtures del ciclo de vida: `before`, `beforeEach`, `after`, `afterEach`

```javascript
describe("Carrito de compras", () => {
  before(() => {
    // se ejecuta UNA sola vez, antes de todos los "it" de este describe
    cy.log("Preparando datos globales del carrito");
  });

  beforeEach(() => {
    // se ejecuta ANTES de CADA "it"
    cy.visit("/carrito");
  });

  it("agrega un producto al carrito", () => {
    cy.get("[data-cy=producto-1]").click();
    cy.get("[data-cy=carrito-contador]").should("have.text", "1");
  });

  it("elimina un producto del carrito", () => {
    cy.get("[data-cy=producto-1]").click();
    cy.get("[data-cy=eliminar-producto]").click();
    cy.get("[data-cy=carrito-contador]").should("have.text", "0");
  });

  afterEach(() => {
    // se ejecuta DESPUÉS de CADA "it"
    cy.log("Test finalizado");
  });

  after(() => {
    // se ejecuta UNA sola vez, al final de todos los "it"
    cy.log("Limpieza final del carrito");
  });
});
```

> **Analogía:** es exactamente el mismo concepto de "abrir y cerrar la tienda cada día" ya visto con `@BeforeEach`/`@AfterEach` de JUnit — `beforeEach` es **barrer el local antes de que entre cada cliente** (cada `it`), y `before` es **la inauguración de la tienda, que solo pasa una vez** al principio de todo el `describe`. La lógica de fondo es idéntica a lo ya conocido; solo cambió el nombre de los métodos y el lenguaje.

| Cypress | Equivalente en JUnit 5 |
|---|---|
| `before()` | `@BeforeAll` |
| `beforeEach()` | `@BeforeEach` |
| `afterEach()` | `@AfterEach` |
| `after()` | `@AfterAll` |

---

## 3. El objeto global `cy` — el control remoto único

Todo lo que se hace en un test de Cypress pasa por el objeto `cy`:

```javascript
cy.visit("/login");           // navegar
cy.get("[data-cy=username]"); // buscar un elemento
cy.contains("Iniciar sesión"); // buscar por texto visible
cy.url();                      // leer la URL actual
cy.reload();                   // recargar la página
```

> **Analogía:** en Selenium, existía el objeto `driver` que representaba "el navegador controlado a distancia". En Cypress, `cy` es parecido, pero recuerda la diferencia arquitectónica ya vista: `cy` no está mandando comandos a un servidor remoto — está **parado dentro del navegador mismo**, dando instrucciones directas. Es el mismo concepto de "control remoto único para todo", pero operado desde adentro en vez de desde afuera.

---

## 4. Comandos encadenados — la sintaxis que más confunde al principio

```javascript
cy.get("[data-cy=username]")
  .should("be.visible")
  .type("usuario123")
  .should("have.value", "usuario123");
```

> **Analogía:** es como dar instrucciones en cadena a un asistente: "busca el campo de usuario, **confirma que está visible**, escribe 'usuario123' en él, y **confirma que efectivamente quedó escrito**". Cada `.` es un paso más en la misma instrucción continua — el asistente no pasa al siguiente paso hasta que el anterior se confirme como exitoso.

**El punto que más rompe la cabeza a quien viene de Selenium:** estos comandos **no son promesas de JavaScript tradicionales**, aunque se parezcan. No se usa `async/await` ni `.then()` como en el resto de JavaScript moderno — Cypress maneja su propia cola interna de comandos y los ejecuta en orden, uno tras otro, de forma asíncrona pero oculta esa complejidad del desarrollador.

```javascript
// ❌ Esto NO funciona como se esperaría en JS normal
const texto = cy.get("[data-cy=mensaje]").text(); // undefined - cy.get no retorna el valor directamente
console.log(texto);

// ✅ Forma correcta: usar .then() para acceder al valor resuelto
cy.get("[data-cy=mensaje]").then(($el) => {
  const texto = $el.text();
  cy.log(texto);
});
```

> **Analogía:** es como pedirle a un mensajero que traiga un paquete — no se puede revisar el contenido del paquete **antes** de que el mensajero regrese con él en la mano. `cy.get()` "despacha al mensajero", pero el contenido real solo está disponible dentro del `.then()`, que es literalmente "cuando el mensajero regrese, haz esto con lo que trajo".

---

## 5. Aserciones con Chai — integradas, no elegidas

```javascript
cy.get("[data-cy=welcome-message]").should("contain", "Bienvenido");
cy.get("[data-cy=btn-login]").should("be.disabled");
cy.get("[data-cy=lista-productos]").should("have.length", 3);
cy.url().should("include", "/dashboard");
```

> **Analogía:** en Selenium, se elegía el "inspector de calidad" (JUnit `Assertions` o TestNG `Assert`) como una herramienta separada. En Cypress, el inspector de calidad **ya viene integrado al mismo control remoto** — `should()` es literalmente parte de la cadena de comandos de `cy`, no una librería aparte que se importa y se usa de forma independiente.

---

## 6. Comparación completa con lo ya conocido

| Concepto | Selenium + JUnit | Cypress |
|---|---|---|
| Agrupar tests relacionados | Clase de test (`LoginTest`) | `describe("Login...")` |
| Un test individual | Método con `@Test` | `it("hace algo...")` |
| Antes de cada test | `@BeforeEach` | `beforeEach()` |
| Después de cada test | `@AfterEach` | `afterEach()` |
| Antes de todos los tests | `@BeforeAll` | `before()` |
| Objeto que controla el navegador | `driver` | `cy` |
| Framework de aserciones | Elegido aparte (JUnit/TestNG/AssertJ) | Integrado (Chai, vía `.should()`) |
| Manejo de asincronía | Bloqueante (cada línea espera a la anterior) | Cola interna de comandos, no promesas tradicionales |

---

## 7. Errores comunes al empezar a escribir tests

| Síntoma | Causa típica |
|---|---|
| `cy.get(...)` devuelve `undefined` al intentar usarlo directamente | No se está usando `.then()` para acceder al valor resuelto |
| El test pasa aunque el elemento nunca apareció | Se usó un `console.log` fuera del flujo de comandos de Cypress, que se ejecuta antes de que los comandos realmente corran |
| Los tests de un `describe` afectan a los de otro | Falta un `beforeEach` que reinicie el estado (por ejemplo, volver a `cy.visit()` la página) |
| Confusión sobre por qué no hace falta `await` | Cypress no usa async/await — su cola de comandos interna maneja la asincronía de forma distinta a como se maneja en JS normal |

---

## 8. Por qué esto importa antes de ver selectors y comandos en detalle

Entender esta estructura es el "esqueleto" sobre el que se construye todo lo demás: cuando en el siguiente tema se vean los selectors recomendados (`data-cy`) y los comandos específicos (`type`, `click`, `check`), todos van a vivir **dentro de un `it()`**, siguiendo exactamente este mismo patrón de `cy.algo().should(...)`. Sin este esqueleto claro, cualquier ejemplo de código suelto parecería magia sin contexto.
