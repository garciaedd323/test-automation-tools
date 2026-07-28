# Ejercicio 03 — Interceptar red con cy.intercept

**Nivel:** 🟡🔴 Intermedio-avanzado
**Requiere haber leído:** Interceptar requests de red con cy.intercept

---

## Objetivo

Escribir un test que:
1. Visite la página oficial de ejemplos de red de Cypress.
2. Intercepte la petición que se dispara al hacer clic en el botón "GET Comments" (sin modificarla), y confirme que responde con status 200.
3. En un segundo test, **mockee** esa misma petición para que devuelva datos inventados (por ejemplo, un solo comentario con un texto que tú elijas), y confirme que ese texto aparece en pantalla.
4. En un tercer test, simula que la petición falla (error de red), y confirme que la app muestra algún estado de error (o al menos que el `cy.wait()` de la petición fallida se resuelve con el error esperado).

## App de práctica

**[example.cypress.io/commands/network-requests](https://example.cypress.io/commands/network-requests)** — la app oficial de ejemplos de Cypress ("Kitchen Sink"), que incluye una sección hecha específicamente para practicar peticiones de red.

## Pistas

<details>
<summary>Pista 1</summary>

Antes de interactuar con cualquier botón de esta página, abre las herramientas de desarrollador del navegador y observa la pestaña "Network" al hacer clic manualmente — así confirmas la URL exacta que deberías interceptar.

</details>

<details>
<summary>Pista 2</summary>

Recuerda el orden correcto: `cy.intercept()` siempre se declara **antes** de la acción que dispara la petición (en este caso, antes del clic en el botón), no después.

</details>

<details>
<summary>Pista 3</summary>

Para el mock, no necesitas replicar la estructura completa de la respuesta real — alcanza con un JSON simplificado que tenga los campos que la interfaz realmente usa para mostrar el texto en pantalla (inspecciona el HTML resultante para saber qué campo se muestra).

</details>

## Reto extra (opcional)

Usa un `fixture` (archivo `.json` en `cypress/fixtures/`) para el mock del segundo test, en vez de escribir el JSON directamente inline en el test.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [03-cy-intercept-solucion.md](./03-cy-intercept-solucion.md)
