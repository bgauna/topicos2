# AGENTS.md — Ticketera virtual

Proyecto del Trabajo Final de Tópicos II — 2026. Responsables: **Briant + Fernando**.
Metodología: Spec-Driven Development (SDD) con OpenSpec.
El agente actúa como asistente de ingeniería dentro del alcance autorizado. Las decisiones y aprobaciones corresponden a los responsables humanos.
Todo ADR lleva exclusivamente `Autores: Briant + Fernando`. El agente carece de rol de autor, firmante o aprobador.

## Fuentes y autoridad

| Consulta | Fuente | Autoridad |
| --- | --- | --- |
| Obligaciones y entregables académicos | [Consigna de Tópicos II](docs/sources/trabajo-final-topicos-ii-2026.pdf) | Alcance y metodología de la cátedra. |
| Comportamiento y garantías | [Requisitos consolidados](docs/requirements.md) | RF, RNF, RN, DF y AMB. Incluye criterios de aceptación y trazabilidad. |
| Arquitectura de referencia | [Panorama](docs/architecture/overview.md), [límites](docs/architecture/service-boundaries.md) y [diagramas](docs/architecture/diagrams.md) | Componentes, responsabilidades y evolución. El panorama registra el estado vigente de aprobación. |
| Propiedad y contratos | [Datos](docs/architecture/data-ownership.md), [comunicación](docs/architecture/communication.md) y [consistencia](docs/architecture/consistency.md) | Autoridad sobre los datos, integración y garantías transaccionales. |
| Decisiones aceptadas | [Índice de ADR](docs/architecture/adr/README.md) | Decisiones aprobadas. Sus detalles diferidos se concretan en los diseños. |
| Auditoría pendiente | [Controles de revisión](docs/architecture/review-controls.md) | OBS y evidencia esperada para cada etapa. |
| Historial de ingeniería | [Índice de bitácora](docs/bitacora/indice.md) y [plantilla](docs/bitacora/plantilla.md) | Entradas, resultados y revisión humana conforme a EA-006. |
| Especificaciones y tareas del incremento | [Configuración de OpenSpec](openspec/config.yaml) | OpenSpec 2.2.0 inicializado. El change activo, cuando se autorice y exista, delimita artefactos y tareas aprobadas. |
| Ramas, validaciones y CI | [CONTRIBUTE.md](CONTRIBUTE.md#convenciones-de-desarrollo-aprobadas) | Flujo de contribución y convenciones DEV aprobadas en P-005. |
| Estado y preparación | [README](README.md), [entorno](docs/preparation/environment.md) y [cierre de Fase 0](docs/preparation/closure.md) | Versión de Node.js, prerrequisitos comprobados y pendientes. |
| Alcance de esta ejecución | Prompt actual | Autoriza la tarea concreta y sus entregables. |

- Consulta el prompt junto con los requisitos, los ADR aceptados y el change pertinente. Las especificaciones respetan estas decisiones vigentes.
- Si una instrucción entra en conflicto con una decisión aceptada, identifica su ID, explica el impacto y comprueba o solicita autorización expresa. Actualiza formalmente la documentación cuando el cambio esté autorizado.
- Las contradicciones de requisitos y políticas de negocio se elevan a Briant + Fernando. Mantén diferidos los parámetros pendientes y trabaja en las partes autorizadas que puedan avanzar.
- Verifica la existencia de cada artefacto antes de usarlo. Las decisiones, los requisitos y los ADR conservan su autoridad frente a esta guía operativa.

## Stack y convenciones

- Usa JavaScript vanilla ES6+, Node.js y Express en backend, y Vue 3 con Vite y JavaScript en frontend. Verifica la versión de Node.js fijada en [.nvmrc](.nvmrc) según el [README](README.md#preparar-el-entorno).
- Usa PostgreSQL con Sequelize, NATS con JetStream y Redis según su etapa. El entorno local usa Docker y Docker Compose.
- Usa JWT, bcrypt, Zod, OpenAPI, Swagger UI, Pino y Server-Sent Events (SSE) según los contratos aceptados.
- Usa Vitest, Supertest y Testcontainers para las verificaciones correspondientes, y k6 para estrés en la Etapa III.
- Usa npm y npm Workspaces, Git y GitHub, GitHub Actions, ESLint, Prettier y OpenSpec.
- Está prohibido incorporar TypeScript, Drizzle ORM, Prisma, React, sustituciones tecnológicas o dependencias de aplicación sin aprobación.
- Sigue ECMAScript Modules (ESM) y las convenciones aceptadas de [ADR-003](docs/architecture/adr/ADR-003.md). Mantén como diferidas las convenciones cuyo diseño siga abierto.

## Flujo de OpenSpec

1. **Etapa I — Core.** Identidad, catálogo, venta, inventario, bitácora HTTP, mockups y frontend del Core.
2. **Etapa II — Recomendaciones y caché.** Preferencias, historial proyectado, recomendaciones, Redis y vistas del incremento.
3. **Etapa III — Escalabilidad.** Colas y turnos por evento, SSE, devoluciones, búsqueda avanzada y estrés.

Cada etapa corresponde a un único change. Consulta el flujo y las [skills y comandos comprobados con OpenSpec 2.2.0](CONTRIBUTE.md#preparar-un-entorno-con-openspec-220) en `CONTRIBUTE.md`.

- Consulta el change activo antes de modificar código. Inicia implementación únicamente con especificaciones y diseño aprobados.
- Implementa solo tareas autorizadas del incremento. Cualquier change adicional o adelanto de etapa requiere autorización expresa.
- Vincula tareas y cambios con requisitos, criterios de aceptación y controles OBS aplicables.
- En cada diseño registra cómo se abordarán los controles de su etapa y qué evidencia permitirá verificarlos.
- Actualiza `tasks.md` con trabajo realmente completado y verificado. Deja abiertas las tareas incompletas.
- Ante una desviación, registra la diferencia y corrige dentro del alcance autorizado o informa el impedimento. Cualquier cambio de especificación requiere el tratamiento formal correspondiente.
- Archiva únicamente con aprobación humana y verificación previa. Usa comandos de OpenSpec solo después de comprobar su existencia y disponibilidad.

## Arquitectura y límites

| Componente | Responsabilidad |
| --- | --- |
| API Gateway | Entrada HTTP, enrutamiento, autenticación, correlación y registro. Proxy SSE desde III. Las reglas de venta pertenecen a Ticketing. |
| Identity | Usuarios, roles, credenciales, registro, administrador inicial y emisión de JWT. |
| Catalog | Metadatos, propiedad y ciclo comercial del evento, publicación y búsqueda en III. |
| Ticketing | Inventario, zonas, asientos, precios, reservas, pagos simulados, compras, tickets y devoluciones en III. Autoridad de habilitación efectiva de venta. |
| Recommendations | Preferencias, proyecciones de catálogo e historial y recomendaciones desde II. |
| Waiting Room | Colas, admisión, turnos FIFO, vigencia y posición por SSE en III. Ticketing verifica las guardas de venta masiva en sus APIs. |
| Web | Vistas por actor, navegación, presentación de resultados e integración mediante Gateway. |

- Mantén cada servicio desplegable de forma independiente y dentro de sus límites aceptados.
- Cada servicio persistente posee su base y escribe sus propios datos. Excluye escrituras directas en bases ajenas y relaciones físicas entre bases.
- Usa contratos explícitos y referencias mediante identificadores. Los paquetes compartidos alojan contratos, sin modelos Sequelize, acceso a persistencia ni lógica de dominio de otros servicios.
- Usa [ADR-001](docs/architecture/adr/ADR-001.md), [ADR-002](docs/architecture/adr/ADR-002.md), [ADR-004](docs/architecture/adr/ADR-004.md) y [ADR-005](docs/architecture/adr/ADR-005.md) para verificar el alcance.

## Invariantes críticas

- Un asiento excluye reservas vigentes incompatibles y admite como máximo una venta activa simultánea. Una zona nunca supera su capacidad comprometida (RN-001, RN-002).
- Cada reserva admite a lo sumo una compra confirmada. Una compra repetida conserva un único efecto. Convertir una reserva en venta conserva el compromiso de inventario (RN-003, RN-009).
- Al vencer una reserva, la confirmación deja de ser válida y el inventario se repone una vez. Una devolución confirmada repone una sola vez, con invalidación del ticket e historial conservado (RN-004, RN-005).
- Solo se vende para eventos publicados y efectivamente habilitados (RN-011). El backend valida propietario, rol y recurso (RN-006, RN-007).
- En venta masiva, conserva el FIFO de turnos, el contrato de procesamiento ordenado y el acceso por turno vigente (RN-008, RN-013).
- Consulta las reglas en [requisitos](docs/requirements.md#7-reglas-de-negocio) y [ADR-007](docs/architecture/adr/ADR-007.md) antes de cambiar las rutas críticas.

## PostgreSQL, Sequelize y concurrencia

- PostgreSQL es la fuente de verdad del inventario. Ticketing protege las operaciones críticas mediante transacciones locales.
- Usa guardas transaccionales y restricciones persistentes. Evita decidir una escritura desde una lectura de disponibilidad sin protección.
- Incluye todas las mutaciones de la operación atómica en la misma transacción y respeta el orden de bloqueos que apruebe el diseño.
- Evalúa la vigencia de reserva con el tiempo efectivo de PostgreSQL después de adquirir las guardas pertinentes. Aplica el diseño de OBS-001 y OBS-002.
- Usa migraciones versionadas. Evita sincronización automática destructiva sobre datos persistentes. Cambiar una migración aplicada exige una estrategia aprobada.
- Sigue [consistencia](docs/architecture/consistency.md) y el diseño OpenSpec correspondiente para bloqueos, aislamiento, reintentos y restricciones concretas.

## Mensajería, idempotencia y consistencia

- Usa NATS con JetStream cuando se requiera persistencia de mensajes. Respeta contratos, versiones y revisiones de agregado cuando correspondan.
- Con Transactional Outbox, guarda la operación y el evento pendiente en la misma transacción local. Publica después del commit con Event ID estable.
- Trata reentregas y reintentos explícitamente. Persiste el efecto y la deduplicación del consumidor como unidad cuando corresponda. Confirma a JetStream después del procesamiento persistente exitoso.
- Diseña para entrega repetida. JetStream no garantiza un único efecto de negocio y el outbox es un patrón de transacción local.
- Distingue reenvío, compra completada y nuevo intento de pago según el contrato. Aplica la unicidad persistente por reserva además de la clave y su vigencia.
- Reserva y confirmación consultan el inventario de PostgreSQL. Redis queda limitado a caché de lectura conforme a la política de II.
- Consulta [ADR-008](docs/architecture/adr/ADR-008.md) y [ADR-009](docs/architecture/adr/ADR-009.md), junto con los controles OBS de la etapa.

## Seguridad y frontend

- Aplica JWT y vigencia de privilegios según el diseño, bcrypt para contraseñas y autorización por rol y recurso en el backend. Verifica la identidad del servicio y aísla los servicios internos.
- Valida entradas con Zod, usa consultas parametrizadas y protege los datos privados de clientes. Conserva las validaciones y controles de seguridad durante las pruebas.
- Excluye contraseñas, tokens y credenciales sensibles de logs, ejemplos y URLs. Mantén secretos reales fuera del control de versiones.
- Usa Vue 3 con Vite y JavaScript, cliente HTTP hacia el Gateway y vistas por actor. Representa éxito, error, fallo de red y disponibilidad de inventario según los contratos.
- Los mockups verifican recorridos visuales. El frontend funcional requiere prueba con APIs. La visibilidad de un botón o ruta solo afecta la interfaz.
- Integra SSE en III con autenticación y recuperación de posición. Vue Router, Pinia y cualquier dependencia adicional requieren aprobación previa.
- Consulta [ADR-006](docs/architecture/adr/ADR-006.md) y [ADR-012](docs/architecture/adr/ADR-012.md).

## Observabilidad

- Usa Pino y propaga Request ID y Correlation ID según el contrato.
- Mide HTTP con reloj monotónico desde la recepción e incluye autenticación, autorización y procesamiento.
- Registra éxito y fallo hasta preparar la respuesta convencional, antes de enviarla. La finalización del transporte corresponde a una medición separada si se instrumenta.
- En SSE registra apertura y latencia de aceptación, y por separado cierre y duración de la conexión. Redacta información sensible en todos los registros.
- La bitácora HTTP es operativa (RNF-009 y [ADR-010](docs/architecture/adr/ADR-010.md)). La bitácora de desarrollo documenta el proceso de ingeniería (EA-006).

## Testing y verificación

- Usa Vitest para reglas y regresiones, Supertest para APIs y permisos, y Testcontainers para integración con PostgreSQL y NATS JetStream reales cuando corresponda.
- Verifica concurrencia, exclusividad, expiración, idempotencia, autorización, reentrega y fallos parciales. Usa PostgreSQL real para garantías de bloqueos y restricciones, y mensajería real para sus garantías de integración.
- Los mocks sirven para comportamientos aislados. Las garantías de exclusividad transaccional requieren evidencia contra el motor real.
- Valida contratos OpenAPI, ESLint y Prettier. En III ejecuta k6 con perfil y umbrales aprobados antes de medir.
- Agrega una prueba de regresión para cada corrección relevante cuando corresponda. Conserva las pruebas y la fuerza de sus aserciones.
- Consulta [CONTRIBUTE.md](CONTRIBUTE.md) y la configuración de las suites cuando se incorporen. Comprueba que los scripts y suites estén disponibles antes de ejecutarlos.
- Informa comandos, resultados reales y verificaciones omitidas o bloqueadas. Usa [ADR-011](docs/architecture/adr/ADR-011.md) como guía.

## Git, alcance y bitácora de desarrollo

- Inspecciona Git antes de editar e informa cambios preexistentes relevantes. Preserva cambios ajenos y limita los tuyos al alcance autorizado.
- Evita refactorizaciones sin relación y operaciones destructivas. Commit, push, merge y cambios de rama requieren autorización correspondiente.
- Aplica las [convenciones DEV aprobadas](CONTRIBUTE.md#convenciones-de-desarrollo-aprobadas) y respeta las autorizaciones de Git de esta guía.
- Registra cada interacción relevante conforme a EA-006 cuando la bitácora esté dentro del alcance autorizado. Conserva entradas y correcciones anteriores.
- Cada entrada incluye identificador, fecha, fase, etapa, objetivo, prompt completo, respuesta, artefactos, decisiones, observaciones, verificación humana, pruebas y resultados cuando correspondan, correcciones, prompts relacionados y commit cuando exista.
- Registra solo resultados y aprobaciones con evidencia. Las entradas históricas conservan sus resultados originales.

## Protocolo de ejecución e informe

**Antes.** Lee las instrucciones aplicables, delimita el objetivo, consulta requisitos, ADR, OBS y change relacionados, inspecciona Git e identifica restricciones, dependencias y decisiones bloqueantes.

**Durante.** Realiza cambios mínimos y autorizados, respeta invariantes y contratos, conserva la trazabilidad y ejecuta las verificaciones aplicables.

**Después.** Revisa los archivos afectados y los criterios de finalización. Actualiza únicamente los artefactos autorizados y termina al completar la tarea. La siguiente tarea o etapa requiere autorización propia.

El informe de ejecución incluye:

- Objetivo y resumen del trabajo realizado.
- Archivos creados o modificados, requisitos y tareas cubiertas, y decisiones aplicadas.
- Comandos ejecutados, resultados reales y verificaciones no realizadas.
- Problemas o bloqueos, riesgos, definiciones pendientes y estado final.

Distingue cambios realizados, propuestas y verificaciones por ejecutar. Usa evidencia de ejecución para respaldar resultados informados.
