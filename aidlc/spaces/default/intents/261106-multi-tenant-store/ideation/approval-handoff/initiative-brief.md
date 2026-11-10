# Brief de Iniciativa — Plataforma de tienda virtual multi-negocio (SaaS multi-tenant)

> Compilación de la fase de Ideación para la aprobación de paso a Inception. Fecha: 2026-11-09.
> Entradas upstream: `intent-statement` y `stakeholder-map` (Intent Capture), `competitive-analysis` (Market Research), `feasibility-assessment` y `constraint-register` (Feasibility), `scope-document` e `intent-backlog` (Scope Definition), `team-assessment` (Team Formation), `wireframes` (Rough Mockups).

## 1. Intent y problema

Plataforma SaaS multi-tenant genérica donde cualquier comercio, al registrarse, obtiene automáticamente su propia tienda en línea (catálogo, carrito, checkout, pagos) con aislamiento de datos entre tiendas. Resuelve la necesidad de comercios pequeños/medianos y emprendedores de bajo presupuesto de vender en línea con tienda propia sin construir su sistema, de forma práctica y monetizable. [intent-statement]

## 2. Validación de mercado (resumen)

Competidores de referencia: plataformas regionales LATAM (Tiendanube/Nuvemshop, Jumpseller; Empretienda por verificar). Diferenciador: practicidad extrema + no retención de fondos (credenciales propias del comercio) + pago manual por transferencia con comprobante. Contexto favorable: e-commerce en Colombia en máximos (CCCE: ~$145,4 billones COP en 2025, +11,1%) con pymes migrando desde redes/WhatsApp a tienda propia. Precios de referencia tratados como hipótesis con fecha, a verificar con fuentes oficiales en diseño. [competitive-analysis] [market-trends]

## 3. Viabilidad y riesgos (resumen)

MVP **viable** técnica y operativamente bajo presupuesto ajustado (techo AWS dev 10 USD/mes) y dedicación parcial (~8 h/semana). Riesgos principales (de diseño, no de factibilidad): [feasibility-assessment] [constraint-register]

- **R1** Sobreventa por fallo en reserva de stock bajo concurrencia (incl. reserva prolongada del pago manual y liberación por vencimiento).
- **R2** Inconsistencia entre estado de orden y de pago ante webhooks duplicados/tardíos y aprobaciones manuales.
- **R3** Fuga de datos entre tiendas por error en el aislamiento multi-tenant.
- **R4** Superar el techo de 10 USD/mes en dev por costos fijos.

Mitigación transversal: walking skeleton primero para validar R1-R3 temprano; AWS Budgets con alerta al 80% en dev y prd para R4. Cumplimiento: Ley 1581 (protección de datos, transferencia internacional por us-east-1), PCI reducido (checkout alojado/tokenización), DIAN fuera del MVP. [feasibility-assessment] [constraint-register]

## 4. Frontera de alcance (MVP)

**Dentro:** 9 unidades — tenants, auth, catalog, inventory (reserva de stock), orders, payments (pago manual + pasarela, sin retención de fondos), admin (básico + super-admin con soporte solo lectura auditado), storefront (≥1 plantilla personalizable), frontend. País piloto Colombia, moneda configurable por tienda. Impuestos parametrizables por producto/categoría. Subdominio de plataforma por tienda. [scope-document] [intent-backlog]

**Fuera:** envíos/logística, facturación electrónica, marketplace multi-vendedor en un carrito, apps móviles nativas, multi-idioma, dominio propio del comercio, comisión por venta, cobro recurrente automático, analítica avanzada/GMV global, conciliación bancaria, CRM completo. [scope-document]

## 5. Concepto visual (resumen)

12 pantallas conceptuales (baja fidelidad) cubriendo tienda pública, detalle, carrito, checkout (métodos según configuración), carga de comprobante, confirmación con reentrada, registro de comercio, formulario de producto, configuración de tienda, paneles del comercio y super-admin, personalización de plantilla y login del comprador. Mobile-first en tienda y checkout; objetivo WCAG 2.1/2.2 AA; avisos de tratamiento de datos en registro, checkout y comprobante. [wireframes]

## 6. Plan de equipo y entrega

Equipo: owner (único ejecutor/decisor, ~8 h/semana) + agente AI-DLC (ejecuta/recomienda). Construcción: una unidad a la vez (unit-major), walking skeleton primero, checkpoints revisados hasta validar el skeleton; paradas obligatorias en auth/aislamiento/pagos/IAM/fallos/apply. Consultas con terceros antes de operar real: contador (impuestos/facturación de suscripciones) y abogado de protección de datos. [team-assessment]

## 7. Verificación de frontera Ideación → Inception

Consistencia Intent → Scope → Intent Backlog verificada; todo el alcance tiene respaldo de viabilidad. Detalle en `../../verification/phase-check-ideation.md`.

## 8. Recomendación Go/No-Go

**GO** — avanzar a la fase de Inception. El intent y el alcance están confirmados, los riesgos reconocidos con mitigación (walking skeleton), y hay compromiso de recursos y entorno dev listo. [Q1] [Q2] [Q3] [Q4]

## Assumptions & Open Questions

- [assumption] Precios de competidores y tarifas de pasarelas son hipótesis con fecha; se verifican con fuentes oficiales en diseño. [competitive-analysis]
- [assumption] Número de plantillas de storefront en el MVP (una, posible segunda) sujeto a la investigación de licencias open source. [scope-document]
- [assumption] Transferencia internacional de datos (us-east-1) frente a la Ley 1581 se analiza en diseño/compliance. [constraint-register]
