# Propiedad de datos

**Estado:** arquitectura de referencia con los doce ADR aceptados por Briant + Fernando en P-003. DF-011 y AMB-005 fijan la separación y la propiedad. Los nombres de bases y la representación descrita se someten a revisión. Véanse [límites](service-boundaries.md) y [ADR-004](adr/ADR-004.md).

**Convención.** Inmediata significa que decide el dato confirmado por el propietario o una validación transaccional local. Eventual significa que la copia puede retrasarse dentro de su contrato. Las copias de lectura jamás habilitan o confirman una venta.

| Dato | Propietario | Fuente de verdad | Consumidores | Método de acceso | Consistencia requerida | Etapa |
| --- | --- | --- | --- | --- | --- | --- |
| Usuarios | Identity | `identity_db` | Gateway y servicios de negocio por referencia | REST de administración o identidad verificada | Inmediata para cambios y comprobaciones de cuenta | I |
| Credenciales | Identity | `identity_db` | Identity | Verificación local con bcrypt; autenticación por API | Inmediata | I |
| Roles | Identity | `identity_db` | Gateway y servicios | JWT más mecanismo de vigencia según ADR-006 | Inmediata en operaciones privilegiadas según propuesta de ADR-006 | I |
| Eventos y estado comercial | Catalog | `catalog_db` | Gateway, Ticketing, Recommendations | REST, comandos de ciclo comercial y proyecciones por eventos | Inmediata al modificar; eventual para recomendaciones | I |
| Propiedad de eventos | Catalog | `catalog_db` | Gateway y Ticketing | Comando autorizado y versionado desde Catalog o REST de Catalog para administración | Inmediata para autorizar gestión en Catalog; las revisiones de control se cercan en Ticketing | I |
| Categorías | Catalog | `catalog_db` | Recommendations y Web vía Gateway | REST y proyección de catálogo | Inmediata al administrar; eventual en recomendaciones | I |
| Organizadores | Catalog | `catalog_db` | Web vía Gateway y búsqueda Catalog | REST y consultas locales por criterio | Inmediata al modificar; resultados dentro de la revisión consultada | I, III |
| Participantes | Catalog | `catalog_db` | Web vía Gateway y búsqueda Catalog | REST y consultas locales por criterio | Inmediata al modificar | I, III |
| Fechas y ubicaciones | Catalog | `catalog_db` | Ticketing, Recommendations y Web | REST, eventos para lectura y comandos versionados para fecha operativa de venta y devolución | Inmediata en operaciones dependientes del horario; eventual en proyecciones | I |
| Zonas y modalidades | Ticketing | `ticketing_db` | Catalog para coordinar preparación; Web vía Gateway | Comandos autorizados y REST | Inmediata para compromiso de inventario | I |
| Asientos vendibles | Ticketing | `ticketing_db` | Web vía Gateway | Lectura REST orientativa; comando de reserva transaccional | Inmediata al reservar, comprar y devolver | I |
| Precios de venta | Ticketing | `ticketing_db` | Web vía Gateway; Catalog para composición si se requiere | REST para precio vigente y lectura de compra para precio histórico | Inmediata en la transición que fija el importe | I |
| Inventario y habilitación efectiva | Ticketing | `ticketing_db` | Catalog, Web y Waiting Room para coordinación de modo en III | Comandos y lecturas de diagnóstico; verificación local al reservar y confirmar | Inmediata; una lectura de disponibilidad externa es orientativa | I, III |
| Reservas y expiraciones | Ticketing | `ticketing_db` | Web vía Gateway | REST privado por titular o rol autorizado | Inmediata en las transiciones | I |
| Compras | Ticketing | `ticketing_db` | Web, análisis operativo y Recommendations | REST por permiso; CompraConfirmada para historial | Inmediata en venta; eventual en historial proyectado | I |
| Pagos simulados e importes históricos | Ticketing | `ticketing_db` | Web vía Gateway y lectura administrativa | Operación de venta y consulta autorizada local | Inmediata para efectos de compra | I |
| Tickets | Ticketing | `ticketing_db` | Web vía Gateway | Consulta del titular o administrativa autorizada | Inmediata para emisión, validez e invalidación | I, III |
| Devoluciones y reintegros simulados | Ticketing | `ticketing_db` | Web vía Gateway y Recommendations si reconcilia historial | REST por rol; DevolucionConfirmada para datos derivados | Inmediata para reintegro, historial e inventario; eventual para proyección | III |
| Preferencias | Recommendations | `recommendations_db` | Web vía Gateway | REST por identidad; invalidación de caché de esa identidad | Inmediata al guardar; refresco de resultado conforme a política de II | II |
| Historial de compras proyectado | Recommendations | `recommendations_db` para lectura derivada; compras originales en `ticketing_db` | Recommendations | JetStream, con copia local deduplicada | Eventual | II, III |
| Proyección de catálogo | Recommendations | `recommendations_db` para lectura derivada; catálogo original en `catalog_db` | Recommendations | Eventos de catálogo y método autorizado de recuperación | Eventual | II |
| Entradas en cola y posición | Waiting Room | `waiting_room_db` | Web vía Gateway | REST y SSE por identidad y evento; posición calculada sobre cola | Inmediata para admisión y capacidad; distribución SSE recuperable | III |
| Turnos | Waiting Room | `waiting_room_db` | Ticketing y Web vía Gateway | Interfaz autenticada de validación o prueba acotada; consulta propia | Inmediata al asignar y validar la política de acceso; Ticketing aplica su propio cerco y vigencia | III |
| Outbox | Cada productor | `catalog_db` o `ticketing_db` desde I; bases adicionales solo con una necesidad de publicar | Publicador del mismo servicio; JetStream recibe mensajes | Lectura y actualización local; confirmación de publicación | Cambio y outbox atómicos; envío posterior eventual | I–III |
| Registros de deduplicación | Ticketing para claves HTTP y cada consumidor para mensajes | Base propietaria del efecto, incluida `ticketing_db` y `recommendations_db` | Solamente su servicio | Consulta y restricción persistentes en transacción local, según el tipo | Inmediata para prevenir un segundo efecto | I–III |
| Caché distribuida | Servicio consumidor que gestiona cada espacio de claves | Redis almacena copias; la fuente de verdad es el servicio propietario | Servicios expresamente autorizados | Cache-Aside e invalidación conforme a la política | Eventual y descartable | II–III |

## Identidad, dinero y tiempo

- **Referencias.** Proponer identificadores opacos, inmutables y estables para identidades, eventos, asientos, reservas y tickets. Un servicio resuelve las referencias remotas mediante contratos o proyecciones autorizadas. Las foreign keys, modelos Sequelize y migraciones quedan dentro de una base y bajo el servicio propietario. Se excluyen lecturas y escrituras SQL directas sobre bases ajenas, foreign keys entre servicios y modelos Sequelize compartidos.
- **Importes.** Proponer monto entero en unidades menores y código de moneda explícito, con aritmética entera o decimal exacta y límites validados. Cada compra conserva su moneda, importe y precio unitario por línea como instantánea. Un reintegro simulado se basa en la compra original. La elección de fijar el importe al reservar o exigir una nueva conformidad antes de confirmar un cambio queda para diseño y revisión. En ambos casos, el importe aceptado por el Cliente coincide con el persistido y no se recalcula después de confirmar.
- **Tiempo.** Proponer instantes UTC con `timestamptz` e intercambio ISO 8601 con zona u offset explícitos. Guardar la zona IANA del evento para presentación, búsquedas y significado comercial local. Validar horas ambiguas o inexistentes al convertir. Las comparaciones de vigencia y del límite de devolución usan instantes y el reloj del propietario transaccional. El alcance de un intervalo de búsqueda y la política de reprogramación que afecte derechos existentes quedan para su diseño y, si modifican reglas de negocio, para validación del equipo.
