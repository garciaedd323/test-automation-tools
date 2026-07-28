# Ejercicio 01 — Tu primer test con getByRole

**Nivel:** 🟢 Básico
**Requiere haber leído:** ¿Qué es Playwright?, Instalación y setup, Anatomía de un test, Locators

---

## Objetivo

Escribir un test que:
1. Visite la página de checkboxes de práctica.
2. Marque el primer checkbox.
3. Confirme que quedó marcado.
4. Visite la página de dropdown.
5. Seleccione "Option 2".
6. Confirme que la opción seleccionada es la correcta.

## App de práctica

**[the-internet.herokuapp.com](https://the-internet.herokuapp.com)** — la misma app que ya usaste en los ejercicios de Selenium y Cypress. Como es un sitio externo sin atributos de accesibilidad perfectos, en algunos casos usarás `page.locator()` con CSS en vez de `getByRole` — es una buena oportunidad para notar cuándo el locator ideal no está disponible.

## Pistas

<details>
<summary>Pista 1</summary>

Inspecciona la página de checkboxes: los inputs no tienen un rol de accesibilidad explícito con nombre, así que probablemente necesites `page.locator('#checkboxes input')` en vez de `getByRole('checkbox', ...)`. Confírmalo abriendo las herramientas de desarrollador.

</details>

<details>
<summary>Pista 2</summary>

Para marcar el checkbox, Playwright tiene `.check()`, igual que Cypress — no necesitas verificar el estado antes como en Selenium.

</details>

<details>
<summary>Pista 3</summary>

Para el dropdown, usa `.selectOption('Option 2')` sobre el `<select>` — es el equivalente directo al `.select()` de Cypress.

</details>

## Reto extra (opcional)

Cuenta cuántos checkboxes hay en la página usando `.count()` sobre el locator, y verifica ese número con un `expect()`.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [01-getbyrole-basico-solucion.md](./01-getbyrole-basico-solucion.md)
