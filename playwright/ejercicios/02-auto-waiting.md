# Ejercicio 02 — Auto-waiting con contenido dinámico

**Nivel:** 🟡 Intermedio
**Requiere haber leído:** Auto-waiting y web-first assertions

---

## Objetivo

Escribir un test que:
1. Visite la página de "Dynamic Loading" (Example 1).
2. Haga clic en el botón "Start".
3. Confirme que aparece un loader (spinner) mientras carga.
4. Confirme que, después de unos segundos, aparece el texto "Hello World!" — **sin usar `page.waitForTimeout()`**.
5. Repita el mismo flujo con el "Example 2" de la misma página.

## App de práctica

**[the-internet.herokuapp.com/dynamic_loading](https://the-internet.herokuapp.com/dynamic_loading)** — el mismo escenario que ya resolviste con Cypress. Vale la pena comparar directamente cómo se ve la misma espera en ambas herramientas.

## Pistas

<details>
<summary>Pista 1</summary>

`expect(locator).toBeVisible()` ya reintenta automáticamente — no necesitas envolver nada en un `waitForTimeout` antes.

</details>

<details>
<summary>Pista 2</summary>

Recuerda que Playwright también ofrece `page.waitForSelector()` como alternativa más explícita, pero en la mayoría de casos el `expect().toBeVisible()` es más idiomático y ya cubre lo mismo.

</details>

<details>
<summary>Pista 3</summary>

Las dos URLs son `/dynamic_loading/1` y `/dynamic_loading/2` — revisa los links de la página principal para confirmarlas.

</details>

## Reto extra (opcional)

Compara tu solución con la que ya escribiste en Cypress para este mismo ejercicio (`cypress/ejercicios/02-retry-ability.md`). Anota, en un comentario dentro de tu archivo de Playwright, qué tan distinta (o parecida) se ve la sintaxis final.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [02-auto-waiting-solucion.md](./02-auto-waiting-solucion.md)
