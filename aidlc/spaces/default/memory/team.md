# Team-Level Rules

> This team's affirmed practices and corrections. Loaded after `org.md` as
> strict-additive guidance; contradictions with broader policy are rejected.
> Populated by the practices-discovery affirmation gate. Edit at the gate,
> not directly.

## Way of Working

<!-- Affirmed during practices-discovery. Example: -->
<!-- We use GitHub Flow with feature branches. Branches live 3-5 days max. -->
<!-- Hotfixes branch from main and merge back via expedited review. -->

## Walking Skeleton

<!-- Affirmed during practices-discovery. Example: -->
<!-- We don't run a walking skeleton — our deployment pipeline is mature -->
<!-- and the slice cost outweighs the value at our maturity stage. -->

## Testing Posture

<!-- Affirmed during practices-discovery. Example: -->
<!-- We use BDD. Specifications drive scenarios; scenarios drive code. -->
<!-- Each Unit ships with feature files in /features/. -->

## Guard Policy

<!-- Affirmed by the team. Mode: strict, relaxed, or off. Strict here holds for every intent, whatever one person asks in chat; relaxed or off here applies to every intent whose policy came from its scope, unless project.md or the person sets another; changing this line changes it. A section under the retired Change Control heading, written by an earlier release, is still read. -->

## Deployment

<!-- Affirmed during practices-discovery. -->

## Code Style

<!-- Team-specific conventions beyond the linter. Example: -->
<!-- - Prefer named exports over default exports -->
<!-- - All async functions return Result<T, E>, never throw -->

## Forbidden

<!-- Team-specific forbidden patterns -->

## Mandated

<!-- Team-specific mandates -->

## Corrections

<!-- Self-learning loop appends here. -->
- ALWAYS verificar tarifas, costos y datos de proveedores externos (p. ej. pasarelas de pago) con fuentes oficiales vigentes en la etapa de diseño, en lugar de darlos por supuestos (learned 2026-11-09) <!-- cid:261106-multi-tenant-store:intent-capture:bf9ce3f79339019edbd4f5ccab95cbb49906814d3ffa80950695b5a39e8a4853 -->
- ALWAYS el acceso de soporte a datos de una tienda o cliente debe ser de solo lectura y quedar registrado en auditoría (learned 2026-11-09) <!-- cid:261106-multi-tenant-store:intent-capture:293ae9f524d4cf18fe7b5a6f394b1eab68dafe3b8163563353c9cbfcfee858da -->
