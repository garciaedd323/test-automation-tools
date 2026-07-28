# Ejercicio 01 — Tu primer locator y clic

**Nivel:** 🟢 Básico
**Requiere haber leído:** ¿Qué es Selenium?, Instalación y setup, Locators

---

## Objetivo

Escribir un test que:
1. Abra la página de práctica.
2. Encuentre el enlace "Form Authentication" usando un locator.
3. Haga clic en él.
4. Confirme que la URL cambió a la página de login.

## App de práctica

**[the-internet.herokuapp.com](https://the-internet.herokuapp.com)** — un sitio hecho específicamente para practicar automatización, con decenas de páginas distintas (formularios, alertas, dropdowns, tablas, etc.). Lo vas a usar en varios de estos ejercicios.

## Pistas (ábrelas solo si te atoras)

<details>
<summary>Pista 1</summary>

Inspecciona la página principal (`the-internet.herokuapp.com`) con las herramientas de desarrollador del navegador (clic derecho → Inspeccionar). Busca el enlace que dice "Form Authentication" y revisa qué atributo único tiene (¿tiene `id`? ¿solo texto?).

</details>

<details>
<summary>Pista 2</summary>

Si el enlace no tiene un `id` o `class` único, recuerda que puedes localizar un link por su texto visible con `By.linkText("...")`.

</details>

<details>
<summary>Pista 3</summary>

Para "confirmar que la URL cambió", puedes usar `driver.getCurrentUrl()` y comparar con un `assert` que contenga la palabra `login`.

</details>

## Reto extra (opcional)

Una vez que funcione, agrega una espera explícita (`WebDriverWait`) antes de hacer clic, en vez de asumir que el elemento ya está listo.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [01-primer-locator-y-clic-solucion.md](./01-primer-locator-y-clic-solucion.md)
