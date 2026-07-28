# Ejercicio 02 — Localizar por texto con UiSelector y agregar al carrito

**Nivel:** 🟡 Intermedio
**Requiere haber leído:** Locators específicos de mobile

---

## Objetivo

Escribir un test que:
1. Abra la app y, en la lista de productos, **busque uno específico por su texto** (no el primero de la lista) usando `-android uiautomator`.
2. Toque ese producto para entrar al detalle.
3. Agregue el producto al carrito.
4. Confirme que el contador del carrito (ícono superior) muestra "1".

## App de práctica

Misma app del ejercicio 01 — [Sauce Labs "My Demo App" (Android)](https://github.com/saucelabs/my-demo-app-android/releases).

## Pistas

<details>
<summary>Pista 1</summary>

Usa el Inspector para confirmar el texto exacto de un producto que no sea el primero de la lista (por ejemplo, uno que necesite scroll para verse).

</details>

<details>
<summary>Pista 2</summary>

Repasa la sintaxis de `UiSelector` con `textContains()` o `text()` combinada con `className()` — el mismo patrón que ya viste en la nota de locators de mobile.

</details>

<details>
<summary>Pista 3</summary>

El contador del carrito suele vivir en la barra superior — confirma su `resource-id` exacto con el Inspector antes de escribir el assert.

</details>

## Reto extra (opcional)

Si el producto que elegiste no es visible sin hacer scroll, combina tu `UiSelector` con `UiScrollable` (como ya viste en la nota de locators) para que el propio Appium se encargue de desplazarse hasta encontrarlo, en vez de asumir que ya está en pantalla.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [02-uiselector-carrito-solucion.md](./02-uiselector-carrito-solucion.md)
