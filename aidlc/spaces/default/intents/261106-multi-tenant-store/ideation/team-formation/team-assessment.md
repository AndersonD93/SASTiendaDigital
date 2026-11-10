# Evaluación de Equipo — Plataforma de tienda virtual multi-negocio

> Nivel de ideación. Fecha: 2026-11-09.
>
> Entradas upstream: `scope-document` e `intent-backlog` (Scope Definition) y `feasibility-assessment` (Feasibility), de los que toma el volumen de trabajo (9 unidades), las restricciones (dedicación parcial, presupuesto ajustado) y la secuencia (walking skeleton, riesgo primero).

## Composición del equipo

- **Owner (el usuario):** único ejecutor humano y decisor. Responsable de aprobaciones, del entorno AWS y de ejecutar `terraform apply`/`destroy`. Experiencia suficiente en AWS; dedicación parcial ~8 h/semana. [Q1] [Q4] [feasibility]
- **Agente AI-DLC:** ejecutor y asesor; genera diseño, código, pruebas e infraestructura como código, y recomienda; no decide ni sustituye consultas a terceros. [Q1] [Q4]
- Sin equipos humanos separados ni contratistas en el MVP. [Q1]

## Capacidad y ritmo

- Capacidad humana: ~8 h/semana del owner, principalmente para revisar, aprobar y ejecutar `apply`. [Q4-feasibility]
- Implicación: avance por fases, unidades pequeñas, **una unidad a la vez** (unit-major), coherente con la dedicación parcial. [Q2]

## Consultas con terceros (previas a operar con comercios reales)

| Consulta | Motivo | Momento |
|---|---|---|
| Contador | Impuestos y facturación de las suscripciones de la plataforma | Antes de operar con comercios reales. [Q4] |
| Abogado de protección de datos | Términos de servicio, política de tratamiento y transferencia internacional de datos (us-east-1 / Ley 1581) | Antes de operar con comercios reales. [Q4] |

El agente no sustituye estas consultas; las deja marcadas como dependencias. [Q4]

## Assumptions & Open Questions

- [assumption] La dedicación de ~8 h/semana es una estimación del owner; el ritmo real se ajustará por fases. [Q4-feasibility]
