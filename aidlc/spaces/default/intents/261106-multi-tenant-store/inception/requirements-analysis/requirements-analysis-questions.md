# Requirements Analysis — Preguntas de aclaración

## Sources

- [desc] Initial description: "Alcance del MVP: gestión de tiendas (tenants), catálogo y landing pública, inventario con reserva de stock, carrito y checkout, pagos (manual + pasarela), panel de administración, autenticación y autorización con aislamiento entre tiendas. Impuestos configurables. País piloto Colombia."
- [scope] Workflow-selected scope: `feature`.

---

> Convertimos el alcance y el intent en requisitos verificables. Ya tengo las decisiones de Ideación y las prácticas afirmadas; estas preguntas cierran huecos concretos para que cada requisito tenga un criterio de pass/fail claro. Marca una opción por pregunta en el `[Answer]:`. Si ninguna encaja, usa `X`.

## Q1. Reserva de stock: ¿cuánto dura la reserva de una orden pendiente antes de liberarse por vencimiento? (define el criterio verificable de R1)

- A. 30 minutos para pago por pasarela; para pago manual por transferencia, un plazo más largo configurable por tienda (p. ej. 24-72 h) para dar tiempo a transferir y subir comprobante.
- B. Un único plazo fijo para ambos métodos (indícalo en Otro).
- C. Plazo configurable por tienda para ambos métodos, con un valor por defecto que propongas.
- D. No lo sé / propón valores por defecto razonables y configurables.
- X. Other (please specify)

[Answer]: C. La duración de la reserva será configurable por tienda y diferenciada por método de pago. Se proponen como valores predeterminados iniciales 30 minutos para pagos mediante pasarela y 24 horas para pagos manuales por transferencia, sujetos a aprobación y posterior definición de límites de configuración. Toda orden creada reservará las unidades disponibles. Cuando el pago se confirme dentro del plazo, la reserva se convertirá en una salida definitiva de inventario. Cuando expire sin confirmación, el sistema liberará explícitamente las unidades reservadas. La expiración no dependerá del TTL de DynamoDB. No se permitirá confirmar órdenes vencidas ni liberar dos veces una misma reserva. La solución deberá garantizar cero sobreventas bajo concurrencia.

## Q2. Estados de una orden: ¿confirmas esta máquina de estados como requisito? (base de R2)

Propuesta: `pending_payment` (stock reservado) → `paid` (pasarela aprobada o comprobante aprobado) → `cancelled` (rechazado por el comercio o por la pasarela) / `expired` (venció sin pago; stock liberado). Sin estados de envío (fuera de alcance).

- A. Sí, esos estados (pending_payment, paid, cancelled, expired).
- B. Sí, pero añade algún estado (indícalo en Otro, p. ej. `refunded`).
- C. Otra máquina de estados (descríbela en Otro).
- D. No lo sé / propón tú la mínima que sirva.
- X. Other (please specify)

[Answer]:B. Se establecen estados independientes para órdenes y pagos. La orden manejará pending_payment, paid, cancelled y expired. El pago manejará pending, pending_review, approved, rejected y failed, diferenciando el rechazo de comprobantes manuales de los fallos de pasarela. Para pagos manuales, el comprador podrá cargar el comprobante y el Owner/Admin o staff de órdenes recibirá una notificación para verificar el ingreso en su banco y aprobarlo o rechazarlo. Cuando un comprobante sea rechazado, el comprador podrá volver a cargarlo mientras la orden y su reserva continúen vigentes. La aprobación válida del pago cambiará la orden a paid. La expiración liberará el stock. Se garantizará idempotencia, manejo de eventos duplicados, tardíos y fuera de orden, y trazabilidad de las transiciones. No se incluye gestión de devoluciones ni estados de envío en el MVP.

## Q3. Roles y permisos (autorización): ¿confirmas este mapa de permisos como requisito?

Propuesta: **super-admin** (plataforma: crear/suspender/reactivar tiendas, plan/estado de pago manual, métricas básicas, soporte solo lectura auditado); **Owner/Admin de tienda** (todo lo de su tienda); **staff de catálogo/inventario** (productos, stock); **staff de órdenes** (órdenes, aprobar/rechazar pago manual); **cliente comprador** (navegar, comprar, ver sus órdenes). Aislamiento estricto entre tiendas.

- A. Sí, ese mapa de roles y permisos.
- B. Sí, pero con un ajuste (indícalo en Otro).
- C. Simplificar: en el MVP solo Owner/Admin + cliente + super-admin (sin staff diferenciado todavía).
- D. No lo sé / propón tú.
- X. Other (please specify)

[Answer]: A. Se confirma el modelo de cinco perfiles funcionales definido en Ideation: super-admin de plataforma, Owner/Admin de tienda, staff de catálogo/inventario, staff de órdenes y cliente comprador. El super-admin podrá administrar tiendas, planes, estados de suscripción, métricas básicas y realizar soporte de solo lectura auditado. El Owner/Admin tendrá administración integral de su tienda y de sus usuarios staff. El staff de catálogo/inventario gestionará productos y existencias. El staff de órdenes gestionará órdenes y aprobará o rechazará comprobantes de pagos manuales. El comprador podrá navegar, comprar y consultar exclusivamente sus propias órdenes. Todos los permisos se validarán en backend bajo el principio de mínimo privilegio y aislamiento estricto por tenant_id. Se mantendrá la resolución de tenant_id en un único punto y la auditoría de operaciones sensibles. Se conserva la decisión D9 de Ideation.

## Q4. Impuestos: ¿qué regla de cálculo exacta debe soportar el MVP? (criterio verificable)

- A. Un impuesto por producto/categoría (tasa %), con opción por tienda de precio con impuesto incluido o agregado al final; redondeo estándar a 2 decimales en COP.
- B. Lo de A, pero además varias tasas por producto (p. ej. base + otra).
- C. Lo de A, con exención posible por producto (tasa 0%).
- D. No lo sé / propón la regla más simple que cubra IVA en Colombia.
- X. Other (please specify)

[Answer]: D. Se adopta como propuesta funcional inicial un modelo tributario sencillo y configurable para el país piloto Colombia. Se permitirá configurar una tasa porcentual por producto o categoría, contemplar productos exentos mediante tasa 0 %, establecer si los precios incluyen impuestos o si estos se agregan al checkout y calcular los valores correspondientes en la orden. La configuración específica del producto prevalecerá sobre la heredada de su categoría y no se acumularán automáticamente ambas tasas. La orden conservará una fotografía de la configuración tributaria y de los valores calculados al momento de la compra. Las tarifas, tratamientos tributarios, bases gravables y reglas de redondeo definitivas quedan sujetas a validación contable antes de operar comercialmente.

## Q5. Descuentos en el MVP: ¿qué nivel de descuento debe soportar? (el alcance marcó cupones complejos como Could)

- A. Solo descuento simple por tienda a nivel de orden (p. ej. % o monto fijo), configurable; sin cupones por código.
- B. Descuento por producto (precio rebajado), sin cupones.
- C. Sin descuentos en el MVP; se difieren.
- D. No lo sé / propón el mínimo.
- X. Other (please specify)

[Answer]:A. Se permitirán descuentos simples a nivel de orden, mediante porcentaje o valor fijo, configurables por el Owner/Admin. No se incluirán cupones por código, campañas promocionales, acumulación de descuentos ni reglas avanzadas. El descuento no podrá superar el subtotal elegible de la orden y su valor aplicado quedará registrado para preservar la consistencia histórica. La interacción tributaria del descuento se definirá conforme a la validación contable.


## Q6. Catálogo/landing públicos: ¿qué debe cumplir la publicación de una tienda para considerarse "publicada"? (criterio verificable del éxito <15 min)

- A. La tienda es pública en su subdominio cuando tiene configuración básica (nombre, moneda, contacto), al menos una categoría y al menos un producto con stock, y el comercio activa "publicar".
- B. Lo de A, pero permitir publicar sin productos (catálogo vacío con aviso).
- C. Otra definición (indícala en Otro).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Una tienda podrá publicarse cuando se encuentre activa por el super-admin, tenga configurados los datos básicos del comercio (nombre, moneda y contacto), disponga de al menos una categoría y un producto activo con stock disponible, y el Owner/Admin ejecute explícitamente la acción de publicar. El subdominio asignado deberá estar habilitado para servir la tienda pública. Una vez publicada, permitirá consultar el catálogo, navegar categorías, visualizar productos y disponibilidad y utilizar el carrito. El criterio de éxito será que un comerciante sin capacitación previa pueda publicar su tienda en menos de 15 minutos, contados desde la activación de su cuenta por el super-admin.

## Q7. Rendimiento y disponibilidad (NFR): ¿qué objetivos ponemos como requisito para el MVP piloto? (medibles pero realistas con presupuesto ajustado)

- A. Objetivos modestos de MVP: páginas públicas p95 < 2 s en condiciones normales; checkout responde en p95 < 3 s; disponibilidad objetivo 99% mensual (sin SLA formal con pilotos). Escala por diseño serverless.
- B. Objetivos más estrictos (indícalos en Otro).
- C. Sin objetivos numéricos en el MVP; solo "funcional y confiable".
- D. No lo sé / usa A como punto de partida, a refinar en NFR.
- X. Other (please specify)

[Answer]:C. Para el MVP se establecen como requisitos cualitativos la estabilidad, confiabilidad y funcionamiento correcto de las operaciones principales: navegación pública, catálogo, autenticación, carrito, checkout, reserva de inventario, creación de órdenes y confirmación de pagos. No se establece en esta etapa un SLA comercial ni objetivos numéricos definitivos de rendimiento y disponibilidad. Los umbrales medibles de latencia, concurrencia, disponibilidad y capacidad se definirán durante NFR Requirements y NFR Design, considerando la arquitectura serverless, el presupuesto disponible, las características de los pilotos y los costos operativos. Estos objetivos deberán quedar establecidos antes de la validación de preparación para producción.

## Q8. Retención y manejo de comprobantes de pago manual (dato sensible, Ley 1581): ¿qué requisito fijamos?

- A. El comprobante se almacena cifrado (S3 + KMS), accesible solo por el staff de la tienda dueña y el super-admin en soporte solo-lectura auditado; con un periodo de retención configurable y borrado tras él.
- B. Lo de A, con retención fija por defecto que propongas.
- C. No almacenar el comprobante, solo registrar la decisión de aprobación/rechazo (más simple, menos evidencia).
- D. No lo sé / propón lo más seguro y simple.
- X. Other (please specify)

[Answer]:A. Los comprobantes de pago manual se almacenarán en un repositorio privado, cifrados en reposo y en tránsito. El acceso estará restringido al Owner/Admin y al staff de órdenes de la tienda propietaria, y al super-admin exclusivamente durante operaciones de soporte de solo lectura auditadas. Los compradores solo podrán cargar comprobantes asociados a sus propias órdenes. No se permitirán accesos públicos permanentes a los archivos. Se registrarán las operaciones de carga, consulta, aprobación y rechazo sin incluir datos personales, comprobantes ni información sensible en los logs. El periodo de retención será configurable y estará sujeto a la política de conservación, finalidades del tratamiento y obligaciones legales y contractuales aplicables. La eliminación deberá respetar los plazos y condiciones de conservación definidos con asesoría jurídica y contable, incluyendo las copias que correspondan.

## Q9. ¿Hay algún requisito o criterio adicional que quieras fijar ahora y que no haya preguntado?

- A. No, con lo anterior y lo ya decidido en Ideación es suficiente para los requisitos del MVP.
- B. Sí (descríbelo en Otro).
- X. Other (please specify)

[Answer]: A. No se incorporan nuevos requisitos funcionales en esta pregunta. Se mantienen como parte del alcance del MVP todas las decisiones aprobadas en Ideation y Practices Discovery, incluyendo aislamiento multi-tenant, pagos sin retención de fondos, adaptadores de pasarela, webhooks firmados e idempotentes, suscripciones de actualización manual, subdominios de plataforma, moneda configurable por tienda, notificaciones transaccionales, experiencia mobile-first, objetivo de accesibilidad WCAG 2.1/2.2 AA, controles de seguridad, observabilidad y pruebas automatizadas. Se mantienen fuera del MVP las funcionalidades expresamente diferidas en Scope Definition. Las dependencias tributarias, jurídicas, de licencias y de costos continúan abiertas para las etapas correspondientes.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
