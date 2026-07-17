# Por qué el reporte de Serenity solo mostraba la última carpeta corrida

## El problema, en una frase

Cada vez que corres los tests, Gradle genera el reporte automáticamente al final (por el `finalizedBy`), **pero ese reporte automático solo mira los resultados que se generaron en esa corrida**, no todo el histórico acumulado en la carpeta.

## Detalle técnico

El `build.gradle` tiene esta línea (línea 102):

```groovy
test.finalizedBy(aggregate)
```

Esto significa que cada vez que corremos:

```bash
./gradlew.bat test --tests <Runner>
```

Al terminar se dispara automáticamente `aggregate`. Pero ese `aggregate` **solo arma el reporte con los resultados de esa sesión de Gradle en particular**, no con el histórico acumulado.

Por eso, al abrir `index.html` después de correr Administradores, solo se veían esos 6 escenarios: el dashboard se reescribió con esa única corrida.

Los JSON de Login y Fuerza de Ventas **nunca se borraron** (siguen en `target/site/serenity/`). Corriendo `./gradlew.bat aggregate` solo (sin `test`), Serenity vuelve a leer todos los JSON del directorio y regenera el reporte combinado. Al ejecutarlo así, el dashboard mostró:

**12 scenarios** — los 4 de Login + los 2 de Fuerza de Ventas + los 6 de Administrador Dinámico, todos en verde.

---

## Analogía: la carpeta de fotos reveladas

Imagina que tienes una cámara antigua y un álbum de fotos:

- Cada vez que tomas fotos (**corres los tests**), las mandas a revelar y las guardas en un cajón (**`target/site/serenity/`**, donde quedan los JSON).
- Al final de cada sesión de fotos, tienes un asistente que automáticamente arma un álbum (**el `aggregate` que se dispara solo, por el `finalizedBy`**)... pero ese asistente es medio distraído: **solo mira las fotos que acabas de traer de esta sesión**, no revisa todo el cajón completo.

Entonces:

1. **Lunes**: tomas fotos de "Login" → el asistente arma un álbum con esas fotos de Login. Perfecto, coincide.
2. **Martes**: tomas fotos de "Fuerza de Ventas" → el asistente arma un álbum, pero **solo con las fotos de Fuerza de Ventas**. Las de Login siguen en el cajón, ¡pero el álbum ya no las muestra! No es que se perdieron, es que el asistente no las miró esta vez.
3. **Miércoles**: tomas fotos de "Administradores" → mismo cuento, el álbum (`index.html`) ahora solo muestra las 6 fotos de Administradores.

Tú, al abrir el álbum, piensas "¡se borraron las fotos de Login y Fuerza de Ventas!" — pero no, **siguen en el cajón**. Solo que el álbum resumen no las incluyó porque el asistente automático solo trabaja con lo de la última sesión.

### La solución: pedirle al asistente que revise TODO el cajón

Cuando corres `./gradlew.bat aggregate` **solo**, sin `test`, le estás diciendo al asistente: *"Oye, no traigas fotos nuevas, solo ve al cajón completo, mira TODAS las fotos que hay ahí (Login + Fuerza de Ventas + Administradores) y arma un álbum con todas."*

Por eso al correrlo así, aparecieron los 12 escenarios: el asistente por fin miró el cajón completo en vez de solo lo último que llegó.

---

## Analogía extra: el resumen bancario

- Cada compra con tu tarjeta (**cada test que corres**) queda registrada en el banco (**los JSON en la carpeta**), eso nunca se borra.
- Pero si le pides al cajero "hazme un resumen" justo después de UNA compra (**`test --tests X`**, que dispara `aggregate` automático), el resumen que te da solo refleja esa compra reciente, aunque tu historial completo siga intacto en el sistema.
- Si en cambio le pides "dame el resumen de TODO mi historial" (**`aggregate` solo**), ahí sí te muestra todas las transacciones acumuladas.

---

## Resumen de comandos

| Acción | Qué hace |
|---|---|
| `./gradlew.bat test --tests X` | Corre tests de X **y** regenera el reporte, pero el reporte queda "miope": solo ve la corrida actual |
| `./gradlew.bat aggregate` (solo) | No corre tests, solo relee **todos** los JSON guardados y arma el reporte completo actualizado |

## Recomendación para seguir corriendo por carpetas

Flujo sugerido cuando se corren varias carpetas de tests:

1. Corres los tests de la carpeta A (`test --tests A`) → se genera el reporte parcial (solo A).
2. Corres los tests de la carpeta B (`test --tests B`) → se genera el reporte parcial (solo B, aunque A siga en el cajón/carpeta de JSON).
3. Al terminar todas las carpetas que quieras correr, ejecutas:

   ```bash
   ./gradlew.bat aggregate
   ```

   (sin `--tests`) → esto "abre el cajón completo" y te da el `index.html` con el histórico acumulado real.

Así nunca te asustas pensando que "se perdieron" escenarios: simplemente el reporte automático de cada corrida es parcial, y el `aggregate` manual es el que te da la foto completa. 📸
