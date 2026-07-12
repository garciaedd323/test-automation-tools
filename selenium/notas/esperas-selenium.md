# Esperas en Selenium: Guía Completa (con analogías de la vida cotidiana)

Este es probablemente el tema #1 que separa un test estable de uno "flaky" (que a veces pasa y a veces falla sin razón aparente).

---

## El problema de fondo

Selenium ejecuta comandos más rápido de lo que el navegador renderiza contenido. Cuando haces clic en un botón que dispara una llamada AJAX, el elemento que buscas después puede no existir todavía en el DOM. Sin una estrategia de espera, tu test falla con `NoSuchElementException` o `ElementNotInteractableException` de forma intermitente.

---

## 1. `time.sleep()` — por qué es el enemigo

```python
time.sleep(5)
elemento = driver.find_element(By.ID, "resultado")
```

Problemas:
- **Lento**: siempre espera el tiempo completo, aunque el elemento aparezca en 0.5s.
- **Frágil**: si el servidor tarda 6s ese día (CI cargado, red lenta), tu test falla igual.
- No hay "cantidad correcta" de sleep — es adivinar a ciegas.

Úsalo *solo* para depurar manualmente, nunca en tests reales.

---

## 2. Implicit Wait

Se configura **una vez** por driver y aplica globalmente a `find_element`/`find_elements`.

```python
driver.implicitly_wait(10)  # segundos
```

Con esto, Selenium reintentará buscar el elemento hasta 10s antes de lanzar la excepción, en lugar de fallar de inmediato.

**Ventajas:** simple, una línea.

**Problemas serios:**
- Solo espera *presencia en el DOM*, no que sea clickeable, visible, etc.
- No te deja esperar condiciones específicas ("que el texto cambie", "que desaparezca un spinner").
- **Mezclarlo con Explicit Wait causa tiempos de espera impredecibles** (a veces suman, a veces hay comportamientos raros según versión de Selenium). La recomendación oficial es: **usa uno u otro, no ambos**.

Por esto, la mayoría de equipos serios evitan implicit wait y van directo a explicit.

---

## 3. Explicit Wait (`WebDriverWait` + `expected_conditions`)

Esta es la herramienta principal. Esperas una **condición específica**, con timeout y polling configurables.

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

wait = WebDriverWait(driver, 10)  # timeout máximo: 10s
elemento = wait.until(EC.presence_of_element_located((By.ID, "resultado")))
```

### Condiciones (`expected_conditions`) más usadas

```python
EC.presence_of_element_located((By.ID, "x"))       # existe en el DOM (no necesariamente visible)
EC.visibility_of_element_located((By.ID, "x"))     # existe Y es visible (display != none, size > 0)
EC.element_to_be_clickable((By.ID, "x"))           # visible Y habilitado (enabled)
EC.invisibility_of_element_located((By.ID, "x"))    # útil para esperar que desaparezca un spinner/loader
EC.text_to_be_present_in_element((By.ID, "x"), "texto")
EC.staleness_of(elemento)                           # el elemento viejo ya no está en el DOM (útil tras un refresh/AJAX)
EC.alert_is_present()
EC.number_of_windows_to_be(2)                        # para manejo de pestañas/popups
```

**Regla práctica:** para hacer clic, usa `element_to_be_clickable`. Para leer texto, `visibility_of_element_located`. `presence_of_element_located` es el más débil — el elemento puede existir pero estar oculto o cubierto por otro (te daría `ElementClickInterceptedException`).

### Condición personalizada

Cuando `expected_conditions` no cubre tu caso:

```python
def numero_de_filas_mayor_que(tabla_locator, minimo):
    def condicion(driver):
        filas = driver.find_elements(By.CSS_SELECTOR, f"{tabla_locator} tr")
        return len(filas) > minimo
    return condicion

wait.until(numero_de_filas_mayor_que("#tabla", 5))
```

O con una lambda simple:
```python
wait.until(lambda d: d.find_element(By.ID, "contador").text == "Completado")
```

### Manejo de timeout

```python
from selenium.common.exceptions import TimeoutException

try:
    elemento = wait.until(EC.element_to_be_clickable((By.ID, "btn")))
    elemento.click()
except TimeoutException:
    # falla con mensaje claro, screenshot, etc.
    driver.save_screenshot("timeout_error.png")
    raise
```

---

## 4. Fluent Wait

Es una versión más configurable del explicit wait: controlas el **intervalo de polling** y qué excepciones ignorar mientras esperas.

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import NoSuchElementException, StaleElementReferenceException

wait = WebDriverWait(
    driver,
    timeout=15,
    poll_frequency=0.5,  # revisa cada 0.5s
    ignored_exceptions=[NoSuchElementException, StaleElementReferenceException]
)

elemento = wait.until(EC.presence_of_element_located((By.ID, "resultado")))
```

En Python, `WebDriverWait` ya acepta `poll_frequency` e `ignored_exceptions` directamente — no necesitas una clase separada como en Java (`FluentWait`). Así que en la práctica, "fluent wait" en Python **es** `WebDriverWait` con esos parámetros extra.

**Cuándo te sirve el `poll_frequency` más largo:** si buscar el elemento es costoso (páginas muy pesadas) y no quieres martillar el DOM cada 0.5s, aunque en la mayoría de casos el default está bien.

**`ignored_exceptions` es clave** para elementos que se re-renderizan (React/Vue): sin esto, un `StaleElementReferenceException` a mitad de polling puede tirar abajo el wait antes de tiempo.

---

## Tabla comparativa rápida

| | Implicit | Explicit | Fluent |
|---|---|---|---|
| Alcance | Global (todo el driver) | Puntual (por llamada) | Puntual |
| Condición | Solo "existe en DOM" | Cualquiera de `EC` o custom | Cualquiera + polling custom |
| Configurable por caso | No | Sí | Sí, más granular |
| Recomendado para tests reales | No (o con mucho cuidado) | **Sí, por defecto** | Sí, casos especiales |

---

## Patrón recomendado en la práctica (Page Object Model)

```python
class BasePage:
    def __init__(self, driver, timeout=10):
        self.driver = driver
        self.wait = WebDriverWait(driver, timeout)

    def click(self, locator):
        self.wait.until(EC.element_to_be_clickable(locator)).click()

    def get_text(self, locator):
        return self.wait.until(EC.visibility_of_element_located(locator)).text

    def esperar_que_desaparezca(self, locator):
        self.wait.until(EC.invisibility_of_element_located(locator))
```

Esto centraliza la lógica de espera en un solo lugar y evita que cada test tenga sus propios `time.sleep()` regados por todos lados.

---

## Errores comunes que causan flakiness incluso usando explicit wait

1. **Esperar `presence` cuando necesitas `clickable`** → el elemento existe pero está tapado por un overlay/modal.
2. **No esperar que desaparezca un loader/spinner** antes de interactuar con lo que carga después.
3. **`StaleElementReferenceException`**: guardaste una referencia al elemento, la página se re-renderizó, y esa referencia ya no es válida. Solución: vuelve a localizar el elemento en vez de reusar la variable vieja.
4. **Mezclar implicit + explicit wait** → comportamiento inconsistente.
5. **Timeout demasiado corto para CI** pero que funciona en tu máquina local — considera timeouts algo generosos (10-15s), ya que el costo de esperar de más es mínimo comparado con un falso negativo.

---

# Ejemplos con analogías de la vida cotidiana

## Ejemplo 1: Tabla que carga por AJAX

Imagina que entras a un restaurante y pides la cuenta. El mesero no te la trae al instante — va a la cocina, calcula el total, y regresa. Si te levantas a leer la cuenta apenas la pediste, la mesa está vacía todavía.

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

driver.find_element(By.ID, "btn-cargar-datos").click()
```
> **Analogía:** esto es como llamar al mesero y pedir la cuenta. Acabas de "disparar" la acción, pero el resultado (la cuenta) todavía no existe en tu mesa.

```python
wait = WebDriverWait(driver, 10)
```
> **Analogía:** es decirle al mesero "voy a estar atento a la mesa durante los próximos 10 minutos, pero no voy a quedarme mirando fijo todo ese tiempo si la cuenta llega antes". Defines tu *paciencia máxima*.

```python
tabla = wait.until(
    EC.presence_of_element_located((By.ID, "tabla-resultados"))
)
```
> **Analogía:** esto es como esperar a que **la mesa exista** — es decir, que al menos hayan puesto el plato de la cuenta sobre la mesa, aunque todavía no puedas leer los números porque el mesero está tapando la vista con la bandeja. El elemento ya está en el DOM, pero no necesariamente listo para usarse.

```python
wait.until(
    lambda d: len(d.find_elements(By.CSS_SELECTOR, "#tabla-resultados tr")) > 1
)
```
> **Analogía:** ahora esperas a que la cuenta tenga **más de un renglón escrito** — no basta con que exista el papel, necesitas que ya tenga los platos listados (los datos ya llegaron, no solo el encabezado `<tr>` de la tabla vacía). Es como asomarte de reojo cada cierto tiempo ("¿ya escribió algo el mesero?") en vez de arrancarle la hoja de las manos apenas la puso en la mesa.

```python
filas = tabla.find_elements(By.TAG_NAME, "tr")
for fila in filas[1:]:  # nos saltamos el encabezado
    celdas = fila.find_elements(By.TAG_NAME, "td")
    print([c.text for c in celdas])
```
> **Analogía:** esto es leer la cuenta renglón por renglón, ignorando el título "CUENTA" de arriba (el encabezado), y anotando cada plato con su precio.

---

## Ejemplo 2: Spinner que aparece y desaparece

Ahora imagina que pides comida para llevar. Te dan un buzzer (vibrador) que se enciende mientras cocinan y se apaga cuando tu pedido está listo. No te acercas a la barra a preguntar mientras el buzzer sigue vibrando — sería absurdo e inútil, solo te dirán "todavía no".

```python
driver.find_element(By.ID, "btn-guardar").click()
```
> **Analogía:** es el momento en que haces el pedido y te entregan el buzzer. La cocina empieza a trabajar.

```python
wait = WebDriverWait(driver, 15)
```
> **Analogía:** decides que vas a esperar hasta 15 minutos antes de ir a reclamar en el mostrador porque algo salió mal ("¡oiga, llevo mucho esperando!").

```python
wait.until(
    EC.visibility_of_element_located((By.CLASS_NAME, "spinner-cargando"))
)
```
> **Analogía:** esto es confirmar que **el buzzer efectivamente se encendió** — es decir, que la cocina sí recibió tu pedido y empezó a trabajar. Si no esperas esto, corres el riesgo de revisar la mesa *antes* de que el buzzer siquiera se active, y podrías pensar erróneamente que tu comida "ya está lista" cuando en realidad ni empezaron a cocinar.

```python
wait.until(
    EC.invisibility_of_element_located((By.CLASS_NAME, "spinner-cargando"))
)
```
> **Analogía:** este es el momento clave — **esperas a que el buzzer deje de vibrar**. Recién ahí sabes que tu comida está lista para recoger. Si te acercas al mostrador mientras el buzzer sigue activo, el empleado te va a decir "todavía no, señor" (equivalente a un `ElementNotInteractableException` si intentas interactuar con algo que aún está cargando).

```python
mensaje_exito = wait.until(
    EC.visibility_of_element_located((By.ID, "mensaje-exito"))
)
assert "Guardado correctamente" in mensaje_exito.text
```
> **Analogía:** finalmente revisas la bolsa de comida y confirmas que trajeron lo que pediste ("Guardado correctamente" = "sí, es la hamburguesa que ordené y no otra cosa por error").

---

## Ejemplo 3: combinando todo, con manejo de error

```python
from selenium.common.exceptions import TimeoutException

try:
    wait.until(EC.invisibility_of_element_located((By.CLASS_NAME, "spinner-cargando")))
except TimeoutException:
    driver.save_screenshot("pedido_nunca_llego.png")
    raise AssertionError("El spinner nunca desapareció: la cocina se tardó demasiado")
```
> **Analogía:** esto es como decir "si pasan 15 minutos y el buzzer sigue encendido, algo está mal en la cocina — no me voy a quedar ahí parado para siempre". Tomas una foto de evidencia (el `screenshot`, como fotografiar el buzzer todavía encendido para reclamar después) y levantas la queja formalmente (`raise`), en lugar de quedarte esperando indefinidamente o asumiendo en silencio que todo salió bien.

---

## El patrón que se repite (y por qué importa)

Fíjate que en **ambos ejemplos** el flujo es el mismo:

1. **Disparas la acción** (pides la cuenta / haces el pedido) → clic.
2. **Esperas una condición de "inicio"** (que la mesa tenga algo / que el buzzer se encienda) → confirmas que el proceso arrancó.
3. **Esperas una condición de "fin"** (que la tabla tenga filas / que el buzzer se apague) → confirmas que terminó.
4. **Recién ahí interactúas** con el resultado.

Si te saltas el paso 2 o el 3, tu test se comporta como alguien impaciente en un restaurante: a veces "le da suerte" y el plato ya estaba listo, y otras veces se lleva un tenedor vacío porque llegó demasiado pronto — exactamente el comportamiento *flaky* que estamos tratando de eliminar.
