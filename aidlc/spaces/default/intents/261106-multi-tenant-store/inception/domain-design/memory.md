<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
- 2026-11-12 — 9 componentes de dominio (1 por unidad de negocio) + NotificationComponent lógico; TenantContext transversal y frontend consumidor, no componentes. AdminComponent coordina sin poseer entidades.
- 2026-11-12 — Order y Payment separados; Payment unificado (manual+pasarela) con estado consolidado derivado del historial de intentos; InventoryComponent dueño de la Reservation (garante de R1).

## Deviations
- 2026-11-12 — Request Changes del humano (acotado): ajustes puntuales en ADR-001 (nueve unidades aprobadas; ocho componentes backend + frontend consumidor; NotificationComponent capacidad lógica, no 10ª unidad), ADR-005 (mecanismos de consistencia: id único de evento, idempotencia, duplicados/reintentos/fuera de orden, no reabrir orden terminal, recuperación sin duplicar efectos; sin tecnologías) y ADR-008 (validación de identidad/permisos/tenant en el límite antes de la operación; operaciones globales del super-admin autorizadas/acotadas/auditadas; no confiar en el origen de la llamada; no recrear ciclos implícitos por eventos). IDs y dirección de los ocho ADRs conservados; propiedad de entidades sin cambios.

## Tradeoffs
- 2026-11-12 — Interacción mixta (sync REST para comandos/consultas inmediatas; eventos para efectos desacoplados) frente a event-first o sync-first, para dar respuesta inmediata donde el usuario la necesita y desacoplar pagos/notificaciones (ancla R2).

## Open questions
- 2026-11-12 — OQ2 (modelo físico/claves DynamoDB) y el esquema completo de entidades se difieren a Functional Design / data-modelling; aquí solo propiedad+forma.
- 2026-11-12 — OQ-US-1 (consulta segura de la orden del invitado) permanece abierta; no se cierra en el modelo de dominio.
- 2026-11-12 — La ubicación física de NotificationComponent entre las 9 unidades se decide en Units Generation.
