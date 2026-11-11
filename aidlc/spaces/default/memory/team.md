# Team-Level Rules

> This team's affirmed practices and corrections. Loaded after `org.md` as
> strict-additive guidance; contradictions with broader policy are rejected.
> Populated by the practices-discovery affirmation gate. Edit at the gate,
> not directly.

## Way of Working

Trabajamos con **trunk-based development**: una sola rama de integración, `main`, que es a la vez la base y el destino de merge de todo el trabajo.

- Base del worktree de Construcción: `main`. Destino de merge: `main`.
- Cada Bolt se integra a `main` con **squash-merge** (un único commit por Bolt, nombrado por el slug del Bolt), para mantener un historial lineal de `main` que mapea 1:1 con la secuencia de Bolts de delivery-planning. Dimensionamos cada Bolt como un corte verificable end-to-end o una unidad/sub-slice coherente, para que el commit único de `main` siga siendo bisectable.
- Ramas de feature cortas, que se resuelven **en una o pocas sesiones de trabajo** (el owner trabaja a dedicación parcial, así que medimos la vida de la rama por sesiones de trabajo más que por días de calendario); nada de ramas de larga vida. El trabajo incompleto que deba convivir en `main` se protege con feature flags **solo cuando hagan falta** (no por defecto): si una rama corta se resuelve dentro de su ventana, no se introduce una flag solo por costumbre.
- Los commits que se piden van donde ya estás, en la rama en que estás; el agente crea una rama solo cuando se pide explícitamente o cuando esta sección lo indique.
- Un único entorno troncal: dev primero, luego producción (ver `## Deployment`); las promociones entre entornos se gobiernan con tags/config de despliegue por entorno, no con ramas de release de larga vida.

Encaje con el proyecto: la construcción es **una unidad a la vez (unit-major)**, por lo que habrá a lo sumo un Bolt en vuelo; trunk-based + squash es el ajuste más limpio para ese ritmo.

**Estructura de repositorio: monorepo.** Mantenemos un único repositorio con `modules/<unidad>/` por unidad (coherente con la estructura ya fijada en Scope Definition): un solo historial, un solo pipeline y refactors atómicos que cruzan unidades, lo que encaja con owner único + agente y evita la sobrecarga de coordinación de un repo por unidad. Fijamos temprano el esqueleto de directorios: `modules/<unidad>/` para IaC Terraform y código de aplicación por unidad.

**Specs OpenAPI versionadas como fuente de verdad de los contratos entre unidades.** Las specs OpenAPI viven versionadas en el repo (p. ej. `docs/openapi.*`) y se validan en CI; son el contrato autoritativo de las interfaces entre unidades y entre backend y frontend, no las implementaciones internas.

## Walking Skeleton

**Sí: construimos primero un walking skeleton** — un corte mínimo end-to-end que recorre todo el flujo (de la petición del comprador hasta la respuesta persistida) antes de incorporar las features reales, para probar que las piezas conectan.

- Es la primera unidad del DAG de Unidades: el corte integrado más pequeño que funciona, pasado por todas las etapas por-unidad aplicables, incluida Code Generation, antes que las unidades posteriores aun cuando se elija orden stage-major.
- Debe ejercitar y validar temprano los riesgos críticos reconocidos en Ideación: reserva de stock bajo concurrencia (R1), consistencia orden↔pago (R2) y aislamiento de datos multi-tenant (R3).
- **Compuerta de salida del skeleton:** las pruebas de riesgo R1-R3 end-to-end **en verde** son la condición de salida del walking skeleton y preceden al checkpoint humano. Una revisión de etapa de diseño **no** demuestra el skeleton; las pruebas ejecutadas y en verde sí. Build and Test no puede dar el skeleton por cumplido sin esa evidencia.
- Se verifica con el Construction Verification Command registrado y autorizado por el humano, que demuestra el resultado end-to-end real del proyecto.
- Los **checkpoints se revisan de forma guiada (gated) hasta validar el skeleton**. Recién tras el skeleton verificado y aprobado por el humano se ofrece la elección explícita de autonomía de Construcción (continuar automáticamente vs. revisar cada checkpoint). Plan Approval, confirmación de resumen, selección del comando de verificación y los fallos siguen siendo paradas humanas en todo caso.

Método de prueba de los riesgos del skeleton (ancla para diseño; herramienta concreta por fijar con el stack):
- **R1 — Reserva de stock / concurrencia:** prueba de integración con N reservas concurrentes sobre un SKU con stock M (N>M) que verifica que exactamente M tienen éxito sin sobreventa ni stock negativo, más la liberación por vencimiento de reservas no confirmadas. Es una prueba de concurrencia (escritura condicional/transaccional de DynamoDB), no un unitario con mocks. **La expiración de reservas NO debe depender del TTL de DynamoDB**: el borrado por TTL es diferido (puede tardar y no es puntual), así que una reserva vencida seguiría consumiendo stock mientras el ítem TTL no se elimina. La liberación por vencimiento se maneja con **lógica explícita** (una reserva lleva su `expiresAt` y se trata como vencida/liberable por esa marca en la ruta de lectura y en un barrido/reclamo explícito), no delegando el vencimiento al TTL; la prueba debe ejercitar esa liberación explícita, no asumir que el TTL la realiza a tiempo.
- **R2 — Idempotencia de pagos / consistencia orden↔pago:** reenvío del mismo webhook (mismo id de evento) con un único efecto sobre la orden, y webhooks fuera de orden/tardíos sin estados inconsistentes; validar la clave de idempotencia y que los estados terminales no retrocedan. Cubrir también el pago manual (aprobación/rechazo del comprobante).
- **R3 — Aislamiento multi-tenant:** pruebas negativas de autorización que, autenticado como tenant A, intentan leer/escribir recursos de tenant B por todas las rutas y verifican negación sistemática y que ninguna consulta omita el filtro de `tenant_id`. El aislamiento es un invariante de seguridad: una sola fuga lo invalida, así que lleva pruebas negativas explícitas, no solo del camino feliz.

## Testing Posture

Scope `feature`, greenfield. Metodología general **test-after** con una excepción afirmada para los módulos de dominio críticos; por combinar cadencias (test-after general con prueba escrita junto con la implementación en los módulos críticos) la metodología agregada se marca como `custom`.

- **Methodology**: custom
- **Ordering**: Test-after como cadencia general — se implementa cada capa comprobable y luego se escriben y ejecutan sus pruebas de unidad e integración; para los módulos de dominio críticos (reserva de stock, estados de orden, idempotencia de pagos y resolución de `tenant_id`) la prueba se escribe **junto con la implementación**, no después; los criterios de aceptación de cada historia se expresan en **Given/When/Then**; se exige un piso del **80% de cobertura de línea** global en CI antes del merge más **cobertura de ramas (branch)** en los módulos de dominio críticos; y las pruebas de riesgo R1-R3 en verde son compuerta de salida del walking skeleton antes del checkpoint humano.
- **Methodology evidence**: No es un valor puro porque mezcla dos cadencias. La general es `test-after`: se implementa la capa y luego se prueban unidad e integración, postura realista para un owner único a dedicación parcial. La excepción de los módulos de dominio críticos escribe la prueba junto con la implementación (cadencia de tipo test-alongside/ATDD), porque los escenarios Given/When/Then de reserva de stock, estados de orden, idempotencia de pagos y resolución de `tenant_id` deben derivar casos verificables que anclen la compuerta de salida del skeleton; tratarlos como test-after perdería ese anclaje. Los criterios de aceptación Given/When/Then al frente especifican el comportamiento antes de implementar (matiz BDD/ATDD en la especificación). La suma de una cadencia test-after general y una test-alongside en los módulos críticos es lo que hace que la metodología agregada sea `custom` y no `test-after`, `bdd` ni `atdd` puros.
- **Coverage floor (global)**: piso del **80% de cobertura de línea** con ejecución en **CI antes del merge** (piso aditivo del scope `feature`; no reduce ni reemplaza la estrategia de pruebas activa, solo fija un mínimo). Se mide sobre el código de dominio/aplicación propio; se excluyen con justificación registrada el IaC (Terraform no se cubre con unitarios de app), los adaptadores finos a servicios AWS (se cubren por integración contra LocalStack/testcontainers) y el glue trivial de handlers. Las exclusiones siguen política de pragmas (proveedor real, verificadas corriendo el comando de cobertura, con rationale); nunca se amplían solo para pasar la compuerta.
- **Branch coverage (módulos de dominio críticos)**: además del piso de línea, se mide **cobertura de ramas** en reserva de stock, máquina de estados de órdenes, idempotencia de pagos y resolución de `tenant_id`, donde las ramas condicionales son lo que puede fallar y la cobertura de línea se infla con glue. El alcance exacto y el umbral concreto se fijan en diseño.
- **Compuertas de calidad en CI**: gate de merge (todas las pruebas de unidad e integración en verde, cobertura no decrece y cumple el piso, linter sin fallos, escaneo de seguridad sin críticos/altos); gate del walking skeleton (R1-R3 end-to-end en verde como condición de salida, antes del checkpoint humano); gate de regresión (la suite existente permanece en verde en todo cambio posterior).
- **Suite existente**: cualquier cambio posterior mantiene la suite existente en verde; Build and Test verifica estos pisos y no se debilitan para hacer pasar un paso.
- **Herramientas de prueba**: por definir junto con el stack/framework en la fase de diseño (medidor de cobertura nativo del runtime con umbral en CI; para la integración contra DynamoDB/EventBridge/Step Functions, **LocalStack es solo un ejemplo y NO una elección hecha**: queda **sujeto a verificar su licencia y costo** antes de adoptarlo, con **DynamoDB Local como alternativa** si la licencia/costo de LocalStack no encaja; property-based testing recomendado para reserva de stock e idempotencia; mutation testing selectivo opcional en módulos de riesgo). La elección de herramienta no cambia la metodología ni el ordering afirmados aquí.

## Guard Policy

<!-- Affirmed by the team. Mode: strict, relaxed, or off. Strict here holds for every intent, whatever one person asks in chat; relaxed or off here applies to every intent whose policy came from its scope, unless project.md or the person sets another; changing this line changes it. A section under the retired Change Control heading, written by an earlier release, is still read. -->

## Deployment

**Desplegamos primero a dev; producción solo tras validar dev**, con aprobación manual del owner.

- Deploy a **dev** como entorno inicial y único punto de partida. El agente **PREPARA** código IaC/aplicación y **RECOMIENDA** los comandos a ejecutar, pero **NO despliega**: en el MVP el pipeline de CI **valida** (`terraform fmt`, `terraform validate`, pruebas, escaneos) y **NO ejecuta `terraform apply`**. Cualquier automatización de despliegue (CD) exige una **decisión y aprobación nuevas del owner**; no se asume ni se habilita por defecto.
- Promoción a **producción** gatillada por una **aprobación manual explícita del owner**, después de validar el comportamiento en dev (incluido el skeleton y los smoke/health checks aplicables).
- **El owner ejecuta `terraform apply` y `terraform destroy`**. El **agente nunca** ejecuta `apply`/`destroy`; solo corre `terraform fmt` y `terraform validate`. Toda intención de `apply` es una parada obligatoria previa (ver `discovered-rules.md`).
- Paradas obligatorias ante cambios en autenticación, aislamiento multi-tenant, pagos e IAM, ante fallos, y antes de cualquier `terraform apply`.
- **`allowed_account_ids` fijado en el provider de Terraform de cada ambiente**, de modo que un `apply` no pueda dirigirse a la cuenta equivocada: dev usa la cuenta tienda-dev `547887911593` y prd usa la cuenta tienda-prd `790152345727`. Cada ambiente declara su `allowed_account_ids` con el ID que le corresponde.
- **El owner revisa el plan de Terraform completo antes de cada `apply`**, con atención especial a los recursos marcados `destroy`/`replace` (un cambio de esa clase puede eliminar o recrear estado y datos, por lo que no se aprueba a la ligera).
- **La alarma de presupuesto debe estar activa ANTES del primer despliegue**: AWS Budgets con techo de **10 USD/mes en dev** y alerta al **80%** del techo. No se despliega a un ambiente sin su alarma de presupuesto activa.
- Un despliegue no está "hecho" hasta que los smoke/health checks confirmen el servicio sano en su entorno; todo despliegue debe tener una ruta de **rollback documentada**.

## Code Style

Diferimos la configuración por lenguaje al stack elegido en diseño, y fijamos ahora las convenciones que no dependen del stack para evitar que cada una de las nueve unidades las reinvente de forma divergente.

Base diferida al stack:
- **Formateador y linter por lenguaje** (p. ej. Prettier/ESLint para JS/TS, Black/Ruff para Python, `gofmt`/golangci-lint para Go), configurados en la raíz del repositorio y ejecutados en CI antes del merge; el fallo del linter bloquea el merge. El linter incluye su **plugin de seguridad** (p. ej. `eslint-plugin-security`, reglas S de `bandit`/`ruff`, `gosec`) y su fallo bloquea el merge igual que el de estilo (shift-left de bajo costo).
- **Nombres idiomáticos** del lenguaje (camelCase en JS/TS, snake_case en Python, etc.); sin reglas de renombrado a nivel de proyecto salvo que el equipo afirme una.
- **El código y los identificadores se escriben en inglés** (nombres de variables, funciones, tipos, archivos). La prosa de artefactos y la conversación con el humano siguen en español.
- Cuando el framework sugiera un ajuste de estilo, el agente lee primero la configuración del linter del proyecto; la sugerencia solo aplica si el linter no la cubre ya.

Convenciones independientes del stack (afirmadas ahora):
- **Organización por feature/unidad, no por capa.** Código relacionado co-localizado bajo `modules/<unidad>/`; cada módulo es candidato a extracción posterior. Se evita el anti-patrón de carpetas globales `models/`, `services/`, `handlers/`.
- **Envelope de error consistente en todas las Lambdas/handlers.** Un único sobre de error en los límites HTTP (`error.code` legible por máquina, `error.message` para humanos, `details[]` por campo, `requestId`/`correlationId`), con mapeo uniforme de códigos HTTP (400 validación de sintaxis, 401/403 auth/permisos, 404, 409 conflicto de estado, 422 regla de negocio, 429 rate limit). Es contrato cruzado entre unidades y debe ser idéntico desde la primera Lambda.
- **Validación en los límites de confianza (boundary validation).** Toda entrada que cruza un límite no confiable (handlers de API Gateway, consumidores de EventBridge/colas, webhooks de pasarela) se valida y se convierte a tipo de dominio una sola vez; el núcleo de dominio confía en el límite y no re-valida. El `tenantId` efectivo se resuelve y valida en el límite, nunca se toma de un campo de cuerpo no verificado.
- **Manejo de errores en Lambdas.** Fail-fast en la entrada del handler (validación de evento) y catch en el límite (el handler mapea a envelope + código HTTP; no se tragan excepciones en silencio); errores tipados de dominio para fallos esperados (p. ej. `StockInsuficiente`, `OrdenYaPagada`, `TenantNoAutorizado`) distinguidos de fallos fatales, habilitando mapeo determinista a 409/422; idempotencia explícita en consumidores (webhooks de pago, eventos) como parte del contrato del handler.
- **Logging estructurado (JSON) con `correlationId`.** Log en fronteras de operación (inicio/éxito/fallo), nunca dentro de bucles; prohibido loguear secretos, PII, tokens o comprobantes de pago. El `correlationId` permite trazar una orden a través de Lambda→EventBridge→Step Functions.
- **Convenciones de nombres estables e independientes del lenguaje:** rutas REST en plural y kebab-case sin verbos (acciones no-CRUD como `POST /orders/{id}/cancel`), IDs estables (`tenantId`, `orderId`), y en el frontend `data-testid` en kebab-case `{componente}-{rol}` para soportar E2E/accesibilidad (objetivo WCAG 2.1/2.2 AA de los wireframes). El patrón de clave DynamoDB (prefijos `PK=TENANT#...`, `SK=...`) se muestra aquí como **ilustrativo** y **no es una decisión de modelo de datos tomada en esta etapa**: queda **sujeto a que la fase de diseño (data-modelling) confirme el modelo de aislamiento multi-tenant** (single-table vs. otras opciones, forma exacta de PK/SK e índices). Lo que se afirma ahora es el valor de acordar temprano una convención de claves transversal; su detalle y su validación corresponden a data-modelling en diseño.

**Regla de frontera entre unidades.** Con nueve unidades construidas en secuencia, fijamos cómo se comunican y qué puede depender de qué:
- **Sin acoplamiento a la base de datos entre unidades.** Una unidad no lee ni escribe el almacenamiento de otra directamente; cruza por contrato explícito (API interna o evento). En single-table DynamoDB, cada unidad posee sus prefijos de clave y nadie consulta fuera de los suyos.
- **Dirección de dependencias alineada al orden de construcción.** `tenants`+`auth` son la base; `inventory` antes de `orders`; `payments` consume órdenes vía evento/contrato, no al revés. Las dependencias van hacia unidades ya construidas, nunca ciclos.
- **`storefront` y `frontend` consumen contratos, no internos.** Dependen de los contratos OpenAPI publicados por el backend, no de sus implementaciones.
- **Resolución de `tenant_id` como preocupación transversal única.** El aislamiento multi-tenant (R3) se resuelve en un único punto (middleware/límite compartido), no se re-implementa por unidad; de lo contrario cada unidad es una superficie de fuga independiente.
## Forbidden

<!-- Team-specific forbidden patterns -->

## Mandated

<!-- Team-specific mandates -->

## Corrections

<!-- Self-learning loop appends here. -->
- ALWAYS verificar tarifas, costos y datos de proveedores externos (p. ej. pasarelas de pago) con fuentes oficiales vigentes en la etapa de diseño, en lugar de darlos por supuestos (learned 2026-11-09) <!-- cid:261106-multi-tenant-store:intent-capture:bf9ce3f79339019edbd4f5ccab95cbb49906814d3ffa80950695b5a39e8a4853 -->
- ALWAYS el acceso de soporte a datos de una tienda o cliente debe ser de solo lectura y quedar registrado en auditoría (learned 2026-11-09) <!-- cid:261106-multi-tenant-store:intent-capture:293ae9f524d4cf18fe7b5a6f394b1eab68dafe3b8163563353c9cbfcfee858da -->
