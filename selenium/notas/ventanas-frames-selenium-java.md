# Manejo de ventanas, pestañas y frames/iframes en Selenium (Java)

Este tema es clave porque Selenium, en cualquier momento dado, solo "ve" **un contexto a la vez** — una ventana/pestaña específica, o un frame específico dentro de ella. Si intentas interactuar con un elemento que está en un contexto distinto al que Selenium tiene activo, obtienes `NoSuchElementException` aunque el elemento esté perfectamente visible en pantalla para un humano.

> **Analogía general:** imagina que estás viendo la televisión y tienes varios canales grabándose en segundo plano. Aunque las grabaciones existen, tú solo puedes **ver e interactuar con el control remoto de un canal a la vez** — el que tienes sintonizado en ese momento. Cambiar de "contexto" en Selenium es como cambiar de canal: hasta que no cambias, es como si los otros canales no existieran para ti.

---

## 1. El concepto de "handle" (identificador de ventana)

Cada ventana o pestaña tiene un identificador único, el **window handle**. Es como el número de habitación de un hotel: no ves el nombre de la persona, solo un código que te permite referirte a esa habitación específica.

```java
String ventanaPrincipal = driver.getWindowHandle();
```
> **Analogía:** esto es anotar en un papel "estoy actualmente en la Habitación 101" antes de salir a explorar el resto del hotel. Si no anotas esto, después no sabrás cómo volver a tu punto de partida.

```java
Set<String> todasLasVentanas = driver.getWindowHandles();
```
> **Analogía:** esto es pedirle al recepcionista la lista completa de habitaciones ocupadas en el hotel en este momento — no te dice qué hay dentro de cada una, solo cuáles existen. Nota que en Java es un `Set<String>` (no garantiza orden), a diferencia de otros lenguajes donde puede venir como lista ordenada.

---

## 2. Cambiar entre pestañas/ventanas

### Caso típico: un link abre una nueva pestaña

```java
driver.findElement(By.linkText("Ver términos y condiciones")).click();
```
> **Analogía:** es como si, estando en la Habitación 101, alguien de repente abriera una puerta nueva al pasillo (la Habitación 102) para mostrarte algo — pero tú **sigues físicamente parado en la 101**. El hecho de que la nueva puerta exista no significa que ya estés adentro.

```java
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.numberOfWindowsToBe(2));
```
> **Analogía:** esperas a que el recepcionista confirme que ahora hay 2 habitaciones activas, en vez de asumir a ciegas que la puerta nueva ya terminó de abrirse (si actúas demasiado rápido, la "habitación" podría no estar lista todavía — de ahí el uso de un explicit wait en vez de asumir instantáneamente que ya se abrió).

```java
for (String handle : driver.getWindowHandles()) {
    if (!handle.equals(ventanaPrincipal)) {
        driver.switchTo().window(handle);
        break;
    }
}
```
> **Analogía:** recorres la lista de habitaciones que te dio el recepcionista, buscas la que **no** es la tuya original (la 101), y **físicamente caminas hacia allá**. Recién cuando "caminas" (`switchTo().window`) es que puedes tocar cosas ahí dentro.

```java
System.out.println(driver.getTitle());
WebElement elemento = driver.findElement(By.id("titulo-terminos"));
```
> **Analogía:** ahora que ya entraste a la Habitación 102, puedes ver lo que hay adentro y tocar los objetos (interactuar con los elementos). Antes de cambiar de ventana, esto habría fallado — como intentar tocar un mueble de la 102 mientras sigues parado en la 101.

### Volver a la ventana original

```java
driver.switchTo().window(ventanaPrincipal);
```
> **Analogía:** regresas caminando a la Habitación 101 usando el número que anotaste al principio. Sin ese número anotado, tendrías que adivinar cuál era tu habitación original entre todas las que existen.

### Cerrar la pestaña actual y volver

```java
driver.close();  // cierra SOLO la ventana/pestaña actual, no todo el navegador
driver.switchTo().window(ventanaPrincipal);
```
> **Analogía:** `driver.close()` es como cerrar la puerta de la Habitación 102 detrás de ti al salir — pero el hotel entero (`driver.quit()` sería derribar el hotel completo) sigue en pie. Aun así, después de cerrar la puerta, **tienes que decirle explícitamente al sistema "ahora estoy en la 101"**, porque Selenium no vuelve automáticamente a la ventana anterior solo porque cerraste la otra.

---

## 3. Frames / iframes

Un `<iframe>` es básicamente una página web completa **incrustada dentro de otra**. Cosas como formularios de pago (Stripe, PayPal), reCAPTCHA, o widgets de chat embebidos suelen vivir dentro de un iframe.

> **Analogía:** si una ventana normal es una habitación del hotel, un iframe es como una **caja fuerte empotrada dentro de esa habitación**, con su propia cerradura independiente. Aunque la caja fuerte esté físicamente dentro de la habitación 101 y la veas ahí, necesitas una llave distinta (`switchTo().frame`) para poder meter la mano y tocar lo que hay adentro. Si intentas `findElement` sin haber "abierto la caja fuerte" primero, Selenium te dirá que ese elemento no existe — aunque lo estés viendo justo enfrente tuyo.

### Cambiar al frame por nombre o ID

```java
driver.switchTo().frame("frame-pago");
```
> **Analogía:** metes la llave específica de esa caja fuerte (identificada por su nombre o ID) y la abres. Ahora tu "radio de acción" para buscar elementos se limita **solo al contenido de esa caja fuerte**.

### Cambiar al frame por elemento web

```java
WebElement frameElemento = driver.findElement(By.cssSelector("iframe.stripe-frame"));
driver.switchTo().frame(frameElemento);
```
> **Analogía:** en vez de usar un nombre grabado en la caja fuerte, primero **señalas físicamente cuál caja fuerte es** (la ubicas en la habitación) y luego la abres. Útil cuando la caja fuerte no tiene un nombre único, pero sí puedes identificarla por su posición o apariencia (un selector CSS).

### Cambiar al frame por índice

```java
driver.switchTo().frame(0);  // el primer iframe de la página
```
> **Analogía:** esto es como decir "abre la primera caja fuerte que encuentres en la habitación, sin importar su nombre" — funciona, pero es frágil: si el día de mañana agregan otra caja fuerte antes que esa, tu referencia por índice ya apunta a la caja equivocada.

### Interactuar dentro del frame

```java
WebElement campoTarjeta = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("numero-tarjeta"))
);
campoTarjeta.sendKeys("4242424242424242");
```
> **Analogía:** ahora que la caja fuerte está abierta, puedes meter la mano y manipular lo que hay adentro con toda normalidad — como si fuera cualquier otro elemento de una página normal.

### Volver al contenido principal

```java
driver.switchTo().defaultContent();
```
> **Analogía:** cierras la caja fuerte y **sales de vuelta al nivel de la habitación normal**. Esto es crucial: si después necesitas tocar un botón que está fuera del iframe (por ejemplo, un botón "Confirmar pago" que vive en la página principal, no dentro del formulario de Stripe), y no vuelves al contexto principal primero, Selenium seguirá buscando dentro de la caja fuerte y no lo va a encontrar.

### Subir un nivel (si hay frames anidados)

```java
driver.switchTo().parentFrame();
```
> **Analogía:** a diferencia de `defaultContent()` (que te saca de *todas* las cajas fuertes de una vez, hasta el nivel más externo), `parentFrame()` es como salir **solo un nivel** — útil si hay una caja fuerte dentro de otra caja fuerte (frames anidados), y quieres subir de la más interna a la intermedia, sin salir del todo hasta la habitación principal.

---

## 4. Manejo de alertas de JavaScript (bonus relacionado)

Las alertas (`alert`, `confirm`, `prompt`) de JavaScript son otro tipo de "contexto especial" — ni son una ventana normal ni un frame, y Selenium no puede interactuar con ellas usando `findElement`.

```java
wait.until(ExpectedConditions.alertIsPresent());
Alert alerta = driver.switchTo().alert();
```
> **Analogía:** es como si alguien tocara la puerta de tu habitación de hotel con un aviso urgente ("¿confirma que desea proceder?") que **bloquea todo lo demás** hasta que respondas. No puedes seguir usando el control remoto de la tele (interactuar con la página) hasta atender ese aviso.

```java
System.out.println(alerta.getText());
alerta.accept();   // equivalente a hacer click en "Aceptar" / "OK"
// alerta.dismiss(); // equivalente a hacer click en "Cancelar"
```
> **Analogía:** lees el aviso que dejaron en la puerta y decides si aceptas ("sí, adelante") o lo rechazas ("no, cancelar"). Solo después de responder puedes volver a tus asuntos normales dentro de la habitación.

```java
alerta.sendKeys("mi respuesta");  // solo aplica a un prompt() de JS
alerta.accept();
```
> **Analogía:** si el aviso en la puerta además te pedía escribir algo en un espacio en blanco (un `prompt`), primero llenas ese espacio y luego aceptas.

---

## 5. Errores comunes

| Error / síntoma | Causa típica | Solución |
|---|---|---|
| `NoSuchElementException` en un elemento que "sí está en pantalla" | El elemento está dentro de un iframe y no hiciste `switchTo().frame(...)` | Localiza el iframe y cambia el contexto antes de buscar el elemento |
| El test interactúa con la ventana equivocada tras un click que abre una nueva pestaña | No se hizo `switchTo().window(...)` tras el click | Esperar `numberOfWindowsToBe` y luego cambiar explícitamente de handle |
| `NoSuchWindowException` | Intentas interactuar con una ventana que ya se cerró | Guardar y validar los handles vigentes antes de cambiar de contexto |
| No puedes volver a interactuar con elementos "fuera" del formulario de pago | Te quedaste dentro del contexto del iframe | Llamar `driver.switchTo().defaultContent()` |
| `NoAlertPresentException` | Intentas manejar una alerta que no existe (o ya se cerró) | Esperar explícitamente con `ExpectedConditions.alertIsPresent()` antes de acceder a `driver.switchTo().alert()` |

---

## 6. Patrón recomendado (Java)

```java
import java.time.Duration;
import java.util.Set;
import java.util.function.Supplier;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.WebDriverWait;

public class VentanaUtils {

    public static String cambiarANuevaVentana(WebDriver driver, String ventanaOriginal, int timeoutSegundos) {
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(timeoutSegundos));
        wait.until(d -> d.getWindowHandles().size() > 1);

        for (String handle : driver.getWindowHandles()) {
            if (!handle.equals(ventanaOriginal)) {
                driver.switchTo().window(handle);
                return handle;
            }
        }
        throw new RuntimeException("No se encontró una nueva ventana");
    }

    public static <T> T conIframe(WebDriver driver, By frameLocator, Supplier<T> accion) {
        driver.switchTo().frame(driver.findElement(frameLocator));
        try {
            return accion.get();
        } finally {
            driver.switchTo().defaultContent();
        }
    }
}
```

Uso del patrón:

```java
String nuevaVentana = VentanaUtils.cambiarANuevaVentana(driver, ventanaPrincipal, 10);

String textoTarjeta = VentanaUtils.conIframe(driver, By.cssSelector("iframe.stripe-frame"), () -> {
    WebElement campo = driver.findElement(By.id("numero-tarjeta"));
    return campo.getText();
});
```

> **Analogía del patrón completo:** son como protocolos de un botones de hotel: "cada vez que entres a una caja fuerte o a otra habitación, anota de dónde saliste, haz lo que tengas que hacer, y **siempre vuelve a tu punto de partida antes de continuar con el resto de tus tareas**". Ese hábito evita que tu test se quede "perdido" en el contexto equivocado — sin importar si el código está en Python o en Java.
