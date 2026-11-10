# Composición de Trabajo y Modo de Construcción — Plataforma de tienda virtual multi-negocio

> Nivel de ideación. Fecha: 2026-11-09.
>
> Entradas upstream: `scope-document` e `intent-backlog` (Scope Definition) y `feasibility-assessment` (Feasibility), que fijan las 9 unidades, la secuencia (walking skeleton, riesgo primero) y las restricciones de capacidad.

## Composición

No hay mobs humanos múltiples: el trabajo lo ejecuta el agente AI-DLC bajo la dirección y aprobación del owner. En términos de roles de mob, el owner actúa como "navigator/decisor" (define intención, revisa, aprueba) y el agente como "driver" (construye). [Q1]

## Modo de Construcción (decisiones que condicionan la fase de Construcción)

- **Iteración:** una unidad a la vez (unit-major) — completar y aprobar cada unidad antes de pasar a la siguiente. [Q2]
- **Arranque:** empezar por el walking skeleton (corte mínimo de punta a punta: crear tienda, 1 producto con stock, 1 orden con pago manual aprobado, con aislamiento entre tiendas). [Q2] [scope-document]
- **Puntos de control:** revisar cada checkpoint hasta que el walking skeleton esté verificado y aprobado; a partir de ahí el owner decide si pasa a un modo más autónomo. [Q3]
- **Paradas obligatorias (incluso en modo autónomo):** siempre detenerse ante cambios en autenticación, aislamiento multi-tenant, pagos e IAM; ante cualquier fallo; y antes de cualquier `terraform apply` (lo ejecuta el owner). [Q3]

## RACI simplificado

| Actividad | Responsable / Aprueba | Ejecuta / Recomienda | Consultado |
|---|---|---|---|
| Decisiones de alcance y prioridad | Owner | Agente | — |
| Diseño y código de cada unidad | Owner (aprueba) | Agente | — |
| Aprobación en puertas y checkpoints | Owner | Agente (presenta) | — |
| `terraform plan` | Owner (revisa) | Agente (genera) | — |
| `terraform apply` / `destroy` | Owner (único que ejecuta) | — (el agente nunca ejecuta) | — |
| Impuestos y facturación de suscripciones | Owner | Agente (recomienda) | Contador (antes de operar real) |
| Términos, política de tratamiento y transferencia internacional de datos | Owner | Agente (recomienda) | Abogado de protección de datos (antes de operar real) |

## Normas de comunicación

- Idioma: español para conversación y documentación; código, identificadores y recursos en inglés. [desc]
- Cadencia: aprobación explícita del owner en cada puerta + resumen al cerrar cada fase. [intent-statement]

## Assumptions & Open Questions

- [assumption] El cambio a modo autónomo en Construcción es una decisión futura del owner, condicionada a que el walking skeleton quede verificado y aprobado; hasta entonces, cada checkpoint se revisa. [Q3]
- [assumption] Las paradas obligatorias (auth, aislamiento, pagos, IAM, fallos, apply) se mantienen aunque se active el modo autónomo. [Q3]
