# Comunicación entre componentes

**Estado:** propuesta P-002. AMB-005, AMB-011 y AMB-012 son restricciones aprobadas. Véanse [límites](service-boundaries.md), [consistencia](consistency.md) y [ADR-005](adr/ADR-005.md).

## HTTP y decisiones inmediatas

| Flujo | Elección y fundamento | Fallo y repetición |
| --- | --- | --- |
| Web → Gateway → servicio | HTTP/REST para autenticación, lecturas con respuesta y comandos que devuelven una decisión inmediata. SSE para posición de cola en III. | Preservar resultados de negocio, correlación y errores normalizados. La Web puede volver a consultar después de una respuesta incierta. |
| Gateway o servicio → Identity | Emisión o consulta autenticada de atributos vigentes para el alcance definido en ADR-006. La firma del JWT se verifica localmente donde corresponda. | Presupuesto de tiempo por salto; las operaciones que requieren vigencia se deniegan temporalmente si no pueden verificarla. |
| Catalog → Ticketing | Comandos de preparación y de puerta de venta versionados. El resultado firme se necesita antes de avanzar la coordinación. | Un timeout no prueba que el comando falló. Consultar o repetir el mismo comando idempotente y mantener el cambio como pendiente hasta confirmar. |
| Web → Gateway → Waiting Room; Ticketing → Waiting Room | SSE vivo y consulta de posición actual; validación o emisión autenticada de prueba de turno, según diseño de III. | Manejar desconexión como evento normal. Reconsultar al reconectar. La API propietaria falla de forma cerrada si falta evidencia válida; el contrato deberá precisar vigencia y revocación. |

Proponer plazos por solicitud y por salto, propagando el presupuesto restante. Definir límites de conexión y lectura diferenciados. SSE tiene tiempo de aceptación y ciclo de conexión propios. Reintentar lecturas y comandos explícitamente idempotentes ante fallos transitorios, con límite, espera escalonada y dispersión. Separar rechazo de negocio, error de autenticación o autorización, saturación, dependencia no disponible y resultado incierto. El alcance, tiempos y códigos restantes se fijarán en OpenAPI con Zod para validación de bordes.

Las llamadas internas autentican al servicio emisor y verifican el usuario cuando corresponda. El Gateway descarta headers de identidad proporcionados por el cliente y propaga credenciales o un contexto autenticado verificable según ADR-006. Las redes internas limitan exposición y cada destino verifica confianza y permisos. Usar TLS en el entorno que cruza un límite de confianza según su configuración. Credenciales de servicio y JWT de usuario tienen alcances distintos.

## Eventos y recuperación

Elegir JetStream para notificar hechos confirmados a consumidores que pueden converger después, alimentar proyecciones e invalidar caché. Confirmar comandos que cambian inventario en Ticketing y registrar sus publicaciones con outbox. En I puede comprobarse la publicación de catálogo y compra con un consumidor de integración. El primer consumidor de producto de esos eventos se incorpora en II. Diseñar el arranque de II mediante replay suficiente o inicialización desde lecturas autorizadas con un punto de corte y reconciliación. La retención de JetStream se coordinará con esa recuperación y con los datos permitidos para replay.

| Evento conceptual | Productor | Consumidores | Propósito y etapa | Información mínima | Garantía requerida |
| --- | --- | --- | --- | --- | --- |
| EventoCreado | Catalog | Recommendations desde II | Inicializar la proyección desde I si se conserva el historial necesario. Puede integrarse en un evento general de revisión cuando simplifique el contrato. | Evento, revisión, estado y resumen público de atributos; ausencia de datos privados innecesarios. | Publicar desde outbox; deduplicar; aplicar la revisión de forma monotónica. |
| EventoActualizado | Catalog | Recommendations e invalidación aplicable desde II | Refrescar atributos o retirar candidatos cuyo estado deja de ser elegible. Emisión conceptual desde I. | Evento, revisión, estado comercial y estado de elegibilidad de lectura cuando aplique, categoría, atributos necesarios y fecha. | Entrega al menos una vez dentro del contrato de retención; conservar máxima revisión y reconciliar vacíos. |
| EventoPublicado | Catalog | Recommendations e invalidación aplicable desde II | Incorporar eventos publicados al catálogo derivado. Emisión conceptual desde I. | Evento, revisión y resumen público para la proyección. | Deduplicación y control de versiones. Su recepción habilita la lectura según política, mientras Ticketing conserva la autoridad de venta. |
| CompraConfirmada | Ticketing | Recommendations desde II | Alimentar historial y perfiles derivados; emisión desde I. | Compra, evento, referencia opaca del Cliente, instante y referencias de líneas o tickets para la reconciliación. | Misma transacción de negocio y outbox; recuperación tras indisponibilidad de NATS; efecto idempotente en el destino. |
| DevolucionConfirmada | Ticketing | Recommendations en III si reconcilia historial; consumidores operativos autorizados si se justifican | Reflejar la devolución conservando historial en III. | Devolución, compra, ticket, evento, referencia opaca y resultado comercial mínimo. | Deduplicar y resolver si llega antes de la proyección de compra; la regla de puntuación sigue diferida. |

UsuarioRegistrado y RolActualizado quedan como candidatos si aparece un consumidor que justifique distribuir identidades o invalidar proyecciones. La vigencia de permisos privilegiados no depende de recibir RolActualizado. InventarioPreparado y VentaHabilitada se resuelven inicialmente por el protocolo de comandos, con posible evento informativo para consumidores futuros. ReservaCreada, ReservaVencida y TicketEmitido carecen de consumidor separado en el alcance inicial; CompraConfirmada comunica la emisión necesaria a la proyección. TurnoAsignado puede permanecer local y notificarse por SSE, sujeto al diseño de coordinación de III. La emisión definitiva se decide por necesidad y exposición de datos.

## Sobre conceptual y entrega

| Campo | Significado |
| --- | --- |
| Event ID | Identificador estable asignado al hecho confirmado. Se reutiliza en reintentos de publicación y es clave de deduplicación del consumidor. |
| Event type y Event version | Hecho y versión compatible del contrato, independientes de la identificación del agregado. |
| Aggregate ID y revisión | Identificador estable y, si afecta orden de proyección, revisión de negocio monotónica. |
| Fecha de ocurrencia | Instante UTC del hecho que se persiste con el outbox. |
| Correlation ID | Relaciona comandos, transacciones y entregas del mismo flujo. |
| Causation ID | Referencia al comando o evento causante cuando existe. |
| Payload | Campos mínimos de dominio. Se excluyen credenciales y secretos. |

- El publicador persiste cambios y outbox en la misma transacción, publica después del commit y marca enviado al recibir confirmación. Un ACK de publicación confirma aceptación por el transporte según su configuración, no el efecto en un consumidor.
- La deduplicación de publicación de JetStream con identificador estable opera dentro de la ventana configurada. Un reenvío fuera de esa ventana o una reentrega pueden volver a presentar el hecho. El consumidor registra de forma persistente su clave y efecto en la misma transacción y confirma procesamiento después del commit. Reentregas, mensajes incompatibles y retrasos requieren observación y manejo recuperable.
- Para consumidores que deban reanudar tras caída, proponer suscripción durable y ACK explícito. La red, la concurrencia y los reintentos pueden alterar el orden percibido; las revisiones protegen proyecciones y las secuencias de negocio protegen la sala de espera. JetStream no garantiza por sí solo exactamente un efecto de negocio.
- Proponer cambios aditivos compatibles y ventanas de coexistencia cuando cambien contratos. Identificar responsables, validación de esquema y estrategia de reproceso antes de retirar versiones. En los changes se precisarán subjects, streams, consumidores, ventanas de deduplicación y retención, reentregas, esquema y permisos.
