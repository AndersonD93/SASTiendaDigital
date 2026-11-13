<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
- 2026-11-12 — Nueve unidades (u1-tenants..u9-frontend); sin décima. NotificationComponent = módulo lógico en u6-payments; TenantContext transversal en u2-auth; contratos OpenAPI/eventos compartidos sin unidad spec.
- 2026-11-12 — Edge-block = dependencias de construcción (acíclico); relaciones por evento (pago→orden, eventos→notificación) documentadas aparte, no como aristas; la arista u5↔u6 es una sola (u6→u5), la reacción por evento no la invierte.

## Deviations
- 2026-11-12 — Humano eligió Q6=B (sin unidad u0-walking-skeleton) frente a mi propuesta A: el skeleton es un hito de corte vertical integrado, no una unidad del DAG. Topología del DAG arranca en u1-tenants.

## Tradeoffs
- 2026-11-12 — Despliegue serverless por unidad (no monolito serverless) para independencia/pago-por-uso; el detalle fino de servicios AWS se difiere a Infrastructure Design.

## Open questions
- 2026-11-12 — Orden económico de construcción y Bolts → Delivery Planning (2.9), no aquí.
- 2026-11-12 — Servicios AWS finos por unidad y modelo físico de datos (claves DynamoDB, OQ2) → Infrastructure Design / data-modelling.
- 2026-11-12 — OQ-US-1 (consulta segura de la orden del invitado) permanece abierta; se aborda en el diseño de u5/u8.
