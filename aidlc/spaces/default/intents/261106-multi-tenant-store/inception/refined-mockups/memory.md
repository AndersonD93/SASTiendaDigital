<!-- INVARIANT: examples are single-line HTML comments so a fresh template parses to total=0 (MEMORY_EMPTY). Do NOT un-comment or split across lines. t100 guards this. -->
> This file is kept up to date automatically while the stage runs. Add observations at the review step, not by editing here directly.

## Interpretations
- 2026-11-11 — Sistema de diseño expresado como tokens agnósticos del framework (Q1); no se elige librería/framework aquí, es de diseño técnico. Los mockups son contrato de media fidelidad en Markdown (Q9).
- 2026-11-11 — Personalización por tienda acotada a 4 ejes (color primario, acento, logo, tipografía) con defaults neutrales y validación de contraste AA (Q2 + design-system §7); espaciado/estructura no personalizables en MVP.
- 2026-11-11 — UI de pago `failed` vs `rejected` con mensajes/CTA distintos y la orden nunca mostrada como "cancelada" por un pago no exitoso (Q5), reflejando la separación de máquinas orden/pago de la ronda de refinamiento de User Stories.

## Deviations
<!-- example: 2026-05-29T10:14:32Z — skipped the optional caching layer the stage prose suggested; the dataset is small enough that it adds risk -->

## Tradeoffs
<!-- example: 2026-05-29T10:14:32Z — picked TDD over BDD this run; the team is unit-first and the domain is well-understood -->

## Open questions
- 2026-11-11 — OQ-US-1: mecanismo de consulta segura de la orden del comprador invitado; representado genéricamente en P6/P12 (componente OrderLookupEntry) sin cerrar la decisión (Q6). Se resuelve en diseño funcional.
