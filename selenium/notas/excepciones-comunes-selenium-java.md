# Manejo de excepciones comunes en Selenium (Java)

Entender estas excepciones es la diferencia entre pasar horas adivinando por qué falla un test, y arreglarlo en 2 minutos porque sabes exactamente qué significa cada mensaje de error.

> **Analogía general:** cada excepción de Selenium es como un código de error que te da un electrodoméstico. Si tu lavadora marca "E4", no sirve de nada solo reiniciarla a ciegas una y otra vez — necesitas saber que "E4" significa "puerta mal cerrada" para saber exactamente qué revisar. Las excepciones de Selenium son iguales: cada una apunta a una causa específica, y una vez que aprendes a leerlas, dejan de ser un misterio.

---

## 1. `NoSuchElementException`

### Qué significa

Selenium buscó el elemento con el locator que le diste, **y no lo encontró en el DOM en absoluto** — ni siquiera oculto, simplemente no existe en ese momento.

```java
org.openqa.selenium.NoSuchElementException: no such element:
Unable to locate element: {"method":"css selector","selector":"#boton-guardar"}
```

> **Analogía:** es como ir a buscar una carta específica en un archivero, revisar carpeta por carpeta, y confirmar que **esa carta no está en ningún cajón, ni siquiera traspapelada**. No es que esté mal guardada — simplemente no existe ahí todavía, o nunca existió con ese nombre.

### Causas típicas

1. **El elemento carga de forma asíncrona** (AJAX/JS) y tu código lo busca antes de que aparezca.
2. **El locator está mal escrito** (typo en el ID, clase que cambió, XPath roto).
3. **El elemento está dentro de un iframe** y no hiciste `switchTo().frame()` antes de buscarlo.
4. **El elemento aparece condicionalmente** (por ejemplo, solo si el usuario tiene cierto rol) y esa condición no se cumple.

### Cómo depurarla

```java
// Paso 1: confirma que no es un problema de timing
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.presenceOfElementLocated(By.cssSelector("#boton-guardar")));

// Paso 2: si sigue fallando, verifica el HTML real en ese momento
System.out.println(driver.getPageSource());

// Paso 3: revisa si el elemento vive dentro de un iframe
List<WebElement> iframes = driver.findElements(By.tagName("iframe"));
System.out.println("Cantidad de iframes en la página: " + iframes.size());
```
> **Analogía:** primero confirmas si simplemente llegaste demasiado temprano al archivero (falta de espera). Si no es eso, revisas el archivero completo con tus propios ojos (`getPageSource`) para confirmar si la carta de verdad existe con ese nombre exacto. Y si sigue sin aparecer, revisas si quizás está guardada dentro de **otro archivero anidado** (un iframe) al que no habías entrado todavía.

---

## 2. `StaleElementReferenceException`

### Qué significa

Ya habías encontrado el elemento antes (tenías una referencia válida a un `WebElement`), pero **el DOM cambió después** (la página se recargó, un framework como React lo re-renderizó, hubo un submit) y esa referencia vieja **ya no apunta a nada real**.

```java
org.openqa.selenium.StaleElementReferenceException: stale element reference:
element is not attached to the page document
```

> **Analogía:** es como si tomaras una foto de un asiento específico en un teatro (guardas la referencia al `WebElement`), pero mientras tanto **reorganizaron todas las butacas del teatro** (el DOM se re-renderizó). Tu foto sigue siendo válida como recuerdo, pero ya no corresponde a ningún asiento físico real en la sala actual — necesitas ir a mirar de nuevo dónde quedó esa butaca.

### Causas típicas

1. Guardaste el `WebElement` en una variable, hiciste algo que refrescó la página (submit, reload, AJAX que reemplaza el DOM), y luego intentas usar la variable vieja.
2. Frameworks como React/Vue **destruyen y recrean** nodos del DOM incluso cuando visualmente parece "el mismo" elemento.
3. Iteraste sobre una lista de elementos (`findElements`), y mientras iterabas la página cambió a mitad de camino.

### Cómo depurarla

```java
// MAL — reutilizar una referencia vieja
WebElement boton = driver.findElement(By.id("btn-guardar"));
boton.click(); // esto dispara un re-render de la página
boton.click(); // 💥 StaleElementReferenceException — "boton" ya no existe

// BIEN — volver a localizar el elemento antes de cada uso
driver.findElement(By.id("btn-guardar")).click();
// ... si necesitas volver a interactuar con "el mismo" botón después de un cambio:
driver.findElement(By.id("btn-guardar")).click(); // se re-localiza, no reutiliza la referencia vieja
```

**Patrón robusto: reintentar automáticamente ante staleness**

```java
public void clickConReintento(WebDriver driver, By locator, int intentos) {
    for (int i = 0; i < intentos; i++) {
        try {
            driver.findElement(locator).click();
            return; // éxito, salimos
        } catch (StaleElementReferenceException e) {
            System.out.println("Elemento stale, reintentando... (" + (i + 1) + "/" + intentos + ")");
        }
    }
    throw new RuntimeException("No se pudo hacer click tras " + intentos + " intentos");
}
```
> **Analogía:** en vez de confiar ciegamente en tu foto vieja de la butaca, cada vez que necesitas sentarte **vuelves a preguntar en la entrada del teatro "¿dónde está la butaca 5B ahora?"**. Si la primera vez que preguntas te dicen que está en remodelación, simplemente vuelves a preguntar un momento después en vez de rendirte.

---

## 3. `TimeoutException`

### Qué significa

Le pediste a un `WebDriverWait` que esperara una condición específica, y **esa condición nunca se cumplió dentro del tiempo límite** que configuraste.

```java
org.openqa.selenium.TimeoutException: Expected condition failed:
waiting for visibility of element located by By.id: mensaje-exito (tried for 10 second(s))
```

> **Analogía:** es como esperar en la parada del bus con un límite mental de 10 minutos. Si el bus no llega en ese tiempo, no significa que el bus no exista en absoluto (como en `NoSuchElementException`) — simplemente **no llegó a tiempo**. Puede que llegue 2 minutos después de que te fuiste, o puede que nunca llegue porque canceló la ruta.

### Causas típicas

1. **Timeout demasiado corto** para lo que realmente tarda la aplicación (especialmente en CI, que suele ser más lento que tu máquina local).
2. **La condición esperada nunca se cumple** porque la acción previa falló silenciosamente (ej. un click no llegó a registrarse).
3. **Backend lento o caído** — el elemento jamás aparecerá porque la petición que lo generaría nunca respondió.
4. Estás esperando la condición equivocada (ej. esperas `visibility` de un elemento que en realidad seguirá `hidden` porque tu lógica de negocio no dispara lo que crees que dispara).

### Cómo depurarla

```java
try {
    wait.until(ExpectedConditions.visibilityOfElementLocated(By.id("mensaje-exito")));
} catch (TimeoutException e) {
    // Diagnóstico: ¿el elemento existe pero no es visible? ¿o no existe en absoluto?
    List<WebElement> elementos = driver.findElements(By.id("mensaje-exito"));
    if (elementos.isEmpty()) {
        System.out.println("El elemento NO existe en el DOM — revisa si la acción previa falló");
    } else {
        System.out.println("El elemento SÍ existe pero no es visible — revisa CSS/estado del elemento");
        System.out.println("display: " + elementos.get(0).getCssValue("display"));
    }
    throw e; // volvemos a lanzar para que el test siga fallando (con más info impresa)
}
```
> **Analogía:** cuando el bus no llega, en vez de solo encogerte de hombros, vas a la oficina de la ruta a preguntar: **"¿el bus fue cancelado por completo, o simplemente va con retraso?"**. Esa distinción (elemento inexistente vs. elemento existente-pero-no-visible) te dice si el problema es de timing o de lógica de la aplicación.

---

## 4. Diagrama comparativo

![Diagrama de excepciones comunes de Selenium](excepciones_selenium_diagrama.svg)

*(Diagrama ilustrativo: las tres excepciones representadas como situaciones de la vida cotidiana — la carta que nunca existió, la butaca reubicada, y el bus que no llegó a tiempo.)*

---

## 5. Tabla resumen para depuración rápida

| Excepción | Pregunta clave para depurar | Herramienta de diagnóstico |
|---|---|---|
| `NoSuchElementException` | ¿El elemento existe en el DOM en ese momento? ¿Está en un iframe? | `driver.getPageSource()`, revisar iframes, esperar `presenceOfElementLocated` |
| `StaleElementReferenceException` | ¿El DOM cambió después de localizar el elemento? | Volver a localizar en vez de reutilizar la variable; patrón de reintento |
| `TimeoutException` | ¿El elemento nunca apareció, o apareció mas tarde de lo esperado? | Revisar si existe pero no es visible; aumentar timeout; revisar red/backend |

---

## 6. Patrón general: capturar contexto extra al fallar

```java
public void diagnosticarFallo(WebDriver driver, Exception e) {
    System.out.println("=== DIAGNÓSTICO DE FALLO ===");
    System.out.println("Excepción: " + e.getClass().getSimpleName());
    System.out.println("URL actual: " + driver.getCurrentUrl());
    System.out.println("Título de la página: " + driver.getTitle());

    try {
        byte[] captura = ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
        Files.write(Paths.get("screenshots/diagnostico_" + System.currentTimeMillis() + ".png"), captura);
    } catch (Exception ex) {
        System.out.println("No se pudo capturar screenshot adicional");
    }
}
```
> **Analogía:** esto es como llenar un pequeño reporte de incidente cada vez que algo falla: qué pasó, dónde estabas parado (URL), y una foto del momento — igual que el tema anterior de screenshots, pero ahora conectado directamente a **cada tipo de excepción** para que el reporte tenga contexto útil desde el primer intento de depuración.

---

## 7. Errores de principiante comunes que agravan estas excepciones

1. **Envolver todo en un `try/catch` genérico y seguir sin investigar** — esto oculta el problema real en vez de arreglarlo; el test "pasa" pero no probó nada.
2. **Aumentar el timeout a un número enorme "para que deje de fallar"** sin entender la causa — a veces sí es necesario un timeout mayor, pero primero hay que confirmar que el problema es de timing y no de lógica rota.
3. **Usar `Thread.sleep()` para "solucionar" un `StaleElementReferenceException`** — el problema no es de tiempo, es de referencia; dormir no arregla una referencia rota, solo retrasa el momento del fallo.
4. **No volver a lanzar la excepción tras capturarla para depurar** — si haces `catch` para imprimir información pero no vuelves a lanzar el error (`throw e`), el test puede terminar "pasando" incorrectamente aunque el problema real siga sin resolverse.
