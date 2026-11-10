# Feasibility & Constraints — Preguntas de aclaración

## Sources

- [desc] Initial description: "Restricciones técnicas: AWS serverless (Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge, Step Functions donde aplique), infraestructura como código con Terraform, frontend web responsivo, pruebas automatizadas desde el inicio y pipeline de CI/CD, buenas prácticas OWASP, mínimo privilegio en IAM, secretos en Secrets Manager o SSM (nunca en el repositorio), trazabilidad de operaciones sensibles, observabilidad básica. Entorno AWS ya preparado (OU tiendaComercial, cuentas tienda-dev 547887911593 y tienda-prd 790152345727, us-east-1, IAM Identity Center, buckets de estado de Terraform, estructura modules/ y envs/dev, envs/prd, tags project/env/owner). Reglas: NO ejecutar terraform apply ni destroy; solo fmt y validate; empezar solo por dev."
- [scope] Workflow-selected scope: `feature`.
- [memory:M1] `aidlc/spaces/default/memory/team.md### Corrections`: "ALWAYS verificar tarifas, costos y datos de proveedores externos (p. ej. pasarelas de pago) con fuentes oficiales vigentes en la etapa de diseño, en lugar de darlos por supuestos"

---

> Nota: la mayoría de las restricciones técnicas y del entorno AWS ya las definiste; no las re-pregunto. Estas preguntas cubren los huecos de viabilidad. Mantén tu respuesta a nivel de problema/restricción (el diseño detallado viene después). Marca una opción por pregunta en el `[Answer]:`. Si ninguna encaja, usa `X`.

## Q1. ¿Hay sistemas existentes con los que esta plataforma deba integrarse además de las pasarelas de pago? (ERP, contabilidad, CRM, correo, etc.)

- A. No, es un sistema nuevo independiente; solo integra pasarelas de pago externas.
- B. Sí, correo transaccional/notificaciones (p. ej. para confirmaciones de orden) — indícalo en Otro si ya tienes proveedor.
- C. Sí, además de pagos, otros sistemas (contabilidad/ERP/CRM) — indícalos en Otro.
- D. No lo sé todavía / que el diseño lo evalúe.
- X. Other (please specify)

[Answer]: X. Para el MVP solo notificaciones transaccionales básicas (confirmación de orden y resultado de verificación de pago), con el canal de menor costo; sin campañas ni SMS masivo. Cada tienda podrá ver su lista de clientes y el historial de órdenes (CRM mínimo, sin automatizaciones). El SMS y un CRM más completo quedan para una fase posterior, posiblemente dentro de planes de mayor valor. El diseño debe dejar puertos de integración para que, a futuro, un comercio conecte su propio CRM, su contabilidad o su facturación electrónica. No se integran sistemas externos en el MVP.

## Q2. Cumplimiento de pagos (PCI DSS): confirmo tu intención de que ningún dato de tarjeta pase por tus sistemas. ¿Es correcto que el alcance de pagos se limita a checkout alojado/tokenización de la pasarela (reduciendo tu exposición a PCI)?

- A. Correcto: usar checkout alojado o tokenización de la pasarela; ningún dato de tarjeta toca nuestros sistemas (minimiza el alcance PCI). El pago manual es por transferencia, sin datos de tarjeta.
- B. Correcto, pero quiero que el diseño documente explícitamente qué nivel/alcance de PCI aplica y cómo se evidencia.
- C. No estoy seguro — que el diseño/compliance lo determine.
- D. Not applicable.
- X. Other (please specify)

[Answer]: A. Usar checkout alojado o tokenización de la pasarela; ningún dato de tarjeta toca nuestros sistemas, lo que reduce (no elimina) el alcance de PCI. El pago manual por transferencia no maneja datos de tarjeta.

## Q3. Requisitos regulatorios además de protección de datos (Ley 1581) ya identificada. ¿Alguno más que debamos registrar como restricción?

- A. No, por ahora solo protección de datos personales (Ley 1581 / habeas data) y el alcance PCI reducido por no manejar tarjetas.
- B. Sí, residencia de datos (dónde se almacenan los datos) — indica la exigencia en Otro.
- C. Sí, otros (facturación/DIAN a futuro, etc.) — pero fuera del MVP.
- D. No lo sé / que compliance lo evalúe.
- X. Other (please specify)

[Answer]: X. Combinación de A y C. Restricciones a registrar: protección de datos personales (Ley 1581), incluyendo la transferencia internacional de datos por usar infraestructura en us-east-1 y el tratamiento de los comprobantes de pago; la relación entre la plataforma y los comercios respecto a los datos de sus clientes (términos y política de tratamiento); y el alcance PCI reducido. Fuera del MVP: facturación electrónica (tanto la de las tiendas como la que yo emita por cobrar las suscripciones), a validar con un contador.

## Q4. Perfil del equipo y experiencia técnica (para dimensionar riesgo de ejecución)

- A. Principalmente yo (owner) con apoyo del agente AI-DLC; experiencia media en AWS/serverless.
- B. Yo con experiencia limitada en AWS; dependeré fuertemente de la guía del agente.
- C. Equipo pequeño con experiencia sólida en AWS serverless y Terraform.
- D. Prefiero no estimarlo ahora / que se asuma un perfil conservador.
- X. Other (please specify)

[Answer]: A, con experiencia suficiente en AWS. Dedicación: parcial aproximada de [8] horas por semana.

## Q5. Restricciones de presupuesto y tiempo (a nivel de señal, no cifras exactas)

- A. Presupuesto ajustado (optimizar costo AWS: serverless/pago por uso, sin recursos siempre encendidos); sin fecha límite dura, avanzar por fases.
- B. Presupuesto moderado; una ventana objetivo para el MVP (indícala en Otro).
- C. Presupuesto flexible; prioridad en calidad y confiabilidad sobre costo.
- D. No definido aún.
- X. Other (please specify)

[Answer]: A. Presupuesto ajustado, sin fecha límite dura, avance por fases. Techo de gasto mensual de AWS en dev: 10 USD.

## Q6. Bloqueos organizacionales o dependencias externas que puedan frenar el avance

- A. Ninguno conocido; tú controlas el entorno AWS y las decisiones.
- B. Dependencia de aprobación/credenciales de pasarela(s) de pago para probar el flujo real.
- C. Dependencia de disponibilidad de comercios piloto para validar.
- D. Una combinación (B y C) — indícalo en Otro.
- E. Not identified.
- X. Other (please specify)

[Answer]: D. Dependencias externas: disponibilidad de comercios piloto para validar; aprobación de las cuentas de pasarela de cada comercio para operar pagos reales (el desarrollo se apoya en entornos de prueba); y la adquisición del dominio de la plataforma para los subdominios. El pago manual por transferencia reduce la dependencia de pasarelas en los pilotos.

## Q7. Confirmación del entorno AWS: ¿las dos cuentas (tienda-dev 547887911593, tienda-prd 790152345727), us-east-1, IAM Identity Center y los buckets de estado de Terraform están listos y accesibles para empezar por dev?

- A. Sí, todo listo; empezar solo por dev como indica la regla.
- B. Sí, pero quiero que el diseño verifique/valide la configuración antes de generar módulos.
- C. Parcialmente — hay algo pendiente (indícalo en Otro).
- D. No lo sé con certeza.
- X. Other (please specify)

[Answer]: B. Ya verifiqué ambas cuentas con sus perfiles SSO y los buckets de estado (versionado, cifrado y acceso público bloqueado). El diseño debe validar el resto de la configuración antes de generar módulos. Pendiente por mi parte: alarmas de presupuesto en cada cuenta [si aún no las creaste; si ya existen, quita esta línea].

## Q8. Mayor incertidumbre técnica a tu juicio (lo que más riesgo de viabilidad tiene en el MVP)

- A. Reserva de stock / control de concurrencia para evitar sobreventa en checkout.
- B. Idempotencia y consistencia de órdenes frente a webhooks de pago duplicados/tardíos.
- C. Aislamiento de datos multi-tenant correcto y seguro en DynamoDB.
- D. Enrutamiento por subdominio automático por tienda (DNS/CloudFront).
- E. Varias de las anteriores — indícalo en Otro (o di "todas").
- X. Other (please specify)

[Answer]:E. Las opciones A, B y C. A: reserva de stock y concurrencia, incluida la reserva prolongada de las órdenes pendientes de verificación manual y su liberación por vencimiento. B: consistencia de órdenes frente a notificaciones de pago duplicadas o tardías y frente a aprobaciones manuales. C: aislamiento de datos multi-tenant en DynamoDB.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
