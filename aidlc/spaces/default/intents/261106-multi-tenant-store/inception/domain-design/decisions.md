# Architecture Decision Records — Domain Design

> Fase: Inception / Domain Design. Fecha: 2026-11-12. Registro durable de las decisiones significativas de diseño de dominio. Cada ADR: Context, Decision, Consequences, Alternatives Rejected. Trazan a `components.md`, a los requisitos/historias y a las decisiones de Q1–Q10.

## ADR-001: Un componente de dominio por unidad de negocio del backend

- **Context:** Scope Definition aprobó **nueve unidades de construcción** (`tenants`, `auth`, `catalog`, `inventory`, `orders`, `payments`, `admin`, `storefront`, `frontend`); team-practices exige fronteras claras sin acoplamiento a BD entre unidades y resolución de `tenant_id` en un único punto. Había que traducir esas unidades a componentes de dominio con propiedad de entidades no ambigua, sin crear unidades nuevas.
- **Decision:** Se mantienen las **nueve unidades de construcción aprobadas**. De ellas se identifican **ocho componentes de dominio del backend** (`TenantComponent`, `AuthComponent`, `CatalogComponent`, `InventoryComponent`, `OrderComponent`, `PaymentComponent`, `StorefrontComponent`, `AdminComponent`), mientras que la novena unidad, `frontend`, es **consumidora de contratos** y no posee entidades de dominio del backend. `NotificationComponent` es una **capacidad lógica adicional**, NO una décima unidad de construcción ni de despliegue: su ubicación física dentro de las nueve unidades aprobadas se decidirá en Units Generation. `TenantContext` (resolución y validación de `tenant_id`) es una preocupación transversal, no un componente ni una unidad.
- **Consequences:** (+) la correspondencia con las nueve unidades aprobadas se preserva (ocho componentes de dominio backend + frontend consumidor), con fronteras limpias y cada entidad con un único dueño; (+) el aislamiento se concentra en TenantContext; (+) la capacidad lógica de notificaciones no infla el número de unidades. (−) más fronteras que coordinar por contrato; se acepta porque la cohesión por concern lo justifica.
- **Alternatives Rejected:** (B) agrupar/fusionar componentes (menos fronteras, pero mezcla concerns con ritmos de cambio distintos); (C) separar más (p. ej. pagos manual vs. pasarela) — rechazado en ADR-003; tratar `NotificationComponent` como una décima unidad — rechazado: es una capacidad lógica cuya ubicación física resuelve Units Generation sin alterar el conteo de unidades.

## ADR-002: Order y Payment como componentes separados (no fusionados)

- **Context:** Las historias refinadas separan la máquina de estados de la orden de la del pago, y exigen que un pago `rejected`/`failed` no cancele ni marque `paid` la orden (R2). Fusionarlos sería más simple pero acoplaría dos ciclos de vida distintos.
- **Decision:** `OrderComponent` posee `Order` (estados `pending_payment`/`paid`/`expired`/`cancelled`) y el snapshot; `PaymentComponent` posee `Payment`/`PaymentAttempt`/`TransferProof`. El pago referencia la orden por `orderId` y publica su resultado como **evento**; la orden reacciona idempotentemente. Ninguna máquina de estados vive en el otro componente.
- **Consequences:** (+) cada componente gobierna su propio invariante; la consistencia orden↔pago (I4) es explícita y verificable; aísla el concern de seguridad/PCI en pagos. (−) la coordinación es asíncrona y exige idempotencia cuidadosa; se asume como parte del riesgo R2 que el walking skeleton valida.
- **Alternatives Rejected:** (B) `OrderPaymentComponent` unificado — rechazado: acopla dos ciclos de vida con ritmos e invariantes distintos y dificulta aislar el concern de pagos.

## ADR-003: PaymentComponent unificado (manual + pasarela), no dividido

- **Context:** El MVP soporta pago manual por transferencia y pago por pasarela. Una opción era modelarlos como dos componentes.
- **Decision:** Un único `PaymentComponent` que posee el estado consolidado `Payment`, el historial `PaymentAttempt` (tanto de intentos de pasarela como de revisiones manuales) y `TransferProof`. El método (`transfer`/`gateway`) es un atributo, no una frontera de componente.
- **Consequences:** (+) un solo lugar deriva el estado consolidado del pago desde el historial de intentos (I5), sin duplicar la máquina de estados; (+) añadir/!cambiar pasarela (adaptador genérico) no crea un componente nuevo. (−) el componente concentra dos flujos; se mitiga con límites internos claros (adaptador de pasarela vs. revisión manual).
- **Alternatives Rejected:** dividir en `ManualPaymentComponent` + `GatewayPaymentComponent` — rechazado: fragmenta el estado consolidado y el historial de intentos, y duplica la lógica de consistencia orden↔pago.

## ADR-004: InventoryComponent dueño de la Reservation

- **Context:** La no-sobreventa (R1) y la liberación por vencimiento sin TTL son invariantes críticos. La reserva podía pertenecer a la orden o al inventario.
- **Decision:** `InventoryComponent` posee `Reservation` (con `expiresAt`), además de `StockItem` y `StockMovement`. `OrderComponent` crea/confirma/libera reservas **por contrato**, sin tocar el almacenamiento de inventario. La reserva referencia la orden por `orderId`.
- **Consequences:** (+) el garante del invariante de concurrencia es también el dueño del dato; reserva/confirmación/liberación atómicas e idempotentes en un solo lugar (I2, I3); (+) acota el riesgo R1 a un componente. (−) el checkout depende síncronamente de inventario para reservar; se acepta porque la reserva debe ser inmediata y consistente.
- **Alternatives Rejected:** (B) `OrderComponent` dueño de la reserva con inventario solo agregado — rechazado: separaría el invariante de no-sobreventa de su dato y abriría una superficie de sobreventa.

## ADR-005: Interacción mixta — síncrono para comandos/consultas, eventos para efectos desacoplados

- **Context:** team-practices prohíbe el acoplamiento a BD entre unidades y permite cruce por API o evento. Había que fijar un estilo por defecto por frontera y los mecanismos de consistencia que lo acompañan, sin fijar tecnologías concretas (mensajería, bus, almacenamiento).
- **Decision:** Modelo mixto, con mecanismos de consistencia precisados:
  - **Síncrono (por contrato, p. ej. REST/OpenAPI)** para las operaciones que requieren **respuesta inmediata**, en especial la **reserva de inventario durante el checkout** (y las consultas de catálogo/disponibilidad y las operaciones administrativas que necesitan confirmación en línea).
  - **Asíncrono (eventos)** para efectos desacoplados y de consistencia eventual (resultado de pago → orden, notificaciones). Los **eventos de pagos y órdenes deben tener un identificador único** y **procesamiento idempotente**.
  - Los **consumidores deben contemplar duplicados, reintentos y eventos fuera de orden**, aplicando siempre un **único efecto de negocio** por evento.
  - La **aprobación de un pago no puede reabrir una orden en estado terminal** (`expired`/`cancelled`) ni producir transiciones inválidas (ancla I4/R2).
  - Los **fallos de comunicación deben contemplar mecanismos de recuperación** (reintento/reproceso) **sin duplicar efectos de negocio**, apoyándose en la idempotencia y en el identificador único de evento.
  - El estilo por frontera se documenta en `components.md`.
- **Consequences:** (+) respuesta inmediata donde el usuario la necesita (reserva en checkout) y desacople donde la consistencia puede ser eventual; (+) la idempotencia y el identificador único de evento anclan R2 y permiten recuperación segura ante fallos. (−) dos mecanismos que mantener y una disciplina de idempotencia en cada consumidor; se asume por el beneficio de desacoplar pagos/notificaciones. (Las tecnologías concretas de mensajería/bus se fijan en diseño técnico/infraestructura, no aquí.)
- **Alternatives Rejected:** event-first en casi todo (añade latencia/complejidad a flujos que necesitan respuesta inmediata como la reserva); sync-first en casi todo (acopla order↔payment y notificaciones, contra R2).

## ADR-006: AdminComponent y super-admin sin componente propietario de entidades

- **Context:** El panel de comercio y el de super-admin necesitan vistas y acciones sobre varias entidades. Un `PlatformAdminComponent` propietario duplicaría propiedad.
- **Decision:** `AdminComponent` es un componente de **coordinación** que consume los contratos de los componentes propietarios y no posee entidades. Las operaciones de plataforma del super-admin (crear/suspender/reactivar/activar tiendas, plan/estado de pago) son **operaciones globales autorizadas** de `TenantComponent`; el soporte solo-lectura auditado es una capacidad transversal de autorización/auditoría.
- **Consequences:** (+) no se duplica propiedad de entidades; una sola fuente de verdad por dato; (+) la autorización por rol gobierna la visibilidad del panel. (−) el panel depende de muchos contratos; se acepta porque es inherente a su rol de composición.
- **Alternatives Rejected:** (B) `PlatformAdminComponent` propietario — rechazado: duplicaría la propiedad de `Store`/usuarios/pagos y rompería el invariante de propiedad única (I7).

## ADR-007: Entidades a nivel propiedad+forma; modelo físico diferido

- **Context:** La restricción C1/OQ2 y team-practices indican que el modelo de claves DynamoDB (single-table vs. otras opciones, PK/SK, índices) se valida en data-modelling. Domain Design no debe fijar el esquema físico.
- **Decision:** Capturar entidades solo a nivel de **propiedad y forma** (dueño, identificador, atributos, referencias). Los tipos, validaciones, cardinalidades y el modelo físico se definen en Functional Design / data-modelling.
- **Consequences:** (+) no se toman decisiones técnicas prematuras; el diseño de dominio permanece estable aunque cambie el modelo físico. (−) el esquema completo queda pendiente; se asume por diseño del proceso.
- **Alternatives Rejected:** fijar el esquema físico aquí — rechazado: decisión prematura que invadiría data-modelling y acoplaría el dominio al almacenamiento.

## ADR-008: Romper el ciclo síncrono Tenant↔Auth (autorización en el límite)

- **Context:** Un primer borrador declaró una dependencia síncrona bidireccional entre `TenantComponent` y `AuthComponent` (Tenant→Auth para "autorizar" y Auth→Tenant para "leer activación"). La regla de frontera afirmada del equipo exige que las dependencias vayan hacia unidades ya construidas y **nunca formen ciclos**; un ciclo síncrono entre los dos componentes base complica el orden de arranque/construcción y arriesga llamadas recíprocas en tiempo de ejecución. La revisión de arquitectura lo marcó (F1, Major).
- **Decision:** La **identidad, los permisos y el contexto de tenant se validan en el límite de confianza ANTES de ejecutar cualquier operación de negocio** (vía `TenantContext` y la base de autorización de `AuthComponent`); la autorización NO es una llamada saliente de `TenantComponent`. La única dirección síncrona del par es `AuthComponent → TenantComponent`, para **leer el estado de activación del tenant** como dato, de modo que el grafo síncrono queda acíclico. Reglas que acompañan la decisión:
  - Las **operaciones globales del super-admin** (crear/suspender/reactivar/activar tiendas, plan/estado de pago, métricas) deben contar con **autorización explícita, alcance limitado y auditoría**; son la única excepción al filtro de tenant.
  - **Ningún componente debe confiar únicamente en que una llamada procede de otro componente interno como prueba suficiente de autorización**: la autorización se verifica en el límite en cada operación, no se asume por el origen de la llamada (defensa en profundidad, coherente con mínimo privilegio).
  - La eliminación de ciclos **síncronos** NO autoriza a reintroducir **dependencias circulares implícitas mediante eventos**: la coordinación por evento Order↔Payment es intencional, acotada y de consistencia eventual, y Notification solo **consume** eventos de Order/Payment (relación unidireccional, sin arista de retorno); ninguna de estas relaciones debe ampliarse a lazos de realimentación que recreen un ciclo de hecho.
- **Consequences:** (+) grafo de dependencias síncronas acíclico, coherente con la regla de frontera; orden de construcción/arranque claro (Tenant es raíz, Auth lo lee); (+) el aislamiento multi-tenant (I1) y la autorización se concentran en el límite de confianza, no en la confianza entre componentes; (+) las operaciones globales quedan acotadas y auditadas. (−) la autorización deja de ser una responsabilidad "interna" de Tenant y pasa a ser una precondición del límite verificada en cada operación; se asume porque es exactamente donde el aislamiento y la seguridad deben vivir.
- **Alternatives Rejected:** mantener el ciclo síncrono bidireccional — rechazado por violar la regla de frontera "nunca ciclos" y crear acoplamiento recíproco entre los dos componentes base; sustituir el ciclo síncrono por un lazo de eventos recíproco entre Tenant y Auth — rechazado por recrear una dependencia circular implícita, solo que asíncrona.
