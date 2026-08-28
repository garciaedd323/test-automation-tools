# De `codegen` a tu primer test corriendo — el camino completo

## Para quién es este tutorial

Este es el recorrido **operativo, paso a paso**, sin teoría adicional — para cuando ya leíste las notas conceptuales y solo necesitas seguir la secuencia exacta de comandos y decisiones, de principio a fin, la primera vez que lo haces. Si algo no queda claro, cada paso enlaza a la nota conceptual correspondiente.

---

## Paso 0 — Tener el proyecto inicializado

Si todavía no existe la carpeta del proyecto, esto se hace una sola vez:

```bash
npm init playwright@latest
```

Responde las 4 preguntas interactivas (TypeScript o JavaScript, nombre de la carpeta de tests, si quieres el workflow de GitHub Actions, si instalar los navegadores ahora). Detalle completo en [instalación y setup](../notas/instalacion-setup-playwright.md).

Al terminar, tendrás esta estructura mínima:
```
mi-proyecto/
├── tests/
│   └── example.spec.ts
├── playwright.config.ts
└── package.json
```

> ⚠️ **Todo lo que sigue se corre desde dentro de esta carpeta**, nunca desde la raíz de tu usuario (`C:\Users\tu-usuario`). Confirma dónde estás con `dir` (Windows) o `pwd`/`ls` (Mac/Linux) antes de cualquier comando.

---

## Paso 1 — Abrir Codegen apuntando a la página que quieres automatizar

```bash
npx playwright codegen https://tu-sitio-de-prueba.com
```

Esto abre **dos ventanas**:
1. Un navegador real, en la URL indicada.
2. El **Playwright Inspector** — un panel aparte con el código que se va generando en vivo.

> Concepto de fondo: [¿Qué es Playwright?](../notas/que-es-playwright.md)

---

## Paso 2 — Interactuar con la página como lo haría un usuario real

Haz clic, escribe en campos, selecciona opciones — **exactamente como usarías la app normalmente**. Cada acción aparece reflejada como código en el Inspector, en tiempo real.

Ejemplo de lo que se genera al escribir en un campo y presionar Enter:
```ts
await page.getByPlaceholder('What needs to be done?').click();
await page.getByPlaceholder('What needs to be done?').fill('Comprar leche');
await page.getByPlaceholder('What needs to be done?').press('Enter');
```

---

## Paso 3 — Usar el Pick Locator para elementos difíciles

Si necesitas el selector de un elemento **sin hacerle clic** (por ejemplo, para usarlo luego en un `expect`), usa el botón de **Pick Locator** en la barra superior del Inspector, y pasa el mouse sobre el elemento. Playwright te muestra el locator recomendado antes de que hagas nada.

> Recuerda: Playwright prioriza `getByRole` sobre CSS/XPath — si vienes de Selenium, esta es la diferencia de filosofía más importante. Detalle completo en [Locators en Playwright](../notas/locators-playwright.md).

---

## Paso 4 — Grabar también la aserción (el paso que más se olvida)

Codegen graba **acciones** automáticamente, pero no aserciones a menos que se lo pidas explícitamente. En la barra de herramientas del Inspector hay un botón para grabar aserciones (ícono de "ojo" o similar, según versión) — actívalo y haz clic sobre el texto o elemento que quieres verificar que aparezca.

Esto genera algo como:
```ts
await expect(page.getByText('Comprar leche')).toBeVisible();
```

> Un test sin `expect` **hace cosas pero no verifica nada** — no sirve como test real.

---

## Paso 5 — Cerrar Codegen y copiar el código generado

Cierra la ventana del navegador o el Inspector. En la terminal donde corriste `codegen`, queda impreso el código completo generado durante la sesión. Cópialo — lo vas a pegar dentro de la estructura de un test real en el siguiente paso.

---

## Paso 6 — Crear el archivo de test con la estructura correcta

Codegen te da **líneas sueltas**, no un archivo de test completo. Hay que envolverlas:

```ts
import { test, expect } from '@playwright/test';

test('agregar una tarea nueva', async ({ page }) => {
  await page.goto('https://tu-sitio-de-prueba.com');

  await page.getByPlaceholder('What needs to be done?').fill('Comprar leche');
  await page.getByPlaceholder('What needs to be done?').press('Enter');

  await expect(page.getByText('Comprar leche')).toBeVisible();
});
```

Guárdalo dentro de la carpeta `tests/` (la que configuraste en el Paso 0), con extensión `.spec.ts` o `.test.ts`:
```
tests/mi-primer-test.spec.ts
```

> Detalle de cada pieza de esta estructura (`test()`, la fixture `page`, por qué no hay `beforeEach`/`afterEach` manual) en [Anatomía de un test](../notas/anatomia-test-playwright.md).

---

## Paso 7 — Correr el test

Desde la terminal, **en la raíz del proyecto** (donde está `package.json`):

```bash
npx playwright test
```

Variantes útiles según lo que necesites ver:

| Comando | Qué hace |
|---|---|
| `npx playwright test` | Corre todo en modo headless (sin ventana visible), en todos los navegadores configurados |
| `npx playwright test --headed` | Igual, pero mostrando el navegador mientras corre |
| `npx playwright test --debug` | Pausa antes de cada acción, con el Inspector abierto para avanzar paso a paso |
| `npx playwright test --ui` | Abre el UI Mode — la forma más visual de ver, repetir y depurar cada paso |
| `npx playwright test mi-primer-test.spec.ts` | Corre solo ese archivo, en vez de la suite completa |

---

## Paso 8 — Si algo falla, no adivinar — mirar la evidencia

Si el test falla, antes de tocar código:

```bash
npx playwright show-report
```

Esto abre el reporte HTML con capturas del momento exacto del fallo. El error casi siempre cae en una de estas causas, en este orden de probabilidad:

1. La página cargó, pero en un idioma/estado distinto al que se grabó con codegen (revisar el nombre accesible exacto del elemento).
2. El elemento está dentro de un iframe o aparece después de una carga asíncrona.
3. La URL no es accesible desde el entorno donde corre el test (probar abrirla a mano, fuera de Playwright).

> Caso real completo, con el proceso de diagnóstico documentado paso a paso: [Debugging real en Playwright](../notas/debugging-real-playwright.md).

---

## Resumen visual del camino completo

```
npx playwright codegen <url>
        │
        ▼
Interactuar con la página (clicks, forms, Pick Locator)
        │
        ▼
Grabar también la aserción (botón de aserciones en el Inspector)
        │
        ▼
Cerrar Codegen → copiar el código generado de la terminal
        │
        ▼
Pegarlo dentro de test('nombre', async ({ page }) => { ... })
        │
        ▼
Guardar como tests/archivo.spec.ts
        │
        ▼
npx playwright test  (o --headed / --debug / --ui)
        │
        ▼
¿Falló? → npx playwright show-report antes de cambiar código
```

---

## Checklist rápido para no saltarte nada

- [ ] Estoy parado en la carpeta del proyecto (no en la raíz del usuario) antes de correr cualquier comando.
- [ ] Grabé también la aserción, no solo las acciones.
- [ ] El archivo quedó dentro de `tests/` con extensión `.spec.ts`/`.test.ts`.
- [ ] Corrí `npx playwright test` desde la raíz del proyecto (donde está `package.json`).
- [ ] Si falló, revisé `show-report` antes de sospechar del código.
