# Locators en Selenium: Estrategias para Encontrar Elementos

## ¿Qué es un Locator?

Un **locator** (localizador) es la forma en que le dices a Selenium **dónde** está el elemento con el que quieres interactuar en la página (un botón, un campo de texto, un enlace, etc.). Es como darle una dirección exacta a alguien para que encuentre una casa específica en una ciudad.

Selenium ofrece la clase `By` con varias estrategias para "apuntar" a un elemento.

## Las 8 estrategias de localización

### 1. `By.ID`
Busca por el atributo `id` del elemento HTML. Es **la opción más rápida y confiable**, porque los IDs deberían ser únicos en toda la página.

```python
driver.find_element(By.ID, "username")
```
```html
<input id="username" type="text">
```

### 2. `By.NAME`
Busca por el atributo `name`. Muy común en formularios, pero **no garantiza unicidad** (puede haber varios elementos con el mismo `name`, como en radio buttons o checkboxes agrupados).

```python
driver.find_element(By.NAME, "email")
```

### 3. `By.CLASS_NAME`
Busca por el atributo `class`. Ojo: si un elemento tiene varias clases (`class="btn btn-primary"`), debes usar solo **una** clase a la vez con este localizador (no `"btn btn-primary"` junto).

```python
driver.find_element(By.CLASS_NAME, "btn-primary")
```

### 4. `By.TAG_NAME`
Busca por la etiqueta HTML (`div`, `a`, `input`, `button`, etc.). Muy genérico, normalmente devuelve muchos resultados si usas `find_elements` (plural).

```python
driver.find_elements(By.TAG_NAME, "a")  # todos los enlaces de la página
```

### 5. `By.LINK_TEXT`
Busca un enlace (`<a>`) por su texto **exacto**.

```python
driver.find_element(By.LINK_TEXT, "Iniciar sesión")
```

### 6. `By.PARTIAL_LINK_TEXT`
Igual que el anterior, pero busca una **coincidencia parcial** del texto.

```python
driver.find_element(By.PARTIAL_LINK_TEXT, "Iniciar")
```

### 7. `By.CSS_SELECTOR`
Usa selectores CSS (los mismos que usarías en una hoja de estilos) para encontrar elementos. Es **muy potente y flexible**, y generalmente **más rápido que XPath**.

```python
driver.find_element(By.CSS_SELECTOR, "input#username")
driver.find_element(By.CSS_SELECTOR, ".btn-primary")
driver.find_element(By.CSS_SELECTOR, "div.form-group > input[type='email']")
```

### 8. `By.XPATH`
Usa expresiones XPath para navegar por la estructura del documento HTML/XML. Es **la opción más poderosa** (puede moverse hacia arriba, abajo, o buscar por texto interno), pero también **la más lenta y frágil**.

```python
driver.find_element(By.XPATH, "//input[@id='username']")
driver.find_element(By.XPATH, "//button[text()='Enviar']")
driver.find_element(By.XPATH, "//div[@class='form-group']/input")
```

---

## ¿Cuál conviene priorizar? (Orden de preferencia)

Existe una especie de "jerarquía" recomendada por la comunidad y buenas prácticas:

```
ID  >  CSS_SELECTOR  >  otros (NAME, CLASS_NAME, LINK_TEXT...)  >  XPATH (última opción)
```

### ¿Por qué este orden?

| Estrategia | Velocidad | Estabilidad | Legibilidad | Cuándo usarla |
|---|---|---|---|---|
| **ID** | Muy rápida | Muy estable | Muy clara | Siempre que exista un ID único |
| **CSS_SELECTOR** | Rápida | Estable | Clara | Cuando no hay ID, pero hay clases o estructura clara |
| **NAME / CLASS_NAME** | Rápida | Media (puede repetirse) | Clara | Formularios, cuando ID no está disponible |
| **TAG_NAME / LINK_TEXT** | Media | Media (depende del contenido) | Clara | Casos muy específicos, pocos elementos |
| **XPATH** | Lenta | Frágil (se rompe fácil con cambios de diseño) | Compleja | Solo cuando no hay otra opción (ej: buscar por texto o relaciones padre-hijo complejas) |

### Razones técnicas del orden:

1. **`ID` es el más rápido** porque el navegador tiene métodos nativos optimizados (`getElementById`) para buscar por ID directamente, sin tener que recorrer todo el árbol del DOM.

2. **`CSS_SELECTOR` es más rápido que `XPATH`** porque los navegadores modernos están altamente optimizados para procesar CSS (lo usan constantemente para renderizar estilos), mientras que XPath requiere un motor de evaluación distinto y más pesado.

3. **`XPATH` es el más frágil** porque muchas veces se escribe usando la posición o jerarquía exacta del HTML (ej: `//div[3]/span[2]/a`). Si el equipo de desarrollo cambia mínimamente la estructura de la página (agregan un `<div>` extra), el XPath se rompe. Además, es más difícil de leer y mantener.

4. Sin embargo, **XPath tiene superpoderes que CSS no tiene**, como:
   - Buscar por texto visible: `//button[text()='Enviar']`
   - Navegar hacia el elemento **padre**: `//input[@id='x']/parent::div`
   - Condiciones lógicas complejas: `//input[@type='text' and @disabled]`
   
   CSS Selector **no puede** subir al padre ni buscar por texto interno fácilmente, así que en esos casos específicos, XPath sigue siendo necesario.

---

## Ejemplo comparativo: mismo elemento, distintas estrategias

Supongamos este HTML:
```html
<div class="form-group">
    <label for="email">Correo electrónico</label>
    <input id="email-input" name="email" class="form-control" type="email">
</div>
```

```python
# 1. Mejor opción: ID (rápido, único, estable)
driver.find_element(By.ID, "email-input")

# 2. Buena alternativa: CSS_SELECTOR
driver.find_element(By.CSS_SELECTOR, "#email-input")
driver.find_element(By.CSS_SELECTOR, "input[name='email']")

# 3. Opción aceptable: NAME
driver.find_element(By.NAME, "email")

# 4. Última opción: XPATH (más lento y frágil)
driver.find_element(By.XPATH, "//input[@id='email-input']")
```

---

## Ejemplos de la vida cotidiana

### Ejemplo 1: Buscar una casa por dirección
- **`ID`** es como darle a alguien la dirección exacta con número de casa: *"Calle Falsa 123"*. Es inequívoco, directo, no hay margen de error.
- **`XPATH`** es como decirle: *"Ve por la avenida principal, gira en la tercera calle a la derecha, la tercera casa después de la panadería"*. Funciona, pero si cambian de lugar la panadería (o cambian el diseño de la página), las instrucciones ya no sirven.

### Ejemplo 2: Buscar un libro en una biblioteca
- **`ID`** es como el código único de catálogo de un libro (ej: "ISBN 978-0-123456"). Vas directo al estante correcto.
- **`CSS_SELECTOR`** es como decir "el libro de tapa azul en la sección de ciencia ficción". Más rápido que buscar por descripción larga, pero no tan directo como el ISBN.
- **`XPATH`** es como decir "el tercer libro del segundo estante de la cuarta fila". Funciona, pero si alguien reorganiza la biblioteca (cambia el diseño del sitio), pierdes el libro.

### Ejemplo 3: Encontrar a una persona en un evento
- **`ID`** es como llamarla por su número de cédula/DNI —inconfundible, un solo resultado.
- **`NAME` / `CLASS_NAME`** es como llamarla por su nombre de pila: puede haber más de una "María" en el evento (no es único).
- **`LINK_TEXT`** sería como identificarla por lo que dice su gafete: "Coordinador de Ventas" (buscas por el texto visible).
- **`XPATH`** sería como decir: "la tercera persona sentada en la segunda mesa, a la izquierda del que tiene camisa roja". Muy específico, pero si mueven las sillas, ya no la encuentras.

### Ejemplo 4: Pedido en un restaurante de comida rápida
- **`ID`** es como pedir por el número de combo: *"Quiero el Combo #5"*. Rápido, exacto, sin ambigüedad.
- **`CSS_SELECTOR`** es como decir: *"Quiero la hamburguesa con queso de la sección de combos"*. Un poco más descriptivo, pero igual de eficiente.
- **`XPATH`** es como decir: *"Quiero lo que pidió la persona que estaba frente a mí en la fila, el segundo ítem en su bandeja"*. Funciona, pero depende de muchos factores que pueden cambiar.

---

## Resumen final

| Prioridad | Estrategia | Cuándo usarla |
|---|---|---|
| 1 | `ID` | Siempre que el elemento tenga uno (más rápido y estable) |
| 2 | `CSS_SELECTOR` | Cuando no hay ID, pero hay clases, atributos o estructura clara |
| 3 | `NAME`, `CLASS_NAME`, `LINK_TEXT`, `TAG_NAME` | Casos específicos donde encajan naturalmente |
| 4 | `XPATH` | Última opción: solo si necesitas buscar por texto, navegar al padre, o lógica compleja que CSS no puede resolver |
