# Arquitectura de referencia

**Proyecto:** ticketera virtual, Trabajo Final de Tópicos II — 2026. **Responsables:** Briant + Fernando. **Estado:** propuesta P-002 para revisión, Fase 0. La referencia es el [documento consolidado de requisitos](../requirements.md) y la [consigna académica](../sources/trabajo-final-topicos-ii-2026.pdf).

## Objetivos y alcance

Diseñar una plataforma con autorización por recurso, venta concurrente sin duplicaciones, recuperación verificable ante fallos y despliegue independiente. La arquitectura habilita la experiencia web por rol, y en incrementos posteriores recomendaciones, caché, búsqueda, devoluciones y venta masiva.

La consigna exige límites de microservicios, comunicación síncrona y asíncrona, manejo de fallos, consistencia eventual, bitácora HTTP, caché y soporte de alta demanda. Las etapas y entregables académicos se conservan. P-001 y P-001.1 aprueban el stack y las políticas DF y AMB, entre ellas datos por servicio, PostgreSQL transaccional, JWT, Docker Compose, simulación de pagos, outbox, colas por evento y Vue. P-002 propone concretar el reparto de componentes, las guardas transaccionales, la coordinación comercial y la organización técnica. Las propuestas requieren revisión y se detallarán en los `design.md` de los tres changes oficiales.

## Estilo y principios

Proponer microservicios por capacidad de negocio, API Gateway con un frontend web, HTTP para decisiones inmediatas y eventos persistentes para proyecciones y recuperación. El estilo cumple el objetivo académico y permite evolucionar Catalog, Recommendations y Waiting Room de forma independiente del núcleo transaccional. La separación se limita a capacidades con propiedad de datos y ritmos de evolución distintos. Reservas, compras, pagos simulados y devoluciones se agrupan en Ticketing para mantener las invariantes en una transacción local.

- **Propiedad.** Un escritor autorizado por dato. Referencias entre servicios mediante identificadores y contratos. Las transacciones de Ticketing consultan PostgreSQL y la puerta efectiva local.
- **Autoridad.** El Gateway autentica y enruta. El backend propietario aplica roles y permisos sobre el recurso. Las copias de lectura y la interfaz son orientativas.
- **Fallos parciales.** Publicar después del commit mediante outbox, deduplicar con el efecto del consumidor y recuperar estados pendientes. Un cierre de venta es efectivo cuando Ticketing confirma el cerco.
- **Despliegue.** Cada servicio tiene proceso, configuración, credenciales, comprobación de salud y artefacto de despliegue propios. En local, Compose usa redes internas y volúmenes persistentes para PostgreSQL y JetStream. Redis es descartable como caché. Solo la entrada web y el Gateway requieren exposición a usuarios. La instancia PostgreSQL inicial implica un punto de fallo y capacidad compartidos aunque las bases estén aisladas.
- **Stack.** Mantener JavaScript vanilla (ES6+), Node.js, Express, Vue 3 con Vite, PostgreSQL, Sequelize, NATS con JetStream, Redis, Docker y Compose, JWT y bcrypt, Zod, OpenAPI y Swagger UI, Vitest, Supertest, Testcontainers, k6, Pino, SSE, npm y npm Workspaces, Git y GitHub, GitHub Actions, ESLint, Prettier y OpenSpec. TypeScript, Drizzle ORM, Prisma y React están excluidos. El diseño tampoco incorpora Kubernetes, Elasticsearch ni servicios externos adicionales.
- **Verificación.** Combinar unitarias, contratos, HTTP, concurrencia real, pruebas de recuperación y k6 en III. La medición HTTP termina al preparar la respuesta; SSE registra por separado aceptación y cierre. [ADR-010](adr/ADR-010.md) y [ADR-011](adr/ADR-011.md).

## Evolución y dependencias

| Etapa oficial | Componentes y mecanismos | Dependencias y comprobaciones |
| --- | --- | --- |
| I · Core | Web Vue, Gateway, Identity, Catalog y Ticketing. PostgreSQL con tres bases, NATS con JetStream, outbox, registro HTTP y pruebas. | Identity habilita actores. Catalog coordina preparación y apertura con Ticketing. Comprobar reservas, compras, idempotencia y recuperación de publicación con NATS caído. Catálogo e historial originan los datos de II y III. |
| II · Recomendaciones y caché | Recommendations y su base, Redis, proyecciones, preferencias, vistas y reglas explicables. | Consumir catálogo y compras de I. Usar Cache-Aside, TTL por dato, invalidación aplicable y degradación acotada; evaluar caché de cliente, CDN, servicio y distribuida. Redis excluido de la decisión de inventario. Probar reentrega, desconexión y convergencia. |
| III · Escalabilidad | Waiting Room y su base, cola por evento, FIFO, procesamiento ordenado, backpressure y SSE. Catalog suma búsqueda PostgreSQL y Ticketing suma devoluciones. Escenarios de estrés con k6. | Reusar identidad, ciclo comercial e invariantes del Core y comprobar permisos y turnos en la escritura propietaria. Verificar orden, reintegros, reconexión SSE, fallos y capacidad con el plan de estrés aprobado previamente. |

Cada etapa corresponde a un único change con flujo `proposal → specs → design → tasks → implement → archive` y verificación del equipo previo a archivar. Esta propuesta de Fase 0 orienta los tres diseños. El [índice de ADR](adr/README.md) identifica los antecedentes y las decisiones pendientes de revisión.

## Ventajas, limitaciones y alternativas

La independencia de despliegue, la propiedad clara y la capacidad de verificar fallos parciales satisfacen la consigna. El coste es coordinar contratos, credenciales, observabilidad y pruebas entre procesos, además de asumir latencia y posibles estados pendientes. Un monolito modular simplificaría el despliegue y permitiría transacciones dentro de un único proceso, pero reduciría la evidencia académica de comunicación y despliegue independiente. Separar pagos, reservas y devoluciones produciría límites de consistencia artificiales al inicio. El alcance inicial del simulador y la invariante de inventario favorecen Ticketing cohesivo. Una base compartida entre servicios simplificaría consultas, pero comprometería el ownership aprobado en DF-011. Los riesgos y compensaciones del diseño propuesto se detallan a continuación.

## Riesgos arquitectónicos

| Riesgo | Impacto | Componentes | Mitigación propuesta | Etapa | Requisito | ADR | Verificación |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Venta duplicada de asiento | Dos compradores obtienen un asiento incompatible | Ticketing | Transición condicional y unicidad local | I | RN-001, RF-012 | [ADR-007](adr/ADR-007.md) | CA-RF-012-01 |
| Sobreventa de zonas | Un compromiso excede cupo o produce contadores negativos | Ticketing | Actualización condicional y restricciones de capacidad | I | RN-002, RN-009 | [ADR-007](adr/ADR-007.md) | CA-RF-013-01 |
| Confirmación simultánea de reserva | Dos compras o tickets para la misma reserva | Ticketing | Transición única y restricción persistente de compra por reserva | I | RN-003, RF-014 | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md) | CA-RF-012-02 |
| Vencimiento contra compra | Unidad vendida y liberada por la carrera | Ticketing | Bloquear reserva y evaluar el tiempo al transicionar | I | RN-004, RF-009 | [ADR-007](adr/ADR-007.md) | CA-RF-009-04 |
| Fallo después del commit | Respuesta perdida o hecho sin publicación | Ticketing, NATS | Compra durable con outbox y repetición segura | I | RNF-006, RNF-022 | [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md) | CA-RF-014-01, CA-RNF-006-02, CA-RNF-006-03 |
| Duplicación de mensajes | Proyección o efecto secundario repetido | Consumidores | Deduplicar en la transacción que aplica el efecto | II, III | RNF-007, RNF-022 | [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | CA-RNF-007-02, CA-RNF-022-02 |
| Consumidor desconectado | Historial incompleto o atrasado | Recommendations, NATS | Consumidor durable y replay o recuperación autorizada | II | RNF-007, RNF-020 | [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | CA-RNF-007-01 |
| Inventario sin preparar al publicar | Listado comprable sin unidades correctamente configuradas | Catalog, Ticketing | Preparación verificable más autorización de apertura protegida | I | RN-011, RF-005 | [ADR-007](adr/ADR-007.md) | CA-RF-005-01 y probar fallo de preparación antes de abrir |
| Estado comercial desactualizado | Compra después del cierre confirmado o reapertura atrasada | Catalog, Ticketing | Cerrar primero y usar generaciones monotónicas en la puerta local | I | RN-011, RF-010 | [ADR-007](adr/ADR-007.md) | CA-RF-005-02 y pruebas de comando atrasado y cierre concurrente |
| JWT vigente tras cambio de rol | Facultades obsoletas permiten cambios indebidos | Identity, Gateway y servicios | Validación fresca para operaciones privilegiadas y vigencia acotada | I | RF-003, RF-004, RF-023 | [ADR-006](adr/ADR-006.md) | CA-RF-004-02 y comprobar acceso con el JWT previo al cambio |
| Acceso directo a servicios internos | Omisión de controles del Gateway | Gateway y servicios | Red privada, autenticación del servicio y permisos por recurso en destino | I | RF-003, RN-007 | [ADR-006](adr/ADR-006.md) | CA-RF-003-02, CA-RF-003-03 y rechazar llamada interna sin identidad de servicio |
| Credenciales en registros | Exposición de contraseñas, JWT o secretos | Gateway y servicios | Campos permitidos, redacción y pruebas de errores y SSE | I, III | RNF-009, RNF-013 | [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md) | CA-RNF-013-01 |
| Cambio de precio posterior | Historial o reintegro calculado con el precio nuevo | Ticketing | Instantánea de línea e importe de compra persistidos | I, III | RF-010, RF-017 | [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md) | CA-RF-010-01 y probar consulta y reintegro después de actualizar el precio |
| Zonas horarias y horas ambiguas | Venta o solicitud en un instante incorrecto | Catalog, Ticketing | UTC, zona IANA y eventos de frontera en pruebas | I, III | RF-025, RN-012 | [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md) | CA-RF-015-01, CA-RF-015-03 y probar fecha local ambigua |
| Proceso de expiración interrumpido | Inventario retenido y riesgo de confirmación tardía | Ticketing | Tarea recuperable y vigencia local independiente de la tarea | I | RF-009, RN-004 | [ADR-007](adr/ADR-007.md) | CA-RF-009-02, CA-RF-009-04 y reprocesar tras detener la tarea |
| Caída de Redis | Sobrecarga del origen o resultado personalizado indisponible | Recommendations, Redis | Degradación acotada, límites de consultas y aislamiento por identidad | II | RNF-010, RNF-018 | [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md) | CA-RNF-010-02, CA-RNF-010-03 y CA-RNF-018-01 |
| Omisión de sala de espera | Reserva o compra sin turno válido | Waiting Room, Ticketing | Validación de turno y generación en la escritura propietaria | III | RF-020, RF-021, RN-013 | [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md) | CA-RF-021-02 y probar API directa sin turno o con turno de otro usuario |
| Pérdida de orden con varios consumidores | Adelantamiento o bloqueo de cabecera | Waiting Room, Ticketing | Secuencia persistente y coordinación por evento con resolución de posiciones terminales | III | RF-021, RN-008, RNF-011 | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | CA-RF-021-01, CA-RF-021-03, CA-RNF-011-01 |

## Decisiones pendientes y diferidas

| Momento | Definición |
| --- | --- |
| Revisión P-002 | Límites de cinco servicios y Gateway, monorepo, contratos y estándares, propiedad física, protocolo de cierre y apertura, guardas locales, idempotencia, outbox, observabilidad, seguridad, estrategia de testing y organización web. La selección propuesta y las alternativas constan en [ADR-001 a ADR-012](adr/README.md). |
| Design de I | Modos de bloqueo, aislamiento y reintentos operativos. Cercos de preparación, publicación y cierre, configuración y manejo de comandos pendientes. Consultas de estado de identidad, caducidad JWT, vigencia de privilegios y credenciales entre servicios. Respuestas idempotentes, solicitudes en curso y semántica de intentos de pago. Conservación de precios históricos y elección del momento de fijación. Política de cancelación con ventas confirmadas antes de habilitarla; validar reprogramación si afecta derechos. Contratos, eventos, despliegue y retenciones de mensajería. |
| Design de II | Inicialización y recuperación de proyecciones, reglas y pesos de recomendaciones, popularidad o proximidad, datos cacheables y ubicaciones, TTL, invalidación, límites de consultas y degradación ante fallo de Redis. |
| Design de III | Habilitación autorizada, capacidad de cada cola, duración y revocación de turnos, abandonos, reintentos, control de procesamiento y admisión con múltiples consumidores, SSE autenticado y reconexión, búsqueda y semántica de fechas y relevancia, contratos de devoluciones, perfiles y umbrales de estrés. |

La coordinación comercial debe confirmar el cierre efectivo antes de que Catalog lo dé por aplicado. Requiere validar el compromiso entre disponibilidad y ausencia de ventas posteriores al cierre, especialmente si Ticketing está temporalmente inaccesible. En III, el contrato de procesamiento ordenado y el alcance de una prueba de turno requieren revisión explícita. La propuesta mantiene la política de cancelación con ventas confirmadas diferida y conserva intacta la especificación aprobada.

## Trazabilidad arquitectónica

Cada fila referencia criterios existentes del [documento de requisitos](../requirements.md#8-criterios-de-aceptación). Las pruebas adicionales de riesgos se concretarán en el change que corresponda.

| Requisito o regla | Componente responsable | ADR | Mecanismo | Etapa | Criterios |
| --- | --- | --- | --- | --- | --- |
| RF-001 | Identity, Gateway | [ADR-006](adr/ADR-006.md) | Emisión y validación de identidad por JWT | I | CA-RF-001-01, CA-RF-001-02 |
| RF-002 | Identity, servicios de recurso | [ADR-006](adr/ADR-006.md) | Roles y autorizaciones sobre el recurso propietario | I | CA-RF-002-01 |
| RF-003 | Gateway, servicios de recurso | [ADR-006](adr/ADR-006.md) | Validar identidad y propiedad en backend; consulta del ticket por titular | I; reaplica en III | CA-RF-003-01, CA-RF-003-02, CA-RF-003-03 |
| RF-004 | Identity | [ADR-006](adr/ADR-006.md) | Promoción autorizada con control de rol vigente | I | CA-RF-004-01, CA-RF-004-02 |
| RF-005 | Catalog, Ticketing | [ADR-001](adr/ADR-001.md), [ADR-007](adr/ADR-007.md) | Propiedad, ciclo comercial y protocolo de puerta versionada | I | CA-RF-005-01, CA-RF-005-02 |
| RF-006 | Ticketing, Catalog | [ADR-007](adr/ADR-007.md) | Configurar recursos vendibles con guardas de inventario | I | CA-RF-006-01, CA-RF-006-02, CA-RF-006-03 |
| RF-007 | Ticketing | [ADR-007](adr/ADR-007.md) | Modelo específico de asiento y de capacidad | I | CA-RF-007-01 |
| RF-008 | Ticketing | [ADR-007](adr/ADR-007.md) | Reserva exclusiva o atómica sobre cupo | I | CA-RF-008-01 |
| RF-009 | Ticketing | [ADR-007](adr/ADR-007.md) | Expiración recuperable y transición única | I | CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04 |
| RF-010 | Ticketing | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md) | Confirmación local, importe histórico y publicación posterior | I | CA-RF-010-01 |
| RF-011 | Ticketing | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md) | Simulador local y distinción de intento nuevo | I | CA-RF-011-01, CA-RF-011-02 |
| RF-012 | Ticketing | [ADR-007](adr/ADR-007.md) | Bloqueo o mutación condicional más unicidad persistente | I | CA-RF-012-01, CA-RF-012-02 |
| RF-013 | Ticketing | [ADR-007](adr/ADR-007.md) | Actualización condicional de zona y restricciones | I | CA-RF-013-01 |
| RF-014 | Ticketing | [ADR-008](adr/ADR-008.md) | Clave por usuario y operación, hash y unicidad persistente | I | CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06 |
| RF-015 | Ticketing | [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md) | Validar titularidad y plazo bajo revisión autoritativa | III | CA-RF-015-01, CA-RF-015-02, CA-RF-015-03 |
| RF-016 | Ticketing | [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md) | Resolución autorizada y conservación tras rechazo | III | CA-RF-016-01, CA-RF-016-02, CA-RF-017-01 |
| RF-017 | Ticketing | [ADR-007](adr/ADR-007.md), [ADR-009](adr/ADR-009.md) | Reposición, invalidación y reintegro atómicos | III | CA-RF-017-01, CA-RF-017-02 |
| RF-018 | Recommendations | [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md) | Preferencias, proyección y puntuación explicable | II | CA-RF-018-01, CA-RF-018-02 |
| RF-019 | Catalog | [ADR-003](adr/ADR-003.md), [ADR-004](adr/ADR-004.md) | Búsqueda AND y relevancia en PostgreSQL | III | CA-RF-019-01, CA-RF-019-02, CA-RF-019-03 |
| RF-020 | Waiting Room, Ticketing | [ADR-001](adr/ADR-001.md), [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | Admisión persistente por evento y activación cercada | III | CA-RF-020-01, CA-RF-020-02, CA-RF-020-03, CA-RF-020-04 |
| RF-021 | Waiting Room, Ticketing | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | FIFO de turnos y coordinación de procesamiento por evento | III | CA-RF-021-01, CA-RF-021-02, CA-RF-021-03 |
| RF-022 | Waiting Room, Gateway, Web | [ADR-005](adr/ADR-005.md), [ADR-010](adr/ADR-010.md), [ADR-012](adr/ADR-012.md) | SSE y reconstrucción de posición por identidad | III | CA-RF-022-01, CA-RF-022-02 |
| RF-023 | Identity, Gateway y servicios | [ADR-006](adr/ADR-006.md) | JWT válido, verificación y control de vigencia | I | CA-RF-023-01 |
| RF-024 | Identity | [ADR-006](adr/ADR-006.md) | Registro de Cliente y administrador inicial | I | CA-RF-024-01, CA-RF-024-02 |
| RF-025 | Catalog, Ticketing | [ADR-004](adr/ADR-004.md) | Datos de evento por propietario, composición y referencia estable | I | CA-RF-025-01 |
| RF-026 | Identity, Catalog, Ticketing, Gateway | [ADR-001](adr/ADR-001.md), [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md) | Operaciones administrativas permitidas y ventas históricas protegidas | I–III | CA-RF-026-01, CA-RF-026-02, CA-RF-026-03 |
| RF-027 | Web, Gateway | [ADR-012](adr/ADR-012.md) | Recorridos de los tres roles integrados en I | I | CA-RF-027-01, CA-RF-027-02, CA-RF-027-03, CA-RF-027-04 |
| RF-028 | Web, Recommendations | [ADR-012](adr/ADR-012.md) | Preferencias y recomendaciones vía APIs | II | CA-RF-028-01, CA-RF-028-02 |
| RF-029 | Web, servicios de III | [ADR-012](adr/ADR-012.md) | Búsqueda, devolución y posición en cola | III | CA-RF-029-01, CA-RF-029-02, CA-RF-029-03 |
| RNF-001 | Gateway y servicios Core | [ADR-001](adr/ADR-001.md), [ADR-003](adr/ADR-003.md) | Backends Express y fachada REST funcional | I | CA-RNF-001-01 |
| RNF-002 | Ticketing | [ADR-007](adr/ADR-007.md) | Transacción local y justificación comparada de mecanismos | I | CA-RNF-002-01, CA-RNF-002-02 |
| RNF-003 | Ticketing y consumidores | [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md) | Claves de idempotencia y deduplicación persistente | I | CA-RNF-003-01, CA-RNF-003-02 |
| RNF-004 | Todos los servicios | [ADR-001](adr/ADR-001.md), [ADR-004](adr/ADR-004.md) | Capacidades cohesivas y propiedad exclusiva | I | CA-RNF-004-01 |
| RNF-005 | Servicios productores y consumidores | [ADR-005](adr/ADR-005.md) | HTTP inmediato y JetStream persistente | I–III | CA-RNF-005-01 |
| RNF-006 | Catalog, Ticketing y consumidores | [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | Recuperación por comandos estables, outbox y replay | I–III | CA-RNF-006-01, CA-RNF-006-02, CA-RNF-006-03, CA-RNF-006-04 |
| RNF-007 | Recommendations y consumidores | [ADR-004](adr/ADR-004.md), [ADR-009](adr/ADR-009.md) | Proyecciones convergentes con guardas fuertes locales | II–III | CA-RNF-007-01, CA-RNF-007-02 |
| RNF-008 | Todos los servicios | [ADR-001](adr/ADR-001.md), [ADR-002](adr/ADR-002.md) | Artefactos desplegables por servicio y compatibilidad | I–III | CA-RNF-008-01 |
| RNF-009 | Gateway y servicios HTTP | [ADR-010](adr/ADR-010.md) | Pino, reloj monotónico, fases HTTP y SSE | I, III | CA-RNF-009-01, CA-RNF-009-02 |
| RNF-010 | Recommendations y dueños de caché | [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | Redis Cache-Aside, aislamiento, límites e invalidación | II | CA-RNF-010-01, CA-RNF-010-02, CA-RNF-010-03, CA-RNF-010-04 |
| RNF-011 | Waiting Room, Ticketing | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | Capacidad acotada, backpressure y procesamiento asíncrono ordenado | III | CA-RNF-011-01 |
| RNF-012 | Equipo, Gateway, Waiting Room, Ticketing | [ADR-011](adr/ADR-011.md) | k6 y verificación de integridad bajo carga | III | CA-RNF-012-01 |
| RNF-013 | Identity, Gateway y servicios | [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md) | bcrypt, política de redacción y pruebas de logs | I–III | CA-RNF-013-01 |
| RNF-014 | Todos los servicios persistentes | [ADR-004](adr/ADR-004.md) | Bases y credenciales independientes | I–III | CA-RNF-014-01 |
| RNF-015 | Infraestructura y servicios | [ADR-001](adr/ADR-001.md), [ADR-002](adr/ADR-002.md), [ADR-011](adr/ADR-011.md) | Compose y dependencias reproducibles | I–III | CA-RNF-015-01 |
| RNF-016 | Gateway y servicios HTTP | [ADR-003](adr/ADR-003.md), [ADR-011](adr/ADR-011.md) | OpenAPI, Zod, ESLint y Prettier | I–III | CA-RNF-016-01 |
| RNF-017 | Equipo y servicios | [ADR-011](adr/ADR-011.md) | Unitarias, HTTP, contratos e infraestructura real | I–III | CA-RNF-017-01 |
| RNF-018 | Ticketing | [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md) | Puerta e inventario se validan en PostgreSQL | II | CA-RNF-018-01, CA-RNF-018-02 |
| RNF-019 | Equipo | [ADR-011](adr/ADR-011.md) | Plan de estrés con entorno, métricas y umbrales acordados | III | CA-RNF-019-01 |
| RNF-020 | Recommendations, Catalog, Ticketing | [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | Contratos de catálogo, preferencias e historial | II | CA-RNF-020-01 |
| RNF-021 | Catalog | [ADR-003](adr/ADR-003.md), [ADR-004](adr/ADR-004.md) | Filtrado, relevancia y desempate en PostgreSQL | III | CA-RNF-021-01 |
| RNF-022 | Productores con outbox y consumidores | [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md) | Publicación recuperable y deduplicación transaccional | I–III | CA-RNF-022-01, CA-RNF-022-02 |
| RNF-023 | Web, Gateway | [ADR-012](adr/ADR-012.md) | Vue con JavaScript e integración incremental | I–III | CA-RNF-023-01 |
| RN-001 | Ticketing | [ADR-007](adr/ADR-007.md) | Asignación exclusiva de asiento | I | CA-RF-008-01, CA-RF-012-01, CA-RF-012-02, CA-RNF-002-01 |
| RN-002 | Ticketing | [ADR-007](adr/ADR-007.md) | Compromiso total dentro del cupo | I | CA-RF-013-01, CA-RNF-002-01 |
| RN-003 | Ticketing | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md) | Un único efecto y compra única por reserva | I | CA-RF-012-02, CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06, CA-RNF-003-01 |
| RN-004 | Ticketing | [ADR-007](adr/ADR-007.md) | Transición y liberación únicas | I | CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04 |
| RN-005 | Ticketing | [ADR-007](adr/ADR-007.md), [ADR-009](adr/ADR-009.md) | Reposición única con reintegro, invalidación e historial | III | CA-RF-017-01, CA-RF-017-02 |
| RN-006 | Catalog, Ticketing | [ADR-004](adr/ADR-004.md), [ADR-006](adr/ADR-006.md) | Propiedad única y autorización sin delegación | I | CA-RF-003-02, CA-RF-005-01, CA-RF-027-02 |
| RN-007 | Identity y servicios de recurso | [ADR-006](adr/ADR-006.md) | Autorización de rol y recurso en backend | I, III | CA-RF-003-01, CA-RF-003-02, CA-RF-003-03, CA-RF-004-02, CA-RF-015-02, CA-RF-016-02 |
| RN-008 | Waiting Room, Ticketing | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | FIFO de turnos y contrato separado de procesamiento | III | CA-RF-021-01, CA-RF-021-03, CA-RNF-011-01 |
| RN-009 | Ticketing | [ADR-007](adr/ADR-007.md) | Conversión de reservado a vendido sin doble conteo | I | CA-RF-008-01, CA-RF-010-01, CA-RF-013-01 |
| RN-010 | Ticketing | [ADR-007](adr/ADR-007.md) | Rechazo conserva ticket, compromiso e historial | III | CA-RF-016-01 |
| RN-011 | Catalog, Ticketing | [ADR-007](adr/ADR-007.md) | Cierre efectivo primero y revisión de cambios | I | CA-RF-005-02, CA-RF-006-03 |
| RN-012 | Ticketing, Catalog | [ADR-004](adr/ADR-004.md), [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md) | Solicitud por titular, ticket individual y plazo | III | CA-RF-015-01, CA-RF-015-02, CA-RF-015-03 |
| RN-013 | Waiting Room, Ticketing | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md) | Unicidad de entrada, capacidad y vigencia | III | CA-RF-020-01, CA-RF-020-02, CA-RF-020-04, CA-RF-021-02 |
| RM-001 | Equipo | [ADR-001](adr/ADR-001.md), [ADR-011](adr/ADR-011.md) | Especificaciones de OpenSpec y verificación antes de archivar | I–III | CA-RM-001-01 |
| RM-002 | Equipo | [ADR-001](adr/ADR-001.md), [ADR-011](adr/ADR-011.md) | Tres incrementos y validación del alcance académico | I–III | CA-RM-002-01 |
| RM-003 | Equipo | [ADR-001](adr/ADR-001.md), [ADR-011](adr/ADR-011.md) | Un change por etapa con el flujo establecido | I–III | CA-RM-003-01, CA-RM-003-02 |

## Detalles de referencia

- [Límites de servicios](service-boundaries.md)
- [Propiedad de datos](data-ownership.md)
- [Comunicación](communication.md)
- [Consistencia e integridad](consistency.md)
- [Diagramas](diagrams.md)
- [Índice de decisiones](adr/README.md)
