# GitLab CI — Conceptos y tu primer pipeline

## La analogía general

Como ya se explicó al comparar arquitecturas: si GitHub Actions es una fábrica de renta por horas y Jenkins es una fábrica propia que hay que construir y mantener, **GitLab CI es básicamente la misma fábrica de renta por horas que GitHub Actions — solo que operada por un dueño distinto** (GitLab en vez de GitHub). La filosofía de fondo es prácticamente idéntica: runners efímeros, YAML declarativo, triggers integrados en la misma plataforma que el código. Por eso esta nota es más corta — el 80% ya se entendió con GitHub Actions.

---

## 1. El archivo: `.gitlab-ci.yml`

```yaml
stages:
  - build
  - test
  - report

build:
  stage: build
  image: eclipse-temurin:17-jdk
  script:
    - ./gradlew build -x test

test:
  stage: test
  image: eclipse-temurin:17-jdk
  script:
    - ./gradlew test -Dheadless=true
  artifacts:
    when: always
    paths:
      - build/site/serenity/
    expire_in: 14 days

report:
  stage: report
  image: eclipse-temurin:17-jdk
  script:
    - ./gradlew aggregate
  when: always
```

> **Analogía:** a diferencia de GitHub Actions (donde el archivo vive en una carpeta específica, `.github/workflows/`), GitLab espera encontrar **un solo archivo con nombre fijo** (`.gitlab-ci.yml`) directamente en la raíz del repositorio — como si la fábrica solo aceptara un único manual de instrucciones, con un nombre exacto, en vez de una carpeta que pueda contener varios.

---

## 2. Tabla de equivalencias con GitHub Actions (y de paso, con Jenkins)

| GitHub Actions | Jenkins | GitLab CI | Rol |
|---|---|---|---|
| `.github/workflows/*.yml` | `Jenkinsfile` | `.gitlab-ci.yml` (nombre fijo, en la raíz) | El manual de instrucciones |
| `on:` | Configurado en el job | Integrado automáticamente (push, MR) | El trigger |
| `jobs:` | `stages` | `stages:` + jobs debajo de cada stage | Las etapas |
| `runs-on:` | `agent` | `image:` (una imagen Docker) | Dónde/con qué se ejecuta |
| `steps:` | `steps` | `script:` | Las instrucciones |
| `uses: actions/...` | Plugins | Imágenes Docker + templates de GitLab | Recetas reutilizables |
| `upload-artifact` | `archiveArtifacts` | `artifacts:` | Guardar el resultado |
| `secrets.ALGO` | Credentials | CI/CD Variables (protegidas/enmascaradas) | La caja fuerte |

Nota clave: GitLab CI no usa `runs-on` como GitHub Actions — en su lugar, cada job especifica una **imagen Docker** (`image: eclipse-temurin:17-jdk`) que define el entorno completo, en vez de elegir entre un puñado de sistemas operativos predefinidos.

---

## 3. La diferencia real más notable: runners propios vs. compartidos

GitLab.com ofrece **runners compartidos** gratuitos (como GitHub Actions), pero GitLab también es la opción más común cuando una empresa quiere **runners propios auto-hospedados** dentro de su propia infraestructura, sin salir de la filosofía "todo en YAML, todo integrado a la plataforma".

> **Analogía:** es como si la misma fábrica de renta por horas **también ofreciera la opción de traer una sucursal propia** y conectarla al mismo sistema de pedidos — se conserva toda la simplicidad del manual de instrucciones en YAML, pero se decide dónde vive físicamente la fábrica que lo ejecuta. Es un punto medio entre la comodidad de GitHub Actions y el control total de Jenkins.

---

## 4. `stages` vs el orden de los jobs

```yaml
stages:
  - build
  - test
  - report
```

Los jobs con el mismo `stage` corren **en paralelo**; jobs de distintos `stages` corren **en orden**, esperando a que termine el stage anterior.

> **Analogía:** es literalmente la cola de estaciones de la fábrica: todas las máquinas de "pintura" trabajan al mismo tiempo entre sí, pero ninguna pieza pasa a "ensamblaje" hasta que termine su turno completo en pintura — el mismo concepto de `stages` ya visto en GitHub Actions y Jenkins, solo que aquí el orden lo determina la lista `stages:` de arriba, no la posición de los `jobs:` en el archivo.

---

## 5. Pipelines de Merge Request — el quality gate integrado

```yaml
test:
  stage: test
  script:
    - ./gradlew test
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

> **Analogía:** es decirle a la fábrica "este control de calidad específico solo se activa cuando alguien propone fusionar una pieza nueva a la línea principal" — el mismo concepto de quality gate ya visto, pero con su propia sintaxis de condición (`rules:`) para decidir cuándo aplica cada job.

---

## 6. Tabla resumen

| Concepto | Una frase para recordarlo |
|---|---|
| `.gitlab-ci.yml` | Nombre fijo, en la raíz — a diferencia de una carpeta como en GitHub Actions |
| `image:` | La imagen Docker que define el entorno del job |
| Runners compartidos | Igual de simples que GitHub Actions |
| Runners propios | El punto medio entre GitHub Actions y Jenkins |
| `artifacts:` | Igual función que `upload-artifact`/`archiveArtifacts` |
| CI/CD Variables | La caja fuerte de credenciales |

---

## 7. Cuándo GitLab CI tiene sentido específicamente

| Escenario | Por qué |
|---|---|
| El repositorio ya vive en GitLab (no GitHub) | Es la opción nativa, sin nada que conectar externamente |
| Se quiere la simplicidad de YAML de GitHub Actions, pero con la opción de runners propios sin montar Jenkins completo | GitLab CI cubre ambos mundos con el mismo archivo |
| Equipo que ya usa GitLab para gestión de proyecto (Issues, MRs) y quiere todo integrado en un solo lugar | Menos herramientas separadas que coordinar |

---

## 8. Cierre del círculo de `ci-cd/`

Con GitHub Actions, Jenkins y GitLab CI cubiertos, el patrón que se repite en las tres herramientas es el mismo, solo con distinta sintaxis: **algo dispara el pipeline → se prepara un entorno → se corren los tests → se guarda evidencia → opcionalmente se bloquea el avance si algo falló**. Ese patrón de fondo es lo verdaderamente importante — la sintaxis específica de cada `.yml`/`Jenkinsfile` se puede consultar cuando se necesite, pero el modelo mental ya está construido.
