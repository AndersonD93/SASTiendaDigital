# User Stories — Plan de historias y preguntas

## Sources

- [desc] Initial description: "Dividir el MVP en unidades independientes; staff por tienda y clientes compradores con aislamiento estricto; carrito y checkout; pagos manual y pasarela; panel de administración."
- [scope] Workflow-selected scope: `feature`.

---

> Plan: derivar historias de usuario (formato "Como [persona], quiero [objetivo], para [beneficio]") con criterios de aceptación en Given/When/Then e IDs estables (US{grupo}.{seq}, AC{g}.{s}.{seq}), trazadas a los requisitos FR/NFR. Criterios INVEST. Prioridad MoSCoW por historia. Estas preguntas afinan el plan; luego genero las historias y las apruebas en la puerta.

## Plan propuesto

- **Personas** (derivadas de roles FR2 y de los wireframes): Comerciante Owner/Admin, Staff de catálogo/inventario, Staff de órdenes, Cliente comprador (y comprador invitado), Super-admin de plataforma.
- **Enfoque de desglose:** por unidad/dominio (tenants, auth, catalog/storefront, inventory, orders, payments, admin), alineado con las 9 unidades y el orden de construcción.
- **Granularidad:** historias verticales (end-to-end) de tamaño pequeño/estimable; las del walking skeleton con más detalle.
- **Prioridad:** MoSCoW por historia (Must para el núcleo del MVP; Should/Could para lo diferible).

## Q1. ¿Apruebas este enfoque de desglose de historias?

- A. Sí, desglose por unidad/dominio (alineado con las 9 unidades y el orden de construcción).
- B. Prefiero desglose por persona (agrupar historias por tipo de usuario).
- C. Prefiero desglose por flujo (comprador end-to-end, comerciante end-to-end).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Las historias se organizarán principalmente por unidad de construcción y dominio funcional, alineadas con las nueve unidades definidas: tenants, auth, catalog, inventory, orders, payments, admin, storefront/templates y frontend. Se mantendrán las dependencias y relaciones entre dominios, permitiendo historias transversales cuando representen un flujo de negocio completo. El agrupamiento no deberá fragmentar capacidades funcionales en tareas exclusivamente técnicas ni modificar el orden de construcción aprobado. Cada historia deberá conservar trazabilidad hacia los requisitos FR/NFR correspondientes.

## Q2. ¿Confirmas el conjunto de personas propuesto?

- A. Sí: Owner/Admin, staff de catálogo/inventario, staff de órdenes, cliente comprador (+ invitado), super-admin.
- B. Sí, pero ajusta algo (indícalo en Otro).
- C. Simplifica personas (indícalo en Otro).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: B. Se confirma el conjunto de personas funcionales: Owner/Admin, staff de catálogo/inventario, staff de órdenes, cliente comprador y super-admin de plataforma. Se contempla adicionalmente al comprador invitado como una variante del recorrido de compra, no como un rol de autorización independiente. Su inclusión como modalidad de checkout deberá estar respaldada por los requisitos aprobados; si aún no está confirmada, se mantendrá como capacidad pendiente de validación. Las historias deberán respetar los cinco perfiles funcionales definidos en Requirements Analysis y sus permisos.

## Q3. Granularidad / tamaño de historia

- A. Historias pequeñas y verticales (una capacidad por historia), con más detalle en las del walking skeleton. Recomendado.
- B. Historias más grandes por épica (menos historias, más gruesas).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Se utilizarán historias pequeñas, verticales, independientes y estimables, cada una representando una capacidad funcional con valor verificable para una persona. Las historias deberán cumplir INVEST, utilizar el formato Como [persona], quiero [objetivo], para [beneficio], incorporar criterios de aceptación Given/When/Then, prioridad MoSCoW, dependencias e IDs estables. Se dará mayor detalle y rigor de aceptación a las historias del walking skeleton. No se crearán historias que representen exclusivamente tareas técnicas de infraestructura, componentes o servicios sin valor funcional independiente; estas se derivarán posteriormente durante Construction.

## Q4. ¿Debo marcar el subconjunto de historias que forman el walking skeleton (el corte end-to-end mínimo)?

- A. Sí, marca explícitamente las historias del walking skeleton (crear tienda + publicar producto + comprar con pago manual + aislamiento).
- B. No hace falta marcarlas por ahora.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Se identificarán explícitamente las historias que integran el walking skeleton: aprovisionamiento y activación de tienda, configuración y publicación de catálogo, gestión inicial de inventario, compra mediante carrito y checkout, creación de orden, reserva de stock, carga y aprobación de comprobante de pago manual y confirmación de la orden. Este subconjunto deberá validar obligatoriamente R1 (sobreventa y concurrencia), R2 (consistencia orden-pago e idempotencia) y R3 (aislamiento entre tiendas). Las pruebas deberán incluir escenarios exitosos, negativos, concurrentes y de reintentos. Las pruebas R1-R3 satisfactorias serán la compuerta obligatoria de salida del walking skeleton.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
