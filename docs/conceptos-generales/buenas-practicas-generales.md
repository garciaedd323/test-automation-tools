# Buenas prácticas generales en automatización de pruebas

Estas son las prácticas de "higiene básica" de cualquier suite de automatización. No dependen de la herramienta (Selenium, Cypress, Playwright) ni del patrón que uses (POM, Screenplay) — son la base sobre la que se sostiene todo lo demás. Ignorarlas es la causa número uno de que una suite se vuelva lenta, inestable y odiada por el equipo.

---

## 1. Naming de tests

### Por qué importa
El nombre de un test es, muchas veces, la **única documentación** que alguien va a leer. Cuando un test falla en un pipeline de CI/CD a las 3 de la mañana, lo primero (y a veces lo único) que ve la persona de guardia es el nombre del test. Un buen nombre le ahorra tener que abrir el código para entender qué se estaba probando.

### Malas prácticas comunes
```javascript
test('test1', ...)
test('login', ...)
test('debería funcionar', ...)
test('test de usuario', ...)
```
Ninguno de estos dice qué se espera que pase, ni bajo qué condición.

### Buena práctica: patrón "Given-When-Then" o "Escenario-Acción-Resultado"
Un buen nombre de test responde tres preguntas: **¿en qué contexto? ¿qué acción? ¿qué resultado se espera?**

```javascript
test('debería mostrar mensaje de error cuando el usuario ingresa una contraseña incorrecta', ...)
test('debería redirigir al dashboard cuando el login es exitoso', ...)
test('debería deshabilitar el botón de pago cuando el carrito está vacío', ...)
```

**Analogía cotidiana**: es la diferencia entre etiquetar una caja de mudanza como "cosas" versus "vajilla de cocina — frágil, cuarto piso". La segunda etiqueta te dice exactamente qué hay adentro y cómo tratarla, sin necesidad de abrirla.

### Estructura recomendada para organizar tests
```
describe('Login', () => {
  describe('cuando las credenciales son correctas', () => {
    test('debería redirigir al dashboard', ...)
    test('debería guardar el token de sesión', ...)
  });
  describe('cuando las credenciales son incorrectas', () => {
    test('debería mostrar un mensaje de error', ...)
    test('debería mantener al usuario en la pantalla de login', ...)
  });
});
```
Esto agrupa por contexto y hace que los reportes de ejecución sean legibles de un vistazo.

---

## 2. Independencia entre pruebas

### La regla de oro
**Cada test debe poder ejecutarse solo, en cualquier orden, sin depender de que otro test haya corrido antes.**

### Por qué es tan importante
Si el test B necesita que el test A haya creado un usuario primero, pasan cosas malas:
- Si corrés solo el test B (por ejemplo, para depurar algo puntual), falla sin razón aparente.
- Si los tests corren en paralelo (algo común para acelerar el pipeline de CI), el orden no está garantizado y todo se vuelve impredecible.
- Un solo fallo en el test A genera un efecto dominó de fallos en cascada, y ahora tenés que investigar 10 tests rotos cuando en realidad el problema real es uno solo.

**Analogía cotidiana**: es como armar un examen de opción múltiple donde la pregunta 5 solo tiene sentido si respondiste bien la pregunta 2. Si alguien entra directamente a la pregunta 5 (o la pregunta 2 se anula), todo el examen se derrumba, aunque el resto de las preguntas estuvieran bien planteadas.

### Cómo lograr independencia
- Cada test crea (y limpia) sus propios datos. No reutilices un usuario o un pedido creado por otro test.
- Usá hooks de `setup` y `teardown` (`beforeEach`, `afterEach`) para preparar el estado necesario y limpiarlo después, en vez de depender del estado que dejó el test anterior.
- Evitá variables globales o estado compartido entre archivos de test.

```javascript
// Mal: depende de que otro test haya corrido antes y haya creado el usuario
test('debería mostrar el perfil del usuario', async () => {
  await page.goto('/perfil/usuario-creado-en-otro-test');
  ...
});

// Bien: el propio test crea lo que necesita
test('debería mostrar el perfil del usuario', async () => {
  const usuario = await crearUsuarioDePrueba();
  await page.goto(`/perfil/${usuario.id}`);
  ...
  await eliminarUsuarioDePrueba(usuario.id);
});
```

---

## 3. Datos de prueba

### El problema de usar datos reales o "hardcodeados"
- Si el test depende de un usuario específico que existe hoy en la base de datos ("juan@test.com"), el día que alguien borre ese usuario (o cambie su contraseña), el test se rompe sin que el código tenga ningún problema real.
- Si dos tests corren en paralelo y ambos usan el mismo email para "crear un usuario", van a chocar entre sí.

### Estrategias recomendadas

**Datos generados dinámicamente**: generar valores únicos en cada ejecución (con librerías como Faker.js) para evitar colisiones.
```javascript
const email = `usuario_${Date.now()}@test.com`;
```

**Datos sembrados (seed data)**: para escenarios que requieren datos consistentes y predecibles (por ejemplo, "un producto que siempre está agotado"), se prepara una base de datos de testing con datos fijos y conocidos antes de correr la suite, en vez de depender de lo que haya en producción o en una base compartida.

**Fixtures**: archivos con datos de prueba reutilizables y versionados junto con el código (JSON, YAML), para casos donde los datos no cambian entre ejecuciones.

**Aislamiento de entornos**: nunca correr pruebas automatizadas contra la base de datos de producción. Usar una base de datos de testing dedicada, que se pueda resetear entre ejecuciones.

**Analogía cotidiana**: es la diferencia entre practicar cirugía en un maniquí (datos de prueba controlados, se puede repetir cuantas veces sea necesario) versus practicar en un paciente real (producción) — en el segundo caso, cualquier error tiene consecuencias reales y no podés simplemente "resetear" para intentar de nuevo.

---

## 4. Evitar flaky tests

### Qué es un test flaky
Un test **flaky** (inestable) es aquel que a veces pasa y a veces falla, **sin que el código haya cambiado**. Es el peor enemigo de una suite de automatización, porque destruye la confianza: cuando un test falla, nadie sabe si es un bug real o "es que ese test es así".

### Causas más comunes

**1. Esperas mal manejadas** (la causa número uno — se profundiza en la sección 5).

**2. Dependencia de tiempo real**: probar algo relacionado con fechas u horarios sin controlar el reloj del sistema puede fallar según el día o la hora en que corra el test.
```javascript
// Riesgoso: puede fallar distinto según cuándo se ejecute
expect(fechaDeVencimiento).toBeGreaterThan(new Date());
```

**3. Dependencia de orden de ejecución** (ver sección 2).

**4. Animaciones y transiciones de UI**: un test que intenta hacer click en un botón mientras todavía se está animando (deslizando, apareciendo con fade) puede fallar de forma intermitente.

**5. Datos compartidos entre tests** que corren en paralelo (ver sección 3).

**6. Dependencias externas inestables**: llamar a una API real de terceros en los tests, en vez de simularla (mock), introduce la inestabilidad de esa API como inestabilidad del test.

### Cómo abordarlos
- Identificar y **marcar** los tests flaky conocidos (muchas herramientas permiten reintentos automáticos o "cuarentena" de tests inestables) mientras se investiga la causa raíz — pero nunca como solución permanente.
- Mockear dependencias externas inestables.
- Usar esperas explícitas en vez de tiempos fijos (sección 5).
- Congelar el tiempo/fecha en tests que dependen de la hora (librerías como `sinon` o `date-fns` con mocks de fecha).
- Ejecutar la suite varias veces seguidas en el pipeline antes de confiar en que un test nuevo es estable.

**Analogía cotidiana**: un test flaky es como una alarma de auto que suena al azar, sin que nadie la toque. Al principio todos se sobresaltan, pero después de la décima vez, la gente empieza a ignorarla — incluso el día que suene por un robo de verdad.

---

## 5. Esperas explícitas vs. implícitas

Este es uno de los temas técnicos más importantes y más mal entendidos en automatización.

### Esperas fijas (lo que hay que evitar siempre)
```javascript
await sleep(5000); // esperar 5 segundos "a ciegas"
await page.click('#boton');
```
El problema: si la página tarda 6 segundos en cargar ese día (por una red más lenta, un servidor con más carga), el test falla igual. Y si tarda 1 segundo, estás perdiendo 4 segundos de espera innecesaria en cada ejecución. Multiplicado por cientos de tests, esto hace que la suite completa tarde mucho más de lo necesario.

### Esperas implícitas
Configuran un tiempo máximo de espera **global**, que aplica automáticamente a todas las búsquedas de elementos durante toda la sesión del navegador.
```javascript
driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
```
Es mejor que una espera fija, pero tiene un problema: aplica el mismo criterio ("existe en el DOM") a todos los elementos, sin importar si lo que realmente hace falta esperar es que sea *visible*, *clickeable*, o que tenga un *texto específico*. Además, mezclar esperas implícitas y explícitas en el mismo test puede generar comportamientos inconsistentes y tiempos de espera sumados de forma impredecible (por eso muchas guías oficiales, como la de Selenium, recomiendan no combinarlas).

### Esperas explícitas (la mejor práctica recomendada hoy)
Esperan una **condición específica** antes de continuar, revisando periódicamente si se cumplió, sin bloquear un tiempo fijo:
```javascript
// Playwright: espera automáticamente a que el botón sea clickeable
await page.click('#boton-pago');

// Selenium: espera explícita a una condición concreta
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.elementToBeClickable(By.id("boton-pago")));
```

La clave: en vez de decir "esperá 5 segundos y después hacé click", decís "esperá **hasta que** el botón sea clickeable (con un máximo razonable de tiempo), y ahí hacé click". Si el botón está listo en 800ms, el test sigue de inmediato; si tarda 4 segundos, espera esos 4 segundos y no falla innecesariamente.

**Nota sobre frameworks modernos**: Playwright y Cypress incorporan esperas explícitas automáticas ("auto-waiting") para la mayoría de las acciones — no necesitás escribirlas manualmente en la mayoría de los casos, lo cual es una de las razones por las que estos frameworks generan menos tests flaky que Selenium "a mano" sin esperas bien configuradas.

**Analogía cotidiana**: una espera fija es poner una alarma para las 8:00 am "porque el pan tarda como una hora en hornearse" — si por algún motivo tardó 70 minutos, sacás el pan crudo igual. Una espera explícita es abrir el horno **cuando el pan está dorado**, sin importar si eso pasó a los 55 o a los 70 minutos — la condición real (que esté listo) es lo que determina cuándo actuar, no un número fijo de minutos.

---

## 6. Tabla resumen

| Práctica | Problema que evita | Regla simple |
|---|---|---|
| Naming descriptivo | Tests ilegibles, difíciles de depurar | El nombre debe decir contexto + acción + resultado esperado |
| Independencia entre tests | Fallos en cascada, orden impredecible | Cada test crea y limpia sus propios datos |
| Datos de prueba controlados | Colisiones, dependencia de datos reales | Generar datos únicos o usar seed data en un entorno aislado |
| Evitar flaky tests | Pérdida de confianza en la suite | Investigar y corregir la causa raíz, nunca ignorar |
| Esperas explícitas | Tests lentos o que fallan por timing | Esperar una condición, no un tiempo fijo |

---

## 7. Regla general

Todas estas prácticas comparten un mismo principio: **un test debe fallar únicamente cuando hay un bug real**, nunca por razones ajenas a la lógica que se está probando (orden de ejecución, datos sucios, timing, o nombres confusos que generan errores de interpretación). Cuanto más se acerque una suite a ese ideal, más confianza le tiene el equipo — y una suite en la que se confía es la única que realmente aporta valor.
