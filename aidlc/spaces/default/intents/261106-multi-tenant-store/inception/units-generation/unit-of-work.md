# Unidades de trabajo — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Units Generation. Fecha: 2026-11-12. Agrupa los componentes de dominio en **unidades de trabajo** (testeable y desplegable por separado). Esta etapa fija la topología; el **orden económico de construcción y los Bolts** los decide Delivery Planning (2.9).
> Entradas upstream: `components.md`, `decisions.md`, `requirements.md`, `stories.md`, `team-practices.md`.

## Tabla de unidades

| Unit ID | Directory | Kind | Complejidad | Componente(s) de dominio |
|---|---|---|---|---|
| U1 | `u1-tenants` | service | M | TenantComponent |
| U2 | `u2-auth` | service | L | AuthComponent (+ TenantContext transversal) |
| U3 | `u3-catalog` | service | M | CatalogComponent |
| U4 | `u4-inventory` | service | L | InventoryComponent |
| U5 | `u5-orders` | service | L | OrderComponent |
| U6 | `u6-payments` | service | XL | PaymentComponent + NotificationComponent (módulo lógico) |
| U7 | `u7-admin` | service | M | AdminComponent (coordinación) |
| U8 | `u8-storefront` | ui | M | StorefrontComponent |
| U9 | `u9-frontend` | ui | L | Interfaces administrativas (comercio + super-admin) |

> Se conservan exactamente las nueve unidades aprobadas en Scope Definition. No hay décima unidad: `NotificationComponent` es un módulo lógico interno de `u6-payments`; `TenantContext` es una capacidad transversal dentro de `u2-auth`; los contratos OpenAPI/eventos son artefactos compartidos versionados en el repo, sin unidad `spec`. El `frontend` administrativo es `u9-frontend`.

## Definiciones de unidad

### U1 — `u1-tenants` (service, M)
- **Responsabilidad:** ciclo de vida y estado de activación de la tienda (`Store`), configuración tributaria (`TaxConfig`), plan/estado de pago de la suscripción, y las operaciones globales autorizadas del super-admin sobre tiendas (auditadas).
- **Posee:** `Store`, `Subscription`, `TaxConfig`.
- **Despliegue:** standalone (serverless; recursos propios bajo `modules/u1-tenants/`).
- **Notas:** es la unidad base del DAG; expone por contrato el estado de activación y los datos de tienda. No llama a otras unidades para autorizar (la autorización se resuelve en el límite — ADR-008).

### U2 — `u2-auth` (service, L)
- **Responsabilidad:** identidad de usuarios (`UserIdentity`), asignación de roles por tienda (`RoleAssignment`) para los cinco perfiles, autorización bajo mínimo privilegio, y la resolución/validación transversal de `tenant_id` (`TenantContext`) usada por todas las unidades en el límite. Registro de auditoría del soporte solo-lectura del super-admin.
- **Posee:** `UserIdentity`, `RoleAssignment`.
- **Despliegue:** standalone (serverless); integra el proveedor de identidad (Cognito u otro, a fijar en diseño).
- **Notas:** la identidad Owner/Admin se crea al registrarse, separada de la activación del tenant (U1). El invitado es modalidad de checkout, no identidad.

### U3 — `u3-catalog` (service, M)
- **Responsabilidad:** productos, categorías y variantes (`Product`, `Category`, `Variant`) con imágenes por referencia a object-store; operaciones administrativas y de consulta del catálogo por contrato.
- **Posee:** `Product`, `Category`, `Variant`.
- **Despliegue:** standalone (serverless).
- **Notas:** el impuesto efectivo respeta la precedencia producto-sobre-categoría (TaxConfig de U1). La disponibilidad de stock se consulta a U4 por contrato.

### U4 — `u4-inventory` (service, L)
- **Responsabilidad:** stock por SKU (`StockItem`), movimientos auditables (`StockMovement`) y reservas (`Reservation` con `expiresAt`). Garante de la no-sobreventa (R1) y de la liberación por vencimiento explícita (sin TTL), atómica e idempotente.
- **Posee:** `StockItem`, `StockMovement`, `Reservation`.
- **Despliegue:** standalone (serverless); escritura condicional/transaccional para la concurrencia.
- **Notas:** U5 solicita crear/confirmar/liberar reservas por contrato; U4 no expone su almacenamiento.

### U5 — `u5-orders` (service, L)
- **Responsabilidad:** carrito (`Cart`), orden (`Order`) con su máquina de estados y el snapshot inmutable, y líneas (`OrderLineItem`). Cálculo de totales (impuestos vía TaxConfig; descuento opcional).
- **Posee:** `Cart`, `Order`, `OrderLineItem`.
- **Despliegue:** standalone (serverless).
- **Notas:** las únicas transiciones dirigidas por pago/tiempo son `paid` (aprobación) y `expired` (vencimiento); reacciona al resultado de pago de U6 **por evento**, idempotentemente (no es dependencia de construcción).

### U6 — `u6-payments` (service, XL)
- **Responsabilidad:** pago consolidado (`Payment`), historial de intentos (`PaymentAttempt`), comprobante manual (`TransferProof`) y configuración de métodos de pago (`PaymentMethodConfig`, credenciales por referencia a secreto). Verificación de firma e idempotencia de webhooks de pasarela; aprobación/rechazo manual por Owner/Admin o staff de órdenes. **Incluye el módulo lógico de notificaciones** (`NotificationComponent`): consume eventos de orden/pago y envía notificaciones transaccionales (`NotificationLog`), sin poseer órdenes ni pagos de otras unidades.
- **Posee:** `Payment`, `PaymentAttempt`, `TransferProof`, `PaymentMethodConfig`, `NotificationLog`.
- **Despliegue:** standalone (serverless); integra la pasarela (adaptador genérico), Secrets Manager/SSM y object-store cifrado para comprobantes.
- **Notas:** complejidad XL por concentrar pago manual + pasarela + idempotencia + notificaciones; límites internos claros (adaptador de pasarela, revisión manual, módulo de notificación).

### U7 — `u7-admin` (service, M)
- **Responsabilidad:** coordinación de vistas y acciones administrativas (paneles de comercio y super-admin) consumiendo los contratos de las unidades propietarias. **No posee entidades de negocio.**
- **Posee:** (ninguna).
- **Despliegue:** standalone (serverless).
- **Notas:** aplica la autorización por rol de U2; deja auditado el soporte solo-lectura del super-admin; las operaciones globales se enrutan a U1.

### U8 — `u8-storefront` (ui, M)
- **Responsabilidad:** experiencia pública de la tienda por subdominio: catálogo, detalle, carrito y checkout; plantilla base funcional + personalización (`StoreTheme`). Consume catálogo (U3) y disponibilidad (U4) por contrato.
- **Posee:** `StoreTheme`.
- **Despliegue:** frontend estático servido por CDN.
- **Notas:** resuelve la tienda por subdominio vía TenantContext; plantillas adicionales (US3.5, Could) fuera del corte mínimo.

### U9 — `u9-frontend` (ui, L)
- **Responsabilidad:** interfaces administrativas del comercio y de la plataforma (super-admin), consumiendo los contratos publicados (principalmente U7-admin y U2-auth).
- **Posee:** (ninguna entidad de dominio del backend).
- **Despliegue:** frontend estático servido por CDN.
- **Notas:** consume contratos, no almacenes de datos ni internos de las unidades service.

## Notas de implementación y restricciones (transversales)

- Monorepo con `modules/<unidad>/` por unidad (IaC Terraform + código de aplicación).
- Sin acoplamiento a BD entre unidades: todo cruce por contrato (API interna o evento).
- `tenant_id` resuelto en un único punto transversal (TenantContext en U2), no re-implementado por unidad.
- Contratos OpenAPI/eventos versionados como fuente de verdad, validados en CI.
- El agente no ejecuta `terraform apply`/`destroy` (solo `fmt`/`validate`); `allowed_account_ids` por ambiente.
- El modelo físico de datos (claves DynamoDB, OQ2) y el mapeo fino de servicios AWS por unidad se cierran en data-modelling / Infrastructure Design.

## Walking skeleton (hito de integración, no unidad)

El walking skeleton NO es una unidad del DAG: es un **corte vertical integrado** que atraviesa U1, U2, U3, U4, U5, U6 y U8, implementando lo mínimo de cada unidad propietaria para ejercitar el flujo end-to-end (crear tienda → activar → cargar producto con stock → publicar → comprar con reserva → pago manual aprobado). Cada funcionalidad mínima reside en su unidad propietaria y cruza por contrato; no se duplica código ni se mueve propiedad de entidades. Las pruebas de riesgo **R1 (U4), R2 (U5+U6), R3 (U2 + TenantContext)** en verde son la condición obligatoria de salida del skeleton antes del checkpoint humano.

## Sources

- [components] 8 componentes de dominio + NotificationComponent + TenantContext; propiedad de entidades.
- [decisions] ADR-001 (nueve unidades / Notification no 10ª unidad), ADR-004 (Inventory dueño de Reservation), ADR-005 (sync/async), ADR-006 (Admin coordinador), ADR-008 (autorización en el límite).
- [scope-document] nueve unidades aprobadas; storefront y frontend.
- [team-practices] monorepo `modules/<unidad>/`, serverless, fronteras, contratos OpenAPI.
- Q1–Q8 de `units-generation-questions.md`.

## Assumptions & Open Questions

- [assumption] Orden económico de construcción y Bolts → Delivery Planning (2.9), no aquí.
- [assumption] Servicios AWS finos por unidad y modelo físico de datos (OQ2) → Infrastructure Design / data-modelling.
- OQ-US-1 — Mecanismo de consulta segura de la orden del comprador invitado: abierto; se aborda en el diseño de U5/U8.
