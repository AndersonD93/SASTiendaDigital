**Collaborator:** aidlc-quality-agent

## Contribution

Revisión ciega del borrador del líder desde la perspectiva de QA, centrada en la postura de pruebas, las herramientas de cobertura, las compuertas de calidad en CI, los patrones de prueba y los vacíos que la entrevista debe cerrar. El borrador es sólido y coherente con el contexto de Ideación (riesgos R1-R3, 9 unidades, walking skeleton primero); mis aportes buscan hacerlo ejecutable y no debilitable en Construcción.

### Metodología `custom` (BDD Given/When/Then al frente + test-after de unidad/integración)

Es la clasificación correcta para este MVP y la regla del framework la respalda explícitamente: cuando se mezclan cadencias (escenarios dirigidos por comportamiento antes de implementar + pruebas de bajo nivel después), el valor agregado debe ser `custom`, no un valor puro. Que la especificación del comportamiento (criterios de aceptación en Given/When/Then por historia, exigidos en Inception) preceda a la implementación, y que las pruebas de unidad e integración se escriban inmediatamente después de cada capa comprobable, es una postura realista para un owner único a dedicación parcial (~8 h/semana): captura el valor de BDD (especificación ejecutable del comportamiento como contrato) sin imponer el costo de disciplina de un TDD red-green-refactor estricto en todas las capas.

Mi única reserva: el borrador deja `custom` y `test-after` como alternativas intercambiables para que el humano elija al afirmar. Desde QA recomiendo **afirmar `custom`** y no diluirlo a "`test-after` con BDD solo como formato", porque los escenarios Given/When/Then de las unidades de riesgo (orders, payments, aislamiento) deben derivar casos de prueba *antes* de implementar para que funcionen como criterio de aceptación verificable del walking skeleton; tratarlos como mero formato de redacción pierde ese anclaje. La entrevista debe resolver esta elección con una recomendación QA explícita, no como empate.

Un matiz sobre `Ordering` que conviene precisar: para las unidades que materializan R1-R3 (inventory/orders/payments), las **pruebas de riesgo end-to-end del skeleton deben existir y ejecutarse como parte de la verificación del skeleton antes de completar esas unidades**, no "después de implementar cada capa". El borrador ya lo insinúa en el bullet de pruebas del skeleton, pero el campo `Ordering` lo subordina a la cadencia general test-after; vale la pena que el `Ordering` afirmado diga, en una frase, que para las capas de riesgo la prueba de comportamiento precede o acompaña la implementación (cadencia ATDD) mientras el resto sigue test-after — que es, de hecho, lo que hace a esta metodología `custom`.

### Piso de 80% de cobertura de línea en serverless

El piso de 80% de cobertura de línea es razonable y está correctamente planteado como **piso aditivo del scope `feature`**, no como objetivo ni como techo. Dos precisiones que debería incorporar el borrador o resolver la entrevista:

1. **Cobertura de línea es señal necesaria pero no suficiente**, y en código serverless orientado a eventos tiende a inflarse: handlers de Lambda con mucho glue code (parsing de eventos, mapeo DTO) suben el porcentaje sin ejercitar la lógica de negocio real. Recomiendo **complementar el piso de línea con cobertura de rama (branch) en los módulos de dominio críticos** (reserva de stock, máquina de estados de órdenes, idempotencia de pagos, resolución de `tenant_id`), donde las ramas condicionales son precisamente lo que puede fallar. La branch coverage es más significativa que la de línea para estos módulos.

2. **Alcance de la medición**: el 80% debe medirse sobre el código de dominio/aplicación propio, **excluyendo con justificación** el código de infraestructura (Terraform no se cubre con tests unitarios de app), los adaptadores finos a servicios AWS (deben cubrirse por tests de integración contra LocalStack/testcontainers, no por unitarios con mocks que solo reflejan el mock), y el glue de handlers trivial. Esas exclusiones deben seguir la política de pragmas (proveedor real, verificadas corriendo el comando de cobertura, con rationale registrado), nunca ampliarse solo para pasar la compuerta.

### Herramientas de cobertura y de prueba

Correcto diferir la herramienta concreta al diseño (dependen del stack Node/TypeScript vs. Python aún no fijado). Para que esa decisión no quede huérfana, dejo la orientación QA que el diseño debe concretar: medidor de cobertura nativo del runtime con umbral configurado en CI (p. ej. c8/nyc para Node, coverage.py para Python), **integración con LocalStack o testcontainers para las pruebas de integración contra DynamoDB/EventBridge/Step Functions** (fakes realistas, no mocks de lo que no poseemos), y un corredor que emita JUnit XML para anotaciones de PR. Para los módulos de riesgo recomiendo además **property-based testing** (fast-check en TS / Hypothesis en Python) en reserva de stock e idempotencia, donde los casos por ejemplo pierden casos límite de concurrencia y de duplicación de eventos.

### Compuertas de calidad en CI

El borrador afirma el piso de cobertura en CI antes del merge y el bloqueo por linter; bien. Lo que falta hacer explícito como compuertas de calidad (QA las considera parte de la postura de pruebas, no solo de DevSecOps):

- **Gate de merge (pre-merge):** todas las pruebas de unidad e integración en verde, cobertura no decrece y cumple el piso, linter sin fallos, escaneo de seguridad sin críticos/altos (coordinar con DevSecOps). Coherente con el modelo de PR sobre `main` del squash-merge.
- **Gate del walking skeleton:** las pruebas de riesgo R1-R3 end-to-end **en verde** es condición de salida del skeleton y precede al checkpoint humano. Una revisión de etapa de diseño no demuestra el skeleton; las pruebas ejecutadas sí. El borrador ya lo dice en prosa — recomiendo elevarlo a compuerta nombrada para que Build and Test no pueda darlo por cumplido sin evidencia.
- **Gate de regresión:** la suite existente permanece en verde en todo cambio posterior (el borrador ya lo recoge; lo endoso).

### Cómo probar los riesgos críticos del skeleton (R1-R3)

Este es el núcleo del valor de QA en esta etapa y el borrador lo nombra pero sin método. Propongo, para que la entrevista y el diseño tengan un ancla:

- **R1 — Reserva de stock / concurrencia:** prueba de integración que lanza N reservas concurrentes sobre un SKU con stock M (N>M) y verifica que exactamente M tienen éxito y el resto se rechaza, sin sobreventa ni stock negativo; prueba de la **liberación por vencimiento** de reservas no confirmadas (expiración → stock vuelve a estar disponible). La condición de carrera se prueba con escritura condicional/transaccional de DynamoDB; property-based para variar N, M y timings. Esta es una prueba de concurrencia, no un unitario con mocks.
- **R2 — Idempotencia de pagos / consistencia orden↔pago:** prueba que **reenvía el mismo webhook** (mismo id de evento) varias veces y verifica un único efecto sobre la orden (una sola transición de estado, un solo registro), y webhooks **fuera de orden/tardíos** (p. ej. confirmación antes que creación, o duplicado tras timeout). Validar la clave de idempotencia y que estados terminales no retroceden. Cubrir también el pago manual: aprobación/rechazo del comprobante no produce estados inconsistentes.
- **R3 — Aislamiento multi-tenant:** prueba de autorización que, autenticado como tenant A, intenta leer/escribir recursos de tenant B por todas las rutas (API, consultas directas, claves de partición) y verifica **negación sistemática**; prueba de que ninguna consulta omite el filtro de `tenant_id`. Este es el caso donde recomiendo property-based / fuzzing de `tenant_id` y, en diseño, considerar pruebas negativas generadas por cada endpoint. El aislamiento es un invariante de seguridad: una sola fuga lo invalida, así que debe tener pruebas negativas explícitas, no solo del camino feliz.

Estas tres clases de prueba deben estar en verde como condición de salida del walking skeleton.

### Vacíos que la entrevista debe resolver

1. Confirmar `custom` vs. `test-after` (recomendación QA: `custom`, ver arriba).
2. Decidir si se adopta **branch coverage** adicional en módulos de dominio críticos (recomendado) y el alcance exacto del piso de 80% (qué se excluye con justificación).
3. Confirmar que las **pruebas de riesgo R1-R3 son compuerta de salida del skeleton** (no opcional).
4. El **stack/runtime** (Node/TS vs. Python) sigue abierto y determina herramientas concretas de prueba/cobertura — correctamente diferido a diseño, pero QA debe participar en esa elección porque condiciona LocalStack/testcontainers y property-based.
5. Postura sobre **mutation testing** en los módulos de riesgo (opcional, selectivo): no obligatorio para el MVP, pero vale registrar la intención para inventory/payments donde las aserciones débiles son más peligrosas.

## Positions

- AGREE: Metodología `custom` (BDD/ATDD Given/When/Then al frente + test-after de unidad/integración) — es la clasificación correcta del framework cuando se mezclan cadencias y encaja con el ritmo del owner a dedicación parcial.
- AGREE: Piso de 80% de cobertura de línea en CI antes del merge como piso aditivo del scope `feature`, no debilitable — razonable y correctamente planteado como piso, no como objetivo.
- AGREE: Walking skeleton primero con pruebas dirigidas a R1-R3 en verde antes de las unidades posteriores — endoso el enfoque riesgo-primero y la no equivalencia entre revisión de diseño y skeleton demostrado.
- AGREE: Suite existente en verde en todo cambio posterior y Build and Test verifica pisos sin debilitarlos — postura de regresión correcta.
- OBJECT: El `Ordering` afirmado subordina las pruebas de riesgo a la cadencia general test-after — para las capas R1-R3 la prueba de comportamiento debe preceder o acompañar la implementación (matiz ATDD); el `Ordering` debe decirlo en una frase, que es lo que hace `custom` a esta metodología.
- OBJECT: El piso de cobertura se expresa solo como cobertura de línea — recomiendo añadir cobertura de rama (branch) en los módulos de dominio críticos (reserva de stock, estados de orden, idempotencia de pagos, resolución de `tenant_id`), donde la línea se infla con glue de handlers y no prueba la lógica real.
- OBJECT: El borrador deja `custom` y `test-after` como alternativas de igual peso para el humano — QA recomienda afirmar `custom` explícitamente, porque degradar BDD a "solo formato" pierde el anclaje de los escenarios como criterio de aceptación verificable del skeleton.
- OBJECT: Falta nombrar las pruebas de R1-R3 como compuerta de salida del skeleton y especificar su método (concurrencia para R1, reenvío/duplicación de webhooks para R2, pruebas negativas de autorización cross-tenant para R3) — el borrador las menciona pero sin elevarlas a gate ni dar método, lo que permite que Build and Test las dé por cumplidas sin evidencia.
