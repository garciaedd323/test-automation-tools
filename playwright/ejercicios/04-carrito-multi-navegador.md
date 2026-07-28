# Ejercicio 04 — Carrito con Page Object, fixtures y multi-navegador

**Nivel:** 🔴 Avanzado
**Requiere haber leído:** El primer test funcional completo (Playwright)

---

## Objetivo

Construir una suite completa que:
1. Use una **fixture personalizada** (`paginaLogueada`) que haga login automáticamente antes de cada test, como ya viste en la nota de anatomía.
2. Agregue **dos productos distintos** al carrito, a través de un Page Object.
3. Confirme que el contador del carrito muestra "2".
4. Vaya al carrito y confirme que ambos productos aparecen listados.
5. Corra automáticamente en **los tres navegadores** configurados en `playwright.config.ts` (Chromium, Firefox, WebKit), sin cambiar una sola línea del test.

## App de práctica

**[saucedemo.com](https://www.saucedemo.com)** — la misma tienda que ya usaste en los ejercicios equivalentes de Selenium y Cypress.

## Pistas

<details>
<summary>Pista 1</summary>

Necesitas al menos dos Page Objects: `LoginPage` e `InventoryPage`/`CartPage`. Revisa la estructura de clase con constructor `(private page: Page)` que ya construiste en la nota final de Playwright.

</details>

<details>
<summary>Pista 2</summary>

La fixture personalizada va en un archivo separado (`fixtures.ts`) usando `base.extend<...>({...})`, y el test la importa desde ahí en vez de `@playwright/test` directamente.

</details>

<details>
<summary>Pista 3</summary>

Para confirmar que corre en los tres navegadores, solo necesitas correr `npx playwright test` sin ningún flag adicional — la configuración de `projects` ya debería estar corriendo los tres por defecto.

</details>

## Reto extra (opcional)

Agrega `test.step()` para dividir el test en pasos legibles ("Agregar productos", "Verificar contador", "Verificar carrito"), y revisa cómo se ve el reporte HTML generado (`npx playwright show-report`) con esos pasos marcados.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [04-carrito-multi-navegador-solucion.md](./04-carrito-multi-navegador-solucion.md)
