# Units Generation — Plan de descomposición y preguntas

## Sources

- [desc] Initial description: "Dividir el MVP en unidades independientes y proponer el orden de construcción; AWS serverless; monorepo con modules/<unidad>/."
- [scope] Workflow-selected scope: `feature`.

---

> Objetivo de esta etapa: agrupar los componentes de dominio en **unidades de trabajo** independientes (testeable y desplegable por separado), definir el **DAG de dependencias** (topología) y mapear historias→unidad. **No** se decide aquí el orden económico de construcción ni el "camino crítico" (eso es Delivery Planning, 2.9); esta etapa dice qué *puede* depender de qué.
>
> Punto de partida ya acordado upstream (se confirma, no se re-discute): Scope Definition aprobó **nueve unidades** (tenants, auth, catalog, inventory, orders, payments, admin, storefront, frontend); Domain Design definió 8 componentes de dominio backend + NotificationComponent (lógico) + TenantContext (transversal); team-practices fijó monorepo con `modules/<unidad>/`, contratos OpenAPI entre unidades, sin acoplamiento a BD, `tenant_id` transversal, dependencias alineadas al orden; walking-skeleton-first con R1/R2/R3 como compuerta.
>
> Responde cada pregunta en su línea `[Answer]:`. Puedes elegir una opción y matizarla; si prefieres que decida yo, dilo ahí.

## Q1. Correspondencia componentes → unidades de trabajo

¿Cómo mapeamos los componentes de dominio a unidades de construcción?

- A. Una unidad por unidad de negocio aprobada, con IDs y directorios explícitos (nombres en minúscula, path-segment): `u1-tenants`, `u2-auth`, `u3-catalog`, `u4-inventory`, `u5-orders`, `u6-payments`, `u7-admin`, `u8-storefront`, `u9-frontend`. Cada unidad backend contiene su componente de dominio homónimo; `u9-frontend` es la unidad de UI que consume contratos. Se mantienen las nueve unidades aprobadas.
- B. Agrupar algunas unidades (menos unidades, más gruesas).
- C. Separar más (p. ej. dividir alguna unidad en dos).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:[Answer]: A. Mantener exactamente las nueve unidades aprobadas en Scope Definition, con identificadores y directorios explícitos:

u1-tenants

u2-auth

u3-catalog

u4-inventory

u5-orders

u6-payments

u7-admin

u8-storefront

u9-frontend

Cada unidad backend tendrá la responsabilidad de su componente de dominio correspondiente y será propietaria de sus entidades y contratos.

u8-storefront representará la experiencia pública de la tienda, sus plantillas y la presentación del catálogo para los compradores.

u9-frontend representará las interfaces administrativas del comercio y de la plataforma, consumiendo los contratos expuestos por las unidades backend, principalmente mediante AdminComponent y AuthComponent.

NotificationComponent se mantendrá como capacidad lógica interna, sin crear una décima unidad de construcción. TenantContext será una capacidad transversal para resolver y validar el contexto del tenant, no una unidad independiente.

Los contratos OpenAPI y los contratos de eventos se mantendrán versionados en el repositorio como artefactos compartidos, sin crear una unidad adicional para ellos.

## Q2. Ubicación física de NotificationComponent (pendiente de ADR-001)

`NotificationComponent` es una capacidad lógica, no una décima unidad. ¿Dónde vive físicamente?

- A. Dentro de `u6-payments` como módulo de notificación reactivo a eventos de orden/pago (es el consumidor natural de los eventos de pago y de los avisos de comprobante del flujo manual Must), exponiendo su envío por eventos sin acoplar a orders. No crea una unidad nueva.
- B. Dentro de `u5-orders`.
- C. Dentro de `u7-admin`.
- D. Como un módulo compartido ligero reutilizable por orders y payments (sin unidad propia).
- E. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Ubicar NotificationComponent como módulo lógico interno de u6-payments, sin convertirlo en una unidad independiente.

Esta ubicación se justifica porque uno de los avisos indispensables del MVP corresponde al flujo de revisión de comprobantes de pago manuales. El módulo podrá consumir eventos publicados por OrderComponent y PaymentComponent, incluyendo la creación de órdenes y los cambios relevantes en el estado de los pagos.

Su responsabilidad será procesar y enviar notificaciones transaccionales, registrar el resultado mínimo necesario para trazabilidad y evitar envíos duplicados mediante idempotencia.

No será propietario de órdenes, pagos, comprobantes ni identidades. Tampoco realizará llamadas síncronas directas para modificar entidades de otros componentes. La comunicación se realizará mediante contratos de eventos versionados.

La ubicación lógica dentro de u6-payments no deberá generar dependencias circulares ni modificar la propiedad de entidades establecida en Domain Design.

## Q3. Granularidad de las unidades

¿Qué granularidad prefieres?

- A. La granularidad de las nueve unidades aprobadas (ni más gruesa ni más fina): cada una es desplegable/testeable por separado y cohesiva por dominio, coherente con el ritmo unit-major (una unidad a la vez) y el monorepo.
- B. Más gruesa (fusionar para menos unidades).
- C. Más fina (sub-dividir unidades grandes).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Mantener la granularidad de las nueve unidades aprobadas, sin fusionarlas ni subdividirlas en esta etapa. Cada unidad deberá tener una responsabilidad cohesiva, contratos explícitos y capacidad de prueba y despliegue independiente conforme a su naturaleza (service o ui).

La organización deberá ser coherente con el monorepo, la estructura modules/<unidad>/ y la estrategia de construcción por unidad.

La independencia de las unidades no implica duplicar infraestructura, contratos, lógica de negocio o mecanismos transversales. Las dependencias entre ellas se resolverán mediante interfaces y contratos versionados.

## Q4. Modelo de despliegue (topología)

El proyecto es AWS serverless. ¿Qué modelo de despliegue proponemos por unidad?

- A. **Serverless por unidad**: cada unidad backend se despliega como su propio conjunto de funciones/recursos (Lambda + API Gateway + tablas propias) bajo `modules/<unidad>/` con IaC Terraform; `u8-storefront` y `u9-frontend` como frontend estático servido por CDN. Independiente por unidad, coherente con C1 (serverless) y pago-por-uso; el detalle fino de servicios se cierra en Infrastructure Design.
- B. Monolito serverless (una sola función/stack para todo el backend).
- C. Híbrido (algunas unidades juntas, otras separadas).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Adoptar un enfoque serverless por unidad backend, con responsabilidades y límites de despliegue independientes, utilizando Terraform y la estructura de módulos definida para el proyecto.

Las unidades backend (u1-tenants a u7-admin) deberán poder evolucionar, probarse y desplegarse independientemente, respetando la propiedad de sus entidades y la ausencia de acoplamiento directo a bases de datos de otras unidades.

u8-storefront y u9-frontend corresponderán a superficies web diferenciadas: la tienda pública para compradores y las interfaces administrativas para comercios y super-admin, respectivamente. Se contempla su entrega como aplicaciones web estáticas mediante infraestructura CDN.

No fijar todavía una API Gateway, tabla DynamoDB, función Lambda o distribución CDN exclusiva por unidad. La selección y agrupación concreta de recursos AWS, así como el modelo físico de datos, se resolverán en Infrastructure Design y data modelling, manteniendo la independencia lógica y los límites de propiedad definidos.

## Q5. Kind de cada unidad (qué ES cada unidad)

El `kind` determina qué artefactos de diseño de Construcción aplican. ¿Confirmas esta asignación?

- A. `u1-tenants`, `u2-auth`, `u3-catalog`, `u4-inventory`, `u5-orders`, `u6-payments`, `u7-admin` = `service` (ejecutables desplegados); `u8-storefront` y `u9-frontend` = `ui` (superficies frontend). Nota: los contratos OpenAPI entre unidades viven versionados en el repo como fuente de verdad, pero no se modela aquí una unidad `spec` separada salvo que lo prefieras (ver opción B).
- B. Añadir una unidad `spec` separada que posea los contratos OpenAPI compartidos (`kind: spec`), de la que dependan las demás.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Confirmar la siguiente asignación:

u1-tenants = service

u2-auth = service

u3-catalog = service

u4-inventory = service

u5-orders = service

u6-payments = service

u7-admin = service

u8-storefront = ui

u9-frontend = ui

Los contratos OpenAPI y de eventos se mantendrán versionados en el repositorio como fuentes de verdad compartidas, sin crear una unidad independiente de tipo spec.

Las unidades ui consumirán los contratos publicados por las unidades service; no accederán directamente a sus almacenes de datos ni incorporarán lógica que corresponda a los componentes propietarios.

## Q6. Primera unidad = walking skeleton (corte integrado mínimo)

El walking-skeleton-first exige que la primera unidad del DAG sea el corte end-to-end mínimo que corre y valida R1/R2/R3. Las historias 🟢 del skeleton cruzan varias unidades (tenants, auth, catalog, inventory, orders, payments, storefront). ¿Cómo lo modelamos?

- A. Modelar el walking skeleton como una **unidad de corte integrado** `u0-walking-skeleton` (`kind: service`) que atraviesa el flujo mínimo end-to-end (crear tienda → activar → cargar producto con stock → publicar → comprar con reserva → pago manual aprobado), implementando lo mínimo de cada dominio para ejercitar R1/R2/R3, y de la que las nueve unidades "completas" dependen o que luego se profundiza por unidad. Es la primera unidad del DAG resuelto y puede correr antes que las demás.
- B. No crear una unidad de skeleton separada: marcar `u1-tenants` (o la base) como primera y tratar el skeleton como un hito dentro de las primeras unidades.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: B. No crear una unidad independiente u0-walking-skeleton, porque el walking skeleton no es un componente de dominio ni una unidad adicional de construcción. Es un corte vertical integrado que atraviesa las nueve unidades aprobadas.

El walking skeleton deberá implementarse como un flujo end-to-end mínimo, utilizando las capacidades esenciales de las unidades involucradas: creación y activación de una tienda, creación y publicación de un producto con stock, compra con reserva de inventario y aprobación de pago manual.

Se deberá construir e integrar tempranamente, sin duplicar código ni trasladar la propiedad de entidades entre unidades. Cada funcionalidad mínima deberá residir en su unidad propietaria y comunicarse mediante los contratos correspondientes.

El walking skeleton será un hito de integración y validación temprana de R1, R2 y R3, no una unidad del DAG ni una dependencia artificial de las nueve unidades completas. Las pruebas críticas deberán ejecutarse sobre el flujo integrado y mantenerse como condición obligatoria de validación.

## Q7. Estructura del DAG de dependencias (topología, no orden económico)

¿Confirmas esta topología de dependencias (coherente con team-practices: tenants+auth base; inventory antes de orders; payments consume órdenes)?

- A. Sí: `u0-walking-skeleton` sin dependencias (corte mínimo primero). Luego, para las unidades completas: `u1-tenants` y `u2-auth` base (dependen del skeleton como cimiento integrado); `u3-catalog` depende de tenants+auth; `u4-inventory` depende de catalog; `u5-orders` depende de catalog+inventory; `u6-payments` depende de orders; `u7-admin` depende de las que coordina (tenants, catalog, inventory, orders, payments, auth); `u8-storefront` depende de catalog+inventory+tenants; `u9-frontend` depende de los contratos (storefront, orders, payments, admin). Sin ciclos. (El orden económico de construcción lo decide Delivery Planning, no aquí.)
- B. Ajustar la topología (dime qué cambiar).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: B. Definir un DAG de dependencias sin ciclos síncronos, respetando la propiedad de entidades, el orden de construcción y las fronteras acordadas en Domain Design.

La topología de referencia será:

u1-tenants: unidad base, sin dependencia funcional de otras unidades.

u2-auth: depende de u1-tenants para consultar el estado de activación del tenant, conforme al ADR-008.

u3-catalog: depende de u1-tenants y u2-auth para operar dentro del contexto de tienda y aplicar autorización en las operaciones administrativas.

u4-inventory: depende de u3-catalog para gestionar el inventario asociado a productos y variantes, además del contexto transversal de tenant y autorización.

u5-orders: depende de u3-catalog y u4-inventory para consultar productos y gestionar reservas durante el checkout; utiliza el contexto de tenant y autorización correspondiente.

u6-payments: depende de u5-orders para asociar los pagos con órdenes existentes y validar las condiciones de procesamiento.

u7-admin: depende de los contratos de u1-tenants, u2-auth, u3-catalog, u4-inventory, u5-orders y u6-payments, ya que coordina las operaciones administrativas de comercio y plataforma.

u8-storefront: depende de u1-tenants, u3-catalog y u4-inventory para resolver la tienda, mostrar el catálogo y consultar disponibilidad.

u9-frontend: depende principalmente de u7-admin y u2-auth para proporcionar las interfaces administrativas y gestionar las operaciones permitidas mediante los contratos publicados.

Los eventos de pago consumidos por órdenes, y los eventos de órdenes o pagos consumidos por notificaciones, se documentarán como interacciones asíncronas independientes. No deberán interpretarse como dependencias síncronas ni generar ciclos de llamadas entre unidades.

El DAG deberá distinguir explícitamente las dependencias de construcción y ejecución síncrona de las relaciones asíncronas por eventos. La comunicación por eventos deberá preservar idempotencia, consistencia y trazabilidad.

Esta topología describe las dependencias posibles entre unidades, no el orden económico de construcción ni la secuencia definitiva de los Bolts, que corresponde a Delivery Planning.

## Q8. Paralelismo permitido entre unidades independientes

¿Quieres que el DAG marque explícitamente las unidades sin dependencia entre sí como construibles en paralelo (aunque el ritmo del equipo sea unit-major)?

- A. Sí, documentar las oportunidades de paralelismo (p. ej. storefront y admin no dependen entre sí) como información para Delivery Planning, aunque en la práctica se construya una unidad a la vez. No altera el ritmo unit-major; solo informa qué órdenes topológicas son válidas.
- B. No documentar paralelismo (ritmo estrictamente secuencial).
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Documentar explícitamente las oportunidades de paralelismo que permite el DAG, sin modificar la estrategia operativa unit-major de construir y validar una unidad a la vez.

Por ejemplo, una vez disponibles las dependencias requeridas, u7-admin y u8-storefront podrán desarrollarse de manera independiente, ya que cumplen responsabilidades distintas y consumen contratos de unidades base compartidas.

También podrán identificarse otras unidades sin dependencias directas entre sí, siempre que sus contratos estén definidos y versionados.

Este paralelismo será una propiedad de la topología y una entrada para Delivery Planning; no implica que deban ejecutarse trabajos simultáneos ni que se modifique el ritmo de construcción aprobado.

## Assumptions & Open Questions

- [assumption] El **orden económico de construcción** y el "camino crítico" NO se deciden aquí; esta etapa fija la topología (DAG) y Delivery Planning (2.9) elige la secuencia de Bolts. [stage]
- [assumption] El detalle fino de servicios AWS por unidad y el modelo físico de datos (claves DynamoDB, OQ2) se cierran en Infrastructure Design / data-modelling, no aquí. [team-practices] [requirements OQ2]
- OQ-US-1 — Mecanismo de consulta segura de la orden del comprador invitado: permanece abierto; no se resuelve en la descomposición de unidades.

## Plan Approval

[Answer]: Approve Plan

## Consolidated Summary Confirmation

Resumen del plan de descomposición aprobado (Q1–Q8). Confírmalo para generar los artefactos.

**Nueve unidades (sin décima; NotificationComponent es módulo lógico dentro de u6-payments):**
- `u1-tenants` (service), `u2-auth` (service), `u3-catalog` (service), `u4-inventory` (service), `u5-orders` (service), `u6-payments` (service, incluye el módulo de notificaciones), `u7-admin` (service), `u8-storefront` (ui), `u9-frontend` (ui).
- TenantContext = capacidad transversal (en u2-auth), no unidad. Contratos OpenAPI/eventos versionados como artefactos compartidos, sin unidad `spec`.

**Despliegue:** serverless por unidad bajo `modules/<unidad>/` con Terraform; u8/u9 como frontend estático por CDN. Detalle fino de servicios → Infrastructure Design.

**Walking skeleton:** NO es una unidad; es un corte vertical integrado / hito de validación temprana que atraviesa las unidades propietarias, con R1/R2/R3 en verde como condición obligatoria.

**DAG de construcción (acíclico):** u1-tenants (base); u2-auth→u1; u3-catalog→u1,u2; u4-inventory→u3; u5-orders→u3,u4; u6-payments→u5; u7-admin→u1,u2,u3,u4,u5,u6; u8-storefront→u1,u3,u4; u9-frontend→u2,u7.

**Relaciones asíncronas (no aristas de construcción, no ciclos):** resultado de pago (u6)→orden (u5) por evento; eventos de orden/pago→notificaciones (módulo en u6); idempotentes.

**Paralelismo:** documentado como propiedad de la topología (p. ej. u7-admin y u8-storefront independientes entre sí), sin alterar el ritmo unit-major.

**Fuera de esta etapa:** orden económico/Bolts → Delivery Planning; servicios AWS finos y modelo de datos → Infrastructure Design / data-modelling. OQ-US-1 abierta.

¿Esto refleja correctamente el plan de unidades que quieres? (`Looks correct` / `Request changes`)

[Answer]:Looks correct
