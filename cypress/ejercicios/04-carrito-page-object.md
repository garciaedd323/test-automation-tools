# Ejercicio 04 — Carrito de compras con Page Object y fixtures

**Nivel:** 🔴 Avanzado
**Requiere haber leído:** El primer test funcional completo (Cypress)

---

## Objetivo

Construir una suite completa (no solo un archivo suelto) que:
1. Inicie sesión en la tienda de práctica, usando credenciales guardadas en un `fixture`.
2. Agregue **dos productos distintos** al carrito.
3. Confirme que el contador del carrito muestra "2".
4. Vaya al carrito y confirme que ambos productos aparecen listados.

Todo esto usando **Page Object** (una clase o módulo de JS, como ya viste en la nota final de Cypress) — el test final no debería tener ningún `cy.get()` con selectores hardcodeados repetidos, solo llamadas a métodos del Page Object.

## App de práctica

**[saucedemo.com](https://www.saucedemo.com)** — la misma tienda que ya usaste en el ejercicio equivalente de Selenium.

## Pistas

<details>
<summary>Pista 1</summary>

Crea un fixture `cypress/fixtures/credenciales.json` con `{ "usuario": "standard_user", "clave": "secret_sauce" }`, y cárgalo con `cy.fixture('credenciales.json')` dentro de un `.then()`.

</details>

<details>
<summary>Pista 2</summary>

Vas a necesitar al menos dos Page Objects: uno para el login y otro para el inventario/carrito. Revisa la estructura de clase (`class LoginPage { ... }`) que ya construiste en la nota final de Cypress.

</details>

<details>
<summary>Pista 3</summary>

Cada botón "Add to cart" en `saucedemo.com` tiene un `id` que incluye el nombre del producto (`add-to-cart-sauce-labs-backpack`) — el mismo patrón que ya viste en el ejercicio equivalente de Selenium.

</details>

<details>
<summary>Pista 4</summary>

Estructura el test con `beforeEach` para el login (usando tu Page Object), y que el `it` en sí solo llame a métodos de alto nivel como `inventoryPage.agregarProducto('sauce-labs-backpack')`.

</details>

## Reto extra (opcional)

Crea un `custom command` (`cy.loginComoUsuarioValido()`) que envuelva el flujo de login completo usando el fixture, y úsalo en el `beforeEach` en vez de llamar directamente al Page Object.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [04-carrito-page-object-solucion.md](./04-carrito-page-object-solucion.md)
