# Intent Capture & Framing — Preguntas de aclaración

## Sources

- [desc] Initial description: "Quiero construir una plataforma de tienda virtual multi-negocio (multi-tenant, SaaS) que cualquier comercio, sin importar su rubro, pueda usar para vender en línea. (...) Cómo quiero trabajar: responder y documentar en español, código e identificadores en inglés; avanzar por fases esperando mi aprobación explícita en cada puerta; antes de diseñar, hacerme las preguntas de requisitos que necesites (en especial modelo multi-tenant y aislamiento de datos, dominios/subdominios, proveedores de pago iniciales y países/monedas, impuestos y roles de usuario); en decisiones de arquitectura importantes presentarme opciones con pros, contras y recomendación; dividir el MVP en unidades independientes y proponer el orden de construcción."
- [scope] Workflow-selected scope: `feature`.

---

> Nota: esta etapa (Captura de Intención) define el **problema, el cliente, el éxito y los interesados** a nivel de negocio. Las decisiones técnicas finas (p. ej. partición exacta en DynamoDB, elección de pasarela) se detallan en etapas posteriores de diseño; aquí solo capturamos las señales que necesitas fijar temprano, tal como pediste. Marca una opción por pregunta en el `[Answer]:`. Si ninguna encaja, usa `X` y escribe tu respuesta.

## Q1. ¿Cuál es el problema de negocio principal que resuelve esta plataforma?

- A. Dar a comercios sin tienda en línea una forma rápida y genérica de vender por internet sin construir su propio sistema.
- B. Reemplazar soluciones existentes (p. ej. tiendas en marketplaces de terceros) por una tienda propia con marca y subdominio.
- C. Ofrecer a un grupo/holding de comercios una plataforma común para operar varias tiendas de forma aislada.
- D. Monetizar como SaaS cobrando a los comercios por usar la plataforma (suscripción/comisión).
- E. Una combinación de las anteriores (indícalo en Otro si quieres precisar).
- X. Other (please specify)

[Answer]:E, una combinación entre las opciones A,B,C,D ,la intensión es que sea una forma practica y monetizable que cualquier comercio que quiera montarse, se crearía una cuenta bajo una organización de forma automatica como una instancia de la tienda existente, para constutuir un modelo rentable para el negocio y este desarrollo. Monetización: suscripción y/o comisión (por definir con tiendas piloto). El registro de un comercio crea su tienda de forma automática.

## Q2. ¿Quién es el cliente principal y qué dolor vive hoy?

- A. Comerciantes pequeños/medianos sin equipo técnico que necesitan vender en línea rápido y barato.
- B. Comercios con catálogo mediano/grande que necesitan inventario y control de stock confiable.
- C. Emprendedores que prueban un negocio nuevo y quieren lanzar una tienda en horas, no semanas.
- D. Comercios ya digitales que quieren migrar a algo más configurable y con su propia marca/subdominio.
- E. Not yet defined — aún estamos validando el segmento.
- X. Other (please specify)

[Answer]:Diría que fuese un nicho en las opciones A, B, C. De bajo presupuesto y con practicidad para su uso y entendimiento del negocio

## Q3. ¿Cómo se ve el éxito del MVP? ¿Qué métricas importan?

- A. Activación: número de tiendas creadas y publicadas (landing en línea con catálogo).
- B. Transacción: número de órdenes completadas y GMV (valor total vendido) por tienda.
- C. Operación: tiempo desde registro hasta primera venta; tasa de checkout completado vs. abandonado.
- D. Confiabilidad: cero sobreventas (reserva de stock funciona) y pagos conciliados vía webhooks.
- E. Una combinación — prioriza en Otro si quieres fijar las 2-3 principales.
- X. Other (please specify)

[Answer]:X. Un MVP funcional y confiable, medido así: cero sobreventas en pruebas de concurrencia; ninguna orden sin estado final claro; y un comerciante nuevo puede publicar su tienda con al menos un producto en menos de 15 minutos. Además, 10 tiendas piloto operando sin intervención manual. (Corrección ronda 2) La plataforma NO retiene fondos, por lo que la conciliación bancaria queda FUERA del alcance. La métrica de confiabilidad de pagos pasa a ser la CONSISTENCIA entre el estado de la orden y el estado del pago: todo pago aprobado deja su orden en el estado correcto, y los webhooks duplicados o tardíos se procesan de forma idempotente (no duplican ni corrompen el estado de la orden).

## Q4. ¿Cuál es el disparador de esta iniciativa (por qué ahora)?

- A. Oportunidad de mercado: demanda de comercios por vender en línea con tienda propia.
- B. Decisión estratégica/producto nuevo: lanzar un SaaS de comercio electrónico.
- C. Necesidad interna: operar varias tiendas propias bajo una misma plataforma.
- D. Presión competitiva frente a plataformas existentes.
- E. Not identified.
- X. Other (please specify)

[Answer]:La opción es la A. 

## Q5. Multi-tenant y aislamiento de datos: ¿qué modelo de aislamiento esperas entre tiendas? (señal temprana; el diseño detallado vendrá después)

- A. Aislamiento lógico fuerte: datos compartidos en las mismas tablas pero particionados por `tenant_id`, con autorización que impide cruzar tiendas (pooled, estándar en serverless/DynamoDB).
- B. Aislamiento por recurso: una partición/tabla o espacio separado por tienda (silo parcial), mayor aislamiento y costo.
- C. Aislamiento máximo: infraestructura separada por tienda (silo completo) — normalmente para enterprise/regulado.
- D. Lógico fuerte por ahora (A), con opción de migrar tiendas grandes a silo más adelante.
- E. Not yet defined — quiero que me propongas opciones con pros/contras en el diseño.
- X. Other (please specify)

[Answer]: D. Aislamiento lógico fuerte por tenant_id en el MVP, con diseño que permita migrar tiendas grandes o premium a un modelo con recursos o cuenta separada más adelante. Quiero que se evalúen las opciones con pros, contras y costo por tienda en la etapa de diseño.

## Q6. Dominios y subdominios: ¿cómo quieres que cada tienda sea accesible en el MVP?

- A. Subdominio propio bajo un dominio de la plataforma (p. ej. `mitienda.tudominio.com`), gestionado automáticamente.
- B. Subdominio de plataforma (A) **y** soporte para dominio propio del comercio (p. ej. `www.mitienda.com`) ya en el MVP.
- C. Ruta/path por tienda (p. ej. `tudominio.com/mitienda`) en vez de subdominio.
- D. Subdominio de plataforma en el MVP; dominio propio queda para una fase posterior.
- E. Not yet defined.
- X. Other (please specify)

[Answer]:D. Subdominio de plataforma automático para cada tienda en el MVP. El dominio propio del comercio queda para una fase posterior, con el costo del dominio asumido por el comercio. Pido que el diseño deje la puerta abierta para soportarlo después.

## Q7. Proveedores de pago iniciales: ¿con qué pasarela(s) arrancamos el MVP? (habrá capa de adaptadores para agregar más)

- A. Stripe (checkout alojado / tokenización).
- B. Un proveedor regional de LATAM (p. ej. Mercado Pago, PayU, dLocal) — indica cuál en Otro.
- C. Stripe + un proveedor regional desde el inicio.
- D. PayPal.
- E. Not yet defined — diseñar el adaptador genérico y decidir el primer proveedor en el diseño.
- X. Other (please specify)

[Answer]:E. Diseñar un adaptador genérico de pasarelas. Cada comercio conecta sus propias credenciales y la plataforma no retiene fondos. Proponer un primer proveedor para Colombia con entorno de pruebas gratuito y sin cuota fija (por ejemplo Wompi), y un segundo proveedor como validación del adaptador. Mostrar el costo por transacción de cada candidato. (Precisión ronda 1: las tarifas de las pasarelas candidatas deben verificarse con fuentes oficiales vigentes en la etapa de diseño, no darse por supuestas.) (Corrección ronda 2) Además de la pasarela, el MVP incluye un método de pago MANUAL que convive con ella: el comprador sube el comprobante de una transferencia bancaria y un administrador de la tienda aprueba o rechaza el pago tras confirmarlo en su banco. El estado de la orden refleja ese flujo de aprobación/rechazo manual.

## Q8. Países y monedas objetivo del MVP

- A. Un solo país y una sola moneda al inicio (indica cuál en Otro, p. ej. Colombia/COP).
- B. Varios países de LATAM, cada tienda opera en una moneda (multi-moneda a nivel plataforma, mono-moneda por tienda).
- C. Global, múltiples monedas por tienda.
- D. Moneda configurable por tienda (como dice la visión), empezando por un país piloto.
- E. Not yet defined.
- X. Other (please specify)

[Answer]: D. iniciando con un pais piloto como Colombia

## Q9. Impuestos: ¿qué necesita soportar el cálculo de impuestos en el MVP?

- A. Un impuesto configurable por tienda, tasa única (p. ej. IVA %) aplicado a todo el carrito.
- B. Impuesto por producto/categoría (tasas distintas según el ítem).
- C. Impuesto por región/ubicación del comprador (reglas por zona).
- D. Precios con impuesto incluido vs. impuesto agregado al final — configurable por tienda.
- E. Not yet defined — proponer el modelo más simple que sirva y dejar reglas avanzadas fuera del MVP.
- X. Other (please specify)

[Answer]:B. Bloque de impuestos parametrizable desde el administrador, por producto/categoría, con opción de precio con impuesto incluido o agregado.

## Q10. Roles de usuario: ¿qué roles necesitas en el MVP? (staff por tienda y clientes compradores, con aislamiento entre tiendas)

- A. Mínimo: Owner/Admin de tienda (gestiona todo su negocio) + Cliente comprador. Un rol de staff único.
- B. Owner/Admin + roles de staff diferenciados (p. ej. gestor de catálogo/inventario, gestor de órdenes) + Cliente.
- C. Lo anterior (B) **más** un rol de plataforma (super-admin del SaaS que administra tiendas/soporte).
- D. A + un super-admin de plataforma, sin diferenciar staff todavía.
- E. Not yet defined.
- X. Other (please specify)

[Answer]: C.

## Q11. Interesados (stakeholders): ¿quiénes son los interesados clave y qué le importa a cada uno?

- A. Solo tú como owner del producto/fundador (decides y validas todo).
- B. Owner de producto + equipo de desarrollo (tú y el agente AI-DLC).
- C. Owner de producto + comercios piloto que darán feedback temprano.
- D. Owner de producto + áreas de negocio (ventas/soporte) + comercios piloto.
- E. Not identified aún.
- X. Other (please specify)

[Answer]: X. (Precisión ronda 1) Owner de producto (el usuario) + comercios piloto que darán feedback temprano. Las funciones de ventas y soporte las ejerce hoy el mismo owner de producto; no hay equipos separados. No asumir áreas de negocio independientes.

## Q12. Decisión de alcance y prioridad: ¿quién decide qué entra al alcance y quién influye?

- A. Tú decides el alcance y la prioridad; el agente recomienda.
- B. Tú decides, con feedback de comercios piloto como influencia.
- C. Decisión compartida con un socio/co-fundador (indícalo en Otro).
- D. Not applicable — soy el único decisor.
- E. Other arrangement.
- X. Other (please specify)

[Answer]:A. Tu seras mi asesor principal

## Q13. Comunicación y cadencia de reportes: ¿cómo quieres recibir avances?

- A. Aprobación explícita en cada puerta de etapa (como ya pediste), sin reportes adicionales.
- B. Aprobación en cada puerta + un resumen al cerrar cada fase (Ideación, Inception, etc.).
- C. Aprobación en cada puerta + resumen por unidad de construcción.
- D. Not applicable — con la aprobación por puerta basta.
- E. Other.
- X. Other (please specify)

[Answer]: B.

## Q14. Límites del producto en esta primera versión: además de lo ya listado como fuera de alcance (envíos/logística, facturación electrónica, marketplace multi-vendedor en un carrito, apps móviles nativas, multi-idioma), ¿qué más debe quedar **fuera** del MVP?

- A. Nada más; los exclusiones ya listadas son suficientes para el MVP.
- B. Dejar fuera también reportes/analítica avanzada (más allá de métricas básicas).
- C. Dejar fuera también promociones/cupones complejos (solo descuentos simples configurables).
- D. Dejar fuera también dominio propio del comercio (solo subdominio de plataforma) — coherente con Q6.
- E. Una combinación — indícala en Otro.
- X. Other (please specify)

[Answer]: X. (Precisión ronda 1) Además de lo ya listado (envíos/logística, facturación electrónica, marketplace multi-vendedor en un carrito, apps móviles nativas, multi-idioma), quedan FUERA del MVP: dominio propio del comercio, comisión por venta, cobro recurrente automático de suscripciones, analítica avanzada y GMV global consolidado. Todo esto queda para fases posteriores.

---

> Preguntas de seguimiento (surgen del análisis de tus respuestas Q1–Q14). Rellena el `[Answer]:` de cada una.

## Q15. Monetización en el MVP vs. "la plataforma no retiene fondos" (de Q7)

En Q1 mencionaste monetización por suscripción y/o comisión, y en Q7 que cada comercio conecta sus propias credenciales y **la plataforma no retiene fondos**. Si la plataforma no es intermediaria del dinero, no puede descontar automáticamente una comisión de cada venta; una comisión tendría que facturarse aparte. ¿Cómo monetiza el MVP?

- A. Solo suscripción (plan mensual/anual por tienda). La comisión por venta queda como idea futura. (Más simple y coherente con "no retener fondos".)
- B. Suscripción en el MVP, pero dejar el diseño preparado para una futura comisión facturada aparte (reportada, no retenida).
- C. Sin monetización activa en el MVP (gratis para los 10 pilotos); el modelo de cobro se decide después.
- D. Comisión retenida sobre cada venta — lo que implicaría que la plataforma **sí** maneje fondos (reabre la decisión de Q7).
- E. Not yet defined.
- X. Other (please specify)

[Answer]: A. Para el MVP el cobro es solo por suscripción por tienda. Planes futuros de mayor valor (reportes, cupones y otras capacidades) se resolverán como suscripciones superiores. El cobro de la suscripción a los comercios puede ser manual al inicio; el MVP solo debe registrar el plan y el estado de pago de cada tienda. No se construye cobro recurrente automático ni comisión por venta en esta fase.

## Q16. Alcance del rol super-admin de plataforma en el MVP (de Q10 = C)

Elegiste incluir un super-admin del SaaS. ¿Qué debe poder hacer ese rol **en el MVP**?

- A. Administración básica: ver/crear/suspender/reactivar tiendas y dar soporte. Sin panel de facturación ni analítica global.
- B. Lo de A + gestión de suscripciones/planes de las tiendas (cobro, estado de pago).
- C. Lo de B + métricas y analítica global de la plataforma (GMV total, tiendas activas, etc.).
- D. Solo lo mínimo para operar los pilotos (crear y suspender tiendas manualmente).
- E. Not yet defined.
- X. Other (please specify)

[Answer]:X. A más lo siguiente: un campo de plan y estado de pago por tienda, que yo actualizo manualmente (sin cobro automático), y un panel con métricas básicas (número de tiendas activas, órdenes totales y por tienda). No incluir en el MVP analítica avanzada ni GMV global consolidado, ni facturación o cobro automático de suscripciones. Estas capacidades quedan para una fase posterior como parte de planes de mayor valor. (Precisión ronda 1) Si el super-admin ejerce soporte accediendo a una tienda, ese acceso debe ser de SOLO LECTURA y quedar registrado en un log de auditoría. No modifica datos de la tienda desde el acceso de soporte.

## Q17. "10 tiendas piloto operando" (de Q3): ¿es métrica de éxito o requisito funcional?

- A. Es una **métrica de éxito/validación** del MVP (meta de adopción), no una funcionalidad a construir. El producto debe soportar múltiples tiendas; alcanzar 10 pilotos es la meta de validación.
- B. Es un **requisito funcional**: el MVP debe incluir herramientas específicas para onboarding/gestión de un programa de pilotos.
- C. Ambas: métrica de éxito y además algunas utilidades mínimas de onboarding de pilotos.
- D. Not applicable.
- X. Other (please specify)

[Answer]:A. Es una métrica de éxito, no una funcionalidad. (Precisión ronda 1) Una tienda está "operando" cuando, dentro de los 14 a 30 días posteriores a publicar su landing, tiene al menos un producto con stock y al menos una orden pagada (que puede ser una compra real del propio comerciante), sin intervención manual del owner en el proceso de compra. Meta: 10 tiendas en ese estado. El producto debe soportar múltiples tiendas; esto es métrica de validación, no una funcionalidad de onboarding de pilotos a construir.

---

## Requested Changes Feedback (ronda 1)

Precisiones del usuario al aprobar el resumen:
1. Fuera de alcance del MVP, además de lo ya listado: dominio propio del comercio, comisión por venta, cobro recurrente automático de suscripciones, analítica avanzada y GMV global.
2. En interesados, ventas y soporte son hoy el mismo owner de producto (yo); no asumir equipos separados.
3. Si el super-admin ofrece soporte con acceso a una tienda, debe ser de solo lectura y con registro de auditoría.
4. Meta de pilotos: una tienda está "operando" cuando, dentro de los 14 a 30 días posteriores a publicar su landing, tiene al menos un producto con stock y al menos una orden pagada (puede ser una compra real del propio comerciante), sin intervención manual del owner en la compra. Meta: 10 tiendas en ese estado.
5. Las tarifas de pasarelas candidatas deben verificarse con fuentes oficiales vigentes en la etapa de diseño.

[Answer]: Looks correct

## Requested Changes Feedback (ronda 2)

Correcciones de decisiones del usuario:
1. La plataforma no retiene fondos, por lo que la conciliación bancaria queda fuera del alcance. La métrica de confiabilidad de pagos pasa a ser la consistencia entre el estado de la orden y el estado del pago (idempotencia ante webhooks duplicados o tardíos), no una conciliación contra extractos bancarios.
2. Agregar en Q7 un método de pago manual que convive con la pasarela: el comprador sube el comprobante de una transferencia y un administrador de la tienda aprueba o rechaza el pago tras confirmarlo en su banco.
3. Mantener los artefactos de ideación a nivel de problema, sin detalles de implementación, y dejar coherentes intent statement, alcance, supuestos y riesgos.

[Answer]: Looks correct

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
