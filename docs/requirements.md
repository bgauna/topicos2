# P-001.1 — Especificación consolidada de requisitos de la ticketera virtual

## 1. Información del proyecto

| Campo | Detalle |
| --- | --- |
| Proyecto | Ticketera virtual para el Trabajo Final de Tópicos II — 2026 |
| Responsables del proyecto y del documento | Briant + Fernando |
| Fase | Fase 0 · Preparación de requisitos para Spec-Driven Development |
| Fecha | 25 de noviembre de 2026 |
| Estado | Versión consolidada pendiente de revisión final de Briant + Fernando |

## 2. Propósito del documento

Establecer la referencia para las futuras especificaciones de OpenSpec. La matriz vincula cada requisito con su origen, etapa y criterios de aceptación. Las referencias a páginas corresponden a las tres páginas físicas del PDF. Briant + Fernando aprobaron las resoluciones de la sección 13 en P-001.1. Los detalles diferidos para el diseño se recogen en la sección 17.

## 3. Alcance del sistema

La plataforma cubre la gestión administrativa, el diseño y la publicación de eventos, y la adquisición de tickets numerados y por zona. El Core incorpora registro, autenticación, reservas, compra con simulación de pagos, controles de inventario y bitácora de solicitudes HTTP, mockups y vistas funcionales conectadas a las APIs. Los incrementos siguientes incorporan recomendaciones personalizadas y caché, y luego venta masiva con colas y notificaciones, devoluciones, búsqueda avanzada y pruebas de estrés.

Esta fase concluye con la especificación de requisitos para revisión. El diseño técnico y la implementación se gestionarán mediante los changes de las tres etapas.

## 4. Actores y responsabilidades

| Actor | Responsabilidades y permisos | Alcance |
| --- | --- | --- |
| Administrador | Gestión, mantenimiento y tareas operativas (CONSIGNA, p. 1). Gestiona usuarios, roles y eventos, promueve Clientes a Event Owner y consulta ventas e información operativa (DF-004, AMB-001 y AMB-014). Resuelve las devoluciones en III (CONSIGNA, p. 1 y AMB-007). | Interviene según sus permisos. Los cambios directos de ventas confirmadas que comprometan inventario o borren historial se rechazan. Las devoluciones usan su flujo autorizado. |
| Event Owner | Diseña y publica eventos (CONSIGNA, p. 1). Gestiona los eventos de su propiedad y su configuración de zonas y asientos (AMB-001 y AMB-015). | Cada evento tiene un único Event Owner propietario. La primera versión excluye la delegación entre Event Owners. |
| Cliente | Se registra públicamente, inicia sesión, consulta eventos, reserva, compra y consulta sus tickets (AMB-001, AMB-015 y CONSIGNA, p. 1 para la compra). Selecciona categorías y consulta recomendaciones en II. Usa búsqueda, cola y devoluciones en III (CONSIGNA, pp. 1–2 y AMB-006 a AMB-009). | Accede únicamente a sus propios tickets y solicita devoluciones de estos dentro del plazo. El registro público crea cuentas de Cliente. La primera versión excluye recuperación de contraseña. |

La matriz de permisos se aplica en el backend. Una cuenta administrativa inicial se genera mediante un seed (RF-024).

## 5. Requisitos funcionales

### RF-001 · Identidad y autenticación

**CONSIGNA**. Identificar a cada usuario que se autentica y habilitar el acceso a las operaciones que le corresponden. El registro público y el administrador inicial se especifican en RF-024 (AMB-001).

### RF-002 · Roles y responsabilidades

**CONSIGNA**. Distinguir Administrador, Event Owner y Cliente. El Administrador tiene a cargo la gestión completa, el mantenimiento y las tareas operativas. El Event Owner diseña y publica eventos. El Cliente compra tickets para eventos listados. RF-026 detalla las facultades administrativas aprobadas.

### RF-003 · Autorización por operación

**DERIVADO**. Verificar en el backend el rol y el acceso al recurso para cada operación restringida, incluso ante solicitudes directas a las APIs (RN-006, RN-007, AMB-001, AMB-007 y AMB-014). Cada Event Owner solo gestiona eventos propios. Cada Cliente solo consulta sus propios tickets y, desde la Etapa III, solo puede solicitar su devolución.

### RF-004 · Asignación de Event Owner

**DECISIÓN APROBADA**. Permitir que un Administrador promueva una cuenta de Cliente al rol Event Owner (DF-004, AMB-001). La asignación de roles es una capacidad administrativa.

### RF-005 · Diseño y publicación de eventos

**CONSIGNA**. Permitir a cada Event Owner diseñar y publicar sus propios eventos. Cada evento tiene un único Event Owner propietario. Usar como mínimo los estados `DRAFT` (en preparación), `PUBLISHED` (publicado), `CANCELLED` (cancelado) y `FINISHED` (finalizado). Solo un evento publicado y habilitado para la venta admite nuevas compras (AMB-001, AMB-002 y RN-011). La política para cancelar eventos con ventas confirmadas debe definirse antes de habilitar ese comportamiento. Un cambio de estado por sí solo no elimina compras ni libera inventario vendido.

### RF-006 · Configuración de zonas y asientos

**DERIVADO**. Permitir la configuración autorizada de zonas, modalidades, capacidades y asientos identificables. Bloquear todo cambio de zona, asiento o capacidad que invalide reservas vigentes o ventas activas (AMB-002 y RN-011).

### RF-007 · Modalidades de tickets

**CONSIGNA**. Vender entradas numeradas asociadas a un asiento específico y entradas no numeradas asociadas a una zona, con reglas de inventario diferenciadas.

### RF-008 · Creación de reservas

**DERIVADO**. Apartar temporalmente inventario disponible mediante una reserva exclusiva sobre los asientos y unidades de cupo afectados. Las reservas duran inicialmente 10 minutos y su duración es configurable. Las unidades reservadas quedan comprometidas durante la vigencia (DF-002, AMB-003, RN-001, RN-002 y RN-009).

### RF-009 · Vencimiento de reservas

**DERIVADO**. Vencer las reservas al terminar su vigencia y liberar el inventario una única vez. Resolver el vencimiento y la confirmación concurrentes mediante transiciones transaccionales mutuamente excluyentes. Una confirmación tardía no puede consumir inventario liberado (AMB-003, RN-004 y RNF-002).

### RF-010 · Compra de tickets

**CONSIGNA**. Permitir al Cliente comprar tickets para eventos listados, publicados y habilitados para la venta. Una compra exige pago aprobado y reserva aún vigente al ejecutar la transición transaccional. La transición convierte las mismas unidades reservadas en venta con un solo efecto de inventario. Una reserva admite a lo sumo una compra confirmada, incluso con claves de idempotencia diferentes (RF-008, RF-011, RN-003 y RN-009).

### RF-011 · Simulación de pagos

**DECISIÓN APROBADA**. Simular pagos aprobados y rechazados (DF-001). Un rechazo conserva la reserva hasta su vencimiento sin emitir tickets ni crear una compra confirmada. El Cliente puede reintentar el pago mientras la reserva siga vigente. El contrato de reintentos se concretará en el diseño de la Etapa I (AMB-003 y AMB-004).

### RF-012 · Unicidad de venta

**CONSIGNA**. Impedir ventas activas simultáneas del mismo ticket o asiento. La exclusividad comienza en la reserva y se conserva en la confirmación, incluso si solicitudes distintas intentan confirmar la misma reserva (RN-001 y RN-003).

### RF-013 · Capacidad de zona

**DERIVADO**. Aplicar el cupo de cada zona no numerada al reservar y vender. La suma de unidades en reservas vigentes y ventas activas nunca supera el cupo. Convertir reservas en ventas conserva el total comprometido (RN-002 y RN-009).

### RF-014 · Idempotencia de compra

**CONSIGNA**. Aplicar idempotencia a las operaciones de compra. La misma identidad, operación, clave y contenido producen un solo efecto aunque las solicitudes sean concurrentes, la respuesta se pierda o se reintente tras completarse. Si se reutiliza la misma clave con otro contenido dentro de su retención, retornar HTTP 409 Conflict. Después de expirar la clave, la unicidad persistente sigue evitando la duplicación de la compra y del inventario (AMB-004, RN-003 y RNF-003).

### RF-015 · Solicitud de devolución

**CONSIGNA**. Permitir al Cliente solicitar la devolución por ticket individual propio hasta 24 horas antes del inicio del evento, inclusive en el límite. Registrar la solicitud para resolución administrativa (AMB-007 y RN-012).

### RF-016 · Resolución administrativa de devoluciones

**CONSIGNA**. Permitir al Administrador aprobar o rechazar la solicitud. La aprobación confirma la devolución y simula el reintegro monetario dentro del flujo de RF-017. El rechazo mantiene activos el ticket y su compromiso de inventario (AMB-007 y RN-010).

### RF-017 · Reposición de inventario por devolución

**DECISIÓN APROBADA**. Confirmar la devolución, registrar el reintegro simulado, invalidar el ticket y reponer su asiento o unidad de zona en una operación de negocio atómica e idempotente. Liberar la unidad exactamente una vez y conservar el historial de compra y devolución (DF-005, AMB-007 y RN-005).

### RF-018 · Recomendaciones personalizadas

**CONSIGNA**. Generar recomendaciones de eventos con categorías seleccionadas por el Cliente, historial de compras y reglas de puntuación explicables. Si faltan tanto preferencias como historial, usar eventos populares o próximos. Las reglas no requieren modelos generativos ni servicios externos de IA (DF-006 y AMB-006).

### RF-019 · Búsqueda avanzada

**CONSIGNA**. Buscar por tipo o categoría, organizador, participante y fecha. Combinar criterios distintos con AND, aplicar intervalos de fechas y comparar texto sin distinguir mayúsculas, minúsculas ni acentos. Ordenar por relevancia ponderada y desempatar de forma determinista por fecha e identificador (DF-008, AMB-008 y RNF-021).

### RF-020 · Ingreso a cola de venta masiva

**CONSIGNA**. Activar manualmente el modo de alta demanda por evento mediante una operación autorizada. Mantener una cola independiente de capacidad configurable por evento y como máximo una entrada activa por Cliente y evento. Ante saturación, responder HTTP 429 Too Many Requests a nuevas solicitudes de ingreso (DF-007, AMB-009 y RN-013).

### RF-021 · Procesamiento ordenado de ventas masivas

**CONSIGNA**. Asignar turnos con FIFO estricto según el orden de ingreso admitido y aplicar vencimiento configurable. Procesar las solicitudes en orden conforme a la consigna. El diseño deberá definir y justificar qué hitos y garantías observables constituyen procesamiento ordenado con múltiples consumidores. Distinguir ingreso y admisión a la cola, asignación de turno, procesamiento y finalización de la compra (AMB-009, RN-008 y RNF-011).

### RF-022 · Notificación de posición en cola

**CONSIGNA**. Notificar por SSE la posición inicial y sus cambios durante la espera. Tras reconexión, entregar la posición actual de la entrada del Cliente en su cola (AMB-009, AMB-011 y RNF-009).

### RF-023 · Autenticación con JWT

**DECISIÓN APROBADA**. Autenticar las solicitudes protegidas con JWT. Rechazar tokens inválidos, vencidos o manipulados (DF-003 y RNF-013).

### RF-024 · Registro e inicialización de cuentas

**DECISIÓN APROBADA**. Permitir el registro público únicamente como Cliente y disponer de un Administrador inicial generado por un seed. La recuperación de contraseña y la delegación entre Event Owners quedan fuera de la primera versión (AMB-001).

### RF-025 · Datos mínimos del evento

**DECISIÓN APROBADA**. Contemplar título, descripción, categoría, organizador, participantes, ubicación, fecha, zonas, asientos cuando correspondan, precios y estado desde el Core. La representación concreta pertenece al diseño (AMB-013).

### RF-026 · Capacidades administrativas

**DECISIÓN APROBADA**. Permitir al Administrador gestionar usuarios, roles y eventos, consultar ventas e información operativa y, desde III, resolver devoluciones. Incorporar las capacidades conforme estén disponibles en cada incremento. Prohibir cambios directos de ventas confirmadas que comprometan el inventario o eliminen su historial. Las devoluciones usan RF-015 a RF-017 (AMB-014).

### RF-027 · Vistas funcionales del Core

**DECISIÓN APROBADA**. Entregar vistas conectadas a las APIs para registro e inicio de sesión del Cliente, listado y detalle de eventos, selección de asiento o zona, reserva, compra y consulta de tickets propios. Para Event Owner, incluir inicio de sesión, gestión de eventos propios, zonas, asientos y publicación. Para Administrador, incluir inicio de sesión, usuarios, roles, eventos, ventas e información operativa. Representar éxito, errores y falta de inventario (DF-010 y AMB-015).

### RF-028 · Vistas funcionales de recomendaciones

**DECISIÓN APROBADA**. Permitir al Cliente seleccionar sus categorías y consultar recomendaciones en vistas conectadas a las APIs correspondientes. Representar éxito, arranque en frío y errores (RF-018, DF-010, AMB-006 y AMB-015).

### RF-029 · Vistas funcionales de escalabilidad

**DECISIÓN APROBADA**. Integrar para Cliente la búsqueda avanzada, devolución por ticket y posición en cola con las APIs y SSE. Incorporar para Administrador la resolución de devoluciones. Representar resultados exitosos, errores y falta de inventario cuando corresponda (DF-010 y AMB-015).

## 6. Requisitos no funcionales y técnicos

### RNF-001 · Backend y APIs del Core

**CONSIGNA**. Entregar un backend con APIs funcionales para las capacidades de la etapa I.

### RNF-002 · Integridad de inventario y concurrencia

**CONSIGNA**. Proteger las invariantes críticas con transacciones locales en el servicio propietario, con PostgreSQL como fuente de verdad transaccional (AMB-005). Justificar por operación la aplicabilidad, selección o descarte de los mecanismos académicos: locks optimistas, locks pesimistas, transacciones distribuidas, claves de idempotencia y mecanismos de deduplicación. Las claves y la deduplicación controlan repeticiones y las transacciones protegen invariantes ante operaciones distintas. Las transacciones locales, el Transactional Outbox y la mensajería asíncrona cumplen funciones propias y no constituyen por sí mismos una transacción distribuida.

### RNF-003 · Claves de idempotencia y deduplicación

**CONSIGNA**. Aplicar claves de idempotencia y deduplicación. Asociar cada clave con usuario y operación, y comparar un hash del contenido para detectar reutilización incompatible. Retener las claves inicialmente 24 horas. Mantener restricciones persistentes de unicidad de compra independientes de la retención de claves (AMB-004, RF-014 y RN-003).

### RNF-004 · Límites de microservicios

**CONSIGNA**. Definir límites y responsabilidades de servicios, con un propietario para cada dato. El diseño de la Etapa I concreta los límites y la propiedad de datos del Core (AMB-005 y DF-011).

### RNF-005 · Comunicación entre servicios

**CONSIGNA**. Definir interacciones síncronas y asíncronas, sus contratos y sus efectos. Usar NATS con JetStream en la comunicación asíncrona que requiera persistencia de mensajes (AMB-005 y RNF-022).

### RNF-006 · Manejo de fallos parciales

**CONSIGNA**. Preservar las compras confirmadas y las invariantes de inventario ante fallos parciales. Recuperar publicaciones pendientes asociadas a operaciones confirmadas y contemplar recuperación, reintentos y compensación cuando sean necesarios (AMB-005 y RNF-022).

### RNF-007 · Consistencia eventual

**CONSIGNA**. Admitir consistencia eventual para datos derivados y operaciones que preserven las invariantes críticas. Definir la convergencia después de interrupciones y la deduplicación en consumidores. Aplicar las invariantes de reserva, compra, vencimiento y devolución en cada transición propietaria (AMB-005 y RNF-002).

### RNF-008 · Despliegue independiente

**CONSIGNA**. Permitir desplegar cada servicio de forma independiente manteniendo contratos compatibles.

### RNF-009 · Registro HTTP con tiempos

**CONSIGNA**. Medir cada solicitud HTTP desde su recepción, incluida la autenticación, la autorización y el procesamiento. Para HTTP convencional, registrar con Pino el resultado y la duración hasta la preparación de la respuesta, antes de retornarla, tanto en éxitos como en fallos. Para SSE, registrar la apertura y la latencia de aceptación, y por separado el cierre y la duración de la conexión. Excluir contraseñas, tokens y demás credenciales sensibles (AMB-011 y RNF-013).

### RNF-010 · Políticas de caché

**CONSIGNA**. Definir qué datos cachear, dónde, con qué TTL y cómo invalidarlos. Evaluar y justificar caché de cliente, CDN, servicio y caché distribuida. Usar Redis como caché distribuida con Cache-Aside, TTL configurable por tipo de dato e invalidación mediante eventos de NATS cuando corresponda. Aislar los datos personalizados por identidad. Ante fallo de Redis, se podrá consultar la fuente autorizada con límites que protejan al propietario, según la política de degradación aprobada durante el diseño (AMB-012).

### RNF-011 · Infraestructura de colas y backpressure

**CONSIGNA**. Procesar ventas masivas de forma asíncrona con backpressure y notificaciones en tiempo real. Coordinar los consumidores para mantener FIFO estricto en la asignación de turnos y cumplir el contrato de procesamiento ordenado. La coordinación de negocio debe verificarse con múltiples consumidores, independientemente de las garantías de entrega de NATS (AMB-009, RF-021 y RF-022).

### RNF-012 · Pruebas de estrés

**CONSIGNA**. Ejecutar pruebas de estrés reproducibles con k6 bajo carga de alta demanda. Registrar throughput, latencias p95 y p99, tasa de errores, longitud de cola, tiempo de espera e integridad del inventario. Comparar con el plan previo de RNF-019 y verificar la integridad y la recuperación ante fallos al cerrar la Etapa III (AMB-010).

### RNF-013 · Protección de credenciales

**DERIVADO**. Guardar contraseñas mediante bcrypt. Excluir contraseñas, tokens y otras credenciales sensibles de los registros operativos, incluso en errores y solicitudes de autenticación o conexiones SSE (AMB-011, RF-023 y RNF-009).

### RNF-014 · Aislamiento de bases de datos

**DECISIÓN APROBADA**. Mantener una base de datos independiente para cada servicio utilizando inicialmente una misma instancia de PostgreSQL (DF-011).

### RNF-015 · Entorno reproducible

**DECISIÓN APROBADA**. Usar Docker Compose para el despliegue local reproducible (DF-012).

### RNF-016 · Mantenibilidad y contratos de API

**DERIVADO**. Mantener los contratos HTTP en OpenAPI, consultables mediante Swagger UI, y comprobar los cambios de JavaScript con ESLint y Prettier según el stack aprobado.

### RNF-017 · Testabilidad de reglas y APIs

**DERIVADO**. Permitir verificar las reglas de negocio mediante pruebas de unidad y los recorridos HTTP mediante pruebas automatizadas con datos controlados. Aplicar las herramientas aprobadas según el nivel de prueba.

### RNF-018 · Caché fuera de la confirmación de inventario

**DERIVADO**. Consultar el inventario transaccional en PostgreSQL para decidir tanto la reserva como la confirmación, aunque Redis muestre disponibilidad obsoleta o no esté disponible (AMB-005 y AMB-012).

### RNF-019 · Objetivos medibles de rendimiento

**DECISIÓN APROBADA**. Antes de las pruebas de estrés, fijar infraestructura, perfil de carga, usuarios concurrentes, duración, tasa de solicitudes, umbrales y criterios de éxito y fracaso (DF-009 y AMB-010). Los valores se concretarán en la Etapa III.

### RNF-020 · Integración con recomendaciones

**CONSIGNA**. Integrar el sistema de recomendación con catálogo de eventos, categorías seleccionadas por el Cliente e historial de compras, aplicando las reglas y el arranque en frío definidos en RF-018 (AMB-006).

### RNF-021 · Búsqueda y relevancia en PostgreSQL

**DECISIÓN APROBADA**. Ejecutar en PostgreSQL el filtrado y ordenamiento de la búsqueda avanzada bajo las reglas de RF-019 (DF-008 y AMB-008).

### RNF-022 · Publicación recuperable y consumo idempotente

**DECISIÓN APROBADA**. Usar Transactional Outbox para persistir la publicación pendiente junto con la operación confirmada en la transacción local. Recuperar y publicar los eventos pendientes tras fallos de NATS, usando JetStream cuando se requiera persistencia. Los consumidores deben tolerar entregas repetidas sin duplicar efectos (AMB-005).

### RNF-023 · Integración web con APIs

**DECISIÓN APROBADA**. Desarrollar mockups y vistas con Vue 3, Vite y JavaScript. Integrar progresivamente las vistas funcionales con APIs reales del incremento y con SSE en III, bajo los permisos validados en el backend. Los mockups se verifican de forma independiente de las pruebas de integración (DF-010 y AMB-015).

## 7. Reglas de negocio

| ID | Regla | Origen | Enunciado |
| --- | --- | --- | --- |
| RN-001 | Unicidad de reserva y venta por asiento | CONSIGNA | Un asiento admite como máximo una venta activa. La reserva exclusiva impide dos reservas vigentes incompatibles o reservar un asiento con venta activa (DERIVADO). |
| RN-002 | Capacidad máxima de zona | DERIVADO | La suma de unidades en reservas vigentes y ventas activas permanece dentro de la capacidad de la zona. |
| RN-003 | Un efecto por solicitud de compra | CONSIGNA | Una solicitud repetida genera un solo efecto. La unicidad persistente por reserva impide una segunda compra confirmada ante claves distintas o vencidas (AMB-004). |
| RN-004 | Liberación por vencimiento | DERIVADO | Una reserva vencida libera inventario una sola vez. Si la confirmación válida gana la transición, el vencimiento no repone unidades vendidas. Si el vencimiento gana, la compra tardía no consume inventario. |
| RN-005 | Liberación única tras devolución | DERIVADO | La devolución confirmada invalida el ticket y repone exactamente una vez el asiento o cupo de forma atómica e idempotente, con historial preservado (DF-005 y AMB-007). |
| RN-006 | Propiedad de eventos de Event Owner | DECISIÓN APROBADA | Cada evento tiene un único Event Owner propietario. El Event Owner solo gestiona sus eventos, sin delegación entre Event Owners en la primera versión. El Administrador puede intervenir conforme a sus permisos (AMB-001). |
| RN-007 | Acceso por rol y recurso | DERIVADO | El backend exige rol habilitado y acceso al recurso. Un Cliente no puede otorgarse privilegios ni consultar o devolver tickets ajenos (AMB-001, AMB-007 y AMB-014). |
| RN-008 | Orden de venta masiva | CONSIGNA | Se exige procesamiento ordenado. FIFO estricto rige la asignación de turnos sin adelantamientos (AMB-009). El diseño distingue el orden de ingreso admitido, asignación, procesamiento y finalización de compras, y justifica el contrato de procesamiento. |
| RN-009 | Contabilidad de reservas y ventas | DERIVADO | Cada unidad comprometida cuenta una sola vez. Confirmar una reserva vigente convierte las mismas unidades en venta sin aumentar el total comprometido. |
| RN-010 | Rechazo de devolución | DERIVADO | Una solicitud rechazada deja activo el ticket y conserva su inventario comprometido, sin reintegro simulado (AMB-007). |
| RN-011 | Venta y cambios de inventario por estado | DECISIÓN APROBADA | Solo se admiten compras para eventos publicados y habilitados para vender. Se bloquean cambios de inventario que invaliden compromisos vigentes. Cancelar con ventas confirmadas requiere una política previa (AMB-002). |
| RN-012 | Elegibilidad de devolución | DECISIÓN APROBADA | Cada solicitud corresponde a un ticket propio y puede ingresarse hasta 24 horas antes del inicio del evento, inclusive (AMB-007). |
| RN-013 | Entrada única y vigencia de turno | DECISIÓN APROBADA | Cada Cliente tiene a lo sumo una entrada activa por evento. La cola respeta su capacidad y los turnos dejan de habilitar acceso al vencer (AMB-009). |

## 8. Criterios de aceptación

Cada criterio se comprueba en la etapa indicada en la matriz. Los escenarios que dependen de definiciones diferidas usan los valores y contratos que se concreten durante el diseño de esa etapa. Para los requisitos transversales con verificación incremental se aplican estas comprobaciones:

| Criterios | Aplicación por etapa |
| --- | --- |
| CA-RF-026-01, CA-RF-026-02 y CA-RF-026-03 | Las gestiones del Core y la protección de ventas se verifican desde I. La resolución administrativa de devoluciones se verifica en III. |
| CA-RNF-006-01, CA-RNF-006-02, CA-RNF-006-03 y CA-RNF-022-01 | En I se comprueba la integridad de la compra ante pérdida de respuesta o caída de NATS, y el registro atómico y recuperación de la publicación pendiente. |
| CA-RNF-007-01, CA-RNF-007-02 y CA-RNF-022-02 | En II se comprueban desconexiones, reentregas y convergencia de datos derivados con el consumidor de historial u otra proyección de la etapa. La integridad del Core se vuelve a verificar bajo estos fallos. |
| CA-RNF-006-04 y CA-RNF-012-01 | En III se verifica la recuperación con consumidores de ventas masivas y se vuelve a comprobar la integridad bajo carga. |
| CA-RNF-009-01, CA-RNF-009-02 y CA-RNF-013-01 | Se verifica HTTP convencional desde I y se amplía a apertura, cierre y protección de credenciales en SSE en III. |

### RF-001

- **CA-RF-001-01**. **DADO** una identidad habilitada con credenciales válidas, **CUANDO** inicia sesión e invoca una operación autorizada, **ENTONCES** la operación se ejecuta bajo su identidad.
- **CA-RF-001-02**. **DADO** credenciales inválidas o ausencia de autenticación, **CUANDO** se solicita acceso protegido, **ENTONCES** se deniega el acceso.

### RF-002

- **CA-RF-002-01**. **DADO** usuarios de los tres roles y un caso autorizado por rol, **CUANDO** el Administrador realiza una operación de gestión, el Event Owner publica un evento autorizado y el Cliente compra para un evento listado, **ENTONCES** cada operación autorizada se completa bajo el rol correspondiente.

### RF-003

- **CA-RF-003-01**. **DADO** una cuenta de Cliente, **CUANDO** intenta publicar un evento por API, **ENTONCES** se deniega la operación y el evento no cambia.
- **CA-RF-003-02**. **DADO** un Event Owner propietario de A y ajeno a B, **CUANDO** envía directamente a la API solicitudes de modificación y publicación de B, **ENTONCES** se rechazan y B conserva sus datos y estado.
- **CA-RF-003-03**. **DADO** un Cliente autenticado y un ticket de otro Cliente, **CUANDO** consulta ese ticket directamente por API, **ENTONCES** se deniega el acceso y la respuesta no contiene datos del ticket.

### RF-004

- **CA-RF-004-01**. **DADO** un Administrador y una cuenta de Cliente, **CUANDO** el Administrador le asigna el rol Event Owner, **ENTONCES** la asignación queda registrada para esa cuenta.
- **CA-RF-004-02**. **DADO** un Cliente autenticado, **CUANDO** intenta asignarse los roles Administrador y Event Owner mediante solicitudes directas a la API, **ENTONCES** se deniegan ambos cambios y conserva su rol.

### RF-005

- **CA-RF-005-01**. **DADO** un evento DRAFT con datos e inventario válidos y un único Event Owner propietario, **CUANDO** el propietario publica y habilita la venta, **ENTONCES** el evento queda PUBLISHED y aparece como comprable para el Cliente.
- **CA-RF-005-02**. **DADO** casos de eventos DRAFT, CANCELLED, FINISHED y PUBLISHED sin venta habilitada, **CUANDO** se intenta comprar por API en cada caso, **ENTONCES** se rechazan las compras y el inventario no cambia.

### RF-006

- **CA-RF-006-01**. **DADO** un usuario autorizado y un evento en preparación, **CUANDO** configura una zona numerada con asientos identificados y otra zona con cupo, **ENTONCES** cada zona queda disponible para su modalidad de venta.
- **CA-RF-006-02**. **DADO** una zona numerada con un identificador de asiento ya usado, **CUANDO** se intenta agregar otro asiento con ese identificador a la misma zona, **ENTONCES** la configuración duplicada se rechaza.
- **CA-RF-006-03**. **DADO** una reserva vigente y una venta activa de prueba, en asientos o cupos respectivos, **CUANDO** se intenta modificar una zona o asiento comprometido o reducir el cupo por debajo de lo comprometido, **ENTONCES** se rechazan los cambios y permanecen intactos la configuración, las reservas y las ventas.

### RF-007

- **CA-RF-007-01**. **DADO** una zona numerada y otra no numerada con disponibilidad, **CUANDO** dos clientes compran, uno por asiento y otro por zona, **ENTONCES** el primer ticket identifica su asiento y el segundo consume una unidad de su zona.

### RF-008

- **CA-RF-008-01**. **DADO** un asiento y una unidad de zona disponibles, **CUANDO** un Cliente obtiene una reserva vigente y después otro intenta reservar los mismos recursos, **ENTONCES** la primera reserva conserva las unidades y el segundo intento sobre ellas se rechaza.

### RF-009

- **CA-RF-009-01**. **DADO** una reserva con inventario apartado que llega a su vencimiento, **CUANDO** se procesa la expiración varias veces, también concurrentemente, **ENTONCES** la reserva queda vencida y las unidades vuelven a estar disponibles exactamente una vez.
- **CA-RF-009-02**. **DADO** una reserva vencida y un pago que llega tarde, **CUANDO** se intenta confirmar la compra con esa reserva, **ENTONCES** no se emite el ticket ni se consume la unidad liberada.
- **CA-RF-009-03**. **DADO** una configuración inicial de 10 minutos y otra con duración alternativa para nuevas reservas, **CUANDO** se crea una reserva con cada configuración, **ENTONCES** la primera vence 10 minutos después de crearse y la segunda vence al cumplirse su duración configurada.
- **CA-RF-009-04**. **DADO** una reserva exclusiva próxima a vencer, pago aprobado y un reloj controlado, **CUANDO** se fuerza la carrera entre confirmación y vencimiento, probando ambos órdenes de transición, **ENTONCES** si la confirmación se concreta con vigencia se emite una compra sin reposición al vencer, y si se concreta el vencimiento se libera una sola vez y la confirmación tardía no emite compra ni tickets.

### RF-010

- **CA-RF-010-01**. **DADO** un Cliente con reserva vigente para un evento listado, PUBLISHED y habilitado para vender, **CUANDO** se aprueba el pago y se ejecuta la confirmación transaccional durante la vigencia, **ENTONCES** se registra una compra, se emiten sus tickets y las mismas unidades pasan de reservadas a vendidas sin aumentar el compromiso total.

### RF-011

- **CA-RF-011-01**. **DADO** una reserva vigente y el simulador configurado para rechazar, **CUANDO** se intenta el pago, **ENTONCES** la reserva sigue vigente hasta su vencimiento y no se crean compras confirmadas ni tickets.
- **CA-RF-011-02**. **DADO** una reserva que sigue vigente después de un rechazo, **CUANDO** el Cliente reintenta el pago conforme al contrato de la Etapa I y el simulador lo aprueba, **ENTONCES** puede confirmar una única compra mientras la reserva sigue vigente.

### RF-012

- **CA-RF-012-01**. **DADO** el último asiento disponible de un evento y dos Clientes sin reserva previa sobre él, **CUANDO** ambos intentan reservar ese asiento simultáneamente, **ENTONCES** uno obtiene la única reserva vigente y el otro recibe un rechazo por indisponibilidad, sin generar ventas.
- **CA-RF-012-02**. **DADO** una única reserva exclusiva vigente y pago aprobado, **CUANDO** dos solicitudes diferentes, con claves distintas, intentan confirmar la misma reserva simultáneamente, **ENTONCES** solo una transición confirma y queda una compra, un ticket activo y un solo consumo de inventario.

### RF-013

- **CA-RF-013-01**. **DADO** una zona con N unidades libres y demanda simultánea superior a N, **CUANDO** se procesan nuevas reservas y se convierten algunas en ventas, **ENTONCES** se rechazan los intentos que exceden el cupo y en todo momento las reservas vigentes más las ventas activas respetan la capacidad, sin contar dos veces las unidades convertidas.

### RF-014

- **CA-RF-014-01**. **DADO** una compra confirmada cuya respuesta se perdió y su clave aún retenida, **CUANDO** el Cliente reintenta con la misma operación, clave y contenido, **ENTONCES** el resultado se vincula a la compra original y no se emiten más tickets ni se afecta de nuevo el inventario.
- **CA-RF-014-02**. **DADO** dos solicitudes idénticas con el mismo Cliente, operación, clave y contenido, **CUANDO** se envían simultáneamente para confirmar la compra, **ENTONCES** se confirma una sola compra y se ejecuta un solo efecto sobre el inventario.
- **CA-RF-014-03**. **DADO** una compra completada con respuesta recibida y clave retenida, **CUANDO** el Cliente repite la operación con la misma clave y contenido, **ENTONCES** se reconoce la compra original sin registrar otra compra ni aplicar efectos adicionales.
- **CA-RF-014-04**. **DADO** una clave retenida para una compra de un Cliente y un contenido determinado, **CUANDO** ese Cliente la reutiliza para la misma operación con contenido distinto, **ENTONCES** la API devuelve HTTP 409 Conflict sin crear otra compra ni modificar el inventario.
- **CA-RF-014-05**. **DADO** dos Clientes con reservas válidas sobre inventario distinto, **CUANDO** ambos confirman con el mismo valor de clave para su propia compra, **ENTONCES** cada clave se resuelve en el alcance de su usuario, sin cruzar resultados ni afectar inventario ajeno.
- **CA-RF-014-06**. **DADO** una compra confirmada y un reloj controlado con la retención inicial de 24 horas ya agotada, **CUANDO** se reintenta la confirmación de la misma reserva cuando la clave ya no está retenida, **ENTONCES** la restricción persistente impide crear una segunda compra, emitir más tickets o volver a consumir unidades.

### RF-015

- **CA-RF-015-01**. **DADO** un Cliente con dos tickets propios y un reloj situado exactamente 24 horas antes del inicio, **CUANDO** solicita por API la devolución de uno de ellos, **ENTONCES** se registra una solicitud pendiente vinculada únicamente a ese ticket y el otro conserva su estado.
- **CA-RF-015-02**. **DADO** un Cliente autenticado, un ticket de otro Cliente y una fecha dentro del plazo, **CUANDO** intenta solicitar su devolución directamente por API, **ENTONCES** se deniega y no se crea la solicitud.
- **CA-RF-015-03**. **DADO** un ticket propio y un reloj situado a menos de 24 horas del inicio, **CUANDO** el Cliente solicita su devolución, **ENTONCES** se rechaza por estar fuera de plazo y el ticket conserva su estado.

### RF-016

- **CA-RF-016-01**. **DADO** dos devoluciones pendientes de tickets distintos, **CUANDO** un Administrador aprueba una y rechaza la otra, **ENTONCES** la primera se confirma con reintegro simulado, ticket invalidado e inventario repuesto, y la segunda mantiene el ticket activo, el inventario comprometido y ningún reintegro.
- **CA-RF-016-02**. **DADO** una solicitud pendiente y un Event Owner sin permiso administrativo, **CUANDO** intenta resolverla, **ENTONCES** se deniega la operación y la solicitud continúa pendiente.

### RF-017

- **CA-RF-017-01**. **DADO** una devolución pendiente por un asiento o una unidad de zona, **CUANDO** el Administrador aprueba y la operación se repite incluso de forma concurrente, **ENTONCES** se registra un solo reintegro simulado, se invalida el ticket, se repone el inventario una vez y se conserva el historial de compra y devolución.
- **CA-RF-017-02**. **DADO** una devolución pendiente y una interrupción controlada antes de completar la transición atómica, **CUANDO** se intenta aprobar, se observa el estado tras el fallo y se vuelve a procesar, **ENTONCES** el fallo no deja efectos parciales visibles y el reintento confirma con un reintegro, una invalidación y una reposición.

### RF-018

- **CA-RF-018-01**. **DADO** dos clientes con categorías e historiales distintos, y un catálogo de prueba, **CUANDO** se consultan sus recomendaciones con las reglas definidas para II, **ENTONCES** los resultados coinciden con la puntuación esperada de cada perfil y pueden explicarse mediante sus señales y reglas.
- **CA-RF-018-02**. **DADO** un Cliente sin categorías seleccionadas ni historial y eventos que cumplen las reglas de popularidad o proximidad definidas para II, **CUANDO** se consultan recomendaciones, **ENTONCES** se obtiene la selección inicial prevista por esas reglas.

### RF-019

- **CA-RF-019-01**. **DADO** eventos que coinciden con todos o solo algunos de los filtros y eventos fuera del intervalo, **CUANDO** se busca combinando categoría, organizador, participante y un intervalo de fechas, **ENTONCES** solo se devuelven las coincidencias de la intersección dentro del intervalo.
- **CA-RF-019-02**. **DADO** una búsqueda válida sin coincidencias, **CUANDO** se envía la consulta, **ENTONCES** se devuelve una lista vacía.
- **CA-RF-019-03**. **DADO** eventos con variantes de mayúsculas y acentos, relevancias de prueba y empates por fecha, **CUANDO** se repite la búsqueda con variantes de texto equivalentes, **ENTONCES** se recupera el mismo conjunto, ordenado por relevancia ponderada y con desempate determinista por fecha e identificador.

### RF-020

- **CA-RF-020-01**. **DADO** dos eventos de alta demanda con capacidades configuradas independientemente, **CUANDO** se admiten solicitudes en ambos y se llena la cola de uno, **ENTONCES** cada solicitud permanece en su propia cola y la saturación de una no impide admitir en la otra cuando tiene cupo.
- **CA-RF-020-02**. **DADO** una cola en su capacidad máxima, **CUANDO** un Cliente sin entrada activa solicita ingresar, **ENTONCES** la API retorna HTTP 429 Too Many Requests y la cola no incorpora la solicitud.
- **CA-RF-020-03**. **DADO** un evento con el modo de alta demanda apagado y operaciones de prueba con y sin el permiso requerido, **CUANDO** se intenta activarlo manualmente por API, **ENTONCES** solo la operación autorizada activa la cola para ese evento.
- **CA-RF-020-04**. **DADO** un Cliente sin entrada activa y capacidad disponible, **CUANDO** envía solicitudes simultáneas de ingreso al mismo evento, **ENTONCES** queda como máximo una entrada activa para ese Cliente y evento.

### RF-021

- **CA-RF-021-01**. **DADO** tres solicitudes admitidas en secuencia A, B y C, **CUANDO** varios consumidores compiten por asignar turnos y B o C puede estar disponible antes que A para un consumidor, **ENTONCES** los turnos se asignan A, B y C sin adelantamientos.
- **CA-RF-021-02**. **DADO** un turno activo y un reloj controlado, **CUANDO** se alcanza su vencimiento configurado y el Cliente intenta usarlo, **ENTONCES** se rechaza el uso del turno vencido.
- **CA-RF-021-03**. **DADO** un contrato de procesamiento ordenado definido y justificado para III con hitos verificables y solicitudes con tiempos de ejecución distintos, **CUANDO** se ejecuta la venta masiva con varios consumidores y se capturan ingreso, asignación, procesamiento y finalización, **ENTONCES** la secuencia observada de procesamiento cumple el contrato justificado y se comprueba por separado de la asignación FIFO y del orden de finalización.

### RF-022

- **CA-RF-022-01**. **DADO** una solicitud autenticada en espera con SSE activo, **CUANDO** se registra la posición inicial y luego avanzan solicitudes anteriores, **ENTONCES** el Cliente recibe en tiempo real tanto la posición inicial como la nueva posición en su propia cola.
- **CA-RF-022-02**. **DADO** un Cliente que pierde la conexión SSE mientras su posición cambia, **CUANDO** se reconecta estando aún en espera, **ENTONCES** recibe la posición actual que corresponde a su entrada activa.

### RF-023

- **CA-RF-023-01**. **DADO** un JWT válido y ejemplos de JWT inválido, vencido y manipulado, **CUANDO** se invocan endpoints protegidos con cada uno, **ENTONCES** se acepta la credencial válida para la operación autorizada y las demás se rechazan antes de ejecutar operaciones protegidas.

### RF-024

- **CA-RF-024-01**. **DADO** un visitante sin cuenta, **CUANDO** se registra por la API pública e intenta indicar un rol privilegiado, **ENTONCES** la única cuenta que puede obtener por ese flujo tiene rol Cliente y no adquiere privilegios.
- **CA-RF-024-02**. **DADO** el Core inicializado en un entorno limpio con el seed previsto, **CUANDO** se verifica el estado inicial e inicia sesión el Administrador generado, **ENTONCES** existe la cuenta administrativa inicial y puede acceder a una operación de su rol.

### RF-025

- **CA-RF-025-01**. **DADO** un Event Owner autenticado que prepara un evento con zonas, una de ellas numerada, **CUANDO** guarda y vuelve a consultar el evento, **ENTONCES** se conservan título, descripción, categoría, organizador, participantes, ubicación, fecha, zonas, asientos, precios y estado.

### RF-026

- **CA-RF-026-01**. **DADO** un Administrador autenticado y datos de prueba del Core, **CUANDO** utiliza las APIs de usuarios, roles y eventos y consulta ventas e información operativa, **ENTONCES** puede realizar las gestiones y consultas conforme a sus permisos.
- **CA-RF-026-02**. **DADO** una venta confirmada con historial e inventario comprometido, **CUANDO** un Administrador intenta editarla directamente para eliminar el historial o alterar el compromiso de inventario, **ENTONCES** se rechaza la modificación y ventas, historial e inventario se conservan.
- **CA-RF-026-03**. **DADO** una devolución elegible pendiente en III, **CUANDO** un Administrador accede a la resolución por el flujo de devoluciones, **ENTONCES** puede resolverla conforme a RF-016 y RF-017 y conservar su trazabilidad.

### RF-027

- **CA-RF-027-01**. **DADO** una instancia del Core con frontend y APIs disponibles, **CUANDO** un Cliente se registra, inicia sesión, lista y consulta eventos, elige asiento o zona, reserva, compra y consulta su ticket, **ENTONCES** las vistas usan las APIs y el ticket consultado corresponde a la compra realmente persistida.
- **CA-RF-027-02**. **DADO** un Event Owner con credenciales válidas, **CUANDO** inicia sesión en el frontend, administra un evento propio, configura zonas y asientos y publica, **ENTONCES** las vistas reflejan las operaciones autorizadas y el evento aparece en la API de listado.
- **CA-RF-027-03**. **DADO** un Administrador autenticado en el frontend del Core, **CUANDO** accede a usuarios, roles, eventos, ventas e información operativa, **ENTONCES** las vistas consultan las APIs y presentan los resultados de los permisos administrativos.
- **CA-RF-027-04**. **DADO** un asiento ya ocupado y una respuesta de error controlada de la API en recorridos separados, **CUANDO** un Cliente intenta reservar el asiento y realiza la operación que falla, **ENTONCES** las vistas permiten distinguir indisponibilidad y error sin mostrar una compra confirmada.

### RF-028

- **CA-RF-028-01**. **DADO** un Cliente con acceso a las vistas de II, **CUANDO** guarda categorías y consulta recomendaciones desde el frontend, **ENTONCES** las vistas llaman a las APIs de preferencias y recomendaciones y muestran los resultados del perfil.
- **CA-RF-028-02**. **DADO** un Cliente sin señales y, en otra ejecución, una API de recomendaciones que falla, **CUANDO** abre la vista de recomendaciones, **ENTONCES** se muestra la selección inicial en el primer caso y el estado de error en el segundo.

### RF-029

- **CA-RF-029-01**. **DADO** un Cliente autenticado y datos válidos en III, **CUANDO** filtra eventos, solicita devolución de un ticket propio y observa su posición en una cola en casos independientes, **ENTONCES** la vista muestra el resultado real de búsqueda, el estado de devolución registrado por API y las actualizaciones de posición emitidas por SSE.
- **CA-RF-029-02**. **DADO** un Administrador con una devolución pendiente y APIs de III disponibles, **CUANDO** inicia sesión y resuelve la solicitud desde la vista administrativa, **ENTONCES** la vista refleja la resolución persistida por el flujo de devoluciones, incluido el error si la API rechaza la operación.
- **CA-RF-029-03**. **DADO** una cola saturada, un evento sin inventario y una solicitud de devolución fuera de plazo en casos independientes, **CUANDO** el Cliente intenta ingresar, comprar y devolver desde sus vistas respectivas, **ENTONCES** la interfaz distingue los errores de cola y devolución y la falta de inventario, sin indicar operaciones exitosas.

### RNF-001

- **CA-RNF-001-01**. **DADO** el entorno del Core disponible, **CUANDO** se ejecuta por API el recorrido autenticación, publicación y compra, **ENTONCES** las respuestas y el estado persistido coinciden con los resultados de negocio.

### RNF-002

- **CA-RNF-002-01**. **DADO** un asiento numerado y una zona con cupo en PostgreSQL, **CUANDO** se ejecutan en paralelo reservas, confirmaciones y expiraciones sobre el mismo inventario, **ENTONCES** la base conserva RN-001, RN-002, RN-003, RN-004 y RN-009 al confirmar las transiciones.
- **CA-RNF-002-02**. **DADO** el diseño técnico de una operación crítica de inventario y de las operaciones relevantes entre servicios, **CUANDO** se revisan las garantías y mecanismos evaluados, **ENTONCES** se justifica por operación la selección, aplicabilidad o descarte de locks optimistas, locks pesimistas, transacciones distribuidas, claves de idempotencia y deduplicación, con identificación de la transacción local propietaria.

### RNF-003

- **CA-RNF-003-01**. **DADO** una compra válida y dos envíos con la misma clave, usuario, operación y contenido, **CUANDO** se procesan de forma concurrente dentro de la retención, **ENTONCES** hay una sola confirmación y un solo efecto persistente sobre el inventario.
- **CA-RNF-003-02**. **DADO** la configuración inicial y una clave registrada a una hora conocida, **CUANDO** se inspecciona su retención y se prueba la clave con contenido distinto antes de que transcurran 24 horas, **ENTONCES** la retención inicial se configura en 24 horas y el hash permite detectar el cambio y producir HTTP 409 Conflict.

### RNF-004

- **CA-RNF-004-01**. **DADO** la documentación técnica del Core y sus contratos, **CUANDO** se revisa quién escribe cada dato de identidad, evento, inventario y compra, **ENTONCES** cada responsabilidad tiene un servicio propietario identificado.

### RNF-005

- **CA-RNF-005-01**. **DADO** los contratos de una interacción síncrona y una asíncrona del incremento correspondiente, **CUANDO** se ejecuta cada intercambio dentro de su contrato, **ENTONCES** el consumidor obtiene la respuesta o procesa el mensaje esperado según el tipo de interacción.

### RNF-006

- **CA-RNF-006-01**. **DADO** una compra con interrupción controlada tras confirmar su estado y antes de devolver la respuesta, **CUANDO** se restablece el servicio y el cliente reintenta, **ENTONCES** se reconoce la compra existente y el inventario refleja un único consumo.
- **CA-RNF-006-02**. **DADO** compras del Core y NATS temporalmente inaccesible, **CUANDO** PostgreSQL confirma una compra mientras el intermediario está caído, **ENTONCES** la compra, su inventario y el evento pendiente quedan persistidos y un servicio ajeno no puede revertir ese estado por su fallo.
- **CA-RNF-006-03**. **DADO** una compra confirmada con publicación pendiente por indisponibilidad de NATS, **CUANDO** se restablece la conexión y el publicador recupera el trabajo pendiente, **ENTONCES** el evento se publica y queda constancia de su despacho sin repetir la compra ni su efecto sobre el inventario.
- **CA-RNF-006-04**. **DADO** una compra confirmada y un consumidor de III temporalmente fallido durante la venta masiva, **CUANDO** se detiene y restablece el consumidor bajo el procedimiento de recuperación, **ENTONCES** la compra confirmada se conserva y los reintentos no crean ventas ni consumos duplicados.

### RNF-007

- **CA-RNF-007-01**. **DADO** un consumidor de historial de II desconectado y una compra confirmada, **CUANDO** se restablece el consumidor y procesa el trabajo retenido, **ENTONCES** el historial converge a la compra confirmada una sola vez y no se modifica el compromiso de inventario de esa compra.
- **CA-RNF-007-02**. **DADO** una compra confirmada y un consumidor de datos derivados que ya aplicó su evento, **CUANDO** se reentrega el mismo mensaje, **ENTONCES** el consumidor deja un solo efecto en la proyección y el inventario transaccional permanece inalterado.

### RNF-008

- **CA-RNF-008-01**. **DADO** dos servicios en ejecución con una actualización compatible de uno de ellos, **CUANDO** se despliega la actualización del servicio elegido, **ENTONCES** el otro servicio conserva su despliegue y supera sus comprobaciones de disponibilidad.

### RNF-009

- **CA-RNF-009-01**. **DADO** solicitudes HTTP con éxito, autenticación fallida, autorización denegada y error del servidor, bajo un reloj controlado, **CUANDO** se procesan hasta preparar sus respuestas, **ENTONCES** Pino registra cada resultado y la duración desde la recepción antes de retornar, incluida toda fase de autenticación, autorización y procesamiento ejecutada.
- **CA-RNF-009-02**. **DADO** una conexión SSE de III con distintos instantes controlados de recepción, aceptación y cierre, **CUANDO** se acepta y luego se cierra el canal, **ENTONCES** Pino registra por separado la apertura con latencia de aceptación y el cierre con duración de la conexión.

### RNF-010

- **CA-RNF-010-01**. **DADO** una entrada Redis para un tipo de dato con TTL definido y un evento de cambio que requiere invalidación, **CUANDO** se consulta con acierto, se consulta tras expirar y en un caso independiente se publica la invalidación en NATS, **ENTONCES** el acierto usa la entrada vigente y Cache-Aside consulta y repuebla desde la fuente en cada fallo de caché, dejando de servir la entrada invalidada.
- **CA-RNF-010-02**. **DADO** dos Clientes con resultados personalizados distintos y la caché habilitada, **CUANDO** el segundo consulta el mismo tipo de dato después del primero, **ENTONCES** cada identidad recibe sus propios datos y la caché no reutiliza resultados personalizados de otra identidad.
- **CA-RNF-010-03**. **DADO** Redis no disponible y varios accesos concurrentes que recurren a la fuente autorizada según la política de II, **CUANDO** se prueba por debajo y por encima de los límites acordados, **ENTONCES** se aplica la degradación definida y la carga que alcanza al servicio propietario se mantiene dentro de esos límites.
- **CA-RNF-010-04**. **DADO** la política de caché de la Etapa II, **CUANDO** se revisa junto con las configuraciones del incremento, **ENTONCES** identifica datos, TTL e invalidación y justifica la aplicabilidad de cliente, CDN, servicio y caché distribuida.

### RNF-011

- **CA-RNF-011-01**. **DADO** productores que superan la tasa de consumo y una cola de capacidad configurada, **CUANDO** la carga alcanza el límite y después el consumidor se recupera, **ENTONCES** se aplica backpressure con capacidad acotada y HTTP 429 para nuevas entradas al saturar, y el procesamiento asíncrono admitido satisface el contrato de RF-021.

### RNF-012

- **CA-RNF-012-01**. **DADO** un plan de estrés versionado con carga, entorno y umbrales definidos, **CUANDO** se ejecuta el perfil mediante k6 y se repite con las mismas condiciones documentadas, **ENTONCES** se puede reproducir el procedimiento y evaluar cada ejecución con throughput, p95, p99, tasa de errores, longitud de cola, tiempo de espera e integridad del inventario.

### RNF-013

- **CA-RNF-013-01**. **DADO** cuentas de prueba y credenciales conocidas usadas en solicitudes exitosas y fallidas, incluidas conexiones SSE desde III, **CUANDO** se inspeccionan las contraseñas persistidas y los registros estructurados, **ENTONCES** las contraseñas están almacenadas como hashes bcrypt y no aparecen contraseñas, tokens ni otras credenciales sensibles de prueba en los logs.

### RNF-014

- **CA-RNF-014-01**. **DADO** varios servicios que persisten datos en el entorno inicial, **CUANDO** se inspecciona su configuración y se ejecuta una escritura válida por servicio, **ENTONCES** cada servicio utiliza su propia base en la instancia PostgreSQL compartida.

### RNF-015

- **CA-RNF-015-01**. **DADO** un entorno local limpio con los prerrequisitos documentados, **CUANDO** se inicia el incremento con Docker Compose siguiendo las instrucciones del proyecto, **ENTONCES** los componentes se ponen en marcha y permiten repetir el recorrido de verificación documentado.

### RNF-016

- **CA-RNF-016-01**. **DADO** las APIs del incremento y un cambio de código, **CUANDO** se revisa el contrato publicado y se ejecutan las comprobaciones del proyecto, **ENTONCES** las operaciones ejercitadas coinciden con sus contratos y las comprobaciones detectan infracciones introducidas como control de prueba.

### RNF-017

- **CA-RNF-017-01**. **DADO** las pruebas de un incremento y datos de prueba conocidos, **CUANDO** se ejecutan dos veces en entornos equivalentes, **ENTONCES** los resultados son repetibles y las verificaciones ejercitan casos exitosos y de rechazo.

### RNF-018

- **CA-RNF-018-01**. **DADO** Redis muestra un asiento disponible que PostgreSQL ya tiene vendido, **CUANDO** se intenta reservarlo, **ENTONCES** la transacción rechaza la reserva y se conserva un solo ticket activo.
- **CA-RNF-018-02**. **DADO** Redis conserva una vista antigua del inventario y PostgreSQL registra la reserva vencida, **CUANDO** se intenta confirmar la compra, **ENTONCES** se rechaza la confirmación sin reconsumir unidades liberadas ni emitir tickets.

### RNF-019

- **CA-RNF-019-01**. **DADO** el plan previo a ejecutar pruebas de estrés en III, **CUANDO** se contrasta con la configuración de k6 y del entorno, **ENTONCES** especifica infraestructura, perfil de carga, usuarios concurrentes, duración, tasa de solicitudes, umbrales medibles y criterios de éxito y fracaso.

### RNF-020

- **CA-RNF-020-01**. **DADO** un evento candidato y una actualización conocida del historial o preferencias del Cliente, **CUANDO** se ejecuta el intercambio previsto y se solicita una recomendación, **ENTONCES** las reglas reciben la información del Cliente y producen el resultado esperado para los datos actualizados.

### RNF-021

- **CA-RNF-021-01**. **DADO** un catálogo de prueba en PostgreSQL, **CUANDO** se ejecutan búsquedas con criterios y relevancia conocidos y se inspecciona la operación de base de datos, **ENTONCES** PostgreSQL ejecuta el filtrado y el ordenamiento y la API entrega ese resultado.

### RNF-022

- **CA-RNF-022-01**. **DADO** una publicación que debe acompañar la confirmación de compra en el Core, **CUANDO** se fuerza primero un fallo antes del commit y luego se ejecuta una confirmación exitosa con NATS caído, **ENTONCES** el intento revertido no deja compra ni evento pendiente, y en el exitoso PostgreSQL conserva ambos desde el mismo commit.
- **CA-RNF-022-02**. **DADO** un evento de una operación confirmada almacenado en el outbox y un consumidor idempotente, **CUANDO** se restablece NATS, se publica por JetStream y se provoca una reentrega, **ENTONCES** la publicación pendiente se recupera y la reentrega produce un solo efecto observable en el consumidor.

### RNF-023

- **CA-RNF-023-01**. **DADO** las vistas funcionales de un incremento y sus APIs en ejecución, **CUANDO** se ejecutan los recorridos de RF-027, RF-028 o RF-029 según la etapa y se inspeccionan dependencias y tráfico, **ENTONCES** las vistas usan Vue 3 con Vite y JavaScript, ejercitan APIs reales y respetan permisos del backend, con SSE en III.

### RM-001

- **CA-RM-001-01**. **DADO** un incremento listo para revisión, **CUANDO** se inspeccionan sus artefactos, **ENTONCES** las especificaciones de OpenSpec describen el comportamiento implementado y sus escenarios.

### RM-002

- **CA-RM-002-01**. **DADO** los tres incrementos revisables, **CUANDO** se contrasta su secuencia y alcance con la sección 12, **ENTONCES** Core precede a recomendaciones y caché, y ambos preceden a escalabilidad.

### RM-003

- **CA-RM-003-01**. **DADO** el historial de trabajo de una etapa, **CUANDO** se revisa su change de OpenSpec, **ENTONCES** existe un único change para la etapa y se identifica el recorrido desde proposal hasta archive.
- **CA-RM-003-02**. **DADO** los artefactos de una etapa implementada y preparada para archivar, **CUANDO** se aplica el control de calidad del equipo previo al archivado, **ENTONCES** quedan registradas la verificación de los criterios aplicables, los resultados de pruebas y la resolución o registro de hallazgos.

### EA-001

- **CA-EA-001-01**. **DADO** la guía `CONTRIBUTE.md`, **CUANDO** se revisa su flujo de contribución, **ENTONCES** se identifican los pasos para trabajar en una feature branch, ejecutar tests y pasar la CI.

### EA-002

- **CA-EA-002-01**. **DADO** una etapa terminada, **CUANDO** se inspecciona el archivo de OpenSpec, **ENTONCES** el change archivado contiene specs, design y tasks.

### EA-003

- **CA-EA-003-01**. **DADO** los mockups del Core para Cliente, Event Owner y Administrador, **CUANDO** se revisan los recorridos de AMB-015 aplicables a la Etapa I, **ENTONCES** los mockups en Vue representan las pantallas y estados de éxito, error e indisponibilidad según corresponda, con la integración verificada por separado.

### EA-004

- **CA-EA-004-01**. **DADO** el incremento desplegado, **CUANDO** se ejecutan los casos de API documentados para la entrega, **ENTONCES** las operaciones devuelven los resultados esperados.

### EA-005

- **CA-EA-005-01**. **DADO** la suite de unidad de un incremento, **CUANDO** se ejecuta con el procedimiento documentado, **ENTONCES** los casos del alcance implementado se ejecutan y reportan sus resultados.

### EA-006

- **CA-EA-006-01**. **DADO** ejecuciones relevantes de Codex con resultados exitosos y otras que necesitaron correcciones, **CUANDO** se revisa la bitácora de desarrollo comprobando los campos requeridos de cada entrada y sus referencias al historial, **ENTONCES** las ejecuciones y correcciones conservan sus registros previos, los datos requeridos y las referencias aplicables, sin confundirse con los registros HTTP.

## 9. Restricciones tecnológicas

**Origen** DECISIÓN APROBADA (stack de P-001). El uso de OpenSpec también es explícito en la consigna (p. 3, Metodología).

| Ámbito | Stack aprobado |
| --- | --- |
| Lenguajes y aplicaciones | JavaScript vanilla (ES6+), Node.js, Express, Vue 3 con Vite |
| Persistencia y mensajería | PostgreSQL, Sequelize como ORM, NATS con JetStream, Redis |
| Despliegue local | Docker, Docker Compose |
| Identidad, validación y contratos | JWT, bcrypt, Zod, OpenAPI, Swagger UI |
| Pruebas | Vitest, Supertest, Testcontainers, k6 |
| Observabilidad y notificaciones | Pino, Server-Sent Events (SSE) |
| Versionado, integración y herramientas | Git, GitHub, GitHub Actions, npm, npm Workspaces, ESLint, Prettier y OpenSpec |

Se excluyen TypeScript, Drizzle ORM, Prisma y React.

## 10. Decisiones aprobadas

Decisiones comunicadas por Briant + Fernando en P-001. Sus vínculos indican dónde se aplican y verifican. Las resoluciones complementarias aprobadas en P-001.1 conservan sus identificadores AMB y se documentan en la sección 13.

| ID | Decisión aprobada | Aplicación |
| --- | --- | --- |
| DF-001 | Representar pagos aprobados y rechazados mediante un simulador. | RF-011 |
| DF-002 | Establecer una duración inicial de 10 minutos, configurable. | RF-008, RF-009 |
| DF-003 | Usar JWT para autenticación. | RF-023 |
| DF-004 | Controlar por medio de Administradores la asignación del rol Event Owner. | RF-004 |
| DF-005 | Liberar el inventario correspondiente después de confirmar la devolución. | RF-017 |
| DF-006 | Aplicar reglas basadas en preferencias e historial de compras. | RF-018 |
| DF-007 | Usar colas independientes por evento con capacidad configurable. | RF-020 |
| DF-008 | Usar PostgreSQL para la búsqueda avanzada y el ordenamiento por relevancia. | RF-019, RNF-021 |
| DF-009 | Establecer objetivos de rendimiento medibles antes de ejecutar pruebas de estrés. | RNF-012, RNF-019 |
| DF-010 | Desarrollar los mockups en Vue y, según AMB-015, las vistas funcionales en Vue 3 con Vite y JavaScript, conectadas progresivamente con las APIs. | EA-003, RF-027, RF-028, RF-029, RNF-023 |
| DF-011 | Mantener una base por servicio, inicialmente en una misma instancia PostgreSQL. | RNF-014 |
| DF-012 | Utilizar Docker Compose como entorno de despliegue local. | RNF-015 |

## 11. Matriz de trazabilidad

**CONSIGNA** indica un requisito explícito del PDF. **DERIVADO** indica una condición necesaria para cumplir un requisito o una decisión. **DECISIÓN APROBADA** identifica una definición de Briant + Fernando, incluida la resolución de cada AMB en P-001.1. **PROPUESTA** identifica una definición de negocio que requiera aprobación. En filas derivadas, la página señala el requisito académico de apoyo. El guion indica ausencia de una página académica aplicable a esa definición.

| ID | Nombre | Clasificación | Origen | Página y sección del PDF | Etapa | Criterios de aceptación |
| --- | --- | --- | --- | --- | --- | --- |
| RF-001 | Identidad y autenticación | Funcional | CONSIGNA | 3 · Metodología I | I | CA-RF-001-01, CA-RF-001-02 |
| RF-002 | Roles y responsabilidades | Funcional | CONSIGNA | 1 · Enunciado | I | CA-RF-002-01 |
| RF-003 | Autorización por operación | Funcional | DERIVADO | 1 · Enunciado y devoluciones | I | CA-RF-003-01, CA-RF-003-02, CA-RF-003-03 |
| RF-004 | Asignación de Event Owner | Funcional | DECISIÓN APROBADA | — | I | CA-RF-004-01, CA-RF-004-02 |
| RF-005 | Diseño y publicación de eventos | Funcional | CONSIGNA | 1 · Enunciado | I | CA-RF-005-01, CA-RF-005-02 |
| RF-006 | Configuración de zonas y asientos | Funcional | DERIVADO | 2 · Entradas numeradas y no numeradas | I | CA-RF-006-01, CA-RF-006-02, CA-RF-006-03 |
| RF-007 | Modalidades de tickets | Funcional | CONSIGNA | 2 · Entradas numeradas y no numeradas | I | CA-RF-007-01 |
| RF-008 | Creación de reservas | Funcional | DERIVADO | 1 · Compra y unicidad de venta, 3 · Metodología I | I | CA-RF-008-01 |
| RF-009 | Vencimiento de reservas | Funcional | DERIVADO | — | I | CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04 |
| RF-010 | Compra de tickets | Funcional | CONSIGNA | 1 · Enunciado, 3 · Metodología I | I | CA-RF-010-01 |
| RF-011 | Simulación de pagos | Funcional | DECISIÓN APROBADA | — | I | CA-RF-011-01, CA-RF-011-02 |
| RF-012 | Unicidad de venta | Funcional | CONSIGNA | 1 · Unicidad de venta, 2 · Entradas numeradas | I | CA-RF-012-01, CA-RF-012-02 |
| RF-013 | Capacidad de zona | Funcional | DERIVADO | 2 · Entradas no numeradas | I | CA-RF-013-01 |
| RF-014 | Idempotencia de compra | Funcional | CONSIGNA | 1 · Unicidad de venta, 2 · Desafíos técnicos | I | CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06 |
| RF-015 | Solicitud de devolución | Funcional | CONSIGNA | 1 · Gestionar devoluciones, 3 · Metodología III | III | CA-RF-015-01, CA-RF-015-02, CA-RF-015-03 |
| RF-016 | Resolución administrativa de devoluciones | Funcional | CONSIGNA | 1 · Gestionar devoluciones | III | CA-RF-016-01, CA-RF-016-02, CA-RF-017-01 |
| RF-017 | Reposición de inventario por devolución | Funcional | DECISIÓN APROBADA | 1 · Gestionar devoluciones | III | CA-RF-017-01, CA-RF-017-02 |
| RF-018 | Recomendaciones personalizadas | Funcional | CONSIGNA | 2 · Sistema de recomendación, 3 · Metodología II | II | CA-RF-018-01, CA-RF-018-02 |
| RF-019 | Búsqueda avanzada | Funcional | CONSIGNA | 2 · Búsquedas por múltiples criterios, 3 · Metodología III | III | CA-RF-019-01, CA-RF-019-02, CA-RF-019-03 |
| RF-020 | Ingreso a cola de venta masiva | Funcional | CONSIGNA | 2 · Soportar eventos masivos, 3 · Metodología III | III | CA-RF-020-01, CA-RF-020-02, CA-RF-020-03, CA-RF-020-04 |
| RF-021 | Procesamiento ordenado de ventas masivas | Funcional | CONSIGNA | 2 · Soportar eventos masivos | III | CA-RF-021-01, CA-RF-021-02, CA-RF-021-03 |
| RF-022 | Notificación de posición en cola | Funcional | CONSIGNA | 2 · Soportar eventos masivos, 3 · Desafíos técnicos | III | CA-RF-022-01, CA-RF-022-02 |
| RF-023 | Autenticación con JWT | Funcional | DECISIÓN APROBADA | — | I | CA-RF-023-01 |
| RF-024 | Registro e inicialización de cuentas | Funcional | DECISIÓN APROBADA | — | I | CA-RF-024-01, CA-RF-024-02 |
| RF-025 | Datos mínimos del evento | Funcional | DECISIÓN APROBADA | — | I | CA-RF-025-01 |
| RF-026 | Capacidades administrativas | Funcional | DECISIÓN APROBADA | — | I–III | CA-RF-026-01, CA-RF-026-02, CA-RF-026-03 |
| RF-027 | Vistas funcionales del Core | Funcional | DECISIÓN APROBADA | — | I | CA-RF-027-01, CA-RF-027-02, CA-RF-027-03, CA-RF-027-04 |
| RF-028 | Vistas funcionales de recomendaciones | Funcional | DECISIÓN APROBADA | — | II | CA-RF-028-01, CA-RF-028-02 |
| RF-029 | Vistas funcionales de escalabilidad | Funcional | DECISIÓN APROBADA | — | III | CA-RF-029-01, CA-RF-029-02, CA-RF-029-03 |
| RNF-001 | Backend y APIs del Core | No funcional o técnico | CONSIGNA | 3 · Metodología I y Entregables | I | CA-RNF-001-01 |
| RNF-002 | Integridad de inventario y concurrencia | No funcional o técnico | CONSIGNA | 2 · Desafíos técnicos, unicidad de venta | I | CA-RNF-002-01, CA-RNF-002-02 |
| RNF-003 | Claves de idempotencia y deduplicación | No funcional o técnico | CONSIGNA | 2 · Desafíos técnicos, unicidad de venta | I | CA-RNF-003-01, CA-RNF-003-02 |
| RNF-004 | Límites de microservicios | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, microservicios | I | CA-RNF-004-01 |
| RNF-005 | Comunicación entre servicios | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, microservicios y Metodología II | I–III | CA-RNF-005-01 |
| RNF-006 | Manejo de fallos parciales | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, microservicios | I–III | CA-RNF-006-01, CA-RNF-006-02, CA-RNF-006-03, CA-RNF-006-04 |
| RNF-007 | Consistencia eventual | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, microservicios | II–III | CA-RNF-007-01, CA-RNF-007-02 |
| RNF-008 | Despliegue independiente | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, microservicios | I–III | CA-RNF-008-01 |
| RNF-009 | Registro HTTP con tiempos | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, bitácora HTTP | I, III | CA-RNF-009-01, CA-RNF-009-02 |
| RNF-010 | Políticas de caché | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, estrategias de cache y Metodología II | II | CA-RNF-010-01, CA-RNF-010-02, CA-RNF-010-03, CA-RNF-010-04 |
| RNF-011 | Infraestructura de colas y backpressure | No funcional o técnico | CONSIGNA | 3 · Desafíos técnicos, eventos masivos | III | CA-RNF-011-01 |
| RNF-012 | Pruebas de estrés | No funcional o técnico | CONSIGNA | 3 · Metodología III | III | CA-RNF-012-01 |
| RNF-013 | Protección de credenciales | No funcional o técnico | DERIVADO | 3 · Metodología I | I–III | CA-RNF-013-01 |
| RNF-014 | Aislamiento de bases de datos | No funcional o técnico | DECISIÓN APROBADA | — | I–III | CA-RNF-014-01 |
| RNF-015 | Entorno reproducible | No funcional o técnico | DECISIÓN APROBADA | — | I–III | CA-RNF-015-01 |
| RNF-016 | Mantenibilidad y contratos de API | No funcional o técnico | DERIVADO | 2 · Desafíos técnicos, introducción, 3 · Metodología I | I–III | CA-RNF-016-01 |
| RNF-017 | Testabilidad de reglas y APIs | No funcional o técnico | DERIVADO | 3 · Entregables | I–III | CA-RNF-017-01 |
| RNF-018 | Caché fuera de la confirmación de inventario | No funcional o técnico | DERIVADO | 3 · Desafíos técnicos, estrategias de cache | II | CA-RNF-018-01, CA-RNF-018-02 |
| RNF-019 | Objetivos medibles de rendimiento | No funcional o técnico | DECISIÓN APROBADA | — | III | CA-RNF-019-01 |
| RNF-020 | Integración con recomendaciones | No funcional o técnico | CONSIGNA | 2 · Sistema de recomendación, 3 · Metodología II | II | CA-RNF-020-01 |
| RNF-021 | Búsqueda y relevancia en PostgreSQL | No funcional o técnico | DECISIÓN APROBADA | — | III | CA-RNF-021-01 |
| RNF-022 | Publicación recuperable y consumo idempotente | No funcional o técnico | DECISIÓN APROBADA | — | I–III | CA-RNF-022-01, CA-RNF-022-02 |
| RNF-023 | Integración web con APIs | No funcional o técnico | DECISIÓN APROBADA | — | I–III | CA-RNF-023-01 |
| RN-001 | Unicidad de reserva y venta por asiento | Regla de negocio | CONSIGNA | 1 · Unicidad de venta, 2 · Entradas numeradas | I | CA-RF-008-01, CA-RF-012-01, CA-RF-012-02, CA-RNF-002-01 |
| RN-002 | Capacidad máxima de zona | Regla de negocio | DERIVADO | 2 · Entradas no numeradas | I | CA-RF-013-01, CA-RNF-002-01 |
| RN-003 | Un efecto por solicitud de compra | Regla de negocio | CONSIGNA | 1 · Unicidad de venta, 2 · Desafíos técnicos | I | CA-RF-012-02, CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06, CA-RNF-003-01 |
| RN-004 | Liberación por vencimiento | Regla de negocio | DERIVADO | — | I | CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04 |
| RN-005 | Liberación única tras devolución | Regla de negocio | DERIVADO | 1 · Gestionar devoluciones | III | CA-RF-017-01, CA-RF-017-02 |
| RN-006 | Propiedad de eventos de Event Owner | Regla de negocio | DECISIÓN APROBADA | 1 · Enunciado | I | CA-RF-003-02, CA-RF-005-01, CA-RF-027-02 |
| RN-007 | Acceso por rol y recurso | Regla de negocio | DERIVADO | 1 · Enunciado y devoluciones | I, III | CA-RF-003-01, CA-RF-003-02, CA-RF-003-03, CA-RF-004-02, CA-RF-015-02, CA-RF-016-02 |
| RN-008 | Orden de venta masiva | Regla de negocio | CONSIGNA | 2 · Soportar eventos masivos | III | CA-RF-021-01, CA-RF-021-03, CA-RNF-011-01 |
| RN-009 | Contabilidad de reservas y ventas | Regla de negocio | DERIVADO | 1 · Unicidad de venta, 2 · Entradas numeradas y no numeradas | I | CA-RF-008-01, CA-RF-010-01, CA-RF-013-01 |
| RN-010 | Rechazo de devolución | Regla de negocio | DERIVADO | 1 · Gestionar devoluciones | III | CA-RF-016-01 |
| RN-011 | Venta y cambios de inventario por estado | Regla de negocio | DECISIÓN APROBADA | — | I | CA-RF-005-02, CA-RF-006-03 |
| RN-012 | Elegibilidad de devolución | Regla de negocio | DECISIÓN APROBADA | — | III | CA-RF-015-01, CA-RF-015-02, CA-RF-015-03 |
| RN-013 | Entrada única y vigencia de turno | Regla de negocio | DECISIÓN APROBADA | — | III | CA-RF-020-01, CA-RF-020-02, CA-RF-020-04, CA-RF-021-02 |
| RM-001 | Desarrollo guiado por especificaciones | Restricción metodológica | CONSIGNA | 3 · Metodología | I–III | CA-RM-001-01 |
| RM-002 | Tres incrementos oficiales | Restricción metodológica | CONSIGNA | 3 · Metodología | I–III | CA-RM-002-01 |
| RM-003 | Un change por etapa | Restricción metodológica | CONSIGNA | 3 · Metodología | I–III | CA-RM-003-01, CA-RM-003-02 |
| EA-001 | Guía de contribución | Entregable académico | CONSIGNA | 3 · Entregables | I–III | CA-EA-001-01 |
| EA-002 | Changes archivados | Entregable académico | CONSIGNA | 3 · Entregables | I–III | CA-EA-002-01 |
| EA-003 | Mockups | Entregable académico | CONSIGNA | 3 · Entregables y Metodología I | I | CA-EA-003-01 |
| EA-004 | APIs funcionales | Entregable académico | CONSIGNA | 3 · Entregables | I–III | CA-EA-004-01 |
| EA-005 | Tests de unidad | Entregable académico | CONSIGNA | 3 · Entregables | I–III | CA-EA-005-01 |
| EA-006 | Bitácora de documentación | Entregable académico | CONSIGNA | 3 · Entregables | I–III y Fase 0 | CA-EA-006-01 |
| DF-001 | Simulador de pagos | Decisión del equipo | DECISIÓN APROBADA | — | I | CA-RF-011-01, CA-RF-011-02 |
| DF-002 | Vigencia de reserva | Decisión del equipo | DECISIÓN APROBADA | — | I | CA-RF-008-01, CA-RF-009-01, CA-RF-009-03, CA-RF-009-04 |
| DF-003 | Autenticación JWT | Decisión del equipo | DECISIÓN APROBADA | — | I | CA-RF-023-01 |
| DF-004 | Asignación administrativa de Event Owner | Decisión del equipo | DECISIÓN APROBADA | — | I | CA-RF-004-01, CA-RF-004-02 |
| DF-005 | Reposición tras devolución confirmada | Decisión del equipo | DECISIÓN APROBADA | — | III | CA-RF-017-01, CA-RF-017-02 |
| DF-006 | Recomendaciones por reglas | Decisión del equipo | DECISIÓN APROBADA | — | II | CA-RF-018-01, CA-RF-018-02 |
| DF-007 | Capacidad y separación por evento | Decisión del equipo | DECISIÓN APROBADA | — | III | CA-RF-020-01, CA-RF-020-02 |
| DF-008 | Búsqueda en PostgreSQL | Decisión del equipo | DECISIÓN APROBADA | — | III | CA-RF-019-01, CA-RF-019-02, CA-RF-019-03, CA-RNF-021-01 |
| DF-009 | Objetivos de rendimiento previos | Decisión del equipo | DECISIÓN APROBADA | — | III | CA-RNF-012-01, CA-RNF-019-01 |
| DF-010 | Mockups e interfaces en Vue | Decisión del equipo | DECISIÓN APROBADA | — | I–III | CA-EA-003-01, CA-RF-027-01, CA-RF-028-01, CA-RF-029-01, CA-RNF-023-01 |
| DF-011 | Bases independientes por servicio | Decisión del equipo | DECISIÓN APROBADA | — | I–III | CA-RNF-014-01 |
| DF-012 | Docker Compose local | Decisión del equipo | DECISIÓN APROBADA | — | I–III | CA-RNF-015-01 |
| AMB-001 | Cuentas y permisos por evento | Resolución del equipo | DECISIÓN APROBADA | — | I | CA-RF-003-02, CA-RF-004-01, CA-RF-004-02, CA-RF-005-01, CA-RF-024-01, CA-RF-024-02, CA-RF-026-01 |
| AMB-002 | Ciclo del evento, zonas y asientos | Resolución del equipo | DECISIÓN APROBADA | — | I | CA-RF-005-01, CA-RF-005-02, CA-RF-006-03, CA-RF-010-01, CA-RF-013-01 |
| AMB-003 | Pago rechazado y carrera con el vencimiento | Resolución del equipo | DECISIÓN APROBADA | — | I | CA-RF-008-01, CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04, CA-RF-010-01, CA-RF-011-01, CA-RF-011-02 |
| AMB-004 | Contrato de idempotencia | Resolución del equipo | DECISIÓN APROBADA | — | I | CA-RF-012-02, CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06, CA-RNF-003-02 |
| AMB-005 | Consistencia y fallos entre servicios | Resolución del equipo | DECISIÓN APROBADA | — | I–III | CA-RNF-002-01, CA-RNF-002-02, CA-RNF-004-01, CA-RNF-005-01, CA-RNF-006-01, CA-RNF-006-02, CA-RNF-006-03, CA-RNF-006-04, CA-RNF-007-01, CA-RNF-007-02, CA-RNF-008-01, CA-RNF-022-01, CA-RNF-022-02 |
| AMB-006 | Preferencias y arranque en frío | Resolución del equipo | DECISIÓN APROBADA | — | II | CA-RF-018-01, CA-RF-018-02, CA-RF-028-01, CA-RF-028-02, CA-RNF-020-01 |
| AMB-007 | Política y cierre de devoluciones | Resolución del equipo | DECISIÓN APROBADA | — | III | CA-RF-015-01, CA-RF-015-02, CA-RF-015-03, CA-RF-016-01, CA-RF-016-02, CA-RF-017-01, CA-RF-017-02, CA-RF-026-03 |
| AMB-008 | Filtros y relevancia | Resolución del equipo | DECISIÓN APROBADA | — | III | CA-RF-019-01, CA-RF-019-02, CA-RF-019-03, CA-RNF-021-01 |
| AMB-009 | Orden, admisión y reintentos de cola | Resolución del equipo | DECISIÓN APROBADA | — | III | CA-RF-020-01, CA-RF-020-02, CA-RF-020-03, CA-RF-020-04, CA-RF-021-01, CA-RF-021-02, CA-RF-021-03, CA-RF-022-01, CA-RNF-011-01 |
| AMB-010 | Objetivos y perfil de estrés | Resolución del equipo | DECISIÓN APROBADA | — | III | CA-RNF-012-01, CA-RNF-019-01 |
| AMB-011 | Medición HTTP y reconexión SSE | Resolución del equipo | DECISIÓN APROBADA | — | I, III | CA-RNF-009-01, CA-RNF-009-02, CA-RNF-013-01, CA-RF-022-02 |
| AMB-012 | Política y fallo de caché | Resolución del equipo | DECISIÓN APROBADA | — | II | CA-RNF-010-01, CA-RNF-010-02, CA-RNF-010-03, CA-RNF-010-04, CA-RNF-018-01, CA-RNF-018-02 |
| AMB-013 | Datos mínimos de evento | Resolución del equipo | DECISIÓN APROBADA | — | I | CA-RF-025-01 |
| AMB-014 | Alcance operativo del Administrador | Resolución del equipo | DECISIÓN APROBADA | — | I–III | CA-RF-026-01, CA-RF-026-02, CA-RF-026-03, CA-RF-027-03, CA-RF-029-02 |
| AMB-015 | Alcance de los mockups | Resolución del equipo | DECISIÓN APROBADA | — | I–III | CA-EA-003-01, CA-RF-027-01, CA-RF-027-02, CA-RF-027-03, CA-RF-027-04, CA-RF-028-01, CA-RF-028-02, CA-RF-029-01, CA-RF-029-02, CA-RF-029-03, CA-RNF-023-01 |

## 12. Distribución por etapas

| ID | Restricción metodológica | Aplicación |
| --- | --- | --- |
| RM-001 | Desarrollo guiado por especificaciones | Usar Spec-Driven Development con OpenSpec. |
| RM-002 | Tres incrementos oficiales | Entregar Core, integración con recomendaciones y caché, y escalabilidad, en ese orden y con el alcance de la sección 12. |
| RM-003 | Un change por etapa | Gestionar cada etapa como un único change con la secuencia `proposal → specs → design → tasks → implement → archive`. Aplicar la verificación del equipo al terminar `implement` y antes de `archive` (P-001.1, CA-RM-003-02). |

| Etapa oficial | Alcance del incremento | Dependencias |
| --- | --- | --- |
| I · Core | Backend y APIs, registro y autenticación, autorización por roles, gestión de eventos, inventario numerado y no numerado, reservas, simulación de pagos, compras, concurrencia, idempotencia, comunicación y recuperación ante fallos del Core, bitácora HTTP, mockups, frontend funcional y pruebas correspondientes. | La autenticación, propiedad y permisos habilitan gestión y compra. Los datos de eventos, reservas y compras habilitan II y III. La integridad del inventario y la recuperación de publicaciones se verifican desde I. |
| II · Integración con sistema de recomendación y caché | Preferencias, historial de compras, recomendaciones, integración, Redis, caché e invalidación, vistas correspondientes y pruebas. | RF-018 y RNF-020 usan catálogo e historial del Core y las preferencias de AMB-006. Las decisiones de reserva y compra siguen consultando PostgreSQL (RNF-018). Se amplían las pruebas de fallos a consumidores y datos derivados. |
| III · Escalabilidad | Colas por evento, asignación FIFO y procesamiento ordenado, backpressure, SSE, devoluciones, búsqueda avanzada, vistas correspondientes, estrés con k6 y verificación final de integridad y recuperación. | Reutiliza identidad, datos, compra, inventario y contratos previos. SSE depende de las colas y turnos. Devoluciones reutiliza las invariantes del Core. La búsqueda se basa en los datos de RF-025 y la prueba de estrés requiere el plan de RNF-019. |

Los requisitos transversales se incorporan al primer incremento aplicable y se vuelven a verificar cuando se amplía su alcance. Cada etapa se entrega como un único change. P-001.1 pertenece a la Fase 0 documental.

## 13. Resoluciones aprobadas y riesgos

Briant + Fernando aprobaron las 15 resoluciones en P-001.1. Cada registro conserva el problema original y sus riesgos. **RESUELTA** indica una política completa para esta especificación. **RESOLUCIÓN APROBADA — DETALLES DIFERIDOS** indica que la política está aprobada y que aún deben precisarse los detalles indicados en la sección 17.

### AMB-001 · Cuentas y permisos por evento

- **Contexto original.** Falta acordar alta y recuperación de cuentas, asignación y revocación de acceso a eventos, y si varios Event Owners pueden compartirlos.
- **Riesgo.** Una autorización excesiva permite cambios sobre eventos ajenos. Una autorización demasiado estrecha impide la colaboración.
- **Resolución aprobada.** El registro es público para Clientes. Un seed genera el Administrador inicial. Los Administradores pueden promover Clientes a Event Owner e intervenir conforme a sus permisos. Cada evento tiene un único Event Owner, que solo gestiona sus eventos y no puede delegar a otros propietarios en la primera versión. La recuperación de contraseña queda fuera de esa versión. El backend verifica la autorización con independencia de la interfaz.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESUELTA.
- **Etapa y requisitos afectados.** I. RF-001, RF-003, RF-004, RF-005, RF-024, RF-026, RN-006, RN-007, DF-004.
- **Criterios afectados.** CA-RF-003-02, CA-RF-004-01, CA-RF-004-02, CA-RF-005-01, CA-RF-024-01, CA-RF-024-02, CA-RF-026-01.
- **Definiciones diferidas.** Ninguna definición de política queda diferida en este registro.

### AMB-002 · Ciclo del evento, zonas y asientos

- **Contexto original.** Faltan estados del evento y reglas para modificar zonas, asientos, cupos o visibilidad tras publicar o vender.
- **Riesgo.** Un cambio puede dejar tickets sin asiento o reducir la capacidad por debajo de lo comprometido.
- **Resolución aprobada.** Los estados mínimos son DRAFT, PUBLISHED, CANCELLED y FINISHED. Se admiten nuevas compras exclusivamente en eventos publicados y habilitados para la venta. Se bloquean cambios de zonas, asientos o cupos que invaliden reservas vigentes o ventas activas. Un evento con ventas confirmadas requiere una política específica antes de habilitar su cancelación. La cancelación por sí sola no elimina compras ni repone inventario vendido.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** I. RF-005, RF-006, RF-010, RF-013, RN-011.
- **Criterios afectados.** CA-RF-005-01, CA-RF-005-02, CA-RF-006-03, CA-RF-010-01, CA-RF-013-01.
- **Definiciones diferidas.** Política de cancelación para eventos con ventas confirmadas, previa a habilitarla (I).

### AMB-003 · Pago rechazado y carrera con el vencimiento

- **Contexto original.** Falta decidir si un rechazo conserva la reserva y qué resultado comunica el sistema cuando el pago se aprueba justo al vencer.
- **Riesgo.** El Cliente puede recibir mensajes incompatibles con la compra o quedar con inventario apartado innecesariamente.
- **Resolución aprobada.** Las reservas duran inicialmente 10 minutos y son configurables. El rechazo de pago conserva la reserva hasta el vencimiento y permite reintentar durante su vigencia, sin emitir tickets ni confirmar compras. La compra solo confirma si la reserva sigue vigente en la transición transaccional. El vencimiento y la confirmación concurrentes tienen efectos mutuamente excluyentes. Una confirmación tardía no consume inventario liberado.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** I. RF-008, RF-009, RF-010, RF-011, RN-004, RN-009, DF-001, DF-002.
- **Criterios afectados.** CA-RF-008-01, CA-RF-009-01, CA-RF-009-02, CA-RF-009-03, CA-RF-009-04, CA-RF-010-01, CA-RF-011-01, CA-RF-011-02.
- **Definiciones diferidas.** Semántica de reintentos tras rechazo, coordinada con AMB-004 (I).

### AMB-004 · Contrato de idempotencia

- **Contexto original.** Faltan alcance y vigencia de clave, tratamiento de solicitudes en curso y respuesta si se reutiliza con datos distintos.
- **Riesgo.** Un reintento legítimo puede fallar o una operación distinta puede confundirse con una compra anterior.
- **Resolución aprobada.** Cada operación de compra usa una clave asociada a usuario y operación. Un hash del contenido detecta reutilización incompatible, que retorna HTTP 409 Conflict. Repetir clave y contenido dentro del mismo alcance conserva un único efecto. La retención inicial es de 24 horas y la unicidad persistente sigue vigente tras la expiración de la clave. La exclusión entre operaciones distintas se verifica por separado.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** I. RF-010, RF-014, RNF-003, RN-003.
- **Criterios afectados.** CA-RF-012-02, CA-RF-014-01, CA-RF-014-02, CA-RF-014-03, CA-RF-014-04, CA-RF-014-05, CA-RF-014-06, CA-RNF-003-02.
- **Definiciones diferidas.** Solicitudes en curso, respuestas almacenadas, expiración y reintentos de pagos rechazados (I).

### AMB-005 · Consistencia y fallos entre servicios

- **Contexto original.** La consigna menciona bloqueos y transacciones distribuidas como mecanismos. Queda por determinar cuándo se necesita coordinar servicios y qué datos pueden converger después.
- **Riesgo.** Un fallo parcial puede duplicar efectos o dejar datos divergentes entre servicios.
- **Resolución aprobada.** PostgreSQL es la fuente de verdad transaccional y cada servicio es propietario de sus datos. Las invariantes críticas se protegen con transacciones locales en el propietario. Transactional Outbox recupera publicaciones pendientes de operaciones confirmadas. NATS con JetStream proporciona comunicación asíncrona cuando se necesita persistir mensajes. Los consumidores son idempotentes. Se admite consistencia eventual para datos derivados y operaciones que preserven las invariantes. Aplicar recuperación, reintento y compensación cuando corresponda. Evaluar por separado la necesidad de transacciones distribuidas.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** I–III. RNF-002, RNF-004, RNF-005, RNF-006, RNF-007, RNF-008, RNF-022.
- **Criterios afectados.** CA-RNF-002-01, CA-RNF-002-02, CA-RNF-004-01, CA-RNF-005-01, CA-RNF-006-01, CA-RNF-006-02, CA-RNF-006-03, CA-RNF-006-04, CA-RNF-007-01, CA-RNF-007-02, CA-RNF-008-01, CA-RNF-022-01, CA-RNF-022-02.
- **Definiciones diferidas.** Límites de servicios, propiedad concreta, bloqueos, garantías locales y aplicabilidad de transacciones distribuidas por operación (I y extensiones en II y III según alcance).

### AMB-006 · Preferencias y arranque en frío

- **Contexto original.** Faltan las fuentes concretas de preferencias, las señales del historial, los pesos y el resultado para clientes nuevos.
- **Riesgo.** Un cliente nuevo puede no obtener sugerencias o recibir resultados poco pertinentes.
- **Resolución aprobada.** Usar categorías seleccionadas, historial de compras y reglas de puntuación explicables. Si faltan ambas señales, recomendar eventos populares o próximos. El alcance se satisface con reglas y no requiere modelos generativos ni servicios externos de IA.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** II. RF-018, RF-028, RNF-020, DF-006.
- **Criterios afectados.** CA-RF-018-01, CA-RF-018-02, CA-RF-028-01, CA-RF-028-02, CA-RNF-020-01.
- **Definiciones diferidas.** Pesos, reglas de puntuación y criterio de popularidad o proximidad (II).

### AMB-007 · Política y cierre de devoluciones

- **Contexto original.** Faltan elegibilidad, plazos, devolución parcial, efecto monetario simulado y si aprobación y confirmación son el mismo paso.
- **Riesgo.** Una reposición prematura puede revender un asiento antes del cierre. Reintentos pueden duplicar la reposición.
- **Resolución aprobada.** Solicitar devoluciones por ticket propio hasta 24 horas antes del inicio. El Administrador aprueba o rechaza. La aprobación confirma la devolución y simula el reintegro. La confirmación, la invalidación del ticket y la reposición exactamente una vez conforman una operación de negocio atómica e idempotente y conservan el historial. El rechazo deja activos el ticket y su inventario comprometido.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESUELTA.
- **Etapa y requisitos afectados.** III. RF-003, RF-015, RF-016, RF-017, RF-026, RN-005, RN-007, RN-010, RN-012, DF-005.
- **Criterios afectados.** CA-RF-015-01, CA-RF-015-02, CA-RF-015-03, CA-RF-016-01, CA-RF-016-02, CA-RF-017-01, CA-RF-017-02, CA-RF-026-03.
- **Definiciones diferidas.** Ninguna definición de política queda diferida en este registro.

### AMB-008 · Filtros y relevancia

- **Contexto original.** Faltan semántica al combinar filtros, significado de fecha, coincidencia textual y desempate.
- **Riesgo.** Distintas interpretaciones producen resultados u órdenes incompatibles.
- **Resolución aprobada.** Filtrar en PostgreSQL por tipo o categoría, organizador, participante y fecha. Combinar criterios distintos con AND, usar intervalos de fechas y coincidencias sin distinción de mayúsculas, minúsculas o acentos. Ordenar en PostgreSQL por relevancia ponderada y desempatar de forma determinista por fecha e identificador.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** III. RF-019, RNF-021, DF-008.
- **Criterios afectados.** CA-RF-019-01, CA-RF-019-02, CA-RF-019-03, CA-RNF-021-01.
- **Definiciones diferidas.** Pesos de relevancia y semántica de intervalos (III).

### AMB-009 · Orden, admisión y reintentos de cola

- **Contexto original.** Faltan activación de alta demanda, disciplina, unidad de capacidad, rechazo por saturación y tratamiento de reintentos y abandonos.
- **Riesgo.** Puede haber adelantamientos, solicitudes duplicadas o bloqueo de la cola por una solicitud fallida.
- **Resolución aprobada.** Activar manualmente por operación autorizada. Usar colas independientes con capacidad configurable y una entrada activa por Cliente y evento. Asignar turnos FIFO sin adelantamientos, con vencimiento configurable. Retornar HTTP 429 Too Many Requests al admitir nuevas solicitudes en una cola saturada. Notificar por SSE posición inicial y cambios, y aplicar backpressure. Distinguir el ingreso admitido, el turno, el procesamiento ordenado y la finalización. Las garantías de entrega de NATS requieren coordinación adicional para verificar el orden de negocio con varios consumidores.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** III. RF-020, RF-021, RF-022, RN-008, RN-013, RNF-011, DF-007.
- **Criterios afectados.** CA-RF-020-01, CA-RF-020-02, CA-RF-020-03, CA-RF-020-04, CA-RF-021-01, CA-RF-021-02, CA-RF-021-03, CA-RF-022-01, CA-RNF-011-01.
- **Definiciones diferidas.** Capacidades y tiempos, abandono y reintentos, coordinación y garantías de procesamiento ordenado con justificación frente a la consigna (III).

### AMB-010 · Objetivos y perfil de estrés

- **Contexto original.** Faltan métricas objetivo, umbrales, perfiles de carga y condiciones de infraestructura para la prueba.
- **Riesgo.** Un resultado puede resultar irrepetible o carecer de una condición clara de cumplimiento.
- **Resolución aprobada.** Fijar antes de ejecutar k6 la infraestructura, perfil, número de usuarios concurrentes, duración, tasa de solicitudes, umbrales y criterios de éxito y fracaso. Medir throughput, latencias p95 y p99, tasa de errores, longitud de cola, tiempo de espera e integridad del inventario bajo condiciones reproducibles.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** III. RNF-012, RNF-019, DF-009.
- **Criterios afectados.** CA-RNF-012-01, CA-RNF-019-01.
- **Definiciones diferidas.** Valores de la infraestructura, perfiles, carga, duración y umbrales antes de ejecutar las pruebas (III).

### AMB-011 · Medición HTTP y reconexión SSE

- **Contexto original.** La respuesta SSE queda abierta, mientras que la consigna pide registrar el resultado y el tiempo antes de retornar la respuesta. Falta definir qué duración se informa, cómo se recupera la posición y qué ocurre ante desconexión.
- **Riesgo.** Una medición puede confundir latencia de conexión con duración del canal. Una reconexión puede mostrar una posición desactualizada.
- **Resolución aprobada.** Medir desde la recepción de HTTP e incluir autenticación, autorización y procesamiento. Registrar con Pino el resultado y la duración hasta preparar y antes de retornar las respuestas convencionales, tanto exitosas como fallidas. Excluir contraseñas, tokens y demás credenciales sensibles. Para SSE, registrar la apertura y su latencia de aceptación, y registrar por separado el cierre y la duración del canal. Al reconectar, recuperar la posición actual.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESUELTA.
- **Etapa y requisitos afectados.** I, III. RF-022, RNF-009, RNF-013.
- **Criterios afectados.** CA-RNF-009-01, CA-RNF-009-02, CA-RNF-013-01, CA-RF-022-02.
- **Definiciones diferidas.** Ninguna definición de política queda diferida en este registro.

### AMB-012 · Política y fallo de caché

- **Contexto original.** Faltan catálogo de datos cacheables, ubicación, TTL, eventos de invalidación y comportamiento ante un fallo de caché.
- **Riesgo.** Puede mostrarse información obsoleta o filtrarse información personalizada entre clientes.
- **Resolución aprobada.** Usar Redis distribuido con Cache-Aside, TTL configurable por tipo de dato e invalidación mediante eventos de NATS cuando corresponda. Aislar los datos personalizados por identidad. Ante un fallo de Redis, podrá consultarse la fuente autorizada con límites protectores. Reservar y confirmar requiere comprobar el inventario transaccional en PostgreSQL. Evaluar y justificar la aplicabilidad de caché en cliente, CDN, servicio y distribución.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** II. RF-008, RF-010, RNF-010, RNF-018.
- **Criterios afectados.** CA-RNF-010-01, CA-RNF-010-02, CA-RNF-010-03, CA-RNF-010-04, CA-RNF-018-01, CA-RNF-018-02.
- **Definiciones diferidas.** Datos, ubicación, TTL, eventos de invalidación y degradación con límites para proteger al origen (II).

### AMB-013 · Datos mínimos de evento

- **Contexto original.** La consigna enumera criterios de búsqueda, pero deja abiertos catálogos, formatos de fecha y representación de organizadores y participantes.
- **Riesgo.** Si el Core modela datos insuficientes, recomendaciones y búsqueda requerirán completar o transformar eventos existentes.
- **Resolución aprobada.** Considerar desde el Core título, descripción, categoría, organizador, participantes, ubicación, fecha, zonas, asientos cuando correspondan, precios y estado para soportar los incrementos posteriores.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESOLUCIÓN APROBADA — DETALLES DIFERIDOS.
- **Etapa y requisitos afectados.** I. RF-005, RF-025, RF-018, RF-019.
- **Criterios afectados.** CA-RF-025-01.
- **Definiciones diferidas.** Representación de entidades, relaciones, identificadores, fechas y restricciones (I).

### AMB-014 · Alcance operativo del Administrador

- **Contexto original.** La consigna asigna gestión completa, mantenimiento y tareas operativas al Administrador. Falta un inventario de operaciones y el alcance de intervención sobre eventos publicados.
- **Riesgo.** Es posible omitir tareas esperadas o habilitar cambios sensibles sin una política clara.
- **Resolución aprobada.** Habilitar gestión de usuarios, roles y eventos, consulta de ventas e información operativa, y resolución de devoluciones en III. Proteger ventas confirmadas de ediciones directas que comprometan inventario o eliminen historial. Las devoluciones administrativas usan el flujo previsto. Incorporar capacidades conforme al alcance de cada etapa.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESUELTA.
- **Etapa y requisitos afectados.** I–III. RF-002, RF-003, RF-016, RF-026, RF-027, RF-029.
- **Criterios afectados.** CA-RF-026-01, CA-RF-026-02, CA-RF-026-03, CA-RF-027-03, CA-RF-029-02.
- **Definiciones diferidas.** Ninguna definición de política queda diferida en este registro.

### AMB-015 · Alcance de los mockups

- **Contexto original.** Falta definir pantallas, estados, recorridos y grado de interacción requerido para los mockups.
- **Riesgo.** La entrega puede omitir pasos que los responsables esperen revisar en forma visual.
- **Resolución aprobada.** Entregar mockups académicos de los tres roles con estados de éxito, error e indisponibilidad. Desarrollar además un frontend funcional únicamente en JavaScript con Vue 3 y Vite, conectado de manera incremental a las APIs. En el Core, cubrir registro, autenticación, listado, detalle, selección, reserva, compra y tickets del Cliente, gestión de eventos propios, zonas, asientos y publicación del Event Owner, y usuarios, roles, eventos, ventas e información operativa del Administrador. Añadir recomendaciones en II y búsqueda, devoluciones y posición en cola en III, incluida la resolución administrativa.
- **Aprobación.** Briant + Fernando, registrada en P-001.1.
- **Estado.** RESUELTA.
- **Etapa y requisitos afectados.** I–III. RF-027, RF-028, RF-029, RNF-023, EA-003, DF-010.
- **Criterios afectados.** CA-EA-003-01, CA-RF-027-01, CA-RF-027-02, CA-RF-027-03, CA-RF-027-04, CA-RF-028-01, CA-RF-028-02, CA-RF-029-01, CA-RF-029-02, CA-RF-029-03, CA-RNF-023-01.
- **Definiciones diferidas.** Ninguna definición de política queda diferida en este registro.

## 14. Entregables académicos

La consigna enumera estos entregables en la página 3. La matriz indica cuándo se incorporan y cómo se comprueban.

| ID | Entregable | Contenido requerido |
| --- | --- | --- |
| EA-001 | Guía de contribución | `CONTRIBUTE.md` documenta el flujo de trabajo con feature branches, tests y CI. Se incorpora desde el Core y se actualiza cuando cambia el flujo. |
| EA-002 | Changes archivados | Archivar el change de cada etapa con sus specs, design y tasks disponibles para revisión. |
| EA-003 | Mockups | Entregar mockups del Core en Vue para los tres roles, con estados de éxito, error e indisponibilidad según el recorrido (DF-010 y AMB-015). La integración del frontend funcional con APIs se comprueba mediante RF-027 a RF-029 y RNF-023. |
| EA-004 | APIs funcionales | Entregar APIs ejecutables del Core y extenderlas según las capacidades de los incrementos posteriores. |
| EA-005 | Tests de unidad | Entregar tests de unidad ejecutables sobre el alcance implementado en cada incremento. |
| EA-006 | Bitácora de documentación | Mantener la bitácora de desarrollo durante la preparación y las tres etapas. P-001.1 precisa su contenido y conservación debajo de esta tabla. |

### Contenido de la bitácora de desarrollo (EA-006)

Registrar cada interacción relevante con Codex para el trabajo de ingeniería asistido por IA, tanto si concluye con éxito como si necesita correcciones. Conservar el historial de prompts anteriores.

| Campo | Contenido |
| --- | --- |
| Identificación y contexto | Identificador del prompt, fecha de ejecución, fase y etapa, y objetivo. |
| Interacción completa | Texto completo del prompt enviado y respuesta obtenida. |
| Cambios y decisiones | Artefactos afectados, decisiones, observaciones y correcciones realizadas. |
| Verificación | Verificación humana y resultados de pruebas cuando correspondan. |
| Trazabilidad | Identificadores de prompts complementarios y commit asociado cuando exista. |

La bitácora de desarrollo documenta el proceso de ingeniería. La bitácora HTTP (RNF-009) registra de forma operativa las solicitudes, sus resultados y sus tiempos.

## 15. Glosario

| Término | Significado en este documento |
| --- | --- |
| Backpressure | Regulación de la entrada cuando el sistema procesa más lentamente de lo que recibe trabajo. |
| Cache-Aside | Patrón que consulta la caché y, si falta la entrada, obtiene el dato de la fuente y repuebla la caché según la política aplicable. |
| Change | Unidad de trabajo de OpenSpec con el ciclo indicado en RM-003. |
| Consistencia eventual | Convergencia de datos entre servicios después de procesar los cambios pendientes. |
| Evento listado | Evento publicado que figura en el catálogo del Cliente. Solo admite nuevas compras cuando la venta está habilitada. |
| Idempotencia | Propiedad por la que repetir una misma operación lógica conserva un solo efecto de negocio. |
| Inventario | Asientos y cupos por zona con estados que determinan su disponibilidad para una venta. |
| Reserva | Compromiso temporal de inventario hasta la confirmación de compra, la liberación o el vencimiento. |
| SSE | Canal HTTP mediante el que el servidor envía eventos al cliente. |
| Ticket activo | Entrada emitida por una compra confirmada cuyo derecho de uso sigue vigente. |
| Transactional Outbox | Patrón que guarda un evento pendiente en la misma transacción local que el cambio de negocio, para publicarlo de forma recuperable. |
| TTL | Vigencia de una entrada en caché antes de su vencimiento. |
| Zona numerada o no numerada | Sector que vende un asiento específico o consume una unidad de un cupo común, respectivamente. |

## 16. Referencias

- [Trabajo final de Tópicos II — 2026](sources/trabajo-final-topicos-ii-2026.pdf), páginas 1 a 3. Fuente académica de alcance, desafíos, metodología y entregables.
- [Bitácora P-001](bitacora/P-001.md), secciones «Prompt utilizado», «Stack tecnológico aprobado» y «Decisiones previamente aprobadas». Registro de las instrucciones y decisiones de los responsables para esta fase.
- [Bitácora P-001.1](bitacora/P-001.1.md), registro contextual del prompt de revisión y de las resoluciones aprobadas.


## 17. Definiciones diferidas para el diseño

Las políticas de la sección 13 están aprobadas. El diseño de cada etapa debe concretar los parámetros y contratos siguientes y usarlos en los criterios aplicables.

| Etapa | Definiciones | Referencias |
| --- | --- | --- |
| I · Core | Política de cancelación para eventos con ventas confirmadas, antes de habilitar dicha operación. Representación de entidades, relaciones, identificadores, fechas y restricciones de eventos. | AMB-002, AMB-013, RF-005 y RF-025. |
| I · Core | Tratamiento de solicitudes idempotentes en curso, respuesta almacenada y expiración de claves. Semántica de reintentos después de pagos rechazados, incluidos el alcance y la reutilización de claves conforme al contrato de idempotencia. | AMB-003, AMB-004, RF-011, RF-014 y RNF-003. |
| I · Core | Límites de servicios, propietarios y contratos de datos. Selección y justificación de bloqueos optimistas o pesimistas y de mecanismos transaccionales locales y distribuidos por operación. Precisar recuperación, reintentos, compensación y deduplicación cuando apliquen. Revisar impactos al incorporar consumidores en II y III. | AMB-005, RNF-002, RNF-004 a RNF-008 y RNF-022. |
| II · Recomendaciones y caché | Pesos y reglas del algoritmo, criterios de popularidad o proximidad y ejemplos de puntuación esperada. | AMB-006, RF-018 y RNF-020. |
| II · Recomendaciones y caché | Datos cacheables, ubicación, TTL por tipo, eventos de invalidación y degradación ante fallos de Redis. Precisar los límites de consultas al servicio propietario y justificar la aplicabilidad de cada ubicación de caché. | AMB-012, RNF-010 y RNF-018. |
| III · Escalabilidad | Pesos de relevancia, límites y semántica exacta de intervalos de fechas y sentido del desempate. | AMB-008, RF-019 y RNF-021. |
| III · Escalabilidad | Permiso concreto de activación, capacidad por evento, duración de turnos, abandonos y reintentos, coordinación entre consumidores y garantías observables de procesamiento ordenado. Justificar frente a la consigna el orden de procesamiento y su relación con ingreso, asignación y finalización. | AMB-009, RF-020 a RF-022, RN-008 y RNF-011. |
| III · Escalabilidad | Infraestructura, perfiles, usuarios concurrentes, duración, tasas de solicitudes, umbrales y criterios de éxito y fracaso. Fijarlos antes de ejecutar las pruebas de estrés. | AMB-010, RNF-012 y RNF-019. |
