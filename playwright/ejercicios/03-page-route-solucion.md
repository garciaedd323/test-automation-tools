# Solución — Ejercicio 03: Interceptar red con page.route

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto. La URL exacta puede variar levemente si la app de ejemplo cambia.

```typescript
import { test, expect } from '@playwright/test';

test.describe('page.route con la Kitchen Sink de Cypress', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('https://example.cypress.io/commands/network-requests');
  });

  test('espía la petición real sin modificarla', async ({ page }) => {
    const [response] = await Promise.all([
      page.waitForResponse('**/comments/*'),
      page.locator('.network-btn').click(),
    ]);

    expect(response.status()).toBe(200);
  });

  test('mockea la respuesta con route.fulfill', async ({ page }) => {
    await page.route('**/comments/*', (route) =>
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          name: 'Comentario de prueba',
          body: 'Este es un comentario inventado para el test',
        }),
      })
    );

    await page.locator('.network-btn').click();

    await expect(page.locator('.network-comment')).toContainText(
      'Este es un comentario inventado para el test'
    );
  });

  test('simula un error de red', async ({ page }) => {
    await page.route('**/comments/*', (route) => route.abort('failed'));

    const [request] = await Promise.all([
      page.waitForRequest('**/comments/*'),
      page.locator('.network-btn').click(),
    ]);

    expect(request.url()).toContain('comments');
    // La petición se disparó pero fue abortada — no hay response exitosa que verificar.
  });
});
```

## Reto extra: modificar la petición saliente

```typescript
test('modifica la petición antes de continuar', async ({ page }) => {
  await page.route('**/comments/*', async (route) => {
    const headers = {
      ...route.request().headers(),
      'x-usuario-de-prueba': 'true',
    };
    console.log('Headers enviados:', headers); // confirmar en la consola del test runner
    await route.continue({ headers });
  });

  await page.locator('.network-btn').click();
});
```

## Puntos clave a revisar en tu solución

- ¿Tu `page.route()` está declarado antes del `.click()` en los tres tests?
- ¿Usaste `Promise.all()` para disparar la acción y esperar la respuesta al mismo tiempo, en vez de esperar primero?
- ¿El texto mockeado en el segundo test aparece exactamente igual en la aserción final?

## Errores comunes al hacer este ejercicio

- Esperar la respuesta (`page.waitForResponse`) **después** del clic en vez de en el mismo `Promise.all` — llegaría tarde si la petición ya resolvió muy rápido.
- Olvidar `contentType: 'application/json'` en `route.fulfill()` — sin esto, algunas apps no interpretan correctamente el `body` como JSON.
