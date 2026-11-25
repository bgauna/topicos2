# Diagramas de arquitectura

Arquitectura de referencia con los doce ADR aceptados por Briant + Fernando en P-003. Las etapas indican incorporación prevista. Para responsabilidades y mecanismos véanse [límites](service-boundaries.md), [consistencia](consistency.md) y el [índice de ADR](adr/README.md).

## 1. Contexto del sistema

I–III. Cliente, Event Owner y Administrador usan los recorridos disponibles en cada etapa.

```mermaid
flowchart LR
  C["Cliente"] --> P["Ticketera virtual"]
  O["Event Owner"] --> P
  A["Administrador"] --> P
  P --> S["Simulación interna de pagos · I"]
  P --> R["Recomendaciones explicables · II"]
  P --> W["Venta masiva y devoluciones · III"]
```

## 2. Contenedores de la Etapa I

Core propuesto. Las flechas a PostgreSQL indican acceso exclusivo a la base identificada.

```mermaid
flowchart LR
  Web["Web · Vue"] -->|REST| G["API Gateway"]
  G --> I["Identity"]
  G --> C["Catalog"]
  G --> T["Ticketing"]
  G -. vigencia cuando se requiere .-> I
  C -->|"comandos comerciales"| T
  subgraph P["PostgreSQL"]
    ID[("identity_db")]
    CD[("catalog_db")]
    TD[("ticketing_db")]
  end
  I --> ID
  C --> CD
  T --> TD
  C -->|"outbox"| N["NATS JetStream"]
  T -->|"outbox"| N
```

## 3. Arquitectura final prevista

I forma el Core. II incorpora Recommendations y Redis. III incorpora Waiting Room y amplía Catalog y Ticketing.

```mermaid
flowchart LR
  Web["Web · I–III"] -->|"REST y SSE en III"| G["Gateway · I–III"]
  G --> I["Identity · I"]
  G --> C["Catalog · búsqueda en III"]
  G --> T["Ticketing · devoluciones en III"]
  C -->|"control comercial"| T
  I --> P["PostgreSQL · bases aisladas"]
  C --> P
  T --> P
  C -->|"outbox"| J["NATS JetStream · I"]
  T -->|"outbox"| J
  G --> R["Recommendations · II"]
  J -->|"proyecciones desde II"| R
  R --> P
  R --> K["Redis · II"]
  G -->|"REST y proxy SSE"| W["Waiting Room · III"]
  T -->|"validación de turno si aplica"| W
  W --> P
```

## 4. Propiedad de datos

Una instancia inicial contiene bases separadas. Las proyecciones tienen propietario local y origen remoto identificable.

```mermaid
flowchart TB
  I["Identity"] --> ID[("identity_db: usuarios, credenciales, roles")]
  C["Catalog"] --> CD[("catalog_db: eventos, propiedad, outbox")]
  T["Ticketing"] --> TD[("ticketing_db: puerta, inventario, reservas, compras, tickets, outbox")]
  R["Recommendations · II"] --> RD[("recommendations_db: preferencias, proyecciones, deduplicación")]
  W["Waiting Room · III"] --> WD[("waiting_room_db: entradas, turnos")]
  C -. eventos de catálogo .-> R
  T -. compra y devolución .-> R
  R --> K[("Redis: copias Cache-Aside · II")]
```

## 5. Despliegue local con Docker Compose

Topología conceptual. Bases y JetStream conservan volúmenes. Se incorporan Redis y servicios de II o III según el incremento.

```mermaid
flowchart TB
  U["Navegador"] --> Web["Publicación web"]
  U --> G["Gateway público"]
  subgraph Compose["Entorno local Compose"]
    Web
    G
    subgraph Privada["Red privada"]
      I["Identity"]
      C["Catalog"]
      T["Ticketing"]
      R["Recommendations · II"]
      W["Waiting Room · III"]
      P[("PostgreSQL")]
      J[("NATS JetStream")]
      K[("Redis · II")]
    end
    V1[("Volumen PostgreSQL")]
    V2[("Volumen JetStream")]
  end
  G --> I
  G --> C
  G --> T
  G --> R
  G --> W
  I --> P
  C --> P
  T --> P
  R --> P
  W --> P
  C --> J
  T --> J
  J --> R
  R --> K
  P --- V1
  J --- V2
```

## 6. Autenticación y acceso a recurso

I. Los detalles de caducidad y comprobación de rol se concretan en ADR-006 y el design correspondiente.

```mermaid
sequenceDiagram
  actor U as Usuario
  participant G as Gateway
  participant I as Identity
  participant D as Servicio propietario
  U->>G: Iniciar sesión
  G->>I: Enviar credenciales por canal protegido
  I->>I: Verificar cuenta y hash bcrypt
  I-->>G: Emitir JWT si autentica
  G-->>U: Retornar sesión
  U->>G: Solicitar recurso protegido
  G->>G: Verificar JWT y descartar headers no confiables
  G->>D: Enviar contexto verificable y credencial de servicio
  opt Operación requiere vigencia de rol
    D->>I: Comprobar privilegios vigentes
    I-->>D: Devolver estado verificable
  end
  D->>D: Autorizar rol y recurso
  D-->>G: Resultado o rechazo
  G-->>U: Respuesta
```

## 7. Creación, publicación y habilitación de evento

I. La preparación, publicación y apertura se coordinan por revisión. Ante incertidumbre, se consulta el estado del mismo comando.

```mermaid
sequenceDiagram
  actor O as Event Owner
  participant G as Gateway
  participant C as Catalog
  participant T as Ticketing
  participant P as ticketing_db
  O->>G: Crear evento propio
  G->>C: Crear evento
  C->>C: Verificar propiedad esperada y guardar DRAFT
  O->>G: Configurar inventario y solicitar publicación
  G->>C: Validar gestión del evento
  C->>T: Preparar inventario con comando versionado
  T->>P: Guardar preparación con venta cerrada
  alt Preparación confirmada
    T-->>C: Confirmar preparación
    C->>C: Guardar PUBLISHED y el progreso
    C->>T: Solicitar apertura de la revisión publicada
    T->>P: Verificar generación y condiciones, abrir
    T-->>C: Confirmar apertura o estado pendiente por reconciliar
    C-->>G: Estado confirmado conocido
  else Preparación no confirmada
    T-->>C: Fallo o resultado incierto
    C-->>G: Preparación pendiente o fallida, sin apertura
  end
  G-->>O: Respuesta
```

## 8. Reserva de asiento

I. Se comprueba la puerta y la exclusividad en la misma transacción de Ticketing.

```mermaid
sequenceDiagram
  actor C as Cliente
  participant G as Gateway
  participant T as Ticketing
  participant P as ticketing_db
  C->>G: Solicitar asiento
  G->>T: Reservar asiento para la identidad autenticada
  T->>P: Abrir transacción y proteger puerta y asiento
  alt Venta habilitada y asiento libre
    T->>P: Crear reserva exclusiva y fijar vencimiento
    P-->>T: Commit
    T-->>G: Reserva vigente
  else Puerta cerrada o asiento ocupado
    T->>P: Revertir
    T-->>G: Rechazo sin efectos
  end
  G-->>C: Resultado
```

## 9. Reserva por zona

I. El compromiso se actualiza de forma condicional y se conserva como máximo la capacidad.

```mermaid
sequenceDiagram
  actor C as Cliente
  participant G as Gateway
  participant T as Ticketing
  participant P as ticketing_db
  C->>G: Solicitar unidades de zona
  G->>T: Reservar cantidad
  T->>P: Proteger puerta y actualizar cupo si alcanza
  alt Actualización válida
    T->>P: Crear reserva y unidades asociadas
    P-->>T: Commit
    T-->>G: Reserva vigente
  else Puerta cerrada o cupo insuficiente
    T->>P: Revertir
    T-->>G: Rechazo sin compromiso
  end
  G-->>C: Resultado
```

## 10. Confirmación de compra

I. El simulador permanece dentro del límite Ticketing. La prueba de turno de III se añade a la guarda local.

```mermaid
sequenceDiagram
  actor C as Cliente
  participant G as Gateway
  participant T as Ticketing y simulador
  participant P as ticketing_db
  C->>G: Confirmar compra con clave y referencia de intento
  G->>T: Enviar operación autenticada
  T->>P: Verificar clave, puerta, reserva y resultado del pago
  alt Misma clave y contenido con compra completada
    T-->>G: Recuperar resultado autorizado de la compra
  else Intento nuevo aprobado y reserva vigente
    T->>P: Convertir compromiso, emitir tickets y guardar outbox
    P-->>T: Commit de compra única
    T-->>G: Compra confirmada y precio histórico
  else Conflicto o rechazo
    T-->>G: Rechazo sin segunda compra ni tickets
  end
  G-->>C: Resultado
```

## 11. Carrera entre confirmación y vencimiento

I. El ejemplo muestra los dos órdenes válidos sobre la misma reserva.

```mermaid
sequenceDiagram
  participant C as Confirmación en Ticketing
  participant P as ticketing_db
  participant E as Expirador de Ticketing
  alt Confirmación logra transición dentro de la vigencia
    C->>P: Bajo bloqueo, reservar a vendida y confirmar
    P-->>C: Commit de compra
    E->>P: Intentar expirar
    P-->>E: Transición no aplicable, sin liberar
  else Vencimiento logra primero la transición al vencer
    E->>P: Marcar vencida y liberar una vez
    P-->>E: Commit de expiración
    C->>P: Intentar confirmar
    P-->>C: Rechazo, sin consumo ni tickets
  end
```

## 12. Compra confirmada con NATS no disponible

I. La respuesta de compra refleja el commit local.

```mermaid
sequenceDiagram
  participant T as Ticketing
  participant P as ticketing_db
  participant O as Publicador de outbox
  participant J as NATS JetStream
  T->>P: Guardar compra e insertar evento pendiente
  P-->>T: Commit confirmado
  T->>T: Devolver éxito por compra confirmada
  O->>P: Obtener pendiente
  O-xJ: Intento con NATS no disponible
  O->>P: Conservar pendiente con mismo Event ID
  Note over O,J: Se restablece la conexión
  O->>J: Publicar con Event ID estable
  J-->>O: Confirmar publicación
  O->>P: Marcar enviado
```

## 13. Reentrega y consumo idempotente

II y III. La reentrega puede ocurrir después del commit local y antes de que JetStream reciba el ACK.

```mermaid
sequenceDiagram
  participant J as NATS JetStream
  participant R as Consumidor durable
  participant P as Base del consumidor
  J->>R: Entregar evento
  R->>P: Guardar deduplicación y aplicar efecto en una transacción
  P-->>R: Commit
  R-xJ: ACK perdido
  J->>R: Reentregar el mismo Event ID
  R->>P: Verificar ID persistido
  P-->>R: Ya aplicado, sin repetir efecto
  R-->>J: Confirmar procesamiento
```

## 14. Estados del evento

I. La puerta efectiva vive en Ticketing. La cancelación con ventas confirmadas queda diferida.

```mermaid
stateDiagram-v2
  [*] --> DRAFT
  DRAFT --> PUBLISHED: Preparación confirmada y publicación
  DRAFT --> CANCELLED: Cancelación autorizada sin ventas
  PUBLISHED --> FINISHED: Cierre efectivo y finalización comercial
  PUBLISHED --> CANCELLED: Cierre efectivo y política aplicable
  note right of PUBLISHED
    Venta efectiva requiere apertura
    de la revisión por Ticketing
  end note
  note right of CANCELLED
    Con ventas confirmadas se espera política
  end note
```

## 15. Estados de la reserva

I. Las transiciones terminales no son reversibles por expiraciones posteriores.

```mermaid
stateDiagram-v2
  [*] --> VIGENTE: Compromiso exclusivo
  VIGENTE --> VIGENTE: Pago rechazado durante vigencia
  VIGENTE --> CONFIRMADA: Pago aprobado y transición vigente
  VIGENTE --> VENCIDA: Expiración y reposición única
  CONFIRMADA --> [*]
  VENCIDA --> [*]
```

## 16. Estados de inventario numerado

I incluye reserva, venta y expiración. III incorpora reposición por devolución.

```mermaid
stateDiagram-v2
  [*] --> LIBRE
  LIBRE --> RESERVADO: Reserva exclusiva · I
  RESERVADO --> LIBRE: Vencimiento confirmado · I
  RESERVADO --> VENDIDO: Compra confirmada · I
  VENDIDO --> LIBRE: Devolución confirmada · III
```

## 17. Evolución hacia recomendaciones y colas

Evolución prevista. Los componentes de cada etapa se incorporan con su change.

```mermaid
flowchart LR
  subgraph I["I · Core"]
    A["Identidad y catálogo"]
    B["Venta con guardas locales"]
    E["Eventos de catálogo y compra"]
    B --> E
    A --> E
  end
  subgraph II["II · Recomendaciones y caché"]
    R["Catálogo e historial proyectados"]
    P["Preferencias y recomendaciones"]
    K["Redis Cache-Aside"]
    R --> P --> K
  end
  subgraph III["III · Escalabilidad"]
    W["Cola y turnos por evento"]
    S["Posición SSE"]
    Q["Procesamiento ordenado y guarda de turno"]
    D["Búsqueda y devoluciones"]
    W --> S
    W --> Q
  end
  E --> R
  B --> D
  Q --> B
```
