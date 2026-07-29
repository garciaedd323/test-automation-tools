# Page Object Model (POM) para principiantes

## ¿Qué es esto y por qué debería importarme?

Si vas a escribir pruebas automatizadas de cualquier tipo (web, móvil, lo que sea), tarde o temprano vas a escribir la misma prueba una y otra vez, ligeramente distinta. Page Object Model (POM) es la forma más común y más probada de **organizar ese código para que no se vuelva un desastre** a medida que crece.

No es una herramienta que se instala. No es un lenguaje de programación. Es **una forma de ordenar tu código** — un patrón de organización, como tener cajones etiquetados en vez de tirar todo en una caja revuelta.

---

## El problema, explicado sin tecnicismos

Imagina que tienes que probar una página web de login **cien veces distintas** (con distintos usuarios, contraseñas correctas e incorrectas, etc.). Cada vez que pruebas, tienes que:

1. Encontrar la casilla de "usuario"
2. Encontrar la casilla de "contraseña"
3. Encontrar el botón de "entrar"

Si escribes el código **sin ningún orden**, terminas con instrucciones como estas repetidas cien veces:

```
busca la casilla que tiene el id "user-box"
busca la casilla que tiene el id "pass-box"
busca el botón que tiene el id "btn-login"
```

**Ahora imagina que el diseñador de la página cambia el `id` de la casilla de usuario.** Hay que ir a corregir eso... en las 100 copias de código que se escribieron. Una locura.

---

## La solución, en una frase

> **Guarda en un solo lugar la información de dónde están las cosas, y en otro lugar separado, lo que quieres hacer con ellas.**

Ese "lugar donde vive la información de dónde están las cosas" es el **Page Object**. El "otro lugar donde se decide qué hacer" es **la prueba** (el test).

---

## La analogía del control remoto universal

Imagina un control remoto universal para la TV, el equipo de sonido, y el aire acondicionado.

- **El control remoto** = el Page Object (algo que "conoce" la pantalla).
- **Los botones del control** = los métodos (`prender()`, `subirVolumen()`, `cambiarCanal()`).
- **Los cables internos que conectan el botón con el aparato real** = los "locators" (dónde está exactamente cada botón físico dentro del aparato).

El usuario **solo aprieta el botón**. No le importa cómo está cableado por dentro. Si mañana el fabricante cambia el cableado interno del televisor, **se sigue apretando el mismo botón** — alguien actualiza el cableado interno una sola vez, pero la forma de usarlo no cambia.

---

## Aplicado a un ejemplo de login (en palabras, sin código)

**Sin POM**, una prueba dice algo como:

> "Busca la casilla con el id 'user-box', escribe ahí 'juan'. Busca la casilla con el id 'pass-box', escribe ahí '1234'. Busca el botón con el id 'btn-login', y apriétalo."

**Con POM**, la prueba dice:

> "Usando el control remoto de la página de Login, haz login con usuario 'juan' y contraseña '1234'."

Y en **otro lugar separado** (el "control remoto"), alguien ya definió una sola vez:

> "Cuando alguien pida 'hacer login', hay que buscar la casilla 'user-box', escribir ahí el usuario, buscar 'pass-box', escribir la contraseña, y apretar 'btn-login'."

---

## Por qué esto es tan valioso — tabla resumen

| Sin POM | Con POM |
|---|---|
| El diseñador cambia un `id` → hay que arreglar 100 lugares | El diseñador cambia un `id` → se arregla **1 solo lugar** |
| La prueba es difícil de leer (puro detalle técnico) | La prueba se lee casi como una frase normal |
| Se copia y pega el mismo código de login en cada prueba | Se escribe "hacer login" y ya, se reutiliza siempre lo mismo |
| Un error de tipeo en un locator puede repetirse sin notarse | El locator vive en un solo lugar, más fácil de revisar |

---

## Ejemplo real y simple (en Python, por ser el más legible)

### El "control remoto" (el Page Object)

```python
# archivo: pagina_login.py

class PaginaLogin:
    def __init__(self, navegador):
        self.navegador = navegador  # el navegador que se está controlando

    def escribir_usuario(self, texto):
        casilla_usuario = self.navegador.find_element("id", "user-box")
        casilla_usuario.send_keys(texto)

    def escribir_contraseña(self, texto):
        casilla_clave = self.navegador.find_element("id", "pass-box")
        casilla_clave.send_keys(texto)

    def hacer_click_en_entrar(self):
        boton = self.navegador.find_element("id", "btn-login")
        boton.click()

    def hacer_login(self, usuario, clave):
        self.escribir_usuario(usuario)
        self.escribir_contraseña(clave)
        self.hacer_click_en_entrar()
```

Los tres "botones del control remoto" son `escribir_usuario`, `escribir_contraseña`, `hacer_click_en_entrar`. Y hay un cuarto botón "combo": `hacer_login`, que aprieta los otros tres en orden.

### La prueba de verdad (usando el control remoto)

```python
# archivo: prueba_login.py

from pagina_login import PaginaLogin

def probar_login_correcto(navegador):
    pagina = PaginaLogin(navegador)          # se toma el control remoto
    pagina.hacer_login("juan", "1234")       # se aprieta UN botón

    assert "Bienvenido" in navegador.page_source
```

Esta prueba nunca toca el cableado interno — solo aprieta botones. Se lee casi como una instrucción normal: *"se toma la página de login, se hace login con juan/1234, y se confirma que diga Bienvenido"*.

---

## La magia real: ¿qué pasa si algo cambia en la página?

Supongamos que el diseñador de la web cambia el `id` de la casilla de usuario, de `"user-box"` a `"username-field"`.

**Sin POM**, habría que buscar y corregir eso en cada una de las 100 pruebas existentes.

**Con POM**, se va a **un solo lugar** (`pagina_login.py`) y se cambia **una sola línea**:

```python
def escribir_usuario(self, texto):
    casilla_usuario = self.navegador.find_element("id", "username-field")  # ← solo cambia aquí
    casilla_usuario.send_keys(texto)
```

Las 100 pruebas **no se tocan para nada** — siguen diciendo `pagina.hacer_login("juan", "1234")`, exactamente igual que antes.

---

## Un segundo ejemplo: varias pantallas conectadas

Después del login, se entra a un "Panel principal". Se sigue el mismo patrón — otro control remoto:

```python
# archivo: pagina_panel.py

class PaginaPanel:
    def __init__(self, navegador):
        self.navegador = navegador

    def obtener_mensaje_bienvenida(self):
        elemento = self.navegador.find_element("id", "mensaje-bienvenida")
        return elemento.text
```

Y la prueba, uniendo las dos "pantallas":

```python
def probar_login_muestra_bienvenida(navegador):
    login = PaginaLogin(navegador)
    login.hacer_login("juan", "1234")

    panel = PaginaPanel(navegador)
    mensaje = panel.obtener_mensaje_bienvenida()

    assert mensaje == "Bienvenido, juan"
```

Cada pantalla tiene **su propio control remoto**. La prueba solo dice "en el login, haz esto" y "en el panel, revisa esto" — nunca necesita saber los detalles técnicos de cómo está construida cada pantalla por dentro.

---

## Reglas de oro que casi todos siguen

1. **Un Page Object no debería hacer verificaciones (`assert`).** Su trabajo es *actuar* sobre la pantalla y *devolver información* (como el texto de un mensaje) — quien decide si esa información es correcta o no es la prueba, no el Page Object.
2. **Los "cables" (locators) son privados, los "botones" (métodos) son públicos.** La prueba nunca debería necesitar saber el `id` exacto de un botón — solo debería poder decir "haz login" o "agrega este producto".
3. **Un método que navega a otra pantalla debería "entregar" esa nueva pantalla.** Así, después de "hacer login", el código puede seguir trabajando directamente con la pantalla nueva (el panel principal) sin tener que crearla desde cero por separado.
4. **Los nombres de los métodos describen la intención, no la acción técnica.** Es mejor `hacer_login()` que `clickear_boton_verde()` — el primero sobrevive aunque cambie el color del botón.

---

## Preguntas frecuentes de quien recién empieza

**¿Esto es exclusivo de un lenguaje de programación o una herramienta?**
No. Es una idea general que se puede aplicar en Selenium, Cypress, Playwright, Appium, y en Java, Python, JavaScript, C# — cualquier lenguaje que soporte "agrupar código relacionado" (lo que técnicamente se llama una clase, aunque el nombre no importa tanto como la idea).

**¿Tengo que usar POM desde el primer día que aprendo a automatizar?**
No necesariamente. Es totalmente válido escribir las primeras pruebas sueltas, sin ningún patrón, mientras se aprende cómo funciona una herramienta. POM se vuelve valioso **cuando el número de pruebas empieza a crecer** y repetir el mismo código en cada una se vuelve incómodo — ahí es el momento de organizar.

**¿Un Page Object representa siempre una "página completa"?**
No necesariamente. También se pueden crear Page Objects más pequeños para piezas reutilizables que aparecen en varias pantallas (un menú de navegación, un encabezado, un modal de confirmación) — la idea es la misma: agrupar "dónde están las cosas" y "qué se puede hacer con ellas" en un solo lugar reutilizable.

**¿Qué pasa si dos pantallas comparten un elemento (como un menú superior)?**
Se puede crear un Page Object separado solo para ese menú, y que las demás pantallas lo usen — evitando duplicar esos mismos locators en cada Page Object de página completa.

---

## Glosario rápido de esta nota

| Término | Qué significa aquí |
|---|---|
| **Page Object** | El "control remoto" — la clase que agrupa locators y acciones de una pantalla |
| **Locator** | La forma de encontrar un elemento específico en la pantalla (un `id`, una clase, un texto) |
| **Método / acción** | Un "botón" del control remoto — algo que el Page Object sabe hacer |
| **Base Page** | Una clase "padre" con funciones comunes a todas las pantallas (esperar, hacer clic, escribir), que los demás Page Objects heredan para no repetir código básico |
| **Assert / aserción** | La verificación de que algo salió como se esperaba — vive en la prueba, no en el Page Object |

---

## Por qué vale la pena aprenderlo bien desde el principio

Cuando una suite de pruebas crece de 5 a 500 casos, la diferencia entre tener POM bien aplicado y no tenerlo es la diferencia entre poder mantener el proyecto cómodamente, o tener que reescribir todo desde cero porque se volvió imposible de tocar sin romper algo. Vale la pena invertir el tiempo en entenderlo bien ahora, con pocas pruebas, antes de que el proyecto crezca.
