# Selenium: ¿Qué es y para qué sirve?

**Selenium** es una herramienta (en realidad un conjunto de herramientas) de código abierto que permite **automatizar la interacción con navegadores web**. Es decir, le permite a un programa "controlar" un navegador como si fuera una persona haciendo clic, escribiendo texto, navegando entre páginas, etc., pero todo de forma automática mediante código.

## ¿Para qué se utiliza?

Los usos principales son:

1. **Pruebas automatizadas (testing)** — el uso más común. Las empresas de software prueban que sus páginas web funcionen correctamente sin tener que hacerlo manualmente cada vez.
2. **Web scraping** — extraer información de páginas web (aunque para esto también existen herramientas más ligeras como BeautifulSoup o Scrapy).
3. **Automatización de tareas repetitivas** en la web (llenar formularios, descargar reportes, etc.)
4. **Monitoreo de sitios web** — verificar que una página siga funcionando correctamente a lo largo del tiempo.

## ¿Cómo funciona?

Selenium funciona mediante un componente llamado **WebDriver**, que actúa como puente entre tu código y el navegador:

```
Tu código (Python, Java, JS...) → WebDriver → Navegador (Chrome, Firefox, etc.)
```

El WebDriver traduce tus instrucciones ("haz clic aquí", "escribe esto") en comandos que el navegador realmente entiende y ejecuta, y luego te devuelve el resultado (por ejemplo, el contenido de la página).

Componentes clave:
- **Selenium WebDriver**: el núcleo que controla el navegador.
- **Selenium IDE**: una extensión para grabar y reproducir acciones sin programar (ideal para principiantes).
- **Selenium Grid**: permite correr pruebas en muchos navegadores y sistemas operativos al mismo tiempo, en paralelo.

## Ejemplos cotidianos para entenderlo mejor

**Ejemplo 1: El "robot" que compra entradas**

Imagina que quieres comprar boletos para un concierto apenas se abran las ventas. En vez de estar refrescando la página a mano, con Selenium podrías programar un script que:
- Abra el navegador automáticamente
- Vaya a la página de venta
- Llene tus datos
- Haga clic en "comprar" en el instante exacto

**Ejemplo 2: El empleado que revisa 500 formularios**

Una empresa necesita verificar que su formulario de registro funcione en Chrome, Firefox y Safari, todos los días. En vez de que una persona lo haga manualmente en los tres navegadores, un script de Selenium lo hace en minutos y avisa si algo se rompió.

**Ejemplo 3: El "comprador comparador" de precios**

Quieres comparar precios de un producto en varias tiendas online. Selenium puede abrir cada página, buscar el producto, "leer" el precio mostrado y guardarlo en una lista, como si tú mismo estuvieras copiando el precio de cada sitio.

**Ejemplo 4: Pruebas de un banco**

Antes de lanzar una actualización de su app web, un banco usa Selenium para simular miles de veces que un usuario inicia sesión, transfiere dinero y cierra sesión — verificando que todo el flujo funcione sin errores antes de que un cliente real lo use.

## Ejemplo básico de código (Python)

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get("https://www.google.com")

buscador = driver.find_element(By.NAME, "q")
buscador.send_keys("Selenium Python")
buscador.submit()

print(driver.title)
driver.quit()
```

Este código abre Chrome, va a Google, escribe "Selenium Python" en el buscador, presiona enter y luego cierra el navegador — todo automáticamente.

## Ventajas y limitaciones

**Ventajas:**
- Soporta múltiples lenguajes (Python, Java, C#, JavaScript, Ruby)
- Funciona con casi todos los navegadores modernos
- Comunidad enorme y mucha documentación

**Limitaciones:**
- Puede ser lento comparado con herramientas más especializadas
- Los sitios con mucha protección anti-bots pueden detectarlo y bloquearlo
- Requiere mantenimiento porque si la página web cambia su diseño, el script puede dejar de funcionar
