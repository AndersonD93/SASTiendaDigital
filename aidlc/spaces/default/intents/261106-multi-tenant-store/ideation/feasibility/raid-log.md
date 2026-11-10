# RAID Log — Plataforma de tienda virtual multi-negocio

> Riesgos, Supuestos, Problemas (Issues) y Dependencias. Nivel de ideación; estimaciones conservadoras. Fecha: 2026-11-09.
>
> Entradas upstream: `intent-statement` (Intent Capture) y `build-vs-buy` / `competitive-analysis` (Market Research).

## Riesgos (Risks)

| ID | Riesgo | Probabilidad | Impacto | Mitigación (a abordar en diseño) |
|---|---|---|---|---|
| R1 | Sobreventa por fallo en la reserva de stock bajo concurrencia, incluida la reserva prolongada de órdenes pendientes de verificación manual y su liberación por vencimiento. | Media | Alto | Diseñar control de concurrencia y reserva con expiración; es una métrica de éxito (cero sobreventas). [Q8] [intent-statement] |
| R2 | Inconsistencia entre estado de orden y estado de pago ante webhooks duplicados/tardíos y ante aprobaciones/rechazos manuales. | Media | Alto | Diseñar idempotencia y máquina de estados de orden clara; es métrica de éxito. [Q8] [intent-statement] |
| R3 | Fuga de datos entre tiendas por error en el aislamiento multi-tenant en DynamoDB. | Baja-Media | Crítico | Disciplina de claves por `tenant_id` y autorización; pruebas de aislamiento. [Q8] |
| R4 | Superar el techo de 10 USD/mes en dev por costos fijos (zona DNS, dominio, mínimos de servicios). | Media | Medio | Diseño serverless-first; AWS Budgets con alerta al 80% en dev y prd; vigilar costos fijos. [Q5] [Q7] |
| R5 | Riesgo de cumplimiento por transferencia internacional de datos (us-east-1) frente a la Ley 1581. | Media | Medio | Análisis en diseño/compliance; evaluar región o salvaguardas. [Q3] |
| R6 | Automatización del subdominio por tienda más compleja de lo previsto (DNS/CloudFront). | Media | Medio | Prueba temprana del enrutamiento por subdominio. [Q8] |
| R7 | Ritmo de avance lento por dedicación parcial (~8 h/semana). | Media | Medio | Avance por fases; alcance acotado del MVP. [Q4] [Q5] |

## Supuestos (Assumptions)

| ID | Supuesto | A validar |
|---|---|---|
| A1 | El entorno AWS (cuentas, SSO, buckets de estado) está listo y accesible para dev. | Verificado por el owner; el diseño valida el resto. [Q7] |
| A2 | El techo de 10 USD/mes en dev es suficiente para desarrollo y pruebas de pilotos con diseño serverless-first. | Vigilancia de costos; AWS Budgets. [Q5] |
| A3 | El pago manual por transferencia reduce la dependencia de pasarelas en los pilotos. | Validar con pilotos. [Q6] |
| A4 | Tarifas y datos de pasarelas se verificarán con fuentes oficiales vigentes en diseño. | Regla de equipo. [memory:M1-team] |

## Problemas (Issues) — abiertos hoy

| ID | Problema | Estado |
|---|---|---|
| I1 | El entorno de trabajo del agente no completa la verificación de versión de la herramienta por red bloqueada, por lo que los comandos del motor los ejecuta el owner. | Workaround activo (owner ejecuta `orchestrate`); no bloquea el avance. |

## Dependencias (Dependencies)

| ID | Dependencia | Responsable | Notas |
|---|---|---|---|
| D1 | Disponibilidad de comercios piloto para validar el MVP. | Owner | Métrica de éxito: 10 pilotos operando. [Q6] [intent-statement] |
| D2 | Aprobación/configuración de la cuenta de pasarela de cada comercio para pagos reales. | Comercio (externo) | Desarrollo con entornos de prueba; pago manual como alternativa. [Q6] |
| D3 | Adquisición del dominio de la plataforma para los subdominios de las tiendas. | Owner | Necesario para el enrutamiento por subdominio. [Q6] |
| D4 | Creación de alarmas AWS Budgets (alerta 80%) en dev y prd. | Owner (apply) | Definidas por Terraform; el owner ejecuta el apply. [Q7] |
| D5 | Creación de alarmas de presupuesto y validación de configuración del entorno antes de generar módulos. | Owner / diseño | [Q7] |

## Assumptions & Open Questions

- [assumption] Los riesgos R1-R3 son las mayores incertidumbres técnicas del MVP (reserva de stock, consistencia de pagos, aislamiento multi-tenant); se abordan como prioridad en diseño. [Q8]
- [assumption] La facturación electrónica (DIAN) y el CRM completo quedan fuera del MVP y no entran en este RAID salvo como contexto futuro. [Q1] [Q3]
