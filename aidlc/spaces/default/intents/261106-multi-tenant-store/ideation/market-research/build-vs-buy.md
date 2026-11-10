# Evaluación Build vs. Buy — Plataforma de tienda virtual multi-negocio

> Alcance: señal temprana a nivel de problema. El detalle técnico y de costos se aborda en diseño/feasibility. Fecha de consulta: 2026-11-09. [Q8]
>
> Entrada upstream: parte del `intent-statement` de Intent Capture (`aidlc/spaces/default/intents/261106-multi-tenant-store/ideation/intent-capture/intent-statement.md`), del que toma el alcance del MVP (núcleo multi-tenant, pagos con no retención de fondos, restricciones AWS) para enmarcar la decisión build-vs-buy.

## Postura del usuario

Construir el núcleo del producto e integrar servicios gestionados de AWS y pagos externos. [Q6]

- **CONSTRUIR (diferenciador propio):** el núcleo multi-tenant (gestión de tiendas, catálogo, inventario con reserva de stock, carrito, checkout, órdenes, panel de administración) y la experiencia de practicidad (publicar en minutos). Es el corazón de la propuesta de valor, por lo que se construye. [Q6][desc]
- **COMPRAR / INTEGRAR (capacidades commodity):**
  - **Pagos**: pasarelas externas vía un adaptador genérico, con credenciales propias del comercio y sin retención de fondos; más el flujo de pago manual por transferencia. No se construye una pasarela propia. [Q6][Q2]
  - **Servicios gestionados de AWS**: autenticación (Cognito), CDN (CloudFront), almacenamiento (S3), cómputo (Lambda), API Gateway, mensajería/orquestación (EventBridge, Step Functions donde aplique). Ya fijados como restricción técnica. [Q6][desc]

## Criterios de decisión (resumen cualitativo)

| Factor | Núcleo e-commerce multi-tenant | Pagos | Servicios de plataforma (auth, CDN, storage, cómputo) |
|---|---|---|---|
| ¿Es diferenciador central? | Sí → Construir | No → Integrar | No → Integrar (gestionado) |
| Tiempo al mercado | Mayor (se asume por ser el core) | Rápido con adaptador | Rápido (servicios gestionados) |
| Control / personalización | Alto, necesario | Medio (vía adaptador) | Medio-alto (configurable) |
| Manejo de dinero / riesgo | No aplica (no retiene fondos) | Queda en la pasarela y el comercio | No aplica |
| Carga de mantenimiento | Propia (aceptada) | Baja (la lleva el proveedor) | Baja (la lleva AWS) |

Regla aplicada: construir donde está la ventaja competitiva (el núcleo y la practicidad); integrar lo commodity (pagos y servicios de plataforma) con proveedores maduros. [Q6]

## Consideración evaluada y descartada

- **Comprar/incrustar una solución de e-commerce existente** en lugar de construir el núcleo (opción C de Q6): el usuario la descartó al elegir construir el núcleo. Se deja registrada como alternativa considerada. [Q6]

## Implicación para la arquitectura (a confirmar en diseño)

- El adaptador genérico de pasarelas es una decisión de diseño que ya tiene señal clara aquí; el detalle (primer proveedor en Colombia, segundo para validar, costos por transacción) se decide y **verifica con fuentes oficiales vigentes** en diseño. [Q6][Q2]
- El uso de servicios gestionados de AWS reduce el trabajo de construcción a medida y es coherente con las restricciones técnicas del proyecto. [Q6][desc]

## Assumptions & Open Questions

- [assumption] La separación build/buy es una señal temprana; la validación de costo a 3 años (desarrollo + mantenimiento vs. licencias/uso) se hace en feasibility/diseño. [Q6]
- [assumption] La elección y los costos de pasarelas se verifican con fuentes oficiales vigentes en diseño (regla de equipo). [Q6][Q2]
