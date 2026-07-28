# Ejercicio 04 — Carrito de compras con Page Object Model

**Nivel:** 🔴 Avanzado
**Requiere haber leído:** Page Object Model, Screenshots y evidencias, Excepciones comunes

---

## Objetivo

Construir una suite completa (no solo un método `main`) que:
1. Inicie sesión en la tienda de práctica.
2. Agregue **dos productos distintos** al carrito.
3. Confirme que el contador del carrito muestra "2".
4. Vaya al carrito y confirme que ambos productos aparecen listados.
5. Si cualquier paso falla, capture un screenshot automáticamente (usando lo que ya viste en la nota correspondiente).

Todo esto usando **Page Object Model** — el test final no debería tener ningún `driver.findElement()` directo, solo llamadas a métodos de tus Page Objects.

## App de práctica

**[saucedemo.com](https://www.saucedemo.com)** — una tienda de demostración hecha específicamente para practicar automatización de e-commerce.

Credenciales válidas:
- Usuario: `standard_user`
- Contraseña: `secret_sauce`

## Pistas

<details>
<summary>Pista 1</summary>

Vas a necesitar al menos 3 Page Objects: `LoginPage`, `InventoryPage` (la lista de productos) y `CartPage`. Revisa la estructura de `BasePage` que ya viste, y haz que los otros tres extiendan de ahí.

</details>

<details>
<summary>Pista 2</summary>

Cada producto en `saucedemo.com` tiene un botón "Add to cart" con un `id` que incluye el nombre del producto (ej. `add-to-cart-sauce-labs-backpack`). Inspecciona la página para confirmar el patrón exacto antes de escribir tus locators.

</details>

<details>
<summary>Pista 3</summary>

Para el screenshot automático en fallos, puedes usar JUnit 5 con la extensión `TestWatcher` que ya construiste en la nota correspondiente — no la reescribas desde cero, reutiliza ese código.

</details>

<details>
<summary>Pista 4</summary>

Estructura el test con JUnit 5: `@BeforeEach` para el login (usando tu `LoginPage`), y el método `@Test` en sí solo debería llamar métodos de alto nivel como `inventoryPage.agregarProducto("nombre")`.

</details>

## Reto extra (opcional)

Agrega un cuarto Page Object para el checkout, y extiende el test para completar una compra de principio a fin.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [04-carrito-page-object-model-solucion.md](./04-carrito-page-object-model-solucion.md)
