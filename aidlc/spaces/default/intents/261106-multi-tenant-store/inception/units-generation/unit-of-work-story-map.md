# Mapa historia → unidad — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Units Generation. Fecha: 2026-11-12. Una fila por historia `USx.y`; cada fila nombra la(s) unidad(es) que la implementan (varias separadas por `;`). Cada unidad nombrada traza todos los criterios de aceptación de esa historia en su Functional Design.
> Entradas upstream: `stories.md`, `components.md`, `unit-of-work.md`.

## Mapa

| Historia | Unidad(es) | Directorio(s) | Nota |
|---|---|---|---|
| US1.1 | U1; U2 | u1-tenants; u2-auth | registro crea tienda (U1) + identidad Owner/Admin (U2) |
| US1.2 | U1 | u1-tenants | configuración básica de la tienda |
| US1.3 | U1 | u1-tenants | plan/estado de pago de la suscripción (super-admin) |
| US1.4 | U1 | u1-tenants | activación del tenant por el super-admin |
| US2.1 | U2 | u2-auth | aislamiento multi-tenant (TenantContext) — R3 |
| US2.2 | U2; U5 | u2-auth; u5-orders | login/registro comprador (U2) + checkout invitado (U5) |
| US2.3 | U2 | u2-auth | gestión de staff y roles por el Owner/Admin |
| US2.4 | U2 | u2-auth | soporte solo-lectura del super-admin, auditado |
| US3.1 | U3; U4 | u3-catalog; u4-inventory | productos/variantes (U3) con stock inicial (U4) |
| US3.2 | U1; U3; U4 | u1-tenants; u3-catalog; u4-inventory | publicar: tienda activa (U1) + ≥1 categoría/producto (U3) con stock (U4) |
| US3.3 | U8; U3 | u8-storefront; u3-catalog | landing pública, navegación y búsqueda (U8 consume U3) |
| US3.4 | U8 | u8-storefront | plantilla predeterminada + personalización básica |
| US3.5 | U8 | u8-storefront | plantillas adicionales (Could) |
| US4.1 | U4 | u4-inventory | reserva de stock al crear la orden — R1 |
| US4.2 | U4 | u4-inventory | liberación por vencimiento (sin TTL), atómica/idempotente — R1 |
| US4.3 | U4 | u4-inventory | movimientos de inventario auditables |
| US5.1 | U5 | u5-orders | carrito persistente |
| US5.2 | U5; U1 | u5-orders; u1-tenants | checkout/totales y snapshot (U5) con TaxConfig (U1) |
| US5.3 | U5 | u5-orders | máquina de estados de la orden — R2 |
| US5.4 | U5 | u5-orders | descuento simple a nivel de orden |
| US6.1 | U6 | u6-payments | pago manual por transferencia con comprobante |
| US6.2 | U6 | u6-payments | aprobación/rechazo del pago manual (Owner/Admin o staff) |
| US6.3 | U6 | u6-payments | pago por pasarela con webhooks firmados e idempotentes — R2 |
| US7.1 | U7; U6; U1; U9 | u7-admin; u6-payments; u1-tenants; u9-frontend | panel del comercio: coordinación (U7) + config pagos (U6) + impuestos (U1) + UI (U9) |
| US7.2 | U7; U1; U9 | u7-admin; u1-tenants; u9-frontend | panel super-admin: coordinación (U7) + operaciones globales (U1) + UI (U9) |
| US8.1 | U6 | u6-payments | notificaciones transaccionales (módulo de notificación en U6) |

## Filas transversales (cross-cutting)

- **US1.1, US3.1, US3.2, US5.2, US2.2** cruzan más de una unidad (registro, publicación, checkout, snapshot tributario) — cada unidad implementa su parte por contrato.
- **US7.1, US7.2** cruzan la capa de coordinación (U7), los propietarios de datos (U1/U6) y la UI (U9).
- **Aislamiento (US2.1)** aplica como invariante transversal a todas las unidades vía TenantContext (U2), aunque su historia se mapea a U2 como dueño del mecanismo.

## Orden de implementación dentro de cada unidad (indicativo)

Dentro de cada unidad, las historias del walking skeleton (🟢) se implementan primero al mínimo para el corte integrado, y luego se completan:
- U1: US1.1 → US1.4 → US1.2 → US1.3.
- U2: US2.1 → US2.2 → US2.3 → US2.4.
- U3: US3.1 → US3.2(parte) → US3.3.
- U4: US4.1 → US4.2 → US4.3.
- U5: US5.1 → US5.2 → US5.3 → US5.4.
- U6: US6.1 → US6.2 → US8.1 → US6.3.
- U7: US7.1 → US7.2.
- U8: US3.3(render) → US3.4 → US3.5.
- U9: US7.1(UI) → US7.2(UI).

> El orden económico entre unidades (qué unidad primero) lo decide Delivery Planning; esto es solo el orden interno indicativo por unidad.

## Verificación de cobertura

- **26 historias** (US1.1–US8.1) asignadas; ninguna sin unidad.
- **Las nueve unidades** tienen al menos una historia (U9 recibe las UIs administrativas US7.1/US7.2; U8 las de storefront).
- Historias del walking skeleton (🟢) distribuidas en U1, U2, U3, U4, U5, U6 y U8, coherente con el corte integrado.

## Sources

- [stories] US1.1–US8.1 con sus criterios y marcas de walking skeleton.
- [unit-of-work] definiciones y propiedad por unidad; [components] propiedad de entidades.

## Assumptions & Open Questions

- [assumption] Cada unidad nombrada traza los criterios de su historia en Functional Design (etapa posterior).
- OQ-US-1 — Mecanismo de consulta de la orden del invitado (parte de US2.2): abierto; se resuelve en el diseño de U5/U8.
