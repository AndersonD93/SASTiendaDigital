# Intent Statement — Plataforma de tienda virtual multi-negocio (SaaS multi-tenant)

## Problem Statement

Los comercios pequeños y medianos, emprendedores y negocios con catálogo mediano o grande que no tienen equipo técnico necesitan una forma práctica y de bajo presupuesto de vender en línea con su propia tienda, sin construir ni operar su propio sistema de comercio electrónico. [Q1] [Q2]

La plataforma resuelve esto como un SaaS multi-tenant genérico (no atado a un tipo de negocio): cuando un comercio se registra, obtiene automáticamente su propia tienda como una instancia aislada bajo la organización, con catálogo, carrito, checkout y pagos en línea. El objetivo es que sea a la vez práctica para el comerciante y monetizable para el negocio que opera la plataforma. [Q1] [desc]

## Target Customer

- Comerciantes pequeños y medianos sin equipo técnico que necesitan vender en línea de forma rápida y económica. [Q2]
- Emprendedores que prueban un negocio nuevo y quieren lanzar una tienda en horas, no semanas. [Q2]
- Comercios con catálogo mediano o grande que necesitan inventario y control de stock confiable. [Q2]

Rasgo común del nicho: bajo presupuesto y prioridad en la practicidad, la facilidad de uso y la comprensión del negocio por parte del comerciante. [Q2]

## Success Metrics

El MVP se considera exitoso cuando es funcional y confiable, medido por: [Q3]

- **Cero sobreventas** bajo pruebas de concurrencia (la reserva de stock durante el checkout funciona). [Q3]
- **Consistencia entre el estado de la orden y el estado del pago**: todo pago aprobado deja su orden en el estado correcto, y los eventos de pago duplicados o tardíos se procesan de forma idempotente, sin duplicar ni corromper el estado de la orden. La plataforma no retiene fondos, por lo que la conciliación bancaria contra extractos queda fuera del alcance. [Q3] [Q7]
- **Ninguna orden sin estado final claro.** [Q3]
- Un comerciante nuevo puede **publicar su tienda con al menos un producto en menos de 15 minutos**, contados desde que el super-admin de la plataforma activa la cuenta del comercio, para un comerciante sin capacitación previa. [Q3] [Q16]

Métrica de validación de adopción (no es una funcionalidad a construir): **10 tiendas piloto "operando"**, donde una tienda está "operando" cuando, dentro de los **14 a 30 días** posteriores a publicar su landing, tiene al menos un producto con stock y al menos una orden pagada (que puede ser una compra real del propio comerciante), sin intervención manual del owner en el proceso de compra. [Q3] [Q17]

## Initiative Trigger

Oportunidad de mercado: existe demanda de comercios por vender en línea con una tienda propia, y esta plataforma busca atenderla con un modelo rentable. [Q4] [Q1]

## Initial Scope Signal

- **Workflow-selected scope:** `feature` (ciclo de vida completo para una funcionalidad nueva, con profundidad práctica, dividido en unidades de construcción). [scope] (workflow-selected)

- **Product boundary — dentro del MVP** (capacidades que el usuario confirmó como parte de esta primera versión):
  - Gestión de tiendas (tenants) con registro que crea la tienda automáticamente, y configuración básica por tienda. [Q1] [desc]
  - Aislamiento multi-tenant lógico fuerte por tienda, con diseño que permita migrar tiendas grandes o premium a un modelo con recursos o cuenta separada más adelante. [Q5]
  - Subdominio de plataforma automático por tienda. [Q6]
  - Catálogo y landing pública, inventario con reserva de stock, carrito y checkout, y órdenes con estado. [desc]
  - Pagos con dos métodos que conviven: (a) pasarela externa mediante un adaptador genérico, con credenciales propias por comercio y sin retención de fondos por la plataforma; y (b) pago manual, donde el comprador sube el comprobante de una transferencia y un administrador de la tienda lo aprueba o rechaza tras confirmarlo en su banco. [Q7]
  - País piloto Colombia con moneda configurable por tienda (COP como punto de partida). [Q8]
  - Impuestos parametrizables desde el administrador, por producto o categoría, con opción de precio con impuesto incluido o agregado. [Q9]
  - Roles: super-admin de plataforma, Owner/Admin de tienda, staff diferenciado (catálogo/inventario y órdenes) y cliente comprador, con aislamiento de datos entre tiendas. [Q10]
  - Alcance del super-admin en el MVP: administración básica (ver, crear, suspender y reactivar tiendas; soporte), campo de plan y estado de pago por tienda actualizado manualmente, y panel de métricas básicas (tiendas activas, órdenes totales y por tienda). El acceso de soporte a una tienda es de solo lectura y queda registrado en auditoría. [Q16]
  - Monetización en el MVP: solo suscripción por tienda; el MVP únicamente registra el plan y el estado de pago de cada tienda (actualización manual, sin cobro automático). [Q15]

- **Product boundary — fuera del MVP** (diferido a fases posteriores): [Q14]
  - Envíos y logística, facturación electrónica, marketplace multi-vendedor en un mismo carrito, apps móviles nativas y multi-idioma. [desc] [Q14]
  - Conciliación bancaria contra extractos (la plataforma no retiene fondos). [Q3] [Q7]
  - Dominio propio del comercio (solo subdominio de plataforma en el MVP; el diseño deja la puerta abierta para soportarlo después, con el costo del dominio asumido por el comercio). [Q6] [Q14]
  - Comisión por venta y cobro recurrente automático de suscripciones. [Q14] [Q15]
  - Analítica avanzada y GMV global consolidado. [Q14] [Q16]

## Assumptions & Open Questions

- [assumption] El modelo de monetización de largo plazo (planes superiores con reportes, cupones y otras capacidades) se resolverá como suscripciones de mayor valor en fases posteriores; el MVP no lo construye. [Q15]
- [assumption] El primer proveedor de pasarela candidato para Colombia (p. ej. Wompi) y un segundo proveedor para validar el adaptador se decidirán en la etapa de diseño; sus tarifas y costos por transacción deben verificarse con fuentes oficiales vigentes en ese momento, no darse por supuestos aquí. [Q7]
- [assumption] "Moneda configurable por tienda" arranca con Colombia/COP como país piloto; el soporte de otros países y monedas se evaluará después. [Q8]
- [assumption] El método de pago manual (comprobante de transferencia con aprobación del administrador) traslada al comercio la responsabilidad de verificar el ingreso en su banco; la plataforma solo registra el estado de aprobación o rechazo de la orden, no valida el movimiento bancario. [Q7]
