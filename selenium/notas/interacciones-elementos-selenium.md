# Interacciones con elementos en Selenium

Ya que dominas las esperas, el siguiente paso es interactuar correctamente con los elementos. Aquí es donde aparecen errores como `ElementNotInteractableException`, `ElementClickInterceptedException` o clics que "no hacen nada" aunque el test no falle.

---

## 1. Clicks

### Click básico

```python
elemento = wait.until(EC.element_to_be_clickable((By.ID, "btn-enviar")))
elemento.click()
```

`element_to_be_clickable` verifica visibilidad + `enabled`, pero **no** garantiza que otro elemento (un modal, un header sticky, un toast) no esté cubriendo físicamente el botón. Si eso pasa, obtienes:

```
ElementClickInterceptedException: element click intercepted
```

### Alternativas cuando el click "normal" falla

**a) Click vía JavaScript** (bypassa la verificación de interceptación de Selenium, pero también bypassa la validación real del navegador — úsalo como último recurso, no como default):

```python
driver.execute_script("arguments[0].click();", elemento)
```

**b) Scroll hacia el elemento primero** (muchas veces el problema es que está fuera del viewport o cubierto por un header fijo):

```python
driver.execute_script("arguments[0].scrollIntoView({block: 'center'});", elemento)
elemento.click()
```

**c) Con `ActionChains`** (simula más fielmente el comportamiento de un usuario real, incluyendo mover el mouse):

```python
from selenium.webdriver.common.action_chains import ActionChains
ActionChains(driver).move_to_element(elemento).click().perform()
```

### Doble click y click derecho

```python
ActionChains(driver).double_click(elemento).perform()
ActionChains(driver).context_click(elemento).perform()  # click derecho
```

---

## 2. `send_keys` — escribir texto

```python
campo = wait.until(EC.visibility_of_element_located((By.ID, "email")))
campo.send_keys("usuario@correo.com")
```

### Detalles importantes

**Limpiar el campo antes de escribir** — si ya tiene texto, `send_keys` lo *agrega*, no lo reemplaza:

```python
campo.clear()
campo.send_keys("usuario@correo.com")
```

`clear()` a veces no funciona bien con inputs controlados por React (el estado interno del framework no se entera). Alternativa más robusta:

```python
campo.send_keys(Keys.CONTROL, "a")  # seleccionar todo
campo.send_keys(Keys.DELETE)
campo.send_keys("nuevo texto")
```

**Teclas especiales** (`Keys`):

```python
from selenium.webdriver.common.keys import Keys

campo.send_keys(Keys.ENTER)
campo.send_keys(Keys.TAB)
campo.send_keys(Keys.ESCAPE)
campo.send_keys(Keys.CONTROL, "c")  # copiar
```

**Subir archivos** — en un `<input type="file">`, no necesitas un file picker del sistema operativo; simplemente:

```python
input_archivo = driver.find_element(By.ID, "input-file")
input_archivo.send_keys("/ruta/absoluta/al/archivo.pdf")
```

Nota: el input debe estar presente en el DOM aunque esté visualmente oculto (`display:none` con CSS). Si el sitio lo elimina del DOM y usa un botón custom para abrir el explorador de archivos nativo, la automatización se vuelve mucho más difícil (ahí se recurre a herramientas del SO o AutoIT en Windows).

---

## 3. Dropdowns con `Select`

**Importante:** `Select` solo funciona con elementos `<select>` HTML nativos. Si el dropdown es un `<div>` custom (común en React con librerías como MUI, Ant Design, etc.), `Select` **no aplica** — hay que tratarlo como clicks normales sobre una lista desplegable (ver sección 3.1).

```python
from selenium.webdriver.support.ui import Select

elemento_select = wait.until(EC.presence_of_element_located((By.ID, "pais")))
select = Select(elemento_select)
```

### Formas de seleccionar una opción

```python
select.select_by_visible_text("Colombia")   # por el texto que ve el usuario
select.select_by_value("CO")                # por el atributo value="CO" en el <option>
select.select_by_index(2)                   # por posición (0-indexed)
```

### Leer información del select

```python
opcion_actual = select.first_selected_option
print(opcion_actual.text)

todas_las_opciones = select.options
for op in todas_las_opciones:
    print(op.text, op.get_attribute("value"))
```

### Multi-select (`<select multiple>`)

```python
select.select_by_visible_text("Rojo")
select.select_by_visible_text("Azul")  # se agrega, no reemplaza

seleccionadas = select.all_selected_options
print([o.text for o in seleccionadas])

select.deselect_by_visible_text("Rojo")
select.deselect_all()
```

### 3.1 Dropdown "custom" (no es un `<select>` real)

Estos suelen ser un `<div>` clickeable que despliega una lista de `<li>` o `<div>`:

```python
driver.find_element(By.ID, "dropdown-custom").click()          # abre el dropdown
opcion = wait.until(EC.element_to_be_clickable(
    (By.XPATH, "//li[text()='Colombia']")
))
opcion.click()
```

Aquí es común necesitar esperar a que la lista de opciones sea visible antes de intentar hacer click en una opción específica, porque a veces se renderiza con una animación.

---

## 4. Checkboxes

```python
checkbox = wait.until(EC.element_to_be_clickable((By.ID, "acepto-terminos")))
```

**Verificar estado antes de actuar** (evita marcar algo que ya está marcado, lo cual lo desmarcaría):

```python
if not checkbox.is_selected():
    checkbox.click()
```

Para **desmarcar**:

```python
if checkbox.is_selected():
    checkbox.click()
```

`is_selected()` funciona tanto para checkboxes como para radio buttons y opciones de `<select>`.

---

## 5. Radio buttons

Un grupo de radio buttons comparte el mismo `name`, pero cada uno tiene un `value` distinto. Solo puedes seleccionar, nunca "deseleccionar" un radio individual (se deselecciona automáticamente al elegir otro del mismo grupo).

```python
radios = driver.find_elements(By.NAME, "metodo_pago")

for radio in radios:
    if radio.get_attribute("value") == "tarjeta_credito":
        radio.click()
        break
```

O de forma más directa si conoces el selector exacto:

```python
driver.find_element(By.CSS_SELECTOR, "input[name='metodo_pago'][value='tarjeta_credito']").click()
```

Verificar cuál quedó seleccionado:

```python
seleccionado = [r for r in radios if r.is_selected()][0]
print(seleccionado.get_attribute("value"))
```

---

## 6. `ActionChains` — acciones complejas

`ActionChains` te permite construir una secuencia de acciones de bajo nivel (mover mouse, presionar, soltar, mantener tecla) que se ejecutan en orden con `.perform()`.

```python
from selenium.webdriver.common.action_chains import ActionChains

actions = ActionChains(driver)
```

### Hover (pasar el mouse por encima, sin hacer click)

Muy común para menús desplegables que se abren al pasar el mouse:

```python
menu = driver.find_element(By.ID, "menu-productos")
submenu = driver.find_element(By.ID, "submenu-electronica")

actions.move_to_element(menu).perform()
wait.until(EC.visibility_of(submenu))
submenu.click()
```

### Drag & drop

**Opción A — método directo** (funciona bien cuando el drag-and-drop es HTML5 nativo simple):

```python
origen = driver.find_element(By.ID, "elemento-arrastrable")
destino = driver.find_element(By.ID, "zona-destino")

actions.drag_and_drop(origen, destino).perform()
```

**Opción B — paso a paso** (más control, necesario cuando el sitio usa librerías JS complejas como SortableJS, react-dnd, etc., que a veces no responden bien al evento sintético de `drag_and_drop`):

```python
actions.click_and_hold(origen) \
       .move_to_element(destino) \
       .pause(0.5) \
       .release() \
       .perform()
```

**Opción C — con offset** (mover una cantidad específica de píxeles, útil para sliders):

```python
actions.drag_and_drop_by_offset(origen, xoffset=100, yoffset=0).perform()
```

> **Nota:** muchas implementaciones de drag & drop en frameworks modernos (React DnD, librerías con HTML5 Drag Events) no reaccionan bien a los eventos simulados de Selenium porque escuchan eventos de bajo nivel específicos del navegador. Si `ActionChains` no funciona, a veces se necesita inyectar los eventos manualmente vía JavaScript (`dispatchEvent` con `DragEvent`), lo cual es más avanzado y específico por caso.

### Mantener tecla presionada mientras haces otra acción

```python
actions.key_down(Keys.SHIFT) \
       .click(elemento1) \
       .click(elemento2) \
       .key_up(Keys.SHIFT) \
       .perform()
```

### Encadenar varias acciones en una sola llamada

```python
actions.move_to_element(elemento1) \
       .click() \
       .send_keys("texto") \
       .move_to_element(elemento2) \
       .click() \
       .perform()
```

Nada se ejecuta hasta llamar `.perform()` — hasta ese punto solo estás *construyendo* la secuencia.

---

## 7. Errores comunes y cómo resolverlos

| Error | Causa típica | Solución |
|---|---|---|
| `ElementNotInteractableException` | Elemento existe pero está oculto, con `display:none`, o fuera del viewport | Esperar `visibility_of_element_located`, hacer `scrollIntoView` |
| `ElementClickInterceptedException` | Otro elemento (modal, overlay, header sticky) tapa el que quieres clickear | Cerrar el overlay primero, o hacer scroll, o click con JS como último recurso |
| `StaleElementReferenceException` | El DOM se re-renderizó después de localizar el elemento | Volver a localizar el elemento justo antes de usarlo |
| `send_keys` no escribe nada visible | El campo es `readonly` o controlado por JS que sobreescribe el valor | Revisar atributos del input; a veces hay que disparar eventos JS adicionales (`input`, `change`) |
| `Select` lanza `UnexpectedTagNameException` | El elemento no es un `<select>` real | Es un dropdown custom — usar clicks normales (sección 3.1) |
| Drag & drop no mueve nada | El sitio usa eventos HTML5 DnD personalizados que `ActionChains` no dispara correctamente | Probar la versión paso a paso con `pause()`, o inyectar `DragEvent` vía JS |

---

## 8. Patrón recomendado (extendiendo el Page Object del tema anterior)

```python
class BasePage:
    def __init__(self, driver, timeout=10):
        self.driver = driver
        self.wait = WebDriverWait(driver, timeout)

    def click(self, locator):
        self.wait.until(EC.element_to_be_clickable(locator)).click()

    def type_text(self, locator, texto):
        campo = self.wait.until(EC.visibility_of_element_located(locator))
        campo.clear()
        campo.send_keys(texto)

    def select_by_text(self, locator, texto_visible):
        elemento = self.wait.until(EC.presence_of_element_located(locator))
        Select(elemento).select_by_visible_text(texto_visible)

    def set_checkbox(self, locator, marcar: bool):
        checkbox = self.wait.until(EC.element_to_be_clickable(locator))
        if checkbox.is_selected() != marcar:
            checkbox.click()

    def hover(self, locator):
        elemento = self.wait.until(EC.visibility_of_element_located(locator))
        ActionChains(self.driver).move_to_element(elemento).perform()
```

Esto te da métodos reutilizables y con la espera ya incorporada, en vez de repetir `wait.until(...)` en cada test.
