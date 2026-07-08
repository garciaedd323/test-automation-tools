# Estrategia de automatización de pruebas

Tener buenas herramientas y buenos patrones (POM, Screenplay) no sirve de nada si automatizás las cosas equivocadas en el orden equivocado. Esta es probablemente la decisión más determinante del éxito o fracaso de un esfuerzo de automatización: **qué automatizar primero, y qué no automatizar nunca.**

---

## 1. El error más común

Los equipos que recién arrancan con automatización suelen cometer el mismo error: automatizan lo que es **fácil de automatizar**, no lo que **más valor aporta**. El resultado típico, meses después: una suite con 200 tests automatizados de pantallas secundarias que casi nadie usa, mientras el flujo de pago (el corazón del negocio) se sigue probando a mano cada release, con miedo, porque "es muy complicado de automatizar".

La estrategia correcta invierte esta lógica: se prioriza por **valor e impacto**, no por facilidad.

---

## 2. Criterios de priorización

Para decidir qué automatizar primero, evaluá cada caso de prueba (o flujo) contra estos criterios:

### 2.1 Criticidad de negocio
¿Qué tan grave es si esto falla en producción? No es lo mismo que se rompa el botón de "compartir en redes sociales" que el flujo de pago.

**Pregunta guía**: si esto falla mañana, ¿perdemos dinero, clientes, o reputación de forma directa?

### 2.2 Frecuencia de ejecución
¿Cuántas veces se corre esta prueba? Un caso que se ejecuta en cada release (varias veces por semana) tiene un ROI de automatización mucho mayor que uno que se corre una vez cada seis meses.

**Pregunta guía**: ¿voy a correr esto más de 5 veces en los próximos meses?

### 2.3 Frecuencia de cambio de la funcionalidad
Si una pantalla cambia todo el tiempo (está en desarrollo activo, se está iterando el diseño), automatizarla ahora significa reescribir el test constantemente. Mejor esperar a que se estabilice.

**Pregunta guía**: ¿esta funcionalidad ya es estable, o todavía está en flujo de diseño/decisión?

### 2.4 Complejidad y esfuerzo de automatizar
Algunos flujos son técnicamente muy costosos de automatizar (por ejemplo, integración con hardware externo, captchas, pagos con validación bancaria real). Hay que sopesar el esfuerzo contra el beneficio.

**Pregunta guía**: ¿el esfuerzo de automatizar esto es razonable comparado con el tiempo que ahorra?

### 2.5 Propensión a errores humanos
Las pruebas manuales repetitivas y tediosas son las que más errores generan por fatiga o distracción (probar 50 combinaciones de un formulario, por ejemplo). Automatizarlas no solo ahorra tiempo, mejora la calidad de la prueba en sí.

**Pregunta guía**: ¿es un caso donde un humano cansado se equivocaría fácilmente?

### 2.6 Tiempo de ejecución manual
Si una prueba manual toma 2 horas cada vez, el ROI de automatizarla es mucho más alto que uno que toma 2 minutos.

---

## 3. Matriz de priorización

Una forma práctica de aplicar estos criterios es cruzar **impacto de negocio** contra **frecuencia de ejecución**:

| | Frecuencia baja | Frecuencia alta |
|---|---|---|
| **Impacto alto** | Automatizar con prioridad media-alta (aunque se corra poco, si falla es grave) | **Automatizar primero** — máximo ROI |
| **Impacto bajo** | No automatizar (dejar manual u omitir) | Automatizar cuando haya tiempo — ROI moderado |

Los flujos de **impacto alto + frecuencia alta** (login, checkout, búsqueda principal, flujos de pago) son casi siempre los primeros candidatos en cualquier producto digital.

---

## 4. ROI de automatización: cómo pensarlo en números

El ROI de automatizar no es gratis ni inmediato — hay una inversión inicial que se recupera con el tiempo. La fórmula conceptual:

```
Costo de hacerlo manual repetidamente (a lo largo del tiempo)
                    vs.
Costo de automatizar una vez + costo de mantenimiento continuo
```

### Ejemplo numérico simple

Supongamos:
- Ejecutar el test manualmente toma **15 minutos**.
- Se ejecuta **1 vez por semana** (en cada release).
- Automatizarlo toma **4 horas** de desarrollo inicial.
- Mantenerlo cuesta, en promedio, **10 minutos por mes** (ajustes cuando cambia la UI).

Cálculo aproximado:
- Costo manual anual: 15 min × 52 semanas = 780 minutos (13 horas) al año.
- Costo automatizado: 4 horas iniciales + (10 min × 12 meses = 120 min = 2 horas) = 6 horas el primer año.

**Se recupera la inversión dentro del primer año**, y a partir del segundo año el ahorro es casi total (solo quedan los 10 min/mes de mantenimiento).

### Cuándo el ROI es negativo (no conviene automatizar)
- La prueba se ejecuta muy pocas veces (por ejemplo, una funcionalidad que se prueba una sola vez antes de un lanzamiento único).
- La funcionalidad cambia tan seguido que el costo de mantenimiento supera constantemente el ahorro de tiempo.
- El esfuerzo de automatizar es desproporcionado frente al tiempo que realmente ahorra (por ejemplo, un caso muy simple de probar a mano en 30 segundos, pero técnicamente complejo de automatizar).

---

## 5. Mantenibilidad: el costo oculto que todos subestiman

La automatización no es "escribir el test y listo" — cada test automatizado es **código que hay que mantener** para siempre, igual que el código de producción. Este es el factor que más subestiman los equipos nuevos en automatización.

### 5.1 Por qué se rompen los tests automatizados
- Cambios en la UI (un `id` o clase CSS que cambia).
- Cambios en la lógica de negocio (una regla que cambia).
- Datos de prueba que quedan obsoletos (un usuario de prueba que se borró de la base).
- Dependencias externas inestables (una API de terceros que cambia su contrato).

### 5.2 Prácticas que mejoran la mantenibilidad

- **Usar patrones como POM**: centralizan los cambios en un solo lugar (ver `patrones-de-diseno.md`).
- **Selectores estables**: preferir atributos dedicados a testing (`data-testid="login-btn"`) en vez de clases CSS o textos que cambian con el diseño.
- **Independencia entre tests**: cada test debe poder correr solo, sin depender del resultado de otro. Si el test B necesita que el test A haya corrido antes, un solo fallo en cascada rompe toda la suite.
- **Datos de prueba controlados**: usar datos generados o sembrados (seed data) específicamente para testing, no depender de datos reales que pueden cambiar.
- **Evitar esperas fijas (`sleep(5000)`)**: usar esperas explícitas basadas en condiciones ("esperar hasta que el botón sea clickeable") en vez de tiempos fijos, que son la causa número uno de tests "flaky" (inestables, que a veces pasan y a veces fallan sin que nada haya cambiado realmente).
- **Revisión periódica de la suite**: eliminar tests obsoletos o duplicados. Una suite grande no es necesariamente una suite buena — a veces es solo deuda técnica acumulada.

### 5.3 El costo de una suite mal mantenida
Cuando los tests son "flaky" (inestables), el equipo empieza a **ignorar los fallos** ("total, siempre falla ese test y no es nada real"). Ese es el peor escenario posible: la suite deja de aportar confianza y se convierte en ruido. En ese punto, es mejor tener menos tests, pero confiables, que muchos tests que nadie toma en serio.

---

## 6. Proceso práctico para decidir qué automatizar primero

Un flujo de trabajo concreto para aplicar todo lo anterior:

1. **Listar los flujos/casos de prueba existentes** (manuales o nuevos).
2. **Puntuar cada uno** según los criterios de la sección 2 (podés usar una escala simple de 1 a 5 por criterio).
3. **Ubicarlos en la matriz de impacto vs. frecuencia** (sección 3).
4. **Empezar por los "smoke tests"**: un pequeño set de pruebas rápidas que verifican que lo esencial funciona (login, carga de la home, flujo principal). Esto da valor inmediato y sirve como red de seguridad básica desde el día uno.
5. **Seguir con los flujos críticos de alto impacto y alta frecuencia** (checkout, búsqueda, registro).
6. **Evaluar el ROI de cada candidato** antes de invertir tiempo (sección 4).
7. **Automatizar, medir el mantenimiento real** durante 1-2 meses, y ajustar la estrategia si un test resulta más costoso de mantener de lo esperado.
8. **Revisar la suite periódicamente** (cada trimestre, por ejemplo) para eliminar tests obsoletos y re-priorizar según cómo cambió el producto.

---

## 7. Resumen ejecutivo

- **No automatices lo más fácil, automatizá lo más valioso.**
- Priorizá por **impacto de negocio × frecuencia de ejecución**.
- Calculá el ROI antes de invertir tiempo: inversión inicial + mantenimiento vs. ahorro acumulado.
- La mantenibilidad no es un detalle técnico secundario — es lo que determina si la automatización sigue dando valor dentro de un año o se convierte en deuda técnica que nadie quiere tocar.
- Empezá con smoke tests para tener una red de seguridad rápida, y avanzá hacia los flujos críticos de negocio.
- Una suite pequeña y confiable vale más que una suite grande y "flaky".
