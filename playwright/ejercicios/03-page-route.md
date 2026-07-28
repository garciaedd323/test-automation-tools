# Ejercicio 03 — Interceptar red con page.route

**Nivel:** 🟡🔴 Intermedio-avanzado
**Requiere haber leído:** Interceptar red con page.route

---

## Objetivo

Escribir un test que:
1. Visite la página oficial de ejemplos de red de Cypress (sí, funciona igual de bien para practicar Playwright — es solo una página web).
2. Intercepte la petición que se dispara al hacer clic en "GET Comments" (sin modificarla), y confirme que responde con status 200.
3. En un segundo test, **mockee** esa misma petición con `route.fulfill()` para que devuelva un comentario inventado por ti, y confirme que aparece en pantalla.
4. En un tercer test, simula que la petición falla (`route.abort()`), y confirme que la petición interceptada efectivamente no llegó a completarse con éxito.

## App de práctica

**[example.cypress.io/commands/network-requests](https://example.cypress.io/commands/network-requests)** — la misma app que usaste en el ejercicio equivalente de Cypress.

## Pistas

<details>
<summary>Pista 1</summary>

`page.route()` se declara antes de la acción que dispara la petición — el mismo orden que ya viste con `cy.intercept()`.

</details>

<details>
<summary>Pista 2</summary>

Para el segundo test, `route.fulfill({ status: 200, contentType: 'application/json', body: JSON.stringify({...}) })` reemplaza la respuesta completa.

</details>

<details>
<summary>Pista 3</summary>

Para confirmar la respuesta real del primer test, combina `page.waitForResponse()` con el clic usando `Promise.all()`, como ya viste en la nota de auto-waiting y en el primer test funcional.

</details>

## Reto extra (opcional)

Modifica el segundo test para que, en vez de reemplazar la respuesta completa con `fulfill()`, uses `route.continue({...})` agregando un header personalizado a la petición saliente, y confirma (con las herramientas de desarrollador o inspeccionando `route.request().headers()` dentro del callback) que el header se agregó correctamente.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [03-page-route-solucion.md](./03-page-route-solucion.md)
