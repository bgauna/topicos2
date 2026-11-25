# Controles de auditoría arquitectónica

P-003 registra las observaciones de la revisión de P-002 para los diseños de las tres etapas. Complementan los [ADR aceptados](adr/README.md) y se trazan a los [requisitos](../requirements.md). Los controles son seguimiento de diseño. Su inclusión conserva los requisitos académicos, las decisiones aceptadas y las definiciones diferidas. Las nuevas políticas de negocio requieren aprobación expresa de Briant + Fernando.

Cada diseño debe indicar cómo aborda los controles de su etapa, qué evidencia deberá producir y qué aspectos siguen abiertos. La evidencia indicada es la esperada para el change correspondiente.

## Matriz de seguimiento

| Control | Título | Etapa | ADR | Estado |
| --- | --- | --- | --- | --- |
| [OBS-001](#obs-001) | Orden y estrategia de bloqueos | I | [ADR-007](adr/ADR-007.md), [ADR-011](adr/ADR-011.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-002](#obs-002) | Tiempo efectivo de vencimiento | I | [ADR-007](adr/ADR-007.md), [ADR-011](adr/ADR-011.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-003](#obs-003) | Idempotencia en procesamiento | I | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-004](#obs-004) | Precio de reserva y precio histórico | I | [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-005](#obs-005) | JWT y vigencia de privilegios | I | [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-006](#obs-006) | Cancelación de eventos | I | [ADR-001](adr/ADR-001.md), [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-007](#obs-007) | Inicialización y recuperación de proyecciones | II | [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-008](#obs-008) | Carreras de invalidación de caché | II | [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-009](#obs-009) | Procesamiento asíncrono de solicitudes | III | [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-010](#obs-010) | Alcance y vigencia del turno | III | [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-011](#obs-011) | Autenticación y reconexión SSE | III | [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md), [ADR-012](adr/ADR-012.md) | Pendiente de resolución en el diseño de la etapa correspondiente |
| [OBS-012](#obs-012) | Solicitudes de devolución duplicadas | III | [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md) | Pendiente de resolución en el diseño de la etapa correspondiente |

## Etapa I

### OBS-001

**Orden y estrategia de bloqueos**

- **Etapa.** I.
- **Riesgo asociado.** Las transacciones concurrentes pueden duplicar ventas, sobrepasar cupos o detenerse en deadlocks.
- **Requisitos y reglas de negocio relacionados.** RF-008, RF-012, RF-013, RNF-002, RN-001, RN-002, RN-009.
- **ADR relacionados.** [ADR-007](adr/ADR-007.md), [ADR-011](adr/ADR-011.md).
- **Aspecto que debe concretarse.** Precisar modos de bloqueo de PostgreSQL, nivel de aislamiento, orden de adquisición por recurso, condiciones de actualización y tratamiento de deadlocks, incluido el reintento seguro cuando corresponda.
- **Evidencia esperada.** Diseño con matriz de transacciones y bloqueos. Pruebas concurrentes contra PostgreSQL real para el último asiento, el último cupo y conflictos de bloqueo. Referencias: CA-RF-012-01, CA-RF-013-01, CA-RNF-002-01 y CA-RNF-002-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-002

**Tiempo efectivo de vencimiento**

- **Etapa.** I.
- **Riesgo asociado.** Una transacción que esperó por un bloqueo podría confirmar una reserva vencida o reponer inventario ya vendido.
- **Requisitos y reglas de negocio relacionados.** RF-009, RF-010, RNF-002, RN-004, RN-009.
- **ADR relacionados.** [ADR-007](adr/ADR-007.md), [ADR-011](adr/ADR-011.md).
- **Aspecto que debe concretarse.** Evaluar la vigencia con el tiempo efectivo de PostgreSQL después de adquirir los bloqueos aplicables. Especificar el instante de decisión y el resultado de la carrera entre confirmación y vencimiento. Un timestamp congelado al iniciar la transacción es insuficiente.
- **Evidencia esperada.** Secuencias y pruebas con esperas controladas en PostgreSQL real. Cubrir los dos órdenes de la carrera, una confirmación que llega al límite y una única reposición ante expiración repetida. Referencias: CA-RF-009-01, CA-RF-009-02 y CA-RF-009-04.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-003

**Idempotencia en procesamiento**

- **Etapa.** I.
- **Riesgo asociado.** Una interrupción puede dejar una clave ocupada indefinidamente o inducir una segunda compra al recuperarla.
- **Requisitos y reglas de negocio relacionados.** RF-011, RF-014, RNF-003, RN-003.
- **ADR relacionados.** [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md).
- **Aspecto que debe concretarse.** Definir estados en curso, recuperación tras interrupciones, plazos o criterios de recuperación y respuestas observables. Diferenciar repetición por transporte, operación completada y nuevo intento de pago. Preservar la unicidad persistente por reserva.
- **Evidencia esperada.** Contrato de estados y pruebas con interrupción durante el procesamiento, pérdida de respuesta después del commit, misma clave en paralelo y nuevo pago tras rechazo. Referencias: CA-RF-011-02, CA-RF-014-01 a CA-RF-014-06.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-004

**Precio de reserva y precio histórico**

- **Etapa.** I.
- **Riesgo asociado.** Un cambio de precio entre reserva y compra podría alterar el importe aceptado. Un cambio posterior podría alterar el historial.
- **Requisitos y reglas de negocio relacionados.** RF-008, RF-010, RF-017, RF-025, RN-009.
- **ADR relacionados.** [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md).
- **Aspecto que debe concretarse.** Definir cuándo se determina y congela el precio de la reserva y qué ocurre cuando el precio cambia antes de confirmar. Conservar importe, moneda y precio de línea históricos. Los cambios posteriores de precio mantienen intactas las compras confirmadas.
- **Evidencia esperada.** Decisión de precio documentada y recorrido con cambios antes y después de confirmar. Verificar el historial y reutilizar el precio original para el reintegro simulado al incorporar III. Referencias: CA-RF-010-01 y CA-RF-017-01.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-005

**JWT y vigencia de privilegios**

- **Etapa.** I.
- **Riesgo asociado.** Un JWT vigente podría conservar permisos obsoletos o dar acceso a una cuenta deshabilitada.
- **Requisitos y reglas de negocio relacionados.** RF-001, RF-003, RF-004, RF-023, RNF-013, RN-007.
- **ADR relacionados.** [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md).
- **Aspecto que debe concretarse.** Definir duración del JWT, tratamiento de cuentas deshabilitadas y cambios de rol, verificación de privilegios vigentes y comportamiento ante indisponibilidad de Identity. El backend valida permisos por rol y recurso con independencia de las vistas.
- **Evidencia esperada.** Contrato de autorización y pruebas HTTP con JWT anterior a un cambio de rol, cuenta deshabilitada, falla de Identity y acceso directo sin permiso. Referencias: CA-RF-003-01 a CA-RF-003-03, CA-RF-004-02 y CA-RF-023-01.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-006

**Cancelación de eventos**

- **Etapa.** I.
- **Riesgo asociado.** Cancelar con ventas confirmadas podría reabrir cupo vendido, borrar historial o dejar compradores sin tratamiento explícito.
- **Requisitos y reglas de negocio relacionados.** RF-005, RF-010, RF-026, RN-011.
- **ADR relacionados.** [ADR-001](adr/ADR-001.md), [ADR-004](adr/ADR-004.md), [ADR-007](adr/ADR-007.md).
- **Aspecto que debe concretarse.** Definir y obtener la aprobación humana de la política de cancelación con ventas confirmadas antes de habilitar dicha operación. Precisar el cierre efectivo, los efectos autorizados y la conservación del historial. Cancelar por sí solo conserva el inventario vendido y los registros históricos.
- **Evidencia esperada.** Política aprobada y escenarios de cierre concurrente y cancelación con ventas, que comprueben rechazo o efectos expresamente autorizados, sin reposiciones automáticas. Referencias: AMB-002, CA-RF-005-02 y CA-RF-026-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

## Etapa II

### OBS-007

**Inicialización y recuperación de proyecciones**

- **Etapa.** II.
- **Riesgo asociado.** Una proyección nueva o recuperada podría omitir compras o duplicar efectos si el historial de JetStream es parcial.
- **Requisitos y reglas de negocio relacionados.** RF-018, RNF-006, RNF-007, RNF-020, RNF-022, RN-003.
- **ADR relacionados.** [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md).
- **Aspecto que debe concretarse.** Definir inicialización con datos previos, punto de corte y reconciliación con la fuente autorizada. Precisar retención, replay, reentregas, deduplicación y reconstrucción. La retención de NATS es acotada por la configuración que se acuerde.
- **Evidencia esperada.** Recorrido desde una base vacía con compras previas y mensajes fuera de la retención. Interrumpir, reconstruir y reconciliar sin duplicados. Referencias: CA-RNF-007-01, CA-RNF-007-02 y CA-RNF-022-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-008

**Carreras de invalidación de caché**

- **Etapa.** II.
- **Riesgo asociado.** Una consulta concurrente puede volver a poblar Redis con una versión anterior después de invalidarla, o sobrecargar el origen ante un fallo.
- **Requisitos y reglas de negocio relacionados.** RNF-010, RNF-018, RN-001, RN-002, RN-009.
- **ADR relacionados.** [ADR-004](adr/ADR-004.md), [ADR-005](adr/ADR-005.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md).
- **Aspecto que debe concretarse.** Definir cómo impedir o acotar la repoblación obsoleta. Cubrir escrituras concurrentes, invalidaciones fuera de orden y degradación ante caída de Redis. PostgreSQL conserva la autoridad para reservar y confirmar.
- **Evidencia esperada.** Pruebas con actualización entre lectura y repoblación, invalidaciones reordenadas, Redis caído y compra con caché desactualizada. Comprobar los límites acordados. Referencias: CA-RNF-010-01, CA-RNF-010-03, CA-RNF-018-01 y CA-RNF-018-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

## Etapa III

### OBS-009

**Procesamiento asíncrono de solicitudes**

- **Etapa.** III.
- **Riesgo asociado.** La pérdida o reentrega de trabajo puede avanzar indebidamente la cola, repetir efectos o exceder la capacidad.
- **Requisitos y reglas de negocio relacionados.** RF-020, RF-021, RNF-011, RNF-022, RN-008, RN-013.
- **ADR relacionados.** [ADR-005](adr/ADR-005.md), [ADR-007](adr/ADR-007.md), [ADR-009](adr/ADR-009.md), [ADR-011](adr/ADR-011.md).
- **Aspecto que debe concretarse.** Definir el transporte persistente de solicitudes de trabajo de venta y distinguirlo de los eventos de dominio. Especificar confirmaciones, reintentos, recuperación, backpressure y garantías de orden con varios consumidores. Conservar la distinción entre FIFO de turnos y procesamiento ordenado.
- **Evidencia esperada.** Contrato de trabajo y pruebas de persistencia, reentrega, caída de consumidor, saturación y secuencias observables con varios consumidores. Referencias: CA-RF-021-01, CA-RF-021-03 y CA-RNF-011-01.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-010

**Alcance y vigencia del turno**

- **Etapa.** III.
- **Riesgo asociado.** Un turno ajeno, vencido o reutilizado podría permitir reservas o compras indebidas por vías directas.
- **Requisitos y reglas de negocio relacionados.** RF-008, RF-010, RF-020, RF-021, RN-008, RN-013.
- **ADR relacionados.** [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-007](adr/ADR-007.md).
- **Aspecto que debe concretarse.** Definir si un turno autoriza una o varias reservas o compras. Vincularlo con cliente, evento, vigencia y contexto de alta demanda. En diseño, precisar reuso, expiración y comprobación en las APIs propietarias de reserva y confirmación.
- **Evidencia esperada.** Contrato de turno y pruebas directas de API con turno ausente, ajeno, expirado o fuera de su alcance. Comprobar la concurrencia y la transición de activación. Referencias: CA-RF-020-04 y CA-RF-021-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-011

**Autenticación y reconexión SSE**

- **Etapa.** III.
- **Riesgo asociado.** Un canal podría revelar posición o credenciales, o continuar con identidad vencida y una posición obsoleta.
- **Requisitos y reglas de negocio relacionados.** RF-003, RF-022, RF-023, RF-029, RNF-009, RNF-013, RN-007.
- **ADR relacionados.** [ADR-005](adr/ADR-005.md), [ADR-006](adr/ADR-006.md), [ADR-010](adr/ADR-010.md), [ADR-012](adr/ADR-012.md).
- **Aspecto que debe concretarse.** Definir autenticación de SSE desde Vue, protección de credenciales fuera de URLs, expiración de identidad, autorización al reconectar y recuperación de la posición actual.
- **Evidencia esperada.** Contrato de conexión y pruebas con expiración, reconexión tras cambios de posición y revisión de URLs y logs de apertura y cierre. Referencias: CA-RF-022-01, CA-RF-022-02, CA-RNF-009-02 y CA-RNF-013-01.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.

### OBS-012

**Solicitudes de devolución duplicadas**

- **Etapa.** III.
- **Riesgo asociado.** Peticiones pendientes incompatibles y aprobaciones concurrentes podrían duplicar el reintegro o liberar inventario varias veces.
- **Requisitos y reglas de negocio relacionados.** RF-015, RF-016, RF-017, RN-005, RN-010, RN-012.
- **ADR relacionados.** [ADR-007](adr/ADR-007.md), [ADR-008](adr/ADR-008.md), [ADR-009](adr/ADR-009.md).
- **Aspecto que debe concretarse.** Definir cómo impedir solicitudes pendientes duplicadas o incompatibles para el mismo ticket y qué resultado ve el Cliente al repetir. Especificar la guarda para aprobaciones concurrentes y la unicidad del reintegro y de la reposición.
- **Evidencia esperada.** Pruebas concurrentes para solicitudes sobre un mismo ticket y aprobaciones repetidas. Comprobar un único reintegro y una única reposición, junto con validez del historial. Referencias: CA-RF-015-01, CA-RF-017-01 y CA-RF-017-02.
- **Estado.** Pendiente de resolución en el diseño de la etapa correspondiente.
