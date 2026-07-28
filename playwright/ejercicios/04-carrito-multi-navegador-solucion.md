# Solución — Ejercicio 04: Carrito con Page Object, fixtures y multi-navegador

> ⚠️ Este es el ejercicio más largo — intenta resolverlo por tu cuenta, aunque te tome varios intentos, antes de leer esto.

## `pages/LoginPage.ts`

```typescript
import { Page } from '@playwright/test';

export class LoginPage {
  constructor(private page: Page) {}

  async visitar() {
    await this.page.goto('https://www.saucedemo.com');
  }

  async login(usuario: string, clave: string) {
    await this.page.locator('#user-name').fill(usuario);
    await this.page.locator('#password').fill(clave);
    await this.page.locator('#login-button').click();
  }
}
```

## `pages/InventoryPage.ts`

```typescript
import { Page, Locator } from '@playwright/test';

export class InventoryPage {
  constructor(private page: Page) {}

  async agregarProducto(nombreProducto: string) {
    await this.page.locator(`#add-to-cart-${nombreProducto}`).click();
  }

  contadorCarrito(): Locator {
    return this.page.locator('.shopping_cart_badge');
  }

  async irAlCarrito() {
    await this.page.locator('.shopping_cart_link').click();
  }

  productosEnCarrito(): Locator {
    return this.page.locator('.inventory_item_name');
  }
}
```

## `fixtures.ts`

```typescript
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type Fixtures = { paginaLogueada: void };

export const test = base.extend<Fixtures>({
  paginaLogueada: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.visitar();
    await loginPage.login('standard_user', 'secret_sauce');
    await use();
  },
});

export { expect } from '@playwright/test';
```

## `carrito.spec.ts`

```typescript
import { test, expect } from './fixtures';
import { InventoryPage } from './pages/InventoryPage';

test.describe('Carrito de compras', () => {
  test('agrega dos productos y confirma que aparecen en el carrito', async ({
    page,
    paginaLogueada,
  }) => {
    const inventoryPage = new InventoryPage(page);

    await test.step('Agregar productos', async () => {
      await inventoryPage.agregarProducto('sauce-labs-backpack');
      await inventoryPage.agregarProducto('sauce-labs-bike-light');
    });

    await test.step('Verificar contador', async () => {
      await expect(inventoryPage.contadorCarrito()).toHaveText('2');
    });

    await test.step('Verificar carrito', async () => {
      await inventoryPage.irAlCarrito();
      const productos = inventoryPage.productosEnCarrito();
      await expect(productos).toContainText(['Sauce Labs Backpack', 'Sauce Labs Bike Light']);
    });
  });
});
```

## Correr en los tres navegadores

```bash
npx playwright test carrito.spec.ts
# Salida esperada: 3 tests pasaron (uno por cada project: chromium, firefox, webkit)

npx playwright show-report
# Muestra el reporte HTML con los test.step() marcados como sub-pasos
```

## Puntos clave a revisar en tu solución

- ¿Tu fixture `paginaLogueada` vive en un archivo separado, y el test importa `test`/`expect` desde ahí, no directamente de `@playwright/test`?
- ¿Tu `InventoryPage` retorna `Locator` en vez de hacer la aserción dentro del propio Page Object?
- ¿Confirmaste que el mismo archivo corrió tres veces (una por navegador) sin haber escrito ningún condicional para eso?

## Errores comunes al hacer este ejercicio

- Hacer la aserción (`expect`) dentro del Page Object en vez del test — rompe la misma regla ya vista en Selenium/Cypress: el Page Object expone información, el test decide si es correcta.
- Olvidar exportar `expect` desde el archivo de fixtures — si el test importa `expect` desde `@playwright/test` en vez de desde `./fixtures`, técnicamente funciona igual en este caso simple, pero rompe el patrón si más adelante se agregan más fixtures personalizadas.
