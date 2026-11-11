# Requisitos — Plataforma de tienda virtual multi-negocio (SaaS multi-tenant)

> Fase: Inception. Fecha: 2026-11-11. Los IDs `FR{n}`/`NFR{n}` son claves de trazabilidad permanentes; las etapas posteriores las preservan.
> Entradas upstream: `intent-statement` y `scope-document` (Ideación) y `team-practices` (Practices Discovery), de las que derivan el alcance, las decisiones y las prácticas afirmadas.

## Análisis de intención

El objetivo es un SaaS multi-tenant genérico donde cualquier comercio, al registrarse, obtiene automáticamente su tienda en línea (catálogo, carrito, checkout, pagos) con aislamiento estricto entre tiendas, de forma práctica para el comerciante y monetizable (suscripción) para el negocio. El MVP debe permitir que un comercio piloto en Colombia publique su tienda y venda con pago manual por transferencia y/o pasarela, sin sobreventas y con órdenes consistentes. [intent-statement] [scope-document]

## Requisitos funcionales

### FR1 — Gestión de tiendas (tenants)
- FR1.1 — El registro de un comercio crea automáticamente su tienda como instancia aislada bajo la organización. [intent-statement]
- FR1.2 — Cada tienda configura datos básicos: nombre, logo, colores, moneda y contacto. [intent-statement]
- FR1.3 — A cada tienda se le asigna un subdominio de plataforma automático. [intent-statement]
- FR1.4 — Cada tienda tiene un campo de plan y de estado de pago de la suscripción, actualizado manualmente (sin cobro automático en el MVP). [intent-statement]
- FR1.5 — Una tienda queda "publicada" cuando está activa (por super-admin), tiene datos básicos (nombre, moneda, contacto), al menos una categoría y al menos un producto activo con stock, su subdominio está habilitado, y el Owner/Admin ejecuta explícitamente la acción de publicar. [Q6]
- FR1.6 — El super-admin puede crear, suspender y reactivar tiendas, y activar la cuenta del comercio. [intent-statement] [Q3]

### FR2 — Autenticación y autorización
- FR2.1 — Existen cinco roles: super-admin de plataforma, Owner/Admin de tienda, staff de catálogo/inventario, staff de órdenes y cliente comprador. [Q3]
- FR2.2 — El Owner/Admin administra su tienda y gestiona sus usuarios staff. [Q3]
- FR2.3 — Los permisos se validan en backend bajo mínimo privilegio; el staff de catálogo/inventario gestiona productos y stock; el staff de órdenes gestiona órdenes y aprueba/rechaza comprobantes de pago manual. [Q3]
- FR2.4 — El cliente comprador puede navegar, comprar y consultar exclusivamente sus propias órdenes. [Q3]
- FR2.5 — El aislamiento de datos entre tiendas se resuelve por `tenant_id` en un único punto transversal; ninguna consulta omite el filtro de tenant. [Q3] [team-practices]
- FR2.6 — El acceso de soporte del super-admin a una tienda es de solo lectura y queda registrado en auditoría. [Q3] [team-practices]

### FR3 — Catálogo y landing pública
- FR3.1 — Cada tienda gestiona productos con variantes, SKU, categorías e imágenes. [intent-statement]
- FR3.2 — La landing pública por subdominio muestra el catálogo con categorías, búsqueda simple y detalle de producto con imágenes. [intent-statement]
- FR3.3 — La tienda pública permite navegar categorías, ver disponibilidad y usar el carrito una vez publicada. [Q6]

### FR4 — Inventario y reserva de stock
- FR4.1 — El inventario registra stock por SKU y movimientos auditables (entradas, salidas, ajustes). [intent-statement]
- FR4.2 — Al crear una orden se reservan las unidades disponibles; la reserva tiene una duración configurable por tienda y diferenciada por método de pago (valores por defecto iniciales: 30 min pasarela, 24 h manual, sujetos a límites de configuración). [Q1]
- FR4.3 — Si el pago se confirma dentro del plazo, la reserva se convierte en salida definitiva de inventario. [Q1]
- FR4.4 — Si la reserva vence sin confirmación, el sistema libera explícitamente las unidades; la expiración no depende del TTL de DynamoDB. [Q1] [team-practices]
- FR4.5 — No se permite confirmar órdenes vencidas ni liberar dos veces la misma reserva. [Q1]

### FR5 — Carrito, checkout y órdenes
- FR5.1 — Carrito persistente para comprador anónimo y autenticado. [intent-statement]
- FR5.2 — El checkout calcula totales con impuestos y descuentos y crea la orden. [intent-statement]
- FR5.3 — La orden tiene estados `pending_payment`, `paid`, `cancelled`, `expired`; sin estados de envío ni devoluciones en el MVP. [Q2]
- FR5.4 — El pago tiene estados `pending`, `pending_review`, `approved`, `rejected`, `failed`, separados de los de la orden; distingue el rechazo de comprobante manual del fallo de pasarela. [Q2]
- FR5.5 — La aprobación válida del pago cambia la orden a `paid`; la expiración la lleva a `expired` liberando el stock. [Q2]
- FR5.6 — Las transiciones de estado son trazables y auditables. [Q2]

### FR6 — Pagos
- FR6.1 — Pago por pasarela mediante un adaptador genérico con credenciales propias del comercio; la plataforma no retiene fondos; sin datos de tarjeta en nuestros sistemas (checkout alojado/tokenización). [intent-statement] [team-practices]
- FR6.2 — Pago manual por transferencia: el comprador carga el comprobante y el Owner/Admin o staff de órdenes lo aprueba o rechaza tras verificar el ingreso en su banco. [intent-statement] [Q2]
- FR6.3 — Tras un rechazo de comprobante, el comprador (incluso invitado) puede volver a cargarlo mientras la orden y su reserva sigan vigentes. [Q2]
- FR6.4 — Los webhooks de pasarela se verifican por firma (HMAC) y se procesan de forma idempotente, tolerando eventos duplicados, tardíos o fuera de orden, sin que un estado terminal retroceda. [Q2] [team-practices]
- FR6.5 — Notificación al staff de la tienda cuando un comprobante queda pendiente de verificación. [Q2]

### FR7 — Impuestos y descuentos
- FR7.1 — Impuesto configurable por producto o categoría (tasa %), con exención posible (0%); la configuración del producto prevalece sobre la de su categoría y no se acumulan. [Q4]
- FR7.2 — Cada tienda configura si los precios incluyen impuesto o se agrega en el checkout; redondeo a 2 decimales en COP (sujeto a validación contable). [Q4]
- FR7.3 — La orden conserva una fotografía (snapshot) de la configuración tributaria y los valores calculados al momento de la compra. [Q4]
- FR7.4 — Descuento simple a nivel de orden (porcentaje o monto fijo) configurable por el Owner/Admin, sin cupones por código; no supera el subtotal elegible; el valor aplicado se registra. [Q5]

### FR8 — Panel de administración
- FR8.1 — Panel del comercio: gestión de productos e inventario, formulario de alta/edición de producto, listado y estado de órdenes, aprobación/rechazo de pagos manuales, y configuración de la tienda (datos de transferencia, conexión de pasarela, parámetros de impuestos). [scope-document] [wireframes]
- FR8.2 — Panel de super-admin: tiendas (crear/suspender/reactivar), plan y estado de pago manual, y métricas básicas (tiendas activas, órdenes totales y por tienda); soporte de solo lectura auditado. [intent-statement] [Q3]
- FR8.3 — Personalización de la tienda: selección de plantilla base y ajuste de colores, logo y tipografía (storefront). [scope-document]

### FR9 — Notificaciones
- FR9.1 — Notificaciones transaccionales básicas por el canal de menor costo: confirmación de orden y resultado de la verificación de pago. Sin campañas ni SMS masivo en el MVP. [feasibility] [scope-document]

## Requisitos no funcionales

### NFR1 — Aislamiento y seguridad multi-tenant
- NFR1.1 — Aislamiento estricto de datos entre tiendas; una fuga entre tenants invalida el MVP. Pruebas negativas de autorización cross-tenant obligatorias. [Q3] [feasibility]
- NFR1.2 — Mínimo privilegio en IAM y trazabilidad (auditoría) de operaciones sensibles. [team-practices] [desc]

### NFR2 — Confiabilidad de inventario y pagos
- NFR2.1 — Cero sobreventas bajo concurrencia: ante N reservas concurrentes sobre stock M (N>M), exactamente M tienen éxito, sin stock negativo. [Q1] [intent-statement]
- NFR2.2 — Consistencia orden↔pago: todo pago aprobado deja la orden en el estado correcto y los eventos duplicados/tardíos se procesan idempotentemente sin corromper el estado. La conciliación bancaria contra extractos queda fuera de alcance (la plataforma no retiene fondos). [Q2] [intent-statement]

### NFR3 — Protección de datos (Ley 1581)
- NFR3.1 — Datos de clientes y comprobantes de pago tratados conforme a la Ley 1581 (consentimiento, finalidad, derechos de los titulares); avisos de tratamiento de datos en registro de comercio, registro de comprador, checkout y carga de comprobante. [market-trends] [wireframes]
- NFR3.2 — Comprobantes de pago manual: almacenamiento privado cifrado en reposo y en tránsito; acceso solo para Owner/Admin y staff de órdenes de la tienda dueña, y super-admin en soporte solo-lectura auditado; el comprador solo carga comprobantes de sus órdenes; sin acceso público permanente; retención configurable sujeta a asesoría jurídica/contable. [Q8]
- NFR3.3 — No registrar PII, tokens, secretos ni comprobantes en los logs. [team-practices] [Q8]

### NFR4 — Seguridad de pagos (PCI reducido)
- NFR4.1 — Ningún dato de tarjeta (PAN/CVV) se procesa, registra ni persiste en nuestros sistemas; se mantiene el alcance PCI reducido mediante checkout alojado/tokenización. [Q2-feasibility] [team-practices]
- NFR4.2 — Cifrado en tránsito (TLS 1.2+) y en reposo (KMS/AES-256) para datos de cliente y comprobantes. [team-practices]

### NFR5 — Rendimiento y disponibilidad
- NFR5.1 — En el MVP se exige estabilidad, confiabilidad y correctitud de las operaciones principales (navegación pública, catálogo, autenticación, carrito, checkout, reserva de inventario, creación de órdenes, confirmación de pagos). Los umbrales numéricos de latencia, concurrencia, disponibilidad y capacidad se definen en NFR Requirements/Design, antes de la validación de preparación para producción. [Q7]

### NFR6 — Observabilidad
- NFR6.1 — Logs estructurados (JSON) con `correlationId` en fronteras de operación; métricas y alarmas básicas. [team-practices] [desc]

### NFR7 — Costo y plataforma
- NFR7.1 — Arquitectura serverless en AWS con diseño pago-por-uso; techo de gasto en dev de 10 USD/mes con alarma de presupuesto (AWS Budgets, alerta al 80%) activa antes del primer despliegue. [feasibility] [team-practices]

### NFR8 — Experiencia y accesibilidad
- NFR8.1 — Frontend web responsivo mobile-first para tienda pública y checkout (incluida carga de comprobante desde móvil); paneles usables en escritorio y móvil. [wireframes]
- NFR8.2 — Objetivo de accesibilidad WCAG 2.1/2.2 AA en tienda pública, carrito y checkout; AA con mejora continua en paneles. La conformidad real requiere pruebas con tecnologías de asistencia y revisión experta. [wireframes]

### NFR9 — Usabilidad / activación
- NFR9.1 — Un comerciante sin capacitación previa puede publicar su tienda con al menos un producto en menos de 15 minutos, contados desde la activación de su cuenta por el super-admin. [Q6] [intent-statement]

## Restricciones

- C1 — AWS serverless (Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge, Step Functions), IaC con Terraform, región us-east-1. [desc]
- C2 — El agente no ejecuta `terraform apply`/`destroy` (solo `fmt`/`validate`); el owner ejecuta apply; `allowed_account_ids` por ambiente. [team-practices]
- C3 — Monorepo con `modules/<unidad>/`; specs OpenAPI versionadas como contrato entre unidades. [team-practices]
- C4 — Secretos en Secrets Manager/SSM, nunca en el repositorio. [team-practices] [desc]
- C5 — País piloto Colombia; moneda configurable por tienda (COP de partida). [intent-statement]

## Supuestos

- A1 — Tarifas, tratamientos tributarios, bases gravables y reglas de redondeo definitivos quedan sujetos a validación contable antes de operar comercialmente. [Q4]
- A2 — La retención y eliminación de comprobantes se define con asesoría jurídica/contable. [Q8]
- A3 — El stack/runtime concreto (Node/TypeScript vs. Python) y las herramientas de prueba/seguridad se fijan en diseño. [team-practices]
- A4 — Número de plantillas de storefront (una, posible segunda) sujeto a investigación de licencias open source. [scope-document]

## Fuera de alcance

- Envíos y logística, facturación electrónica, marketplace multi-vendedor en un carrito, apps móviles nativas, multi-idioma, dominio propio del comercio, comisión por venta, cobro recurrente automático, analítica avanzada/GMV global, conciliación bancaria, CRM completo, cupones por código, devoluciones (refunds). [scope-document] [Q2] [Q5]

## Preguntas abiertas

- OQ1 — Umbrales numéricos de rendimiento/disponibilidad (NFR5): se fijan en NFR Requirements/Design. [Q7]
- OQ2 — Modelo exacto de claves y aislamiento en DynamoDB (single-table vs. otras opciones): se confirma en data-modelling/diseño. [team-practices]
- OQ3 — Proveedor(es) de pasarela inicial(es) para Colombia y sus tarifas: se verifican con fuentes oficiales en diseño. [team-practices]
