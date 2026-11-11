**Collaborator:** aidlc-developer-agent

## Contribution

Revisión ciega del borrador del líder desde el foco de desarrollo: convenciones de nombres, límites de capas, manejo de errores, organización de archivos y estilo de código. En conjunto el borrador es sólido y coherente con las decisiones de Ideación (unit-major, walking skeleton primero, dev→prod con compuerta manual, trunk-based + squash por Bolt). Las observaciones abajo no contradicen nada afirmado; proponen fijar temprano convenciones estables que hoy quedan diferidas y que, por ser independientes del stack, conviene no posponer.

### 1. `## Way of Working` (trunk-based, squash por Bolt, unit-major) — adecuado

Trunk-based con base/destino `main` y squash-merge por Bolt es el ajuste correcto para un solo desarrollador + agente con construcción unit-major: a lo sumo un Bolt en vuelo, sin coordinación de ramas de larga vida, historial lineal de `main` que mapea 1:1 con la secuencia de Bolts. Las feature flags para trabajo incompleto en `main` son la mitigación correcta ante la ausencia de ramas largas. No veo riesgo en esta sección para el perfil del equipo.

Matiz menor (no bloqueante): con squash por Bolt, cada Bolt debería corresponder a un corte verificable end-to-end o a una unidad/sub-slice coherente, para que el commit único de `main` siga siendo bisectable. Vale la pena que delivery-planning lo tenga presente al dimensionar Bolts; no es una regla de esta etapa.

### 2. `## Code Style` diferido al stack — adecuado en lo diferible, pero incompleto en lo que NO depende del stack

Diferir formateador/linter/nombres idiomáticos al stack elegido en diseño es correcto: Prettier/ESLint, Black/Ruff, `gofmt`/golangci-lint son decisiones que dependen del lenguaje y no deben fijarse antes de elegirlo. La regla de código/identificadores en inglés y prosa en español también es acertada y estable.

El hueco es que varias convenciones **no dependen del stack** y, si no se fijan ahora, cada unidad las reinventará de forma divergente (nueve unidades construidas por separado es justo el escenario donde la deriva se acumula). Recomiendo añadir a `## Code Style` (o a una sección de convenciones de proyecto) estos acuerdos independientes del lenguaje, para afirmarlos en la entrevista:

- **Organización por feature, no por capa.** Alinea con la estructura `modules/` por unidad ya decidida en Scope Definition y con la guía de organización por feature (código relacionado co-localizado; cada módulo es candidato a extracción posterior). Evita el anti-patrón de carpetas globales `models/`, `services/`, `handlers/`.
- **Envelope de error consistente en todas las Lambdas/handlers.** Un único sobre de error en los límites HTTP (`error.code` legible por máquina, `error.message` para humanos, `details[]` por campo, `requestId`/`correlationId`), con mapeo uniforme de códigos HTTP (400 validación de sintaxis, 401/403 auth/permisos, 404, 409 conflicto de estado, 422 regla de negocio, 429 rate limit). Esto es contrato cruzado entre `orders`, `payments`, `catalog`, etc., y debe ser idéntico desde la primera Lambda, no negociado unidad por unidad.
- **Validación en los límites de confianza (boundary validation).** Validar toda entrada que cruza un límite no confiable — handlers de API Gateway, consumidores de EventBridge/colas, webhooks de pasarela — y convertir a tipo de dominio una sola vez; el núcleo de dominio confía en el límite y no re-valida. Crítico para `payments` (webhooks idempotentes R2) y para el aislamiento multi-tenant (R3): el `tenantId` efectivo debe resolverse y validarse en el límite, nunca tomarse de un campo de cuerpo no verificado.
- **Logging estructurado (JSON) con `correlationId`.** Nivel de log en fronteras de operación (inicio/éxito/fallo), nunca dentro de bucles; prohibido loguear secretos/PII/tokens/comprobantes de pago. En serverless, el `correlationId` es lo que permite trazar una orden a través de Lambda→EventBridge→Step Functions.
- **Convenciones de nombres estables e independientes del lenguaje:** rutas REST en plural y kebab-case sin verbos (acciones no-CRUD como `POST /orders/{id}/cancel`), IDs estables (`tenantId`, `orderId`), y en el frontend `data-testid` en kebab-case `{componente}-{rol}` para soportar E2E/accesibilidad (el objetivo WCAG 2.1/2.2 AA de los wireframes lo pide). El patrón de clave DynamoDB (prefijos `PK=TENANT#...`, `SK=...`) es una convención transversal que también gana con un acuerdo temprano, aunque su detalle corresponda a data-modelling en diseño.

### 3. Límites entre las 9 unidades — falta fijar la regla de frontera

El borrador trata el conjunto de unidades como ritmo de construcción (unit-major, un Bolt en vuelo) pero no fija **cómo se comunican y qué puede depender de qué**. Con nueve unidades y un agente construyéndolas en secuencia, la ausencia de una regla de frontera es el mayor riesgo de deriva de diseño. Propongo afirmar como convención (encaja en `## Way of Working` o `## Code Style`, o como regla mandada si el humano la considera dura):

- **Sin acoplamiento a la base de datos entre unidades.** Una unidad no lee/escribe el almacenamiento de otra directamente; cruza por contrato explícito (API interna o evento). En single-table DynamoDB esto significa delimitar qué prefijos de clave posee cada unidad y que nadie consulte fuera de los suyos.
- **Dirección de dependencias alineada al orden de construcción.** `tenants`+`auth` son la base; `inventory` antes de `orders`; `payments` consume órdenes vía evento/contrato, no al revés. Las dependencias deben ir hacia unidades ya construidas, nunca ciclos.
- **`storefront` y `frontend` consumen contratos, no internos.** El frontend incremental (y las plantillas de storefront) dependen de contratos de API publicados por las unidades de backend, no de sus implementaciones.
- **Resolución de `tenant` como preocupación transversal única.** El aislamiento multi-tenant (R3) debe resolverse en un punto (middleware/límite compartido), no re-implementarse por unidad; de lo contrario cada unidad es una superficie de fuga independiente.

### 4. Estructura de repositorio — decidir monorepo ahora

Scope Definition ya fijó una estructura `modules/` por unidad para la parte AWS, lo que implica **un único repositorio (monorepo)** y no un repo por unidad. El borrador no lo hace explícito y conviene afirmarlo en esta etapa porque condiciona rutas de CI, el squash por Bolt y la organización de código:

- Un **monorepo** es el ajuste correcto para owner único + agente: un solo historial, un solo pipeline, refactors atómicos cruzando unidades. Un repo por unidad multiplicaría la sobrecarga de coordinación sin beneficio para este equipo.
- Recomiendo fijar el esqueleto de directorios temprano: `modules/<unidad>/` para IaC Terraform por unidad (coherente con la estructura ya citada), código de aplicación organizado por feature/unidad, y specs OpenAPI versionadas en el repo (`docs/openapi.*`) validadas en CI como fuente de verdad de los contratos entre unidades.

### 5. Manejo de errores en Lambdas — fijar el patrón, no solo el envelope

Más allá del sobre de error de salida (punto 2), recomiendo afirmar el patrón interno para serverless:

- **Fail fast en la entrada del handler** (validación de evento) y **catch en el límite** (el handler es el lugar correcto para el catch-all que mapea a envelope + código HTTP); no tragar excepciones en silencio.
- **Errores tipados de dominio** para fallos esperados (p. ej. `StockInsuficiente`, `OrdenYaPagada`, `TenantNoAutorizado`) distinguidos de fallos fatales; esto habilita el mapeo determinista a 409/422 y es directamente relevante para R1/R2.
- **Idempotencia explícita en consumidores** (webhooks de pago, eventos): clave de idempotencia y manejo de reintentos como parte del contrato del handler, no como parche posterior — es la mitigación de R2.

## Positions

- AGREE: `## Way of Working` (trunk-based, base/destino `main`, squash por Bolt) — es el ajuste más limpio para owner único + agente con construcción unit-major y a lo sumo un Bolt en vuelo.
- AGREE: `## Walking Skeleton` sí, primero, verificado con el Construction Verification Command y gated hasta validarlo — ejercitar R1/R2/R3 temprano es la decisión de ingeniería correcta.
- AGREE: Diferir formateador/linter/nombres idiomáticos al stack elegido en diseño, y la regla de código en inglés / prosa en español — correcto y estable.
- AGREE: `## Deployment` dev→prod con compuerta manual del owner y `terraform apply`/`destroy` solo por el owner — alineado con el techo de costo y las paradas obligatorias.
- OBJECT: `## Code Style` está incompleto — difiere al stack convenciones que NO dependen del stack (organización por feature, envelope de error, validación en límites, logging estructurado con `correlationId`, nombres de rutas/IDs/`data-testid`). Con 9 unidades construidas por separado, no fijarlas ahora garantiza deriva; deben afirmarse en la entrevista.
- OBJECT: Falta una **regla de frontera entre las 9 unidades** (sin acoplamiento a BD cruzado, dirección de dependencias, consumo por contrato, resolución de tenant transversal única) — es el mayor riesgo de deriva de diseño en construcción unit-major y debería afirmarse aquí.
- OBJECT: Falta hacer explícita la **estructura de repositorio (monorepo con `modules/` por unidad)** — ya está implícita en Scope Definition; afirmarla ahora condiciona CI, squash por Bolt y organización de código.
- OBJECT: Falta el **patrón de manejo de errores en Lambdas** (fail-fast + catch en el límite, errores tipados de dominio, idempotencia explícita en webhooks/eventos) — es independiente del stack y mitiga directamente R1/R2; conviene fijarlo temprano, no unidad por unidad.
