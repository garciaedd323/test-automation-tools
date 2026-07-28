# Ejercicio 02 — Retry-ability con contenido dinámico

**Nivel:** 🟡 Intermedio
**Requiere haber leído:** Reintentos automáticos (retry-ability)

---

## Objetivo

Escribir un test que:
1. Visite la página de "Dynamic Loading" (Example 1).
2. Haga clic en el botón "Start".
3. Confirme que aparece un loader (spinner) mientras carga.
4. Confirme que, después de unos segundos, aparece el texto "Hello World!" — **sin usar ningún tiempo fijo de espera**.
5. Repita el mismo flujo con el "Example 2" de la misma página (que oculta el elemento en vez de no renderizarlo — un caso sutilmente distinto).

## App de práctica

**[the-internet.herokuapp.com/dynamic_loading](https://the-internet.herokuapp.com/dynamic_loading)** — esta página fue creada específicamente para practicar esperas: el texto final tarda unos segundos en aparecer después de un clic, simulando una carga real.

## Pistas

<details>
<summary>Pista 1</summary>

No necesitas escribir ningún `cy.wait(numero)`. El retry-ability de `.should()` ya maneja la espera por ti — solo asegúrate de encadenar la aserción correcta (`should('be.visible')`, `should('contain', ...)`).

</details>

<details>
<summary>Pista 2</summary>

Para confirmar que el loader "estuvo presente" en algún momento, puedes verificarlo justo después del clic, antes de esperar el texto final.

</details>

<details>
<summary>Pista 3</summary>

"Example 1" y "Example 2" tienen URLs distintas (`/dynamic_loading/1` y `/dynamic_loading/2`) — revisa los links de la página principal para confirmar las rutas exactas.

</details>

## Reto extra (opcional)

Reduce el timeout por defecto a un valor muy bajo (`{ timeout: 500 }`) en la aserción del texto final, y observa cómo el test falla — luego vuelve a subirlo. Esto te ayuda a confirmar que el retry-ability realmente está esperando, no que el texto aparece instantáneamente.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [02-retry-ability-solucion.md](./02-retry-ability-solucion.md)
