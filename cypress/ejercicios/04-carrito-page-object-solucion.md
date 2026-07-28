# Solución — Ejercicio 04: Carrito de compras con Page Object y fixtures

> ⚠️ Este es el ejercicio más largo — intenta resolverlo por tu cuenta, aunque te tome varios intentos, antes de leer esto.

## `cypress/fixtures/credenciales.json`

```json
{ "usuario": "standard_user", "clave": "secret_sauce" }
```

## `cypress/support/pages/LoginPage.js`

```javascript
class LoginPage {
  visitar() {
    cy.visit('https://www.saucedemo.com');
  }

  login(usuario, clave) {
    cy.get('#user-name').type(usuario);
    cy.get('#password').type(clave);
    cy.get('#login-button').click();
  }
}

export default new LoginPage();
```

## `cypress/support/pages/InventoryPage.js`

```javascript
class InventoryPage {
  agregarProducto(nombreProducto) {
    cy.get(`#add-to-cart-${nombreProducto}`).click();
  }

  obtenerContadorCarrito() {
    return cy.get('.shopping_cart_badge');
  }

  irAlCarrito() {
    cy.get('.shopping_cart_link').click();
  }

  obtenerProductosEnCarrito() {
    return cy.get('.inventory_item_name');
  }
}

export default new InventoryPage();
```

## `cypress/support/commands.js` (reto extra)

```javascript
import LoginPage from './pages/LoginPage';

Cypress.Commands.add('loginComoUsuarioValido', () => {
  cy.fixture('credenciales.json').then((credenciales) => {
    LoginPage.visitar();
    LoginPage.login(credenciales.usuario, credenciales.clave);
  });
});
```

## `cypress/e2e/carrito.cy.js`

```javascript
import InventoryPage from '../support/pages/InventoryPage';

describe('Carrito de compras', () => {
  beforeEach(() => {
    cy.loginComoUsuarioValido(); // custom command del reto extra
  });

  it('agrega dos productos y confirma que aparecen en el carrito', () => {
    InventoryPage.agregarProducto('sauce-labs-backpack');
    InventoryPage.agregarProducto('sauce-labs-bike-light');

    InventoryPage.obtenerContadorCarrito().should('have.text', '2');

    InventoryPage.irAlCarrito();

    InventoryPage.obtenerProductosEnCarrito()
      .should('contain', 'Sauce Labs Backpack')
      .and('contain', 'Sauce Labs Bike Light');
  });
});
```

### Versión sin el reto extra (login directo en `beforeEach`)

```javascript
import LoginPage from '../support/pages/LoginPage';
import InventoryPage from '../support/pages/InventoryPage';

describe('Carrito de compras', () => {
  beforeEach(() => {
    cy.fixture('credenciales.json').then((credenciales) => {
      LoginPage.visitar();
      LoginPage.login(credenciales.usuario, credenciales.clave);
    });
  });

  it('agrega dos productos y confirma que aparecen en el carrito', () => {
    InventoryPage.agregarProducto('sauce-labs-backpack');
    InventoryPage.agregarProducto('sauce-labs-bike-light');

    InventoryPage.obtenerContadorCarrito().should('have.text', '2');

    InventoryPage.irAlCarrito();

    InventoryPage.obtenerProductosEnCarrito()
      .should('contain', 'Sauce Labs Backpack')
      .and('contain', 'Sauce Labs Bike Light');
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Tu archivo `.cy.js` final tiene **cero** selectores hardcodeados repetidos? Todo debería pasar por los Page Objects.
- ¿Usaste el fixture para las credenciales, en vez de escribirlas directamente en el test?
- ¿`agregarProducto` construye el selector dinámicamente a partir del nombre del producto, sin duplicar el método por cada producto?

## Errores comunes al hacer este ejercicio

- Olvidar que las clases de Page Object en Cypress no reciben `cy` por constructor (a diferencia de Playwright) — simplemente se usa el objeto global `cy` directamente dentro de los métodos.
- Poner los `.should()` (aserciones) dentro del Page Object en vez del test — igual que en Selenium/Playwright, el Page Object expone información, el test decide si es correcta.
