# Glosario de testing y automatización

Referencia rápida de 100 términos que aparecen a lo largo de todo el repo. Ordenado alfabéticamente.

---

**Aceptación, prueba de (UAT)**
Ver *UAT*.

**Accesibilidad (a11y)**
Cualidad de un software que puede ser usado por personas con distintas capacidades (visuales, auditivas, motrices, cognitivas). Ver `tipos-de-pruebas.md`.

**Actor**
En el Screenplay Pattern, quien realiza las acciones en un test. Tiene *habilidades* (abilities), ejecuta *tareas* (tasks) y hace *preguntas* (questions). Ver `patrones-de-diseno.md`.

**Aserción (assertion)**
Instrucción dentro de un test que verifica que algo sea cierto, y hace fallar el test si no lo es.
```javascript
expect(resultado).toBe(42);
```

**Auto-waiting (espera automática)**
Comportamiento de frameworks modernos (Playwright, Cypress) que esperan automáticamente a que un elemento esté listo antes de interactuar con él. Ver `buenas-practicas-generales.md`.

**BDD (Behavior-Driven Development)**
Metodología que describe el comportamiento esperado del sistema en lenguaje natural y compartido con el negocio, típicamente con sintaxis Given-When-Then. Herramientas: Cucumber, SpecFlow.

**Black box testing (caja negra)**
Enfoque de testing donde se prueba el sistema únicamente a través de sus entradas y salidas, sin conocer ni depender de su implementación interna.

**Boundary value analysis (análisis de valores límite)**
Técnica de diseño de pruebas que se enfoca en los valores en los extremos de un rango válido (por ejemplo, probar 0, 1, 99 y 100 en un campo que acepta de 1 a 100), porque ahí es donde suelen aparecer los bugs.

**Bug / defecto**
Un comportamiento del software que no coincide con lo esperado. Se distingue de un "issue" o "feature request" en que representa algo roto, no una mejora deseada.

**CI/CD (integración continua / entrega continua)**
Práctica de ejecutar automáticamente pruebas y otros procesos cada vez que se sube código nuevo a un repositorio, para detectar problemas lo antes posible.

**Cobertura de pruebas (test coverage)**
Métrica que indica qué porcentaje del código o de los casos de uso está cubierto por pruebas. No garantiza ausencia de bugs, solo indica cuánto se ejecuta.

**Contract testing (pruebas de contrato)**
Verifica que dos servicios (por ejemplo, un frontend y una API) respeten un "contrato" acordado de formato de datos, sin necesidad de levantar ambos sistemas completos a la vez. Herramienta típica: Pact.

**Continuous testing**
Práctica de ejecutar pruebas automatizadas en cada etapa del pipeline de desarrollo, no solo al final, para dar feedback constante.

**Cross-browser testing**
Verificación de que una aplicación web funcione correctamente en distintos navegadores (Chrome, Firefox, Safari, Edge).

**Cuarentena (test quarantine)**
Práctica de aislar temporalmente tests flaky conocidos mientras se investiga su causa raíz. No debe usarse como solución permanente.

**Data-driven testing (pruebas basadas en datos)**
Técnica donde el mismo test se ejecuta múltiples veces con distintos conjuntos de datos de entrada, sin duplicar el código del test.

**Datos de prueba (test data)**
Información utilizada para ejecutar un test: usuarios, productos, transacciones, etc. Ver `buenas-practicas-generales.md`.

**Defecto**
Ver *Bug*.

**Dependency injection (inyección de dependencias)**
Técnica de diseño de software donde las dependencias de un componente se le "inyectan" desde afuera en vez de crearlas internamente, lo cual facilita reemplazarlas por mocks o stubs durante el testing.

**Dry run**
Ejecución de prueba de un proceso (como un pipeline o un script) sin aplicar cambios reales, solo para verificar que la lógica funcione.

**End-to-end (E2E)**
Prueba que simula el flujo completo de un usuario real a través de toda la aplicación, de principio a fin. Ver `tipos-de-pruebas.md`.

**Entorno de pruebas (test environment)**
Infraestructura separada de producción (staging, QA, sandbox) donde se ejecutan las pruebas sin afectar datos ni usuarios reales.

**Equivalence partitioning (partición de equivalencia)**
Técnica de diseño de pruebas que agrupa las entradas posibles en clases donde se espera el mismo comportamiento, para probar un representante de cada clase en vez de todos los valores posibles.

**Escenario (scenario)**
Una situación concreta que se describe y prueba, generalmente en formato Given-When-Then dentro de BDD.

**Estrés, prueba de (stress test)**
Lleva al sistema más allá de su capacidad esperada para identificar en qué punto falla y cómo se recupera. Ver `tipos-de-pruebas.md`.

**Falso negativo (false negative)**
Un test que pasa (dice que todo está bien) cuando en realidad hay un bug real que no fue detectado.

**Falso positivo (false positive)**
Un test que falla cuando en realidad el sistema funciona correctamente — típicamente causado por un test flaky o mal diseñado.

**Feature flag (bandera de funcionalidad)**
Mecanismo que permite activar o desactivar una funcionalidad en producción sin desplegar código nuevo, útil para probar features de forma controlada.

**Fixture**
Conjunto de datos o estado predefinido que se prepara antes de ejecutar un test, para que corra en condiciones conocidas y repetibles.

**Flaky test**
Un test que a veces pasa y a veces falla sin que el código haya cambiado. Ver `buenas-practicas-generales.md`.

**Fuzz testing**
Técnica que alimenta al sistema con datos aleatorios, inválidos o inesperados para encontrar fallos, especialmente vulnerabilidades de seguridad o crashes.

**Gherkin**
Lenguaje de sintaxis simple usado en BDD para escribir escenarios en formato Given-When-Then, legible tanto por personas técnicas como no técnicas.

**Given-When-Then**
Estructura para describir un escenario de prueba: *Given* (contexto inicial), *When* (acción realizada), *Then* (resultado esperado).

**Golden master testing (prueba de máster dorado)**
Técnica que compara la salida actual del sistema contra una salida previamente aprobada ("golden master") para detectar cambios no intencionados, útil cuando la lógica es muy compleja de verificar con aserciones tradicionales.

**Gray box testing (caja gris)**
Enfoque intermedio entre caja blanca y caja negra: quien prueba tiene conocimiento parcial de la implementación interna, aunque prueba principalmente a través de entradas y salidas.

**Happy path (camino feliz)**
El flujo principal y esperado de uso de una funcionalidad, sin errores ni condiciones excepcionales (por ejemplo, un login con credenciales correctas).

**Headless**
Modo de ejecución de un navegador sin interfaz gráfica visible, común en pipelines de CI/CD por ser más rápido.
```javascript
await chromium.launch({ headless: true });
```

**Hook**
Función que se ejecuta automáticamente en un momento determinado del ciclo de vida de un test (`beforeEach`, `afterEach`, `beforeAll`, `afterAll`), usada típicamente para preparar y limpiar el estado.

**Humo, prueba de (smoke test)**
Chequeo rápido de las funcionalidades más básicas de una aplicación. Ver `tipos-de-pruebas.md`.

**i18n / l10n (internacionalización / localización)**
*i18n*: preparar el software para soportar múltiples idiomas y regiones. *l10n*: adaptar el contenido a un idioma o región específica. Las pruebas de l10n verifican traducciones, formatos de fecha/moneda, etc.

**Idempotencia**
Propiedad de una operación que produce el mismo resultado sin importar cuántas veces se ejecute. Importante en tests que pueden reintentarse sin generar efectos secundarios duplicados.

**Integración, prueba de**
Prueba que verifica que varios módulos funcionen correctamente en conjunto. Ver `tipos-de-pruebas.md`.

**Keyword-driven testing (pruebas basadas en palabras clave)**
Técnica donde los pasos de un test se definen mediante palabras clave reutilizables (por ejemplo, "IniciarSesion", "AgregarAlCarrito"), permitiendo que personas no técnicas armen casos de prueba combinándolas.

**Linting**
Análisis estático de código para detectar errores de estilo o problemas potenciales antes de ejecutar cualquier prueba.

**Locator (localizador)**
Forma en que un framework de automatización identifica un elemento en la interfaz.
```javascript
page.locator('#login-btn');
```

**Manual, testing**
Proceso de probar software ejecutando casos de prueba a mano, sin scripts de automatización.

**Matriz de trazabilidad (traceability matrix)**
Documento que vincula cada requisito del sistema con los casos de prueba que lo verifican, para asegurar que no queden requisitos sin cubrir.

**Mock**
Objeto simulado que reemplaza a una dependencia real y verifica cómo fue usado (cuántas veces, con qué argumentos).
```javascript
expect(mockFn).toHaveBeenCalledWith('usuario123');
```

**Mutation testing (pruebas de mutación)**
Técnica que introduce pequeños cambios deliberados ("mutantes") en el código para verificar si la suite de tests los detecta. Si un mutante sobrevive sin que ningún test falle, indica un hueco en la cobertura real.

**Negative testing (prueba negativa)**
Verifica que el sistema se comporte correctamente ante entradas inválidas o inesperadas (por ejemplo, un login con contraseña vacía).

**No funcional, prueba**
Prueba que verifica cómo el sistema hace lo que hace: rendimiento, seguridad, accesibilidad, usabilidad. Ver `tipos-de-pruebas.md`.

**Oráculo de prueba (test oracle)**
La fuente que determina cuál es el resultado correcto esperado de una prueba (una especificación, un cálculo manual, un sistema de referencia), usada para comparar contra el resultado real obtenido.

**Orquestación (orchestration)**
Coordinación de la ejecución de múltiples pruebas o procesos, incluyendo su orden, paralelización y dependencias, típicamente dentro de un pipeline de CI/CD.

**Page Factory**
Implementación del patrón Page Object Model en Selenium (Java/C#) que usa anotaciones para inicializar elementos de forma automática. Ver `patrones-de-diseno.md`.

**Page Object Model (POM)**
Patrón de diseño que encapsula los localizadores y acciones de una página en una clase dedicada. Ver `patrones-de-diseno.md`.

**Parametrized test (test parametrizado)**
Ver *Data-driven testing*.

**Pipeline**
Secuencia automatizada de pasos (compilar, testear, desplegar) que se ejecuta cada vez que hay un cambio de código, como parte de CI/CD.

**Pipeline as code**
Práctica de definir el pipeline de CI/CD como un archivo de configuración versionado junto con el código (por ejemplo, un `.yml`), en vez de configurarlo manualmente en una interfaz.

**Positive testing (prueba positiva)**
Verifica que el sistema se comporte correctamente ante entradas válidas y esperadas. Complementa a la prueba negativa.

**Precondición / postcondición**
*Precondición*: estado que debe cumplirse antes de ejecutar una prueba. *Postcondición*: estado que debe cumplirse después de que la prueba termina.

**Producción**
El entorno real donde los usuarios finales usan la aplicación, en contraposición a los entornos de testing o staging.

**Quality gate (puerta de calidad)**
Un umbral o condición (por ejemplo, "cobertura mínima del 80%" o "cero vulnerabilidades críticas") que el código debe cumplir antes de poder avanzar a la siguiente etapa del pipeline.

**Race condition (condición de carrera)**
Error que ocurre cuando el resultado de una operación depende del orden o timing impredecible en que se ejecutan procesos concurrentes. Fuente común de flaky tests.

**Regresión, prueba de**
Verifica que cambios nuevos no hayan roto funcionalidades que antes funcionaban. Ver `tipos-de-pruebas.md`.

**Reporte de pruebas (test report)**
Documento o dashboard generado tras la ejecución de una suite, mostrando qué tests pasaron, fallaron u omitieron, y con qué detalles de error.

**Reproducibilidad**
Capacidad de obtener el mismo resultado al ejecutar el mismo test en las mismas condiciones, las veces que sea necesario.

**Retry logic (lógica de reintento)**
Mecanismo que vuelve a ejecutar un test (o una operación) automáticamente si falla, antes de reportarlo como fallo definitivo. Debe usarse con cuidado: puede enmascarar flaky tests en vez de resolverlos.

**Riesgo, testing basado en (risk-based testing)**
Estrategia que prioriza qué probar en función de la probabilidad y el impacto de que algo falle, en vez de intentar cubrir todo por igual.

**Rollback**
Acción de revertir un despliegue a una versión anterior estable, generalmente disparada cuando las pruebas post-despliegue detectan un problema grave.

**ROI de automatización**
Relación entre el costo de automatizar y el ahorro de tiempo que genera frente a probar manualmente de forma repetida. Ver `estrategia-de-automatizacion.md`.

**Sanity testing (prueba de cordura)**
Verificación rápida y focalizada de que un cambio o fix específico funciona como se espera, sin probar todo el sistema (más acotada que un smoke test).

**Sandbox**
Entorno aislado donde se pueden ejecutar pruebas o experimentos sin riesgo de afectar sistemas reales.

**Screenplay Pattern**
Patrón de diseño para automatización centrado en actores, tareas e intenciones. Ver `patrones-de-diseno.md`.

**Seed data (datos sembrados)**
Conjunto de datos predefinidos que se cargan en una base de datos de testing antes de ejecutar una suite.

**Selector**
Ver *Locator*.

**Severidad / prioridad (de un bug)**
*Severidad*: qué tan grave es el impacto técnico de un bug (crashea la app vs. un detalle visual). *Prioridad*: qué tan urgente es arreglarlo. Un bug puede ser de alta severidad pero baja prioridad, o viceversa.

**Sharding (particionamiento de tests)**
División de una suite de pruebas en partes ("shards") que se ejecutan en paralelo en distintas máquinas, para reducir el tiempo total de ejecución.

**Shift-left testing**
Práctica de mover las actividades de testing lo más temprano posible en el ciclo de desarrollo (por ejemplo, probar requisitos y diseño antes de escribir código), en vez de dejarlas para el final.

**Shift-right testing**
Práctica complementaria a shift-left: extender el testing hacia producción (monitoreo, pruebas con usuarios reales, feature flags) para detectar problemas que solo aparecen en el mundo real.

**Sesión (browser session)**
El estado de un navegador durante una ejecución de test, incluyendo cookies, almacenamiento local y la ventana abierta.

**Snapshot testing (prueba de instantánea)**
Técnica que guarda una "foto" del resultado de un componente o función (HTML renderizado, estructura de datos) y compara ejecuciones futuras contra esa referencia para detectar cambios no intencionados.

**Spy (espía)**
Tipo de test double que envuelve una función real y registra cómo fue llamada, sin necesariamente reemplazar su comportamiento (a diferencia del mock o el stub).

**Stress test**
Ver *Estrés, prueba de*.

**Stub**
Objeto simulado que reemplaza a una dependencia real y devuelve respuestas predefinidas, sin verificar cómo fue utilizado.
```javascript
sinon.stub(servicioDePago, 'procesar').returns({ exito: true });
```

**Suite de pruebas (test suite)**
Conjunto de tests agrupados, generalmente por funcionalidad o tipo, que se ejecutan juntos.

**Synthetic monitoring (monitoreo sintético)**
Ejecución periódica y automatizada de pruebas contra el ambiente de producción real, para detectar problemas de disponibilidad o rendimiento de forma proactiva, simulando el comportamiento de un usuario.

**Tag (etiqueta de test)**
Marca asignada a un test (`@smoke`, `@regression`, `@slow`) que permite ejecutar subconjuntos específicos de la suite según necesidad.

**Task (tarea)**
En el Screenplay Pattern, una acción de alto nivel con sentido de negocio. Ver `patrones-de-diseno.md`.

**TDD (Test-Driven Development)**
Práctica de escribir el test antes que el código de producción, siguiendo el ciclo "rojo (test falla) → verde (código mínimo para pasar) → refactor".

**Test case (caso de prueba)**
Descripción concreta de qué se va a probar, con qué datos de entrada, y qué resultado se espera obtener.

**Test double (doble de prueba)**
Término general que agrupa a mocks, stubs, spies y fakes: cualquier objeto que reemplaza a una dependencia real durante un test.

**Test harness (arnés de pruebas)**
Conjunto de herramientas y código de soporte (no el test en sí) que permite ejecutar pruebas de forma automatizada: configuración, entornos simulados, generadores de datos.

**Test plan (plan de pruebas)**
Documento que define el alcance, los objetivos, los recursos y el cronograma de una campaña de testing.

**Test runner**
Herramienta que descubre, organiza y ejecuta los tests de un proyecto, y reporta los resultados. Ejemplos: Jest, Mocha, JUnit, pytest.

**Test script**
El código concreto que implementa un caso de prueba automatizado.

**Testing automatizado**
Proceso de probar software mediante scripts o herramientas que ejecutan las pruebas sin intervención humana directa durante la ejecución.

**Timeout**
Tiempo máximo que se espera a que ocurra algo antes de considerar que la operación falló.

**Trophy de testing**
Variante de la pirámide de testing propuesta por Kent C. Dodds, que prioriza pruebas de integración sobre unitarias puras.

**Pirámide de testing (test pyramid)**
Modelo que recomienda muchas pruebas unitarias, menos de integración, y pocas end-to-end, complementadas con testing manual/exploratorio.

**UAT (User Acceptance Testing / pruebas de aceptación de usuario)**
Fase final de testing donde usuarios reales o representantes del negocio validan que el sistema cumple con lo que necesitan, antes de aprobar el lanzamiento.

**Unitaria, prueba (unit test)**
Prueba que verifica una pieza aislada de código. Ver `tipos-de-pruebas.md`.

**Usabilidad, prueba de**
Evalúa qué tan fácil e intuitivo es usar el sistema para una persona real. Ver `tipos-de-pruebas.md`.

**Visual regression testing (prueba de regresión visual)**
Técnica que compara capturas de pantalla de la interfaz entre versiones para detectar cambios visuales no intencionados (elementos desalineados, colores incorrectos).

**WebDriver**
Protocolo/API estándar (W3C) para controlar navegadores de forma programática, base sobre la que se construyó Selenium.

**White box testing (caja blanca)**
Enfoque de testing donde quien prueba conoce y utiliza el conocimiento de la implementación interna del código (estructura, rutas de ejecución) para diseñar los casos de prueba.

**XPath**
Lenguaje de consultas usado para navegar y seleccionar elementos dentro de un documento XML o HTML, una de las formas posibles de definir un locator.
```
//button[@id='login-btn']
```
