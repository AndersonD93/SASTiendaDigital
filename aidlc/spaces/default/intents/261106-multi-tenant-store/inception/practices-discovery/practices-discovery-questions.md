# Practices Discovery — Entrevista de prácticas del equipo

## Sources

- [desc] Initial description: "Pruebas automatizadas desde el inicio y pipeline de CI/CD; buenas prácticas OWASP; mínimo privilegio en IAM; secretos en Secrets Manager o SSM; trazabilidad de operaciones sensibles. Avanzar por fases esperando aprobación en cada puerta. Dividir el MVP en unidades independientes."
- [scope] Workflow-selected scope: `feature`.

---

> Afirmamos cómo trabajará el equipo (se guarda en team.md/project.md y condiciona toda la Construcción). El borrador parte de defaults y de lo ya decidido en Ideación; los tres especialistas (calidad, desarrollo, seguridad) revisaron y propusieron ajustes. Marca una opción por pregunta en el `[Answer]:`.

## Q1. Forma de trabajo (Way of Working)

Propuesta: trunk-based development, rama única `main` (base y destino), squash-merge por Bolt, ramas de feature cortas, feature flags para trabajo incompleto. Encaja con construcción unit-major (un Bolt en vuelo). El desarrollador sugiere además fijar **monorepo** con `modules/` por unidad (ya implícito en el alcance) y specs OpenAPI versionadas como fuente de verdad de los contratos.

- A. Apruebo trunk-based + squash por Bolt **y** fijar monorepo con `modules/` por unidad y OpenAPI versionado.
- B. Apruebo trunk-based + squash, pero el monorepo/OpenAPI lo decidimos en diseño.
- C. Otra forma de trabajo (indícala en Otro).
- X. Other (please specify)

[Answer]: A.

## Q2. Walking skeleton

Propuesta (ya decidida en Ideación): sí, construir primero un corte mínimo end-to-end que valide los riesgos R1 (reserva de stock/concurrencia), R2 (consistencia orden↔pago) y R3 (aislamiento multi-tenant), con checkpoints revisados hasta validarlo. QA recomienda que las **pruebas R1-R3 en verde sean compuerta de salida del skeleton** (no opcionales).

- A. Sí, walking skeleton primero **y** las pruebas R1-R3 en verde son compuerta de salida del skeleton.
- B. Sí, walking skeleton primero, pero sin elevar R1-R3 a compuerta formal.
- C. Otra (indícala en Otro).
- X. Other (please specify)

[Answer]: A.

## Q3. Postura de pruebas (metodología y orden)

Propuesta: metodología `custom` — criterios de aceptación Given/When/Then al frente para las historias (y, para las unidades de riesgo inventory/orders/payments, la prueba de comportamiento precede/acompaña la implementación), y test-after de unidad/integración en el resto. Piso de 80% de cobertura de línea en CI antes del merge. QA recomienda además **cobertura de rama (branch)** en los módulos de dominio críticos (reserva de stock, estados de orden, idempotencia de pagos, resolución de `tenant_id`).

- A. Afirmo `custom` como se describe **y** añado branch coverage en los módulos de dominio críticos, además del piso de 80% de línea.
- B. Afirmo `custom` con el piso de 80% de línea, sin exigir branch coverage adicional (se evalúa en diseño).
- C. Prefiero `test-after` simple (BDD solo como formato de los criterios de aceptación), piso 80% de línea.
- D. Otra (indícala en Otro).
- X. Other (please specify)

[Answer]: D. Test-after como metodología general, con piso de 80 % de cobertura de línea en CI y criterios de aceptación en formato Given/When/Then. Excepción para los módulos de dominio críticos (reserva de stock, estados de orden, idempotencia de pagos y resolución de tenant_id): sus pruebas se escriben junto con la implementación, no después, y se mide cobertura de ramas en ellos.

## Q4. Despliegue (Deployment)

Propuesta (ya decidida en Ideación): deploy a dev primero; producción solo tras validar dev, con aprobación manual del owner; el owner ejecuta `terraform apply`/`destroy`, el agente solo `fmt`/`validate`; rollback documentado; smoke/health checks antes de dar un deploy por hecho.

- A. Apruebo tal cual.
- B. Apruebo con un ajuste (indícalo en Otro).
- C. Otra (indícala en Otro).
- X. Other (please specify)

[Answer]: A.

## Q5. Estilo de código y convenciones (Code Style)

Propuesta base: formateador/linter por lenguaje (a fijar con el stack en diseño), nombres idiomáticos, código e identificadores en inglés. El desarrollador propone fijar ya convenciones **independientes del stack**: organización por feature/unidad, envelope de error consistente en las Lambdas, validación en los límites de confianza, logging estructurado JSON con `correlationId` (sin PII/secretos en logs), y una **regla de frontera entre unidades** (sin acoplamiento directo a la BD de otra unidad; comunicación por contrato/evento; resolución de `tenant_id` en un único punto transversal).

- A. Apruebo la base **y** fijo ya las convenciones independientes del stack y la regla de frontera entre unidades.
- B. Apruebo solo la base; las convenciones adicionales se deciden en diseño.
- C. Otra (indícala en Otro).
- X. Other (please specify)

[Answer]:A.

## Q6. Controles de seguridad como práctica (de la revisión de DevSecOps)

El especialista de seguridad propone fijar ya, como práctica de CI/CD (la herramienta concreta se elige en diseño): escaneo de secretos (pre-commit + CI), SAST + linting de seguridad, escaneo de dependencias + SBOM, escaneo de IaC (Terraform), DAST a dev antes de producción, verificación de firma (HMAC) e idempotencia de webhooks de pago, prohibición de procesar/registrar/persistir datos de tarjeta, y cifrado en tránsito (TLS 1.2+) y en reposo (KMS/AES-256) sin registrar PII/tokens en logs.

- A. Sí, afirmo todos estos controles de seguridad como práctica/reglas duras del proyecto (herramienta concreta en diseño).
- B. Afirmo solo un subconjunto (indícalo en Otro).
- C. No los fijo ahora; se deciden en diseño/NFR.
- X. Other (please specify)

[Answer]: B. Afirmo como reglas del proyecto: escaneo de secretos (antes del commit y en CI); escaneo de dependencias y de la infraestructura como código; verificación de firma e idempotencia de los webhooks de pago; prohibición de procesar, registrar o guardar datos de tarjeta; cifrado en tránsito y en reposo; y no registrar datos personales ni tokens en los logs. Las herramientas concretas se eligen en diseño. SAST, SBOM y DAST se evalúan en diseño y deben quedar resueltos antes del primer despliegue a producción.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
