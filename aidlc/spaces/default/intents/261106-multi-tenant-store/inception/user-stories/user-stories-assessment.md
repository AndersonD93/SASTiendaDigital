# User Stories — Evaluación de necesidad

> Fase: Inception. Fecha: 2026-11-11.

## Decisión

**Execute** — se generan historias de usuario.

## Rationale

El MVP es intensamente de cara al usuario, con múltiples personas (Owner/Admin, staff de catálogo/inventario, staff de órdenes, cliente comprador e invitado, super-admin) y lógica de negocio no trivial (reserva de stock con concurrencia, máquina de estados orden/pago, aislamiento multi-tenant, pago manual con verificación). Las historias con criterios Given/When/Then aportan el anclaje verificable que la postura de pruebas afirmada exige, en especial para el walking skeleton y los riesgos R1-R3.

## Factores considerados

- **Tipo de proyecto:** producto nuevo (greenfield), SaaS multi-tenant.
- **Alcance de cara al usuario:** tienda pública, checkout, paneles de comercio y plataforma.
- **Señales de complejidad:** concurrencia (R1), consistencia de pagos (R2), aislamiento (R3); varios roles con permisos.

## Áreas donde las historias aportan más valor

- Flujo del walking skeleton end-to-end (crear tienda → publicar producto → comprar con pago manual), que fija el criterio de salida del skeleton.
- Reserva/liberación de stock, estados orden/pago e idempotencia de webhooks, donde los criterios de aceptación guían las pruebas de riesgo.
- Autorización y aislamiento entre tiendas, con criterios negativos explícitos.

## Ronda de refinamiento (2026-11-11)

Segunda ronda de refinamiento sobre hallazgos de la revisión funcional. Se preservaron todos los IDs estables de historias y criterios; los criterios nuevos recibieron IDs nuevos. No se eliminaron funcionalidades ni se amplió el MVP; no se tomaron decisiones técnicas prematuras ni se cerraron preguntas abiertas sin autorización. Ajustes:

1. US5.3 (AC5.3.1, +AC5.3.5): se eliminó la transición automática orden→`cancelled` ante pago `rejected`/`failed`; las únicas transiciones de orden dirigidas por pago/tiempo son `paid` (aprobación) y `expired` (vencimiento). Máquinas de orden y pago separadas (coherente con FR5.3/FR5.4/FR5.5).
2. US2.1 (+AC2.1.3/2.1.4/2.1.5): se precisó el aislamiento distinguiendo operación privada de tenant (tenant_id desde la sesión), catálogo público resuelto por subdominio, y operaciones globales autorizadas y auditadas del super-admin.
3. US6.3 (+AC6.3.5/6.3.6): se separaron los intentos individuales de pago del estado consolidado del pago de la orden, preservando el historial de intentos y evitando transiciones inconsistentes.
4. US2.3: prioridad ajustada de Should a **Must (gestión básica)**, alineada con Auth (Must) del alcance aprobado; +AC2.3.4 acota el MVP a gestión básica.
5. US3.4 / nueva US3.5: plantilla predeterminada funcional + personalización básica como **Must** (una plantilla); selección de plantillas adicionales separada como **Could** diferible.
6. US8.1 (+AC8.1.2): se mantuvo Should para el conjunto general y se explicitó la coherencia con la notificación de comprobante `pending_review` de US6.1 (FR6.5, parte del flujo manual Must).
7. US5.2 (AC5.2.1) / US5.4 (+AC5.4.2): se resolvió la contradicción checkout-obligatorio-vs-descuento-Could; el descuento es opcional y su ausencia no bloquea el checkout.
8. US1.1 (+AC1.1.4) / US1.4 (+AC1.4.2): se diferenció la identidad Owner/Admin (creada al registrarse) del estado de activación del tenant (lo activa el super-admin).
9. US6.1 (AC6.1.2) / US6.2 (título, AC6.2.1/6.2.2, +AC6.2.5): se incluyó expresamente al Owner/Admin como autorizado a consultar/aprobar/rechazar comprobantes (FR6.2, NFR3.2).
10. US4.2 (+AC4.2.3/4.2.4): se precisó que la liberación de stock vencido es atómica, idempotente y ejecutada una sola vez.
11. US2.2 (+AC2.2.3/2.2.4) + OQ-US-1: se aclaró la consulta de órdenes del comprador autenticado y se registró como **pregunta abierta** el mecanismo de consulta segura para el invitado (no se decidió).
12. US7.1 (+AC7.1.3): se restringió la configuración de pasarela, credenciales, impuestos y datos bancarios al Owner/Admin.
13. US5.2 (+AC5.2.5): se amplió la fotografía histórica de la orden (precios, cantidades, moneda, descuento, config tributaria e impuestos, totales).
14. US6.3 (AC6.3.2): se dejó de asumir HMAC como obligatorio; se usa el mecanismo oficial de verificación del proveedor (HMAC como ejemplo), a fijar en diseño (OQ3).

### Trazabilidad y skeleton tras el refinamiento

- `traceability.json`: FR8.3 ahora traza a US3.4 y US3.5; el resto de FR/NFR conserva su cobertura sin huérfanos. Prioridades MoSCoW preservadas salvo el ajuste explícito de US2.3 (→Must) y la separación US3.4 (Must) / US3.5 (Could), ambos alineados con el alcance aprobado.
- Walking skeleton sin cambios de composición (US1.1, US1.4, US2.1, US3.1, US3.2, US4.1, US4.2, US5.1, US5.2, US5.3, US6.1, US6.2); US3.5 y US6.3 permanecen fuera. R1, R2 y R3 siguen siendo la compuerta de salida obligatoria.
- Concern pendiente: OQ-US-1 (mecanismo de consulta de órdenes del comprador invitado), a resolver en diseño funcional.
