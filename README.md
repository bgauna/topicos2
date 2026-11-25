# Ticketera virtual

Trabajo Final de Tópicos II — 2026. Responsables: **Briant + Fernando**.

Proyecto de una plataforma de venta de entradas para eventos. El Cliente consulta eventos, reserva y compra tickets. El Event Owner administra y publica sus eventos. El Administrador gestiona usuarios, roles y operaciones autorizadas. El objetivo es mantener la integridad del inventario bajo concurrencia y ampliar el sistema con recomendaciones, devoluciones y venta masiva.

**Estado actual:** cierre documental de la Fase 0 preparado para revisión humana. OpenSpec 2.2.0 está configurado para Codex y todavía no hay changes. El motor Docker respondió en la verificación, pero falta activar Node.js 24.22.3 en una terminal convencional. El repositorio contiene documentación y configuración preparatoria. Las aplicaciones, APIs, bases, despliegues y pipelines corresponden a incrementos futuros. [Cierre de Fase 0](docs/preparation/closure.md) · [Verificación del entorno](docs/preparation/environment.md)

## Arquitectura y etapas

La arquitectura aceptada separa responsabilidades por servicio. La Web consume APIs a través del Gateway. Identity gestiona identidad y roles. Catalog administra eventos. Ticketing es propietario de inventario, reservas y compras, con PostgreSQL como fuente de verdad. Los servicios persistentes usan bases propias. Los casos asíncronos usan NATS JetStream y Transactional Outbox según el diseño.

| Etapa | Change previsto | Alcance |
| --- | --- | --- |
| I · Core | `core-ticketing` | Web Vue, Gateway, Identity, Catalog y Ticketing. Identidad, eventos, inventario, reservas, compra y observabilidad. |
| II · Recomendaciones y caché | `recommendations-cache` | Recommendations, preferencias, proyecciones, Redis y vistas de recomendaciones. |
| III · Escalabilidad | `scalable-sales` | Waiting Room, colas y SSE. Devoluciones en Ticketing, búsqueda avanzada en Catalog y pruebas de estrés. |

Cada etapa se gestiona como un solo change de OpenSpec. Consulta el [panorama de arquitectura](docs/architecture/overview.md), los [ADR aceptados](docs/architecture/adr/README.md) y los [controles de auditoría](docs/architecture/review-controls.md).

## Stack aprobado

- JavaScript vanilla ES6+ con ESM, Node.js **24.22.3 LTS**, Express, Vue 3 y Vite.
- PostgreSQL, Sequelize, NATS con JetStream, Redis desde II, Docker y Docker Compose.
- JWT, bcrypt, Zod, OpenAPI, Swagger UI, Pino y Server-Sent Events desde III.
- Vitest, Supertest, Testcontainers y k6 en III. npm y npm Workspaces, Git, GitHub, GitHub Actions, ESLint, Prettier y OpenSpec.

## Estructura

**Presente:** `docs/` contiene la consigna, requisitos, arquitectura, bitácora y preparación. [openspec/](openspec/) contiene la configuración y los directorios de especificaciones y changes aún vacíos. [.agents/skills/](.agents/skills/) contiene la integración de Codex. En la raíz están esta guía, las instrucciones y la configuración básica de desarrollo.

**Objetivo conceptual** según [ADR-002](docs/architecture/adr/ADR-002.md):

| Rutas previstas | Etapa o finalidad |
| --- | --- |
| `apps/api-gateway/`, `apps/web/` | I · Entrada de APIs y Web. |
| `services/identity/`, `services/catalog/`, `services/ticketing/` | I · Servicios del Core. |
| `services/recommendations/` | II · Recomendaciones. |
| `services/waiting-room/` | III · Sala de espera. |
| `packages/contracts/`, `infra/` | Contratos y configuración de infraestructura a medida que se autoricen. |
| `.github/workflows/`, `tests/load/` | Automatización de CI en el incremento correspondiente y cargas completas en III. |

Estas rutas futuras se materializarán mediante tareas autorizadas. La estructura de npm Workspaces aún es conceptual.

## Preparar el entorno

1. Desde la raíz, activa **Node.js 24.22.3** con tu gestor de versiones o con la distribución oficial disponible para tu sistema. [.nvmrc](.nvmrc) fija la versión, y su lectura depende del gestor. Con un gestor compatible que ya tenga la versión instalada, usa `nvm use` si reconoce `.nvmrc`; en Windows con nvm-windows, indica la versión explícitamente con `nvm use 24.22.3`. El archivo por sí solo no instala ni cambia Node.js.
2. Comprueba la versión efectiva en una terminal convencional. En PowerShell, `Get-Command node` y `where.exe node` ayudan a identificar qué ejecutable tiene prioridad. El runtime interno de Codex sirve como alternativa temporal para comprobaciones, y la preparación del proyecto requiere Node.js 24.22.3 disponible fuera de Codex.
3. Comprueba Git, npm, Docker CLI, Docker Compose, el motor de Docker y OpenSpec 2.2.0 con los comandos de abajo. Usa `openspec.cmd` si Windows restringe el shim de PowerShell. La guía de contribución explica cómo usar las skills y la configuración existente.

```text
node --version
npm --version
git --version
docker --version
docker compose version
docker info
docker ps
openspec --version
openspec list
```

La referencia de Node.js deberá guiar los futuros manifiestos, imágenes y workflows de CI. [.nvmrc](.nvmrc) · [Distribución oficial Node.js 24.22.3 LTS](https://nodejs.org/en/download/archive/v24.22.3) · [Contribución y uso de OpenSpec](CONTRIBUTE.md)

## Documentación

- [Consigna académica](docs/sources/trabajo-final-topicos-ii-2026.pdf) y [requisitos consolidados](docs/requirements.md).
- [Guía de contribución](CONTRIBUTE.md) e [instrucciones para agentes](AGENTS.md).
- [Índice de bitácora](docs/bitacora/indice.md) y [plantilla](docs/bitacora/plantilla.md).
