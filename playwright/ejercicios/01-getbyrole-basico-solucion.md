# Solución — Ejercicio 01: Tu primer test con getByRole

> ⚠️ Intenta resolverlo por tu cuenta antes de leer esto.

```typescript
import { test, expect } from '@playwright/test';

test.describe('Checkboxes y dropdown', () => {
  test('marca el primer checkbox y confirma que quedó marcado', async ({ page }) => {
    await page.goto('https://the-internet.herokuapp.com/checkboxes');

    const checkboxes = page.locator('#checkboxes input');
    await expect(checkboxes).toHaveCount(2); // reto extra

    await checkboxes.first().check();
    await expect(checkboxes.first()).toBeChecked();
  });

  test('selecciona Option 2 en el dropdown', async ({ page }) => {
    await page.goto('https://the-internet.herokuapp.com/dropdown');

    await page.locator('#dropdown').selectOption('Option 2');
    await expect(page.locator('#dropdown')).toHaveValue('2');
  });
});
```

## Puntos clave a revisar en tu solución

- ¿Notaste que aquí `page.locator()` con CSS fue la opción correcta, no por descuido sino porque la página no tiene roles de accesibilidad claros? Eso es exactamente la jerarquía de la nota de locators aplicada a un caso real.
- ¿Usaste `.check()` en vez de `.click()`?
- ¿Tu aserción de "quedó marcado" usa `toBeChecked()`, que ya trae el auto-waiting incorporado?

## Errores comunes al hacer este ejercicio

- Intentar forzar `getByRole('checkbox')` cuando el HTML real no expone ese rol correctamente — a veces la página real no está bien construida, y hay que caer al siguiente nivel de la jerarquía sin culpa.
- Usar `.selectOption('2')` (el value interno) pensando que es lo mismo que el texto visible — en este caso coinciden, pero no siempre será así.
