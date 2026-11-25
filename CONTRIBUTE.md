# CONTRIBUTE.md — Ticketera virtual

Trabajo Final de Tópicos II — 2026. Responsables: **Briant + Fernando**.

Esta guía aplica [AGENTS.md](AGENTS.md), los [requisitos](docs/requirements.md) y los [ADR aceptados](docs/architecture/adr/README.md).

## Convenciones de desarrollo aprobadas

| ID | Decisión | Estado | Responsables | Referencia de aprobación |
| --- | --- | --- | --- | --- |
| DEV-001 | Usar `main` para versiones estables y verificadas. | Aprobada | Briant + Fernando | P-005 |
| DEV-002 | Usar una rama de integración por incremento: `stage/core-ticketing`, `stage/recommendations-cache` y `stage/scalable-sales`. | Aprobada | Briant + Fernando | P-005 |
| DEV-003 | Usar `feature/*` para funcionalidades, `fix/*` para correcciones y `docs/*` para documentación. | Aprobada | Briant + Fernando | P-005 |
| DEV-004 | Usar Conventional Commits. | Aprobada | Briant + Fernando | P-005 |
| DEV-005 | Incluir el identificador del prompt en el cuerpo del commit y, cuando corresponda, el change y los requisitos relacionados. | Aprobada | Briant + Fernando | P-005 |
| DEV-006 | Integrar mediante Pull Requests con revisión humana y verificaciones aplicables. | Aprobada | Briant + Fernando | P-005 |
| DEV-007 | Automatizar las comprobaciones con GitHub Actions. | Aprobada | Briant + Fernando | P-005 |
| DEV-008 | Ejecutar las pruebas completas de estrés con k6 en un procedimiento separado de la CI habitual. | Aprobada | Briant + Fernando | P-005 |
| DEV-009 | Versionar los archivos de integración y skills oficiales de OpenSpec del proyecto. | Aprobada | Briant + Fernando | P-005 |
| DEV-010 | Usar Node.js 24 LTS como base. La versión exacta seleccionada para el proyecto es 24.22.3, registrada en `.nvmrc`. | Aprobada | Briant + Fernando | P-005 |

Las convenciones definen el flujo del equipo. El agente requiere autorización correspondiente para crear o cambiar ramas, hacer commits, push o merge. El archivado requiere aprobación humana.

## Flujo de contribución

El proyecto usa Spec-Driven Development con OpenSpec. Cada etapa académica tiene un único change. Se prevén estos identificadores:

| Etapa | Change previsto | Alcance |
| --- | --- | --- |
| I · Core | `core-ticketing` | Identidad, catálogo, venta, inventario y frontend del Core. |
| II · Recomendaciones y caché | `recommendations-cache` | Recomendaciones, proyecciones y caché. |
| III · Escalabilidad | `scalable-sales` | Venta masiva, SSE, devoluciones, búsqueda avanzada y estrés. |

Estos son nombres reservados para cuando se autorice cada etapa. El flujo del equipo es `proposal → specs → design → tasks → implement → verify → archive`. `verify` es el control de calidad adicional previo a archivar dentro de la misma etapa.

1. Revisa y confirma con los responsables objetivo, alcance y exclusiones de la propuesta, y luego los comportamientos y escenarios de las especificaciones.
2. Revisa el diseño, sus decisiones pendientes y la evidencia para los [controles OBS](docs/architecture/review-controls.md) de la etapa. Divide la implementación en grupos de tareas con dependencias y verificaciones.
3. Implementa solo tareas autorizadas con especificaciones y diseño aprobados. Registra el avance real en `tasks.md` después de verificar cada grupo. Mantén la relación entre requisitos, CA, prompts, tareas, código, pruebas y PR.
4. Verifica el incremento contra sus artefactos. Registra resultados, hallazgos y limitaciones. Archiva únicamente con aprobación expresa de Briant + Fernando.

## Ramas

| Rama | Uso aprobado |
| --- | --- |
| `main` | Versiones estables y verificadas. |
| `stage/core-ticketing` | Integración de la Etapa I. |
| `stage/recommendations-cache` | Integración de la Etapa II. |
| `stage/scalable-sales` | Integración de la Etapa III. |
| `feature/*` | Grupos de tareas del incremento. |
| `fix/*` | Correcciones. |
| `docs/*` | Documentación. |

Las ramas de trabajo se integran mediante Pull Request con revisión humana hacia su rama correspondiente. Las ramas de etapa llegan a `main` después de verificación, archivado autorizado y revisión del incremento. Las ramas de trabajo agrupan tareas y cada etapa conserva su único change de OpenSpec.

## Commits

Usa Conventional Commits con títulos concretos. Tipos habituales: `feat`, `fix`, `docs`, `test`, `refactor`, `chore` y `ci`. Incluye en el cuerpo el identificador del prompt y, cuando corresponda, el change y los requisitos asociados. Ejemplo ilustrativo:

```text
feat(ticketing): implementar reserva exclusiva de asientos

Prompt: I-006
Change: core-ticketing
Requisitos: RF-008, RF-012
```

## Pull Requests

Integra mediante Pull Requests con revisión humana y verificaciones aplicables. Incluye el objetivo, el change, las tareas cubiertas, los requisitos y CA, y los prompts utilizados. Resume las modificaciones y explica los cambios de especificación autorizados. Registra comandos, resultados reales, verificaciones pendientes, riesgos y limitaciones.

## Validación e integración continua previstas

| Etapa | Comprobaciones |
| --- | --- |
| I | ESLint y Prettier. Vitest para unitarias, Supertest para HTTP y permisos, Testcontainers para integración. Concurrencia contra PostgreSQL real, mensajería y recuperación con NATS JetStream real. Validación de OpenAPI, build de Vue y validación de artefactos OpenSpec. |
| II | Mantener las comprobaciones aplicables del Core. Verificar recomendaciones, Redis, carreras de invalidación, degradación, reconstrucción y reconciliación de proyecciones, y regresiones. |
| III | Mantener las comprobaciones aplicables de etapas anteriores. Verificar admisión, FIFO de turnos, procesamiento ordenado, backpressure, SSE, devoluciones y búsqueda. Ejecutar estrés con k6 conforme al procedimiento y los objetivos acordados antes de medir. |

GitHub Actions es la herramienta aprobada para automatizar validaciones en Pull Requests y ramas de integración. Los scripts de aplicación y workflows ejecutables de CI se incorporarán durante la implementación correspondiente. Las pruebas completas de estrés con k6 se ejecutarán por separado de la CI habitual mediante el procedimiento de la Etapa III. Consulta los manifiestos y configuraciones antes de documentar comandos o resultados.

## Cierre de tareas y archivado

Una tarea se completa cuando satisface su especificación, respeta los ADR y las invariantes, supera las verificaciones aplicables, conserva la trazabilidad y registra sus limitaciones. Marca `tasks.md` según la evidencia.

Antes de archivar:

- [ ] Propuesta, especificaciones, diseño y tareas revisados, con alcance de una sola etapa.
- [ ] Implementación y pruebas vinculadas a requisitos, criterios de aceptación y controles OBS.
- [ ] `tasks.md` refleja el trabajo comprobado y las excepciones están documentadas.
- [ ] Verificación del incremento registrada y hallazgos resueltos o aceptados explícitamente.
- [ ] Validación de los artefactos de OpenSpec aprobada y aprobación expresa de Briant + Fernando para archivar.

## Bitácora de desarrollo

Registra cada interacción relevante conforme a EA-006 y [AGENTS.md](AGENTS.md#git-alcance-y-bitácora-de-desarrollo). Usa `P-XXX` en preparación, `I-XXX` en Etapa I, `II-XXX` en Etapa II y `III-XXX` en Etapa III. Usa sufijos `.1`, `.2` y siguientes para correcciones. Conserva entradas anteriores y enlaza prompts, tareas, requisitos, resultados, revisión humana y commits cuando existan.

## Preparar un entorno con OpenSpec 2.2.0

1. Abre el repositorio en su raíz y activa Node.js **24.22.3 LTS**, fijado en [.nvmrc](.nvmrc). Comprueba `node --version` y `openspec --version` en una terminal convencional. La versión seleccionada satisface el mínimo `>=22.20.0` de OpenSpec 2.2.0. Consulta la [preparación del entorno](README.md#preparar-el-entorno).
2. Si ya dispones de una instalación compatible, reutilízala. Si falta en un entorno nuevo, consulta el procedimiento oficial para instalar la herramienta. La versión de referencia comprobada es 2.2.0. En Windows, el ejecutable npm `openspec.cmd` es una alternativa si el shell restringe el shim de PowerShell.
3. Mantén bajo versionado la [configuración de proyecto](openspec/config.yaml) y los archivos oficiales de la [integración local de Codex](.agents/skills/), incluidas sus skills y referencias. Abre Codex desde la raíz. Si una sesión abierta carece del inventario actualizado, vuelve a abrir el proyecto o inicia una sesión desde la raíz. La regeneración solo corresponde a un encargo autorizado.
4. Para un proyecto nuevo que carezca de integración, la ayuda de 2.2.0 admite `openspec init --scope project --tools codex --profile custom`. El selector `openspec config profile` tiene alcance global y `--profile custom` usa su selección. Para un proyecto ya inicializado, la ayuda ofrece `openspec update --dry-run --json` para inspeccionar la actualización y `openspec update` para aplicarla cuando se autorice. Conserva project scope y los archivos del equipo.

### Skills de Codex

Invócalas en el prompt de Codex con `$nombre-de-skill`. Cada nombre corresponde a `.agents/skills/<nombre>/SKILL.md`. Los workflows locales habilitados son `propose`, `explore`, `new`, `continue`, `apply`, `update`, `ff`, `sync`, `archive` y `verify`, además de la skill de entrada `openspec`. `bulk-archive` y `onboard` quedaron deshabilitados.

| Invocación | Uso |
| --- | --- |
| `$openspec` | Orientar el flujo y seleccionar la skill adecuada. |
| `$openspec-explore` | Explorar el problema y aclarar decisiones. |
| `$openspec-new-change` | Iniciar un change autorizado y preparar el flujo incremental. |
| `$openspec-continue-change` | Generar el siguiente artefacto del change. |
| `$openspec-propose` | Preparar un borrador de propuesta, especificaciones, diseño y tareas. |
| `$openspec-ff-change` | Completar los artefactos de planificación pendientes. |
| `$openspec-update-change` | Actualizar artefactos de planificación de un change existente. |
| `$openspec-apply-change` | Implementar las tareas autorizadas de un change aprobado. |
| `$openspec-verify-change` | Contrastar implementación, artefactos y evidencia antes de archivar. |
| `$openspec-sync-specs` | Sincronizar los deltas con las especificaciones principales conservando el change abierto, cuando se autorice. |
| `$openspec-archive-change` | Archivar un change completo, verificado y aprobado. |

Prioriza `$openspec-new-change` y `$openspec-continue-change` para revisar artefactos de uno en uno. Si se usa `$openspec-propose` o `$openspec-ff-change` para preparar borradores, la revisión humana sigue el orden del equipo. `$openspec-verify-change` contrasta la implementación y la evidencia antes del archivado.

### CLI comprobada y ejemplos de uso

Las referencias a `<id>` son ilustrativas e identifican un change que exista y esté autorizado cuando se use la operación.

| Consulta u operación | CLI o skill |
| --- | --- |
| Versión, comandos e inicialización | `openspec --version`, `openspec --help`, `openspec init --help`. |
| Consultar perfil y workflows | `openspec config get profile`, `openspec config get workflows`. Son consultas de la configuración global. |
| Consultar changes y especificaciones | `openspec list --json`, `openspec list --specs`. |
| Consultar esquema y plantillas | `openspec schema validate spec-driven`, `openspec templates --schema spec-driven --json`. |
| Iniciar un change autorizado | `openspec new change <id>` o `$openspec-new-change`. |
| Generar o revisar artefactos | `$openspec-continue-change`, `openspec status --change <id>` y `openspec instructions <artefacto> --change <id>`. |
| Validar especificaciones | `openspec validate <id> --type change --strict`. Valida el formato y contenido que comprueba el CLI. |
| Implementar y verificar | `$openspec-apply-change` para las tareas aprobadas y `$openspec-verify-change` antes del archivado. |
| Archivar un change aprobado | `$openspec-archive-change` después de la verificación y aprobación humana. El CLI también ofrece `openspec archive <id>`. |

Referencias oficiales: [OpenSpec](https://openspec.dev/), [versión 2.2.0](https://github.com/Fission-AI/OpenSpec/releases/tag/v2.2.0), [CLI](https://github.com/Fission-AI/OpenSpec/blob/v2.2.0/docs/cli.md), [configuración del proyecto](https://github.com/Fission-AI/OpenSpec/blob/v2.2.0/docs/customization.md) e [integración de Codex](https://github.com/Fission-AI/OpenSpec/blob/v2.2.0/docs/supported-tools.md#how-to-invoke).
