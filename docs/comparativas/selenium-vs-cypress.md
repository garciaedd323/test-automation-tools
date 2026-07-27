# Selenium vs Cypress: comparativa completa

## La analogía general

Elegir entre Selenium y Cypress es como elegir entre **contratar una agencia de seguridad con sucursales en todo el mundo** (Selenium) o **contratar a un guardaespaldas personal que vive contigo y conoce cada rincón de tu casa** (Cypress). Ambos "protegen" (prueban) la aplicación, pero con filosofías de trabajo completamente distintas — y la decisión correcta depende de qué se necesita proteger y en qué condiciones.

---

## 1. Resumen ejecutivo

| Si la prioridad es... | Elegir... |
|---|---|
| Velocidad de ejecución y feedback rápido | Cypress |
| Cobertura de navegadores (incluyendo Safari real, navegadores legacy) | Selenium |
| Testing mobile nativo (Android/iOS) | Selenium + Appium |
| Simular respuestas de red / mockear el backend fácilmente | Cypress |
| Equipo con experiencia en Java/Python/C# (no solo JS) | Selenium |
| Un solo lenguaje para todo el equipo (frontend + QA) | Cypress |
| Proyectos con mucha automatización cross-browser en paralelo | Selenium (+ Grid) |

---

## 2. Arquitectura — la raíz de todas las demás diferencias

Selenium habla por HTTP con un servidor externo (WebDriver); Cypress vive dentro del navegador, en el mismo proceso que la app.

> **Analogía:** Selenium es la agencia de seguridad que **llama por radio desde una central** para coordinar a sus guardias en cualquier sucursal del mundo — funciona en todos lados, pero cada instrucción tarda un poco en llegar. Cypress es el guardaespaldas que **vive en la misma casa** que protege — reacciona al instante porque nunca salió de ahí, pero solo conoce esa casa específica, no las 50 sucursales de la empresa.

Casi todas las diferencias siguientes son consecuencia directa de esta decisión arquitectónica de origen.

---

## 3. Velocidad de ejecución

Cypress es notablemente más rápido en la mayoría de escenarios, porque no paga el costo de la latencia de red en cada comando.

> **Analogía:** es la diferencia entre pedirle algo a alguien que está **en la misma habitación** (respuesta inmediata) versus pedírselo **por radio a través de la central** (siempre hay una fracción de segundo de "viaje" en cada mensaje). En una suite con miles de comandos, esa fracción de segundo por comando se acumula en minutos reales de diferencia.

---

## 4. Lenguajes soportados

| | Selenium | Cypress |
|---|---|---|
| Lenguajes | Java, Python, JavaScript/TypeScript, C#, Ruby, Kotlin | Solo JavaScript/TypeScript |

> **Analogía:** la agencia de seguridad global (Selenium) contrata personal que hable **cualquier idioma según el país** — se adapta al equipo que ya existe. El guardaespaldas personal (Cypress) solo habla un idioma, pero lo domina perfectamente — si el equipo entero ya habla ese idioma (JavaScript, típico en equipos frontend), no hay pérdida; si el equipo QA viene de un mundo Java/Python, hay una barrera de entrada real.

---

## 5. Multi-navegador y multi-dominio

Selenium soporta de forma nativa Chrome, Firefox, Safari (WebKit real, no solo el motor), Edge, e incluso navegadores legacy en configuraciones empresariales. Cypress soporta Chrome/Edge/Firefox bien, pero su soporte de Safari históricamente ha sido más limitado, y ya se vio la limitación de multi-dominio derivada de su arquitectura.

> **Analogía:** la agencia global tiene **sucursales certificadas en cada país**, incluyendo los más antiguos y particulares. El guardaespaldas personal se mueve rapidísimo dentro de su territorio conocido, pero cruzar a "otra propiedad con dueño distinto" (otro dominio) requiere trámites especiales que la agencia global nunca necesitó.

---

## 6. Interceptar red / mocking

Aquí Cypress tiene una ventaja clara y nativa con `cy.intercept()`, algo que en Selenium requiere herramientas externas (proxies, DevTools Protocol) para lograr un resultado comparable.

> **Analogía:** el guardaespaldas que vive en la casa puede **escuchar y hasta intervenir la línea telefónica interna** sin instalar nada extra — está ahí mismo. La agencia externa necesitaría **pedir permiso a la compañía telefónica** e instalar equipo adicional para lograr lo mismo.

---

## 7. Testing mobile

Selenium tiene una extensión natural hacia mobile: **Appium**, que reutiliza el mismo protocolo WebDriver y buena parte de los conceptos ya aprendidos (esperas, locators, Page Object Model). Cypress **no tiene un equivalente mobile nativo** — su arquitectura de "vivir dentro del navegador" no se traduce a apps nativas de Android/iOS.

> **Analogía:** la agencia global de seguridad tiene una **división hermana especializada en protección de vehículos en movimiento** (Appium), que usa el mismo manual de operaciones. El guardaespaldas personal, en cambio, solo sabe operar dentro de una casa fija — no tiene forma de subirse a un vehículo en movimiento (una app móvil) con la misma filosofía.

---

## 8. Curva de aprendizaje y setup

| | Selenium | Cypress |
|---|---|---|
| Setup inicial | Horas (drivers, IDE, dependencias) | Minutos (`npm install`) |
| Estructura del proyecto | La define el desarrollador | Autogenerada por la herramienta |
| Ver el test corriendo con inspección en vivo | Requiere plugins/config extra | De fábrica |

> **Analogía:** contratar a la agencia global implica **firmar contratos, coordinar con múltiples sucursales, configurar protocolos** antes del primer día de trabajo. Contratar al guardaespaldas personal es casi tan simple como **entregarle las llaves de la casa** — empieza a trabajar casi de inmediato.

---

## 9. Ecosistema y madurez

Selenium existe desde 2004 — dos décadas de comunidad, integraciones, Stack Overflow con prácticamente cualquier error ya resuelto, y soporte en absolutamente cualquier lenguaje/framework de testing. Cypress es mucho más joven (2017), con un ecosistema más acotado pero creciendo rápido, y una filosofía de documentación oficial muy cuidada.

> **Analogía:** la agencia global lleva **20 años operando** — cualquier situación rara que se pueda imaginar, ya la resolvieron antes en algún país. El guardaespaldas personal es de una generación más nueva, con métodos más modernos y eficientes, pero con menos "casos raros ya resueltos" acumulados a lo largo de los años.

---

## 10. Tabla de decisión final

| Escenario | Recomendación |
|---|---|
| App web moderna (React/Vue/Angular), equipo 100% JS/TS, prioridad en velocidad | **Cypress** |
| Necesidad de cubrir Safari real, navegadores legacy, o testing mobile | **Selenium (+ Appium)** |
| Equipo QA con background en Java/Python que no quiere migrar todo a JS | **Selenium** |
| Mucho mockeo de backend, tests de estados de error/carga | **Cypress** |
| Suite gigante corriendo en paralelo contra muchos navegadores/SOs | **Selenium + Grid** |
| Proyecto nuevo, equipo pequeño, prioridad en velocidad de desarrollo | **Cypress** |

---

## 11. Diagrama comparativo

![Diagrama comparativo Selenium vs Cypress](../capturas/selenium-vs-cypress-diagrama.svg)

*(Diagrama ilustrativo: las fortalezas distintivas de cada herramienta, agrupadas bajo "elegir Selenium si..." y "elegir Cypress si...".)*

---

## 12. Nota al margen: ¿y Playwright?

Playwright (todavía no documentado a fondo en este repo) suele mencionarse como "lo mejor de ambos mundos" — corre fuera del navegador como Selenium (multi-lenguaje, sin la limitación de dominio de Cypress) pero fue diseñado desde cero con velocidad y `intercept` nativo en mente. Cuando se documente a fondo, esta comparativa se ampliará con una tercera columna.
