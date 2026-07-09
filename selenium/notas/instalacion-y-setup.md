# Instalación y Setup de Selenium

## 1. Instalación básica con pip (Python)

Para instalar Selenium en Python, es tan simple como:

```bash
pip install selenium
```

Esto instala la librería de Selenium, pero **no** instala automáticamente los navegadores ni sus drivers (aunque, como veremos, Selenium 4.6+ resuelve gran parte de ese problema).

Para verificar la versión instalada:
```bash
pip show selenium
```

## 1.1 Instalación en Java (con Maven)

Si trabajas con Java, en lugar de `pip` usas un gestor de dependencias como **Maven** o **Gradle**. Con Maven, agregas esto a tu archivo `pom.xml`:

```xml
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.21.0</version>
</dependency>
```

Con Gradle sería:

```groovy
implementation 'org.seleniumhq.selenium:selenium-java:4.21.0'
```

Luego, Maven/Gradle descarga automáticamente todas las librerías necesarias (similar a lo que hace `pip` en Python).

## 2. ¿Qué es un "driver" de navegador y por qué es necesario?

Selenium no controla el navegador directamente. Necesita un **intermediario** (driver) que traduzca las instrucciones de Selenium a acciones reales dentro del navegador.

- **Chrome** → necesita `chromedriver`
- **Firefox** → necesita `geckodriver`
- **Edge** → necesita `msedgedriver`
- **Safari** → usa `safaridriver` (viene integrado en macOS)

### El problema en Selenium 3 y versiones anteriores a 4.6

Antes tenías que:
1. Revisar manualmente qué versión de Chrome tenías instalada (ej: Chrome 114).
2. Ir a la página de descargas de ChromeDriver y buscar la versión **exacta** compatible (114).
3. Descargar el ejecutable.
4. Colocarlo en una carpeta específica o agregarlo al `PATH` del sistema.
5. Si Chrome se actualizaba automáticamente (cosa que hace solo), el driver quedaba desactualizado y las pruebas empezaban a fallar con errores como `SessionNotCreatedException`.

Esto era una fuente constante de dolores de cabeza, sobre todo en equipos donde cada desarrollador tenía una versión distinta de Chrome.

## 3. Selenium Manager (la solución desde Selenium 4.6)

Desde la versión **4.6**, Selenium incluye una herramienta llamada **Selenium Manager**, que se ejecuta automáticamente por debajo y hace lo siguiente:

1. Detecta qué navegador y versión tienes instalado en tu máquina.
2. Busca (o descarga) el driver exacto y compatible con esa versión.
3. Lo coloca en caché para no tener que descargarlo cada vez.
4. Configura todo para que tu script simplemente funcione.

### Código en Python: antes vs ahora

**Antes (Selenium 3 / principios de Selenium 4):**
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service

# Tenías que especificar la ruta manual al chromedriver descargado
service = Service(executable_path="C:/drivers/chromedriver.exe")
driver = webdriver.Chrome(service=service)
```

**Ahora (Selenium 4.6+):**
```python
from selenium import webdriver

# ¡Eso es todo! Selenium Manager se encarga del resto
driver = webdriver.Chrome()
driver.get("https://www.google.com")
```

Ya no necesitas descargar nada, ni preocuparte por rutas, ni por el `PATH`. Selenium Manager hace todo el trabajo detrás de escena.

### Código en Java: antes vs ahora

**Antes (Selenium 3 / principios de Selenium 4):**
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class Main {
    public static void main(String[] args) {
        // Tenías que especificar la ruta manual al chromedriver descargado
        System.setProperty("webdriver.chrome.driver", "C:/drivers/chromedriver.exe");
        WebDriver driver = new ChromeDriver();
        driver.get("https://www.google.com");
    }
}
```

**Ahora (Selenium 4.6+):**
```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class Main {
    public static void main(String[] args) {
        // ¡Eso es todo! Selenium Manager se encarga del resto
        WebDriver driver = new ChromeDriver();
        driver.get("https://www.google.com");
    }
}
```

Exactamente igual que en Python: ya no hace falta la línea `System.setProperty(...)` apuntando a una ruta local del driver. Selenium Manager detecta la versión de Chrome instalada y descarga el driver compatible automáticamente.

### ¿Y librerías como `webdriver-manager`?

Antes de Selenium Manager, muchos desarrolladores usaban una librería externa llamada `webdriver-manager` (en Python) para automatizar la descarga de drivers:

```bash
pip install webdriver-manager
```

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
```

En Java existía un equivalente llamado **WebDriverManager** (de Boni García), que se usaba así:

```java
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class Main {
    public static void main(String[] args) {
        WebDriverManager.chromedriver().setup();
        WebDriver driver = new ChromeDriver();
        driver.get("https://www.google.com");
    }
}
```

Con Selenium 4.6+, **estas librerías ya casi no son necesarias** para casos básicos, porque Selenium Manager cumple la misma función de forma nativa. Siguen siendo útiles en casos avanzados (versiones muy específicas, entornos corporativos con reglas particulares), pero para el 90% de los casos ya no hace falta.

---

## Ejemplos de la vida cotidiana

### Ejemplo 1: Comprar un electrodoméstico universal
Imagina que compras una lavadora y, en el pasado, tenías que buscar el adaptador de enchufe **exacto** para tu país, comprarlo por separado, e instalarlo tú mismo antes de poder usarla.
- **Antes de Selenium Manager:** Es como comprar la lavadora (Selenium) pero tener que ir a otra tienda a buscar el adaptador correcto (`chromedriver`) que combine exactamente con el voltaje de tu casa (versión de Chrome).
- **Con Selenium Manager:** Es como si la lavadora **detectara automáticamente** el tipo de enchufe de tu pared y viniera con el adaptador correcto ya integrado. Solo la conectas y funciona.

### Ejemplo 2: Actualización automática de Waze o Google Maps
Cuando tu app de mapas se actualiza sola en el celular, normalmente sigue funcionando bien porque el sistema se encarga de la compatibilidad.
- **Antes:** Era como si cada vez que Waze se actualizaba, tuvieras que ir manualmente a descargar un "traductor" nuevo (`chromedriver`) para que la app siguiera hablando con el GPS de tu teléfono. Si Chrome se actualizaba solo (cosa común) y tú no actualizabas el driver a tiempo, todo se rompía.
- **Ahora:** Selenium Manager es como si ese "traductor" se actualizara solo, sin que tengas que hacer nada.

### Ejemplo 3: Equipo de trabajo con distintas computadoras
En una empresa de QA (control de calidad), cada tester tiene su laptop con una versión de Chrome distinta (uno tiene la 120, otro la 122, otro la 118). Esto aplica igual si el equipo automatiza pruebas en Python o en Java.
- **Antes:** Cada persona debía descargar manualmente el `chromedriver` correspondiente a SU versión específica de Chrome. Un dolor de cabeza para el equipo de IT, sobre todo al hacer onboarding de gente nueva.
- **Con Selenium 4.6+:** Cada laptop, al correr el script (sea Python o Java), detecta su propia versión de Chrome y descarga automáticamente lo que necesita. Cero configuración manual, cero errores de "versión incompatible".

### Ejemplo 4: Pedido de comida a domicilio
Cuando pides comida por una app, no necesitas saber qué moto o repartidor específico te la traerá — la app lo asigna automáticamente según quién está disponible cerca.
- De la misma forma, ya no necesitas saber ni preocuparte por qué versión exacta de `chromedriver` necesitas: Selenium Manager "asigna" automáticamente el driver correcto según lo que detecta en tu sistema.

---

## Resumen rápido: pasos para instalar y correr tu primer script

### En Python

```bash
# 1. Instalar Selenium
pip install selenium
```

```python
# 2. Escribir tu script (sin preocuparte por drivers)
from selenium import webdriver

driver = webdriver.Chrome()  # Selenium Manager hace su magia aquí
driver.get("https://example.com")
print(driver.title)
driver.quit()
```

```bash
# 3. Ejecutar
python mi_script.py
```

### En Java

```xml
<!-- 1. Agregar la dependencia en pom.xml -->
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>4.21.0</version>
</dependency>
```

```java
// 2. Escribir tu script (sin preocuparte por drivers)
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

public class Main {
    public static void main(String[] args) {
        WebDriver driver = new ChromeDriver(); // Selenium Manager hace su magia aquí
        driver.get("https://example.com");
        System.out.println(driver.getTitle());
        driver.quit();
    }
}
```

```bash
# 3. Compilar y ejecutar (con Maven)
mvn compile exec:java -Dexec.mainClass="Main"
```

¡Eso es todo! No hay pasos extra de "ir a descargar el driver correcto" como antes, sin importar si trabajas en Python o en Java.
