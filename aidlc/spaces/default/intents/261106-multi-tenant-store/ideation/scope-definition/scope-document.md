# Documento de Alcance — Plataforma de tienda virtual multi-negocio (MVP)

> Nivel de ideación. Fecha: 2026-11-09.
>
> Entradas upstream: `intent-statement` (Intent Capture), `feasibility-assessment` y `constraint-register` (Feasibility), de los que toma el alcance del MVP, las restricciones y los riesgos.

## Alcance mínimo viable (corte más pequeño que entrega valor)

Una tienda piloto puede **operar** con: crear la tienda (con aislamiento entre tiendas), publicar al menos un producto con stock, completar al menos una orden pagada por **pago manual** (comprobante de transferencia aprobado por el administrador de la tienda), y un **panel de administración básico** del comercio (gestión de productos e inventario, listado y estado de órdenes, aprobación/rechazo de pagos manuales). [Q4] [intent-statement]

La pasarela de pago y los paneles más ricos completan el resto del MVP, pero no son requisito para operar el primer piloto. [Q4] [Q3]

## Dentro del MVP (In scope)

| Capacidad | Prioridad MoSCoW | Fuente |
|---|---|---|
| Gestión de tiendas (tenants): registro, creación automática de tienda, configuración básica, subdominio, plan y estado de pago (manual) | Must | [desc] [Q3] |
| Autenticación y autorización (staff por tienda, clientes, super-admin); aislamiento de datos entre tiendas | Must | [intent-statement] [Q3] |
| Catálogo: productos, variantes, categorías, imágenes, landing pública, búsqueda simple | Must | [desc] [Q3] |
| Inventario: SKU, stock, movimientos auditables, reserva de stock con liberación por vencimiento | Must | [intent-statement] [Q3] |
| Carrito y checkout: carrito persistente, impuestos y descuentos configurables, órdenes y estados | Must | [desc] [Q3] |
| Pagos: pago manual por transferencia con comprobante **y** pasarela (checkout alojado/tokenización, adaptador genérico, sin retención de fondos); webhooks idempotentes | Must (el pago manual es el primero en entregarse; la pasarela es lo primero que pasaría a Should si hay que recortar) | [intent-statement] [Q3] |
| Panel de administración del comercio (básico): productos, inventario, órdenes, aprobación/rechazo de pagos manuales, config de pagos | Must | [Q4] [desc] |
| Panel de super-admin: tiendas (crear/suspender/reactivar), plan y estado de pago manual, métricas básicas, soporte de solo lectura auditado | Must (básico) | [intent-statement] [Q3] |
| Storefront: ≥1 plantilla base bien diseñada y personalizable por tienda (colores, logo, etc.) | Must (una plantilla); plantillas adicionales = Could | [Q1] |
| Frontend web responsivo (construcción incremental junto a cada unidad) | Must | [desc] [Q3] |
| Notificaciones transaccionales básicas (confirmación de orden, resultado de verificación de pago), canal de menor costo | Should | [Q3] [feasibility] |
| Métricas básicas de plataforma (tiendas activas, órdenes totales y por tienda) | Should | [Q3] |
| Búsqueda avanzada; descuentos/cupones complejos; plantillas adicionales de storefront | Could | [Q3] [Q1] |

## Fuera del MVP (Won't have — this time)

| Excluido | Fuente |
|---|---|
| Envíos y logística, facturación electrónica, marketplace multi-vendedor en un carrito, apps móviles nativas, multi-idioma | [desc] |
| Dominio propio del comercio (solo subdominio de plataforma en el MVP; diseño deja la puerta abierta) | [intent-statement] |
| Comisión por venta y cobro recurrente automático de suscripciones | [intent-statement] |
| Analítica avanzada y GMV global consolidado | [intent-statement] |
| Conciliación bancaria contra extractos (la plataforma no retiene fondos) | [feasibility-assessment] |
| CRM completo y SMS/campañas; integraciones con sistemas externos del comercio (puertos previstos para futuro) | [feasibility-assessment] |

## Secuencia de entrega (preferencia)

Dependencias primero (tenants + auth como base) → atacar el riesgo con un walking skeleton (aislamiento multi-tenant, reserva de stock, consistencia de órdenes/pagos) → valor visible (catálogo, tienda pública con plantilla, paneles). En caso de conflicto, **el riesgo tiene prioridad sobre el valor visible**. Sin fechas límite duras; avance por fases. [Q5] [Q2] [Q6]

## Assumptions & Open Questions

- [assumption] La unidad `storefront` requiere, antes de construir el frontend, una investigación de repositorios públicos de plantillas que valide licencia compatible con uso comercial en SaaS (evitando copyleft fuerte que obligue a abrir el código), mantenimiento, accesibilidad y rendimiento. Esa investigación se realiza en diseño/construcción, no aquí. [Q1]
- [assumption] "Panel básico del comercio" en el mínimo viable excluye reportes y configuraciones avanzadas, que llegan en el resto del MVP o en planes posteriores. [Q4]
