# Ejercicio 01 — Capabilities y tu primer tap

**Nivel:** 🟢 Básico
**Requiere haber leído:** ¿Qué es Appium?, Instalación y setup del entorno, Capabilities, Appium Inspector

---

## Objetivo

Escribir un test que:
1. Arranque una sesión de Appium apuntando a la app de práctica.
2. Toque el primer producto de la lista.
3. Confirme que se abrió la pantalla de detalle del producto (por ejemplo, verificando que aparece el botón "Add To Cart").

## App de práctica

**[Sauce Labs "My Demo App" (Android)](https://github.com/saucelabs/my-demo-app-android/releases)** — descarga el `.apk` de la última release. Es una app de catálogo de productos hecha oficialmente por Sauce Labs para practicar automatización (nativa, no una web).

> Si configuraste tu entorno para iOS en vez de Android, existe el equivalente: [my-demo-app-ios](https://github.com/saucelabs/my-demo-app-ios/releases). Los pasos son análogos, cambiando `automationName` a `XCUITest` y usando los locators correspondientes.

## Paso previo obligatorio: usa el Inspector primero

Antes de escribir una sola línea de test, **abre Appium Inspector** con las capabilities de abajo, navega manualmente por la app, y confirma:
- El `resource-id` o `accessibility id` del primer producto en la lista.
- El `resource-id` del botón "Add To Cart" en la pantalla de detalle.

No asumas que los IDs de ejemplo abajo son exactos — cada versión de la app puede variar levemente.

## Capabilities de partida

```java
DesiredCapabilities caps = new DesiredCapabilities();
caps.setCapability("platformName", "Android");
caps.setCapability("appium:automationName", "UiAutomator2");
caps.setCapability("appium:deviceName", "emulator-5554");
caps.setCapability("appium:app", "/ruta/a/mda.apk"); // ajusta el nombre real del apk descargado
```

## Pistas

<details>
<summary>Pista 1</summary>

Los productos de la lista suelen tener un `content-desc` que incluye el nombre del producto — confírmalo con el Inspector antes de escribir el locator.

</details>

<details>
<summary>Pista 2</summary>

Recuerda la jerarquía de locators ya vista: prioriza `accessibility id`/`resource-id` sobre coordenadas o XPath.

</details>

<details>
<summary>Pista 3</summary>

Para "confirmar que se abrió la pantalla de detalle", basta con verificar que un elemento característico de esa pantalla (el botón de agregar al carrito) esté presente — no necesitas comparar títulos de pantalla completos.

</details>

## Reto extra (opcional)

Agrega una espera explícita (`WebDriverWait`) antes de hacer el tap, en vez de asumir que la app ya terminó de renderizar la lista completa.

---

**¿Ya lo intentaste?** Compara tu solución aquí: [01-capabilities-y-tap-solucion.md](./01-capabilities-y-tap-solucion.md)
