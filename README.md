# Automatización de Pruebas 🧪

Repositorio de apuntes, tutoriales y recursos sobre automatización de pruebas (testing automatizado), cubriendo conceptos generales, comparativas entre herramientas, y guías prácticas para las principales tecnologías del ecosistema: **Selenium**, **Appium**, **Cypress** y **Playwright**, además de integración con **CI/CD**.

## 📁 Estructura del repositorio

```
automatizacion-pruebas/
├── README.md
├── docs/
│   ├── conceptos-generales/       # Qué es automatización, tipos de testing, pirámide de pruebas, etc.
│   └── comparativas/              # Selenium vs Playwright vs Cypress, tablas comparativas, benchmarks
│
├── selenium/
│   ├── notas/
│   ├── recursos/
│   │   ├── prompts/
│   │   └── capturas/
│   └── tutoriales/
│
├── appium/
│   ├── notas/
│   ├── recursos/
│   │   ├── prompts/
│   │   └── capturas/
│   └── tutoriales/
│
├── cypress/
│   ├── notas/
│   ├── recursos/
│   │   ├── prompts/
│   │   └── capturas/
│   └── tutoriales/
│
├── playwright/
│   ├── notas/
│   ├── recursos/
│   │   ├── prompts/
│   │   └── capturas/
│   └── tutoriales/
│
├── ci-cd/
│   ├── notas/
│   ├── jenkins/
│   ├── github-actions/
│   └── gitlab-ci/
│
└── recursos-generales/
    ├── capturas/                  # Capturas que no son de una herramienta específica
    └── prompts/                   # Prompts reutilizables (ej. "convertir captura a nota")
```

## 📖 Descripción de las carpetas

### `docs/`
Contenido teórico y transversal, independiente de cualquier herramienta específica.

- **`conceptos-generales/`**: Fundamentos de automatización de pruebas — qué es, tipos de testing (unitario, integración, e2e, etc.), la pirámide de pruebas, buenas prácticas, terminología clave.
- **`comparativas/`**: Análisis y comparación entre herramientas (Selenium vs Playwright vs Cypress), tablas comparativas de características, benchmarks de rendimiento.

### `selenium/`, `appium/`, `cypress/`, `playwright/`
Cada herramienta tiene su propia carpeta con la misma estructura interna:

- **`notas/`**: Apuntes personales, ideas sueltas, aprendizajes durante el estudio de la herramienta.
- **`recursos/`**
  - **`prompts/`**: Prompts usados (por ejemplo, para IA) relacionados específicamente con esta herramienta.
  - **`capturas/`**: Capturas de pantalla asociadas a esta herramienta (código, resultados de tests, errores, etc.).
- **`tutoriales/`**: Guías paso a paso, ejercicios prácticos y ejemplos de uso.

### `ci-cd/`
Integración de las pruebas automatizadas en pipelines de integración continua.

- **`notas/`**: Apuntes generales sobre CI/CD aplicado a testing.
- **`jenkins/`**: Configuraciones y ejemplos específicos de Jenkins.
- **`github-actions/`**: Workflows y ejemplos con GitHub Actions.
- **`gitlab-ci/`**: Pipelines y ejemplos con GitLab CI.

### `recursos-generales/`
Material transversal que no pertenece a una herramienta en particular.

- **`capturas/`**: Capturas de pantalla genéricas (diagramas, conceptos, comparativas).
- **`prompts/`**: Prompts reutilizables que aplican a cualquier herramienta (por ejemplo, "convertir una captura de pantalla en una nota estructurada").

## 🎯 Objetivo del repositorio

Centralizar el aprendizaje y la documentación personal sobre automatización de pruebas, facilitando:

- Consultar conceptos y comparativas antes de elegir una herramienta.
- Mantener notas y tutoriales organizados por tecnología.
- Reutilizar prompts y capturas como material de apoyo.
- Documentar la integración de pruebas automatizadas en pipelines de CI/CD.

## 🛠️ Herramientas cubiertas

| Herramienta | Tipo de testing principal |
|-------------|---------------------------|
| Selenium    | Web (multi-lenguaje)      |
| Appium      | Móvil (iOS / Android)     |
| Cypress     | Web (JavaScript/TypeScript) |
| Playwright  | Web (multi-lenguaje)      |

## ✍️ Convenciones

- Los archivos de notas y tutoriales se escriben en formato Markdown (`.md`).
- Las capturas se guardan en formato `.png` o `.jpg` dentro de la carpeta `capturas/` correspondiente.
- Los prompts se documentan en archivos `.md` indicando el contexto de uso y el resultado esperado.

---

*Repositorio en construcción — se irá actualizando conforme avance el aprendizaje en cada herramienta.*
