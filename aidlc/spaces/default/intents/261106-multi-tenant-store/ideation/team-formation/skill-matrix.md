# Matriz de Habilidades y Análisis de Brechas — Plataforma de tienda virtual multi-negocio

> Nivel de ideación. Fecha: 2026-11-09.
>
> Entradas upstream: `scope-document` e `intent-backlog` (Scope Definition) y `feasibility-assessment` (Feasibility), que definen las capacidades a construir (9 unidades) y sus riesgos técnicos.

## Habilidades requeridas vs. disponibles

| Habilidad | Requerida por | Owner | Agente AI-DLC | Brecha / Mitigación |
|---|---|---|---|---|
| AWS serverless (Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge, Step Functions) | Todas las unidades | Suficiente | Alta | Sin brecha crítica; el agente guía el diseño. [desc] [Q1] |
| Terraform (IaC) | Infra de todas las unidades | Media (ejecuta apply) | Alta | El agente genera módulos; el owner valida y aplica. [desc] |
| Diseño multi-tenant y aislamiento de datos | tenants, auth, todas | Media | Alta | Riesgo R3; el diseño lo aborda como prioridad. [Q8-feasibility] |
| Concurrencia / reserva de stock | inventory, orders | Media | Alta | Riesgo R1; walking skeleton lo valida temprano. [Q8-feasibility] |
| Integración de pagos (pasarela + pago manual, webhooks idempotentes) | payments | Media | Alta | Riesgo R2; verificar tarifas con fuentes oficiales en diseño. [memory:M1-team] |
| Frontend web responsivo | frontend, storefront | Media | Alta | Framework a proponer en diseño. [desc] |
| Plantillas de storefront y su licenciamiento | storefront | Baja | Media | Requiere investigación de licencias (uso comercial en SaaS). [Q1-scope] |
| Seguridad (OWASP, mínimo privilegio IAM, secretos en SM/SSM) | Todas | Media | Alta | El diseño aplica buenas prácticas; paradas obligatorias en cambios de auth/IAM/pagos. [desc] [Q3] |
| Cumplimiento (Ley 1581, PCI reducido) | tenants, orders, payments | Baja | Media | Consulta con abogado de datos antes de operar real. [Q4] |
| Contabilidad / facturación de suscripciones | Operación | Baja | N/A | Consulta con contador antes de operar real. [Q4] |

Escala: Alta / Media / Baja / N/A.

## Brechas relevantes

- **Licenciamiento de plantillas (storefront):** brecha de conocimiento a cerrar con una investigación formal antes de construir el frontend. [Q1-scope]
- **Cumplimiento legal y contable:** no se cubre con el equipo técnico; se cierra con las consultas a contador y abogado de datos. [Q4]

Ninguna brecha bloquea el inicio: el walking skeleton y las unidades base (tenants, auth) no dependen de las consultas con terceros, que se necesitan antes de operar con comercios reales. [Q4] [Q2]

## Assumptions & Open Questions

- [assumption] Los niveles de habilidad del owner son autoevaluados ("suficiente en AWS"); se ajustarán según avance. [Q1-feasibility]
