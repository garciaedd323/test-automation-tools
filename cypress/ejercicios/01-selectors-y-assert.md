# Ejercicio 01 — Tu primer test con selectors y assert

**Nivel:** 🟢 Básico
**Requiere haber leído:** ¿Qué es Cypress?, Instalación y setup, Anatomía de un test, Selectors

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

**[the-internet.herokuapp.com](https://the-internet.herokuapp.com)** — la misma app que ya usaste en los ejercicios de Selenium. Como es un sitio externo, no tiene atributos `data-cy` — para este ejercicio puntual usarás selectors CSS normales (en un proyecto real, le pedirías al equipo de desarrollo que agregue `data-cy`, como ya viste en la nota de selectors).

## Pistas

<details>
<summary>Pista 1</summary>

`cy.visit()` acepta la URL completa. No necesitas configurar `baseUrl` para este ejercicio si vas a visitar dominios distintos en cada test.

</details>

<details>
<summary>Pista 2</summary>

Para el checkbox, inspecciona la página y busca el selector de los inputs dentro de `#checkboxes`. Recuerda que `.check()` es el comando específico de Cypress para checkboxes (no hace falta usar `.click()` ni verificar el estado antes).

</details>

<details>
<summary>Pista 3</summary>

Para el dropdown (un `<select>` nativo), Cypress tiene un comando específico: `.select('texto visible')` — no necesitas una clase auxiliar como `Select` en Selenium.

</details>

## Reto extra (opcional)

Agrega una tercera parte al test: en la misma página de checkboxes, verifica cuántos checkboxes hay en total usando `.should('have.length', ...)`.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [01-selectors-y-assert-solucion.md](./01-selectors-y-assert-solucion.md)
