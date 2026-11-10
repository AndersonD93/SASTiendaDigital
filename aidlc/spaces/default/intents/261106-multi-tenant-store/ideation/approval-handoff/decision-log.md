# Decision Log — Fase de Ideación

> Registro de las decisiones tomadas durante Ideación. Fecha: 2026-11-09.
> Entradas upstream: artefactos de las 6 etapas de Ideación.

## Intent Capture

| # | Decisión | Fuente |
|---|---|---|
| D1 | Plataforma SaaS multi-tenant genérica; el registro crea la tienda automáticamente; monetizable. | intent-statement [Q1] |
| D2 | Cliente: pymes/emprendedores de bajo presupuesto, foco en practicidad. | intent-statement [Q2] |
| D3 | Éxito: cero sobreventas, consistencia orden/pago, toda orden con estado final, publicar en <15 min (desde activación por super-admin), 10 pilotos operando (14-30 días). | intent-statement [Q3] [Q17] |
| D4 | Aislamiento multi-tenant lógico fuerte por `tenant_id`, con opción de silo futuro. | intent-statement [Q5] |
| D5 | Subdominio de plataforma por tienda; dominio propio diferido. | intent-statement [Q6] |
| D6 | Pagos: adaptador genérico + no retención de fondos + pago manual por transferencia con comprobante. | intent-statement [Q7] (ronda 2) |
| D7 | País piloto Colombia, moneda configurable por tienda. | intent-statement [Q8] |
| D8 | Impuestos parametrizables por producto/categoría (incluido o agregado). | intent-statement [Q9] |
| D9 | Roles: super-admin + Owner/Admin + staff diferenciado + cliente; super-admin con soporte solo lectura auditado. | intent-statement [Q10] [Q16] |
| D10 | Monetización MVP: solo suscripción, registro manual de plan/estado de pago. | intent-statement [Q15] |
| D11 | Métrica de pagos = consistencia orden/pago (sin conciliación bancaria; la plataforma no retiene fondos). | intent-statement [Q3] (ronda 2) |

## Market Research

| # | Decisión | Fuente |
|---|---|---|
| D12 | Competidores de referencia: plataformas regionales LATAM. | competitive-analysis [Q1] |
| D13 | Diferenciador: practicidad + no retención de fondos + pago manual. | competitive-analysis [Q2] |
| D14 | Precio de referencia: suscripción mensual baja fija por tienda. | competitive-analysis [Q3] |
| D15 | Foco regulatorio: protección de datos (Ley 1581). | market-trends [Q5] |
| D16 | Build-vs-buy: construir el núcleo; integrar pagos y servicios AWS gestionados. | build-vs-buy [Q6] |

## Feasibility & Constraints

| # | Decisión | Fuente |
|---|---|---|
| D17 | MVP viable; riesgos de diseño (R1-R4), no de factibilidad. | feasibility-assessment |
| D18 | Integraciones MVP: solo notificaciones transaccionales + CRM mínimo; puertos para futuro. | constraint-register [Q1] |
| D19 | PCI reducido (checkout alojado/tokenización, sin datos de tarjeta). | constraint-register [Q2] |
| D20 | Techo de gasto AWS dev: 10 USD/mes; serverless-first. | constraint-register [Q5] |
| D21 | AWS Budgets con alerta al 80% en dev y prd (Terraform; apply del owner). | constraint-register [Q7] |

## Scope Definition

| # | Decisión | Fuente |
|---|---|---|
| D22 | 9 unidades (incluye storefront con plantillas). | intent-backlog [Q1] |
| D23 | Orden con walking skeleton primero. | intent-backlog [Q2] |
| D24 | MoSCoW: núcleo Must; pasarela es lo primero en pasar a Should si hay recorte. | scope-document [Q3] |
| D25 | Mínimo viable: A + panel básico del comercio. | scope-document [Q4] |
| D26 | Secuencia: dependencias → riesgo → valor. | scope-document [Q5] |

## Team Formation

| # | Decisión | Fuente |
|---|---|---|
| D27 | Equipo: owner + agente; sin equipos separados. | team-assessment [Q1] |
| D28 | Construcción unit-major, walking skeleton primero. | mob-composition [Q2] |
| D29 | Checkpoints revisados hasta el skeleton; paradas obligatorias en auth/aislamiento/pagos/IAM/fallos/apply. | mob-composition [Q3] |
| D30 | Consultas con contador y abogado de datos antes de operar real. | team-assessment [Q4] |

## Rough Mockups

| # | Decisión | Fuente |
|---|---|---|
| D31 | 12 pantallas conceptuales (walking skeleton con más detalle). | wireframes [Q1] |
| D32 | Flujo principal = walking skeleton con variantes (pago rechazado, orden vencida). | user-flow [Q2] |
| D33 | Mobile-first en tienda/checkout; WCAG 2.1/2.2 AA objetivo. | wireframes [Q3] [Q5] |
| D34 | Sistema de diseño y plantillas open source, sujeto a investigación de licencias. | wireframes [Q4] |
| D35 | Correcciones R-01/R-02/R-03 aplicadas + notas (métodos por configuración, reentrada tras rechazo, avisos de datos, confirmación/auditoría en aprobación de pago). | wireframes (approval) |

## Aprendizajes registrados (reglas de equipo)

| # | Aprendizaje | Destino |
|---|---|---|
| L1 | Verificar tarifas/datos de proveedores externos con fuentes oficiales vigentes en diseño. | team.md ## Corrections |
| L2 | Acceso de soporte a datos de tienda/cliente: solo lectura y auditado. | team.md ## Corrections |

## Pendientes que pasan a Inception

- Investigación de plantillas open source y sus licencias (uso comercial en SaaS), accesibilidad, mantenimiento, rendimiento — antes del frontend.
- Análisis de transferencia internacional de datos (us-east-1) frente a la Ley 1581.
- Verificación de tarifas/costos de pasarelas candidatas con fuentes oficiales.
- Diseño detallado de reserva de stock, consistencia de órdenes y aislamiento multi-tenant (riesgos R1-R3).
- Consultas con contador y abogado de datos (antes de operar con comercios reales).
