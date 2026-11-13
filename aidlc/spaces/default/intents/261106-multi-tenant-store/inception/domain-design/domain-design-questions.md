# Domain Design — Plan y preguntas de diseño de dominio

## Sources

- [desc] Initial description: "Dividir el MVP en unidades independientes; aislamiento estricto entre tiendas; catálogo/inventario/carrito/checkout/órdenes/pagos; panel de administración; staff por tienda y clientes."
- [scope] Workflow-selected scope: `feature`.

---

> Objetivo de esta etapa: identificar los **componentes lógicos** (los bloques de software que escribiremos), con sus responsabilidades, la propiedad de cada entidad (una entidad = un componente dueño), las interacciones entre componentes y los invariantes de dominio clave. **No** se decide aquí el topología de despliegue (eso es Units Generation), ni el stack, ni los patrones NFR.
>
> Punto de partida ya acordado upstream (no se re-pregunta, solo se confirma al final): Scope Definition definió **9 unidades** (tenants, auth, catalog, inventory, orders, payments, admin, storefront, frontend) y team-practices fijó reglas de frontera (sin acoplamiento a BD entre unidades; `tenant_id` resuelto en un único punto transversal; dependencias alineadas al orden de construcción; contratos OpenAPI entre unidades). Estas preguntas afinan cómo se traduce eso en componentes de dominio y resuelven las fronteras con más de una opción viable.
>
> Responde cada pregunta en su línea `[Answer]:`. Puedes elegir una opción y matizarla; si prefieres que decida yo, dilo ahí.

## Q1. Correspondencia unidades → componentes de dominio

Las 9 unidades de Scope Definition son la base. ¿Cómo quieres mapearlas a componentes de dominio?

- A. Un componente de dominio por unidad de negocio del backend, con nombres explícitos: `TenantComponent`, `AuthComponent`, `CatalogComponent`, `InventoryComponent`, `OrderComponent`, `PaymentComponent`, `AdminComponent`, `StorefrontComponent`; el `frontend` se trata como consumidor de contratos (no como componente de dominio con entidades propias). La resolución de `tenant_id` es una **preocupación transversal** (`TenantContext`), no una entidad de negocio.
- B. Agrupar algunos (p. ej. fusionar admin dentro de cada dominio) para menos componentes.
- C. Separar más (p. ej. dividir pagos en manual vs. pasarela como componentes distintos).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Mantener la correspondencia con las nueve unidades definidas en Scope Definition. Los componentes lógicos de dominio del backend serán TenantComponent, AuthComponent, CatalogComponent, InventoryComponent, OrderComponent, PaymentComponent, AdminComponent y StorefrontComponent. El frontend será consumidor de contratos, sin propiedad de entidades de dominio del backend.La resolución y validación de tenant_id será una preocupación transversal (TenantContext), no una entidad ni un componente de negocio independiente.Se permite identificar capacidades lógicas internas adicionales, como notificaciones, sin que esto implique crear nuevas unidades de construcción o despliegue. La correspondencia entre componentes lógicos y unidades se mantendrá alineada con las nueve unidades aprobadas.

## Q2. Propiedad de la entidad Order y de la entidad Payment (frontera orders ↔ payments)

Las historias separan la máquina de estados de **orden** (`pending_payment`, `paid`, `expired`, `cancelled`) de la de **pago** (`pending`, `pending_review`, `approved`, `rejected`, `failed`), y los **intentos** de pago del estado consolidado. ¿Cómo repartimos la propiedad?

- A. `OrderComponent` posee la entidad `Order` (y su máquina de estados de orden); `PaymentComponent` posee `Payment` (estado consolidado del pago), `PaymentAttempt` (historial de intentos) y `TransferProof` (comprobante). El pago referencia a la orden por `orderId`; `PaymentComponent` consume/produce eventos hacia `OrderComponent` (no acoplamiento directo a datos). Ninguna de las dos máquinas de estados vive en el otro componente.
- B. Un único `OrderPaymentComponent` que posea orden y pago juntos (más simple, menos fronteras, pero acopla dos ciclos de vida distintos).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. OrderComponent será propietario de la entidad Order y de su máquina de estados (pending_payment, paid, expired, cancelled). PaymentComponent será propietario de Payment (estado consolidado), PaymentAttempt (historial individual de intentos) y TransferProof (comprobante de transferencia).El pago referenciará la orden mediante orderId, sin acceder directamente a los datos internos de OrderComponent. Cada componente será responsable exclusivamente de las transiciones de estado de sus propias entidades.La comunicación entre ambos se realizará mediante contratos explícitos y eventos o comandos idempotentes. Se deberá preservar la consistencia del flujo: un pago rejected o failed no cancela automáticamente la orden ni la marca como paid; una aprobación válida puede generar la transición de la orden a paid, siempre que se cumplan las reglas de vigencia y consistencia. Los intentos individuales deberán conservar su historial, sin que un intento fallido sobrescriba el resultado de otro intento posterior.

## Q3. Propiedad del stock y la reserva (frontera inventory ↔ orders)

La reserva de stock (R1, no sobreventa; liberación por vencimiento sin TTL) es un invariante crítico. ¿Quién posee la reserva?

- A. `InventoryComponent` posee `Product`, `Variant/SKU`, `StockItem`, `StockMovement` **y** `Reservation` (con su `expiresAt`), porque el invariante de no-sobreventa y la liberación por vencimiento son responsabilidad del inventario; `OrderComponent` solicita/confirma/libera reservas **a través del contrato** de inventario, sin tocar su almacenamiento. La reserva referencia la orden por `orderId`.
- B. `OrderComponent` posee la `Reservation` (la reserva es parte del ciclo de la orden) e `InventoryComponent` solo lleva el stock agregado.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. InventoryComponent será propietario de Product, Variant/SKU, StockItem, StockMovement y Reservation, incluyendo su estado, cantidad reservada, referencia a orderId y expiresAt.La responsabilidad de garantizar la no sobreventa, controlar la disponibilidad y ejecutar la liberación explícita de reservas vencidas corresponde a InventoryComponent.
OrderComponent solicitará la creación, confirmación o liberación de reservas mediante contratos de inventario, sin acceder directamente a su almacenamiento. Las operaciones de reserva, confirmación y liberación deberán ser atómicas e idempotentes, evitando dobles liberaciones, consumos duplicados o confirmaciones de reservas vencidas.
La reserva se mantendrá asociada a la orden, pero su propiedad y ciclo de vida pertenecerán a inventario.

## Q4. Catálogo vs. storefront (frontera catalog ↔ storefront)

El catálogo se administra en el panel y se muestra en la tienda pública. ¿Cómo separamos?

- A. `CatalogComponent` posee las entidades de catálogo (`Product`, `Category`, `Variant`, imágenes como referencia a object-store) y expone lectura/escritura administrativa; `StorefrontComponent` es la capa de presentación pública (resuelve tienda por subdominio, personalización/plantilla) y **consume** el catálogo por contrato, sin poseer entidades de producto. `StorefrontComponent` posee solo la configuración de presentación (`StoreTheme`/tokens de marca).
- B. Fusionar catálogo y storefront en un solo componente.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. CatalogComponent será propietario de las entidades Product, Category y Variant, así como de las referencias a las imágenes almacenadas en el servicio de objetos. Expondrá las operaciones de administración y consulta del catálogo mediante contratos.StorefrontComponent será responsable de la presentación pública de cada tienda, la resolución de la tienda mediante su subdominio, la selección de la plantilla y la aplicación de la personalización visual. Será propietario únicamente de la configuración de presentación, como StoreTheme y los tokens de marca.El storefront consumirá la información del catálogo mediante contratos, sin duplicar ni asumir la propiedad de las entidades de producto. La disponibilidad de inventario también deberá consultarse a través del contrato correspondiente de InventoryComponent.

## Q5. Propiedad de la configuración de impuestos y de pagos de la tienda

Los mockups restringen la config sensible (pasarela, credenciales, datos bancarios, impuestos) al Owner/Admin (P14). ¿Dónde viven esas entidades de configuración?

- A. `TenantComponent` posee la configuración general de la tienda (`Store`, datos básicos, estado de activación, plan/estado de pago de suscripción) y la **configuración tributaria** (`TaxConfig` por producto/categoría); `PaymentComponent` posee la **configuración de métodos de pago** de la tienda (`PaymentMethodConfig`: datos de transferencia, conexión/credenciales de pasarela — las credenciales como referencia a un secreto, nunca el secreto en claro). El checkout (`OrderComponent`) y el storefront leen estas configs por contrato.
- B. Un `StoreConfigComponent` separado que posea toda la configuración (impuestos + pagos + marca).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. TenantComponent será propietario de la configuración general de la tienda (Store), sus datos básicos, estado de activación y la información del plan y estado de pago de la suscripción. También será propietario de la configuración tributaria (TaxConfig) asociada a productos y categorías.PaymentComponent será propietario de la configuración de métodos de pago de cada tienda (PaymentMethodConfig), incluyendo los datos de transferencia y la configuración de conexión con pasarelas. Las credenciales sensibles se manejarán exclusivamente mediante referencias a secretos protegidos, nunca almacenando valores secretos en claro dentro de las entidades de configuración.OrderComponent utilizará las configuraciones tributarias necesarias para calcular y conservar el snapshot de la orden, mientras que el checkout y StorefrontComponent consumirán únicamente la información necesaria mediante contratos.La autorización para consultar o modificar configuraciones sensibles corresponderá al perfil Owner/Admin, aplicando las reglas de autorización y aislamiento definidas en AuthComponent.

## Q6. Usuarios, roles y aislamiento (componente auth)

¿Cómo modelamos identidad, roles y el aislamiento multi-tenant a nivel de componente?

- A. `AuthComponent` posee `UserIdentity` (Owner/Admin, staff de catálogo/inventario, staff de órdenes, cliente comprador, super-admin) y la asignación de roles por tienda; la **identidad Owner/Admin** se crea al registrarse (separada del estado de activación del tenant, que es de `TenantComponent`). La resolución y validación de `tenant_id` es un **servicio transversal** (`TenantContext`) usado por todos los componentes en el límite, no una entidad; el comprador invitado es una modalidad de checkout, no una identidad con credenciales.
- B. Modelar el comprador invitado como una entidad/rol propio.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. AuthComponent será propietario de UserIdentity y de las asignaciones de roles y permisos por tienda. Se contemplarán los cinco perfiles funcionales aprobados: super_admin, owner_admin, catalog_inventory_staff, orders_staff y customer.La identidad del Owner/Admin se creará durante el registro, independientemente del estado de activación de la tienda, cuya administración corresponde a TenantComponent.La resolución y validación de tenant_id se implementará como una capacidad transversal (TenantContext) utilizada en los límites de acceso de todos los componentes. Ninguna operación de un usuario de una tienda podrá consultar o modificar información de otra tienda, salvo las operaciones globales expresamente autorizadas para el super-admin.El comprador invitado será una modalidad de checkout sin cuenta ni credenciales propias, no una identidad ni un rol adicional. El mecanismo seguro para consultar posteriormente sus órdenes permanecerá sujeto a OQ-US-1.

## Q7. Notificaciones: ¿componente propio o capacidad transversal?

Las notificaciones transaccionales (confirmación de orden, resultado de pago, aviso de comprobante pendiente) son Should, salvo el aviso de comprobante que es parte del flujo manual Must.

- A. `NotificationComponent` como componente de dominio ligero que **reacciona a eventos** de orden/pago (consume eventos, no es llamado síncronamente) y posee solo el registro mínimo de notificación enviada; así las notificaciones no acoplan a orders/payments.
- B. Tratar las notificaciones como una capacidad interna de order/payment (sin componente propio).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Definir NotificationComponent como un componente lógico ligero, orientado a eventos, que reaccione a eventos de órdenes y pagos para enviar notificaciones transaccionales sin acoplar directamente los ciclos de vida de OrderComponent y PaymentComponent.Será responsable del procesamiento de notificaciones, su estado de envío y el registro mínimo necesario para trazabilidad e idempotencia. No será propietario de órdenes, pagos ni datos de clientes.Este componente lógico no implica crear una décima unidad de construcción o despliegue: su ubicación física dentro de las nueve unidades aprobadas se definirá en Units Generation, manteniendo las fronteras y dependencias existentes.Se respetará la prioridad funcional ya establecida: los avisos indispensables para el flujo de comprobantes manuales son Must. Las demás notificaciones transaccionales conservarán la prioridad definida en el alcance aprobado, sin ampliar automáticamente el MVP.

## Q8. Super-admin / administración de plataforma: ¿componente propio?

El super-admin administra tiendas (crear/suspender/reactivar/activar), plan/estado de pago y métricas, con soporte solo-lectura auditado.

- A. No crear un componente nuevo: las operaciones de plataforma del super-admin son **operaciones globales autorizadas** de `TenantComponent` (gestión del ciclo de vida de `Store`) más vistas de métricas agregadas; el soporte solo-lectura auditado es una capacidad transversal de autorización/auditoría. Evita un componente que duplique propiedad de entidades.
- B. Crear un `PlatformAdminComponent` separado que posea las operaciones de super-admin.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. No crear un nuevo componente propietario de entidades para el super-admin. Las operaciones globales sobre el ciclo de vida de las tiendas (crear, activar, suspender y reactivar) corresponderán a operaciones autorizadas de TenantComponent, que conserva la propiedad de Store.AdminComponent actuará como componente de coordinación de las operaciones administrativas y de las vistas de gestión, consumiendo los contratos de los componentes propietarios. No duplicará entidades ni asumirá la propiedad de usuarios, pagos, configuraciones tributarias u órdenes.La gestión de identidades y permisos corresponderá a AuthComponent; la configuración de métodos de pago, a PaymentComponent; y la información de planes y estados de suscripción, a TenantComponent.Las operaciones de soporte del super-admin serán de solo lectura, estarán limitadas a la autorización global correspondiente y deberán quedar auditadas. Las métricas agregadas se construirán mediante consultas o contratos autorizados, sin habilitar acceso indiscriminado a datos de las tiendas.

## Q9. Interacción entre componentes: síncrono vs. eventos

team-practices fija "sin acoplamiento a BD entre unidades; cruce por contrato (API interna o evento)". ¿Qué estilo por defecto quieres que proponga en el diseño?

- A. **Síncrono (REST por contrato OpenAPI)** para consultas y comandos que necesitan respuesta inmediata (checkout consulta catálogo/inventario, reserva de stock, aprobación de pago manual); **asíncrono (eventos)** para efectos desacoplados y de consistencia eventual (orden creada/pagada → notificación; resultado de pago → actualización de orden; idempotencia de webhooks). Se documenta el estilo por frontera en `components.md` y las decisiones en ADRs.
- B. Preferir eventos en casi todo (event-driven primero).
- C. Preferir síncrono en casi todo (menos eventos).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Adoptar un modelo mixto, utilizando comunicación síncrona mediante contratos explícitos (API interna/OpenAPI) para consultas y comandos que requieren respuesta inmediata, y comunicación asíncrona mediante eventos para efectos desacoplados y procesos que admiten consistencia eventual.Se utilizará comunicación síncrona, entre otros casos, para consultar catálogo y disponibilidad durante el checkout, solicitar reservas de inventario y validar operaciones administrativas que requieren una respuesta inmediata.Se utilizarán eventos para notificaciones transaccionales, registro de resultados de procesamiento y comunicación entre los componentes de órdenes y pagos cuando corresponda.La aprobación de pagos deberá coordinar las transiciones de Payment y Order sin permitir estados contradictorios. Los comandos y eventos relacionados deberán ser idempotentes, contemplar duplicados y eventos tardíos, y evitar que una orden en estado terminal sea reabierta automáticamente.La definición detallada de contratos, eventos, responsabilidades de publicación y consumo, y estrategias de recuperación quedará documentada en components.md y en los ADR correspondientes. No se permitirá acoplamiento directo a bases de datos entre unidades.

## Q10. Invariantes de dominio a declarar explícitamente

¿Confirmas que el diseño debe declarar como invariantes de dominio (reglas que el modelo garantiza), para anclar R1–R3 y la consistencia?

- A. Sí, declarar explícitamente: (i) **aislamiento multi-tenant** — ninguna operación cruza `tenant_id` salvo operación global autorizada del super-admin (R3); (ii) **no sobreventa** — la suma de reservas activas + salidas nunca excede el stock (R1); (iii) **liberación por vencimiento explícita** sin depender de TTL; (iv) **consistencia orden↔pago** — un pago `rejected`/`failed` no cancela la orden ni la marca `paid`; estados terminales no retroceden; eventos duplicados son idempotentes (R2); (v) **snapshot inmutable** de la orden (precios, impuestos, descuento, moneda).
- B. Declarar solo un subconjunto (dime cuáles).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Declarar explícitamente los siguientes invariantes de dominio:
Aislamiento multi-tenant (R3): ninguna operación de una tienda podrá consultar o modificar información de otra tienda. Las operaciones globales del super-admin serán excepcionales, autorizadas, limitadas y auditadas. El tenant_id deberá resolverse y validarse en un único punto transversal.
No sobreventa (R1): las operaciones concurrentes de reserva y actualización de inventario deberán garantizar que nunca se comprometa una cantidad superior al stock disponible. La reserva y el consumo de stock deberán ser consistentes incluso ante solicitudes simultáneas.
Ciclo de vida de reservas: las reservas tendrán una fecha de vencimiento explícita (expiresAt). Su liberación deberá ejecutarse mediante una transición controlada, atómica e idempotente, sin depender de TTL como mecanismo de consistencia. Una reserva vencida no podrá confirmarse posteriormente.
Consistencia orden-pago (R2): Order y Payment mantendrán máquinas de estados independientes. Un pago rejected o failed no cancelará automáticamente la orden ni la marcará como pagada. La aprobación válida de un pago podrá actualizar la orden conforme a las reglas de vigencia y consistencia. Los eventos y comandos duplicados deberán procesarse de forma idempotente. Los estados terminales no podrán retroceder ni reabrirse automáticamente.
Historial de intentos de pago: cada intento individual conservará su resultado y trazabilidad. Un intento fallido o rechazado no deberá sobrescribir el resultado de un intento posterior ni impedir una aprobación válida cuando la orden y sus condiciones sigan vigentes.
Snapshot inmutable de la orden: al crear la orden se conservará una copia de los valores comerciales utilizados en el cálculo: moneda, productos y variantes, cantidades, precios unitarios, descuentos, base gravable, impuestos y total. Los cambios posteriores en el catálogo o en la configuración tributaria no modificarán órdenes históricas.
Propiedad de entidades: cada entidad tendrá un único componente propietario. Los demás componentes accederán a ella mediante contratos explícitos, sin consultar o modificar directamente su almacenamiento.
Estos invariantes deberán conservarse en el diseño funcional y técnico y servirán como criterios de validación para las pruebas críticas R1, R2 y R3.

## Assumptions & Open Questions

- [assumption] El modelo físico de datos (single-table DynamoDB vs. otras opciones, forma de PK/SK e índices, OQ2) NO se decide aquí; Domain Design captura entidades a nivel de **propiedad y forma** (dueño, identificador, atributos, referencias), y el esquema completo + el modelo de claves son de Functional Design / data-modelling. [team-practices] [requirements OQ2]
- [assumption] El topología de despliegue (cómo se agrupan los componentes en unidades desplegables) es de Units Generation, no de esta etapa.
- OQ-US-1 — Mecanismo de consulta segura de la orden del comprador invitado: permanece abierto; no se cierra en el modelo de dominio.

## Consolidated Summary Confirmation

Resumen de las decisiones de diseño de dominio a partir de tus respuestas (Q1–Q10). Confírmalo para que la etapa genere los artefactos.

**Componentes de dominio (del backend):**
- `TenantComponent` — dueño de `Store` (datos básicos, estado de activación, plan/estado de pago de suscripción) y `TaxConfig` (impuestos por producto/categoría); operaciones globales autorizadas del super-admin sobre el ciclo de vida de la tienda.
- `AuthComponent` — dueño de `UserIdentity` y asignaciones de rol por tienda (5 perfiles: super_admin, owner_admin, catalog_inventory_staff, orders_staff, customer); identidad Owner/Admin separada de la activación del tenant; invitado = modalidad de checkout, no identidad.
- `CatalogComponent` — dueño de `Product`, `Category`, `Variant` (imágenes como referencia a object-store).
- `InventoryComponent` — dueño de `StockItem`, `StockMovement` y `Reservation` (con `expiresAt`); responsable de no-sobreventa y liberación explícita por vencimiento.
- `OrderComponent` — dueño de `Order` y su máquina de estados (`pending_payment`, `paid`, `expired`, `cancelled`) y del snapshot inmutable.
- `PaymentComponent` — dueño de `Payment` (estado consolidado), `PaymentAttempt` (historial), `TransferProof` (comprobante) y `PaymentMethodConfig` (transferencia + credenciales de pasarela por referencia a secreto).
- `StorefrontComponent` — presentación pública por subdominio; dueño solo de `StoreTheme`/tokens de marca; consume catálogo e inventario por contrato.
- `AdminComponent` — componente de coordinación de operaciones/vistas administrativas; no posee entidades, consume contratos.
- `NotificationComponent` — componente lógico ligero orientado a eventos; su ubicación física entre las 9 unidades la fija Units Generation (no crea una 10ª unidad).
- `TenantContext` — preocupación transversal: resolución y validación de `tenant_id` en el límite; no es entidad ni componente de negocio.
- El `frontend` es consumidor de contratos, sin entidades de dominio del backend.

**Interacción (Q9):** síncrono (REST por contrato OpenAPI) para consultas/comandos con respuesta inmediata (catálogo/inventario en checkout, reserva de stock, operaciones admin); asíncrono (eventos) para efectos desacoplados (resultado de pago → orden, notificaciones, idempotencia de webhooks). Sin acoplamiento directo a BD entre unidades.

**Invariantes de dominio declarados (Q10):** (i) aislamiento multi-tenant (R3); (ii) no sobreventa (R1); (iii) ciclo de vida de reservas con liberación atómica/idempotente sin TTL; (iv) consistencia orden↔pago (R2: `rejected`/`failed` no cancela ni marca `paid`; terminales no retroceden; idempotencia); (v) historial de intentos de pago preservado; (vi) snapshot inmutable de la orden; (vii) propiedad única de entidad (acceso solo por contrato).

**Se mantiene fuera de esta etapa:** modelo físico de datos / claves DynamoDB (OQ2 → Functional Design/data-modelling), topología de despliegue (→ Units Generation), stack y patrones NFR. OQ-US-1 permanece abierta.

¿Esto refleja correctamente el diseño de dominio que quieres? (`Looks correct` / `Request changes`)

[Answer]: Looks correct
