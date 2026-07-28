# Solución — Ejercicio 02: Auto-waiting con contenido dinámico

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```typescript
import { test, expect } from '@playwright/test';

test.describe('Dynamic Loading', () => {
  test('Example 1: espera a que el elemento se renderice y aparezca', async ({ page }) => {
    await page.goto('https://the-internet.herokuapp.com/dynamic_loading/1');

    await page.locator('#start button').click();
    await expect(page.locator('#loading')).toBeVisible();

    // Auto-waiting real: sin waitForTimeout, solo la condición
    await expect(page.locator('#finish h4')).toBeVisible();
    await expect(page.locator('#finish h4')).toHaveText('Hello World!');
  });

  test('Example 2: espera a que el elemento oculto se vuelva visible', async ({ page }) => {
    await page.goto('https://the-internet.herokuapp.com/dynamic_loading/2');

    await page.locator('#start button').click();
    await expect(page.locator('#loading')).toBeVisible();

    await expect(page.locator('#finish h4')).toBeVisible();
    await expect(page.locator('#finish h4')).toHaveText('Hello World!');
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Tu test NO tiene ningún `page.waitForTimeout()`?
- ¿Verificaste el loader como paso intermedio?

## Comparación con la versión de Cypress (reto extra)

La lógica es prácticamente idéntica — la diferencia real está en la sintaxis: Cypress encadena comandos sin `await` (`cy.get(...).should(...)`), mientras que Playwright usa `async/await` real y separa el locator de la aserción (`expect(page.locator(...)).toBeVisible()`). Ambas formas tienen el mismo comportamiento de fondo: reintentar hasta cumplir la condición o agotar el timeout.

## Errores comunes al hacer este ejercicio

- Agregar `await page.waitForTimeout(3000)` "para estar seguro" — el mismo antipatrón ya visto en las tres herramientas.
- Usar `toBeAttached()` en vez de `toBeVisible()` en Example 2 — el elemento está "atado" al DOM desde el principio (solo oculto), así que `toBeAttached()` pasaría de inmediato sin esperar realmente a que se muestre.
