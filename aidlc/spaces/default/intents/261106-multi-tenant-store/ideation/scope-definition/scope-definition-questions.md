# Scope Definition & Prioritization — Preguntas de aclaración

## Sources

- [desc] Initial description: "Dividir el MVP en unidades independientes y proponer el orden de construcción. Alcance del MVP: gestión de tiendas (tenants), catálogo y landing pública, inventario, carrito y checkout, pagos, panel de administración por negocio, autenticación y autorización con aislamiento entre tiendas. Entorno AWS: estructura modules/ con módulos por unidad (tenants, catalog, inventory, orders, payments, admin, frontend)."
- [scope] Workflow-selected scope: `feature`.

---

> Nota: aquí fijamos el límite del MVP (dentro/fuera), la prioridad de cada capacidad y el orden de construcción en unidades independientes. Ya tengo el alcance de las etapas previas; estas preguntas son de decisión. Marca una opción por pregunta en el `[Answer]:`. Si ninguna encaja, usa `X`.

## Q1. Unidades de construcción propuestas. Propongo dividir el MVP en estas unidades independientes (alineadas con tu estructura `modules/`). ¿Las apruebas como base?

Propuesta:
1. **tenants** — registro de comercios, creación automática de tienda, configuración básica (nombre, logo, colores, moneda, contacto), subdominio, plan y estado de pago (manual).
2. **auth** — autenticación y autorización (Cognito): staff por tienda, clientes compradores, super-admin de plataforma; aislamiento de datos entre tiendas.
3. **catalog** — productos, variantes, categorías, imágenes, landing pública y búsqueda simple.
4. **inventory** — SKU, stock, movimientos auditables y reserva de stock (con liberación por vencimiento).
5. **orders** — carrito persistente, checkout, impuestos y descuentos, órdenes y estados.
6. **payments** — adaptador de pasarela (checkout alojado/tokenización) + pago manual por transferencia con comprobante; webhooks idempotentes.
7. **admin** — panel por negocio (productos, inventario, órdenes, config de pagos) + panel de super-admin (tiendas, plan/estado de pago, métricas básicas, soporte de solo lectura auditado) + notificaciones transaccionales.
8. **frontend** — app web responsiva que consume todo lo anterior (landing, tienda, carrito, checkout, paneles).

- A. Sí, apruebo estas 8 unidades tal cual.
- B. Sí, pero fusiona algunas (indícalo en Otro, p. ej. auth dentro de tenants).
- C. Sí, pero separa algo más (indícalo en Otro).
- D. No, prefiero otra división (descríbela en Otro).
- X. Other (please specify)

[Answer]:C. Separar una unidad de “plantillas y personalización de la tienda pública” (storefront). Quiero que cada tienda pueda escoger entre plantillas de diseño y que no se vean genéricas. Antes de construir el frontend, hacer una investigación de repositorios públicos de plantillas, evaluando licencia (compatible con uso comercial en un SaaS, evitando licencias que obliguen a abrir mi código), mantenimiento, accesibilidad y rendimiento. En el MVP: al menos una plantilla base bien diseñada y personalizable por tienda; plantillas adicionales como mejora posterior. 

## Q2. Orden de construcción propuesto. Propongo este orden (dependencias primero, valor temprano). ¿Lo apruebas?

Propuesta de orden:
1. **tenants** + **auth** juntos como base (sin tienda ni identidad aislada no hay nada).
2. **catalog** (publicar productos y landing — primer valor visible).
3. **inventory** (stock y reserva, necesario antes de vender).
4. **orders** (carrito y checkout).
5. **payments** (pago manual primero, luego pasarela — el manual desbloquea ventas en pilotos sin depender de credenciales).
6. **admin** (paneles y notificaciones, sobre lo ya construido).
7. **frontend** se construye de forma incremental junto con cada unidad (no al final), culminando la experiencia.

- A. Sí, apruebo este orden.
- B. Sí, pero con cambios menores (indícalo en Otro).
- C. Prefiero priorizar el "walking skeleton" (una tienda mínima end-to-end: crear tienda, 1 producto, 1 orden con pago manual) antes de completar cada unidad.
- D. No, prefiero otro orden (descríbelo en Otro).
- X. Other (please specify)

[Answer]:C. Primero un walking skeleton: crear una tienda, publicar un producto con stock y completar una orden con pago manual aprobado, con aislamiento entre tiendas. Luego completar cada unidad en el orden propuesto (tenants y auth, catálogo, inventario, órdenes, pagos, admin), con el frontend incremental.

## Q3. Priorización MoSCoW del alcance del MVP. ¿Confirmas qué es "Must Have" (imprescindible para el MVP) vs. "Should/Could"?

- A. Must: tenants, auth, catalog, inventory (con reserva), orders, payments (manual + pasarela), admin básico, frontend. Should: notificaciones transaccionales, panel de métricas básicas. Could: búsqueda avanzada, descuentos complejos.
- B. Must: todo lo anterior EXCEPTO la pasarela (solo pago manual en el MVP; pasarela pasa a Should). 
- C. Must: todo lo anterior EXCEPTO el panel de super-admin (pasa a Should).
- D. Otra distribución — indícala en Otro.
- X. Other (please specify)

[Answer]:A. Si el alcance debe recortarse, la pasarela se entrega después del pago manual y es lo primero en moverse a Should.

## Q4. Alcance mínimo viable (el corte más pequeño que entrega valor y permite "operar" una tienda piloto). ¿Cuál prefieres?

- A. Mínimo estricto: crear tienda + publicar 1 producto con stock + 1 orden pagada por pago manual, con aislamiento entre tiendas. (Pasarela y paneles ricos vienen después, dentro del mismo MVP.)
- B. Mínimo ampliado: lo de A + pasarela de pago operativa desde el inicio.
- C. Mínimo ampliado: lo de A + panel de administración completo del comercio.
- D. No lo sé / que lo proponga yo (sería A).
- X. Other (please specify)

[Answer]: X. Mínimo estricto (A) más un panel de administración básico para el comercio: gestión de productos e inventario, listado y estado de órdenes, y aprobación o rechazo de pagos manuales. Los paneles más ricos y la pasarela completan el resto del MVP, sin ser requisito para operar el primer piloto.

## Q5. Secuencia de entrega preferida (cómo priorizar entre unidades cuando haya que elegir)

- A. Dependencia primero (construir lo que desbloquea a lo demás), luego valor.
- B. Riesgo primero (atacar antes lo más incierto: reserva de stock, consistencia de pagos, aislamiento multi-tenant).
- C. Valor primero (lo que antes deja ver/usar la tienda).
- D. Una combinación equilibrada — indícala en Otro.
- X. Other (please specify)

[Answer]: D. Primero dependencias (tenants y auth como base). Después atacar el riesgo con el walking skeleton: aislamiento multi-tenant, reserva de stock y consistencia de órdenes y pagos. Luego construir el valor visible (catálogo, tienda pública y plantillas, paneles). En caso de conflicto, priorizar el riesgo sobre el valor visible.

## Q6. ¿Hay alguna fecha límite dura atada a alguna capacidad específica?

- A. No, sin fechas duras; avance por fases (coherente con feasibility).
- B. Sí, hay una fecha objetivo (indícala en Otro).
- C. Not applicable.
- X. Other (please specify)

[Answer]:A.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
