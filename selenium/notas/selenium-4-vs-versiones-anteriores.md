# Selenium 4 vs Versiones Anteriores (Selenium 3 y anteriores)

## ¿Qué es Selenium?

Antes de comparar, un contexto rápido: Selenium es una herramienta para automatizar navegadores web. Se usa mucho para pruebas automatizadas de sitios web (verificar que los botones funcionen, que los formularios se envíen correctamente, etc.), pero también para tareas como web scraping.

## Principales diferencias

### 1. Protocolo W3C WebDriver (el cambio más importante)

- **Antes (Selenium 3 y anteriores):** Usaba un protocolo propio llamado JSON Wire Protocol (JWP) para comunicarse con los navegadores. Cada navegador interpretaba las instrucciones de forma un poco distinta, lo que generaba inconsistencias.
- **Selenium 4:** Adopta el estándar oficial **W3C WebDriver**. Esto significa que ahora Selenium "habla" el mismo idioma que entienden nativamente Chrome, Firefox, Edge, Safari, etc. Resultado: menos errores, más estabilidad y comportamiento más predecible entre navegadores.

### 2. Relative Locators (Localizadores relativos)

- **Antes:** Solo podías ubicar elementos con XPath, CSS, ID, etc., de forma "absoluta".
- **Selenium 4:** Puedes ubicar elementos relativos a otros, usando términos como:
  - `above()` (arriba de)
  - `below()` (debajo de)
  - `toLeftOf()` (a la izquierda de)
  - `toRightOf()` (a la derecha de)
  - `near()` (cerca de)

### 3. Soporte nativo para múltiples ventanas/pestañas

- **Antes:** Manejar varias pestañas era más engorroso.
- **Selenium 4:** Trae métodos más simples como `newWindow()` para abrir una nueva pestaña o ventana directamente.

### 4. Chrome DevTools Protocol (CDP)

- **Selenium 4** permite acceso directo a las herramientas de desarrollador de Chrome/Edge, lo que habilita:
  - Simular geolocalización
  - Capturar solicitudes de red (network requests)
  - Simular condiciones de red lentas (3G, sin conexión, etc.)
  - Emitir eventos de consola

### 5. Selenium Grid renovado

- **Antes:** Configurar Selenium Grid (para correr pruebas en paralelo en distintas máquinas/navegadores) era complicado, requería configurar "hub" y "nodos" manualmente.
- **Selenium 4:** Grid es mucho más fácil de desplegar, tiene una interfaz visual mejorada, soporta Docker de forma nativa y se integra mejor con Kubernetes.

### 6. Mejor soporte para pruebas visuales

Con Selenium 4 puedes tomar screenshots no solo de la página completa, sino de **elementos específicos**, algo que antes requería trucos o librerías externas.

### 7. Eliminación de clases obsoletas

Cosas como `findElementByXXX` (ej. `findElementById`) fueron deprecadas/eliminadas a favor de un enfoque más limpio y unificado con `findElement(By.xxx())`.

---

## Ejemplos de la vida cotidiana

Para que se entienda mejor, pensemos en Selenium como un "robot" que controla un navegador tal como lo haría una persona.

### Ejemplo 1: Compra en línea (E-commerce)
Imagina que trabajas en una tienda online y quieres asegurarte de que el botón "Agregar al carrito" funcione en Chrome, Firefox y Safari.
- **Antes de Selenium 4:** Podías tener resultados distintos en cada navegador porque el protocolo JWP se comunicaba diferente con cada uno (como hablar con 3 personas en 3 idiomas parecidos, pero no iguales).
- **Con Selenium 4:** Como usa el estándar W3C, es como si ahora todos hablaran el mismo idioma exacto. Menos "falsos errores" en las pruebas.

### Ejemplo 2: Banca en línea
Quieres verificar que el mensaje de error "Contraseña incorrecta" aparezca **justo debajo** del campo de contraseña.
- Con los **localizadores relativos** de Selenium 4, puedes escribir algo como: "busca el elemento que está *debajo* del campo de contraseña", en lugar de depender de un XPath complicado y frágil que se rompe si cambian el diseño.

### Ejemplo 3: Reserva de vuelos
Estás probando un sitio de reservas y necesitas simular que el usuario tiene una conexión a internet lenta (como en un aeropuerto con mal wifi) para ver si la página sigue cargando bien.
- Gracias al **CDP (Chrome DevTools Protocol)** en Selenium 4, puedes simular esa red lenta directamente desde el script, sin necesidad de herramientas externas.

### Ejemplo 4: Formulario de registro con múltiples pestañas
Al hacer clic en "Términos y condiciones", se abre una nueva pestaña.
- **Antes:** Cambiar el control entre pestañas requería más código.
- **Selenium 4:** Con `newWindow()` y el manejo mejorado de ventanas, es mucho más directo cambiar de una pestaña a otra, como si simplemente "hicieras clic" en la pestaña correcta.

---

## En resumen (tabla comparativa)

| Característica | Selenium 3 | Selenium 4 |
|---|---|---|
| Protocolo | JSON Wire Protocol | W3C WebDriver (estándar) |
| Localizadores | Solo absolutos (XPath, CSS) | Absolutos + relativos (`above`, `near`, etc.) |
| Acceso DevTools | No | Sí (CDP) |
| Selenium Grid | Configuración manual compleja | Más simple, con UI y soporte Docker/K8s |
| Screenshots | Solo página completa | Página completa + elementos específicos |
| Manejo de ventanas | Más código necesario | Métodos nativos simplificados |
