# Reglas Duras Descubiertas — Practices Discovery

> Restricciones duras establecidas por el humano en Ideación, en las decisiones de entrega y afirmadas en la entrevista de esta etapa. Al aprobar la compuerta, estas reglas se promueven (con sello) a `project.md` bajo `## Mandated` y `## Forbidden`. Las dos reglas ya afirmadas en `team.md` (`## Corrections`) sobre verificar datos de terceros con fuentes oficiales vigentes y sobre el acceso de soporte de solo lectura y auditado **no** se duplican aquí. SAST, SBOM y DAST **no** se afirman aquí como regla dura: se evalúan en diseño y deben quedar resueltos antes del primer despliegue a producción (registrado en `evidence.md`).

## Mandated

- ALWAYS aplicar mínimo privilegio en IAM y mantener trazabilidad (auditoría) de las operaciones sensibles.
- ALWAYS detenerse ante cambios en autenticación, aislamiento multi-tenant, pagos e IAM, ante fallos, y antes de cualquier `terraform apply`.
- ALWAYS verificar la firma (HMAC) de todo webhook de pago entrante y garantizar su idempotencia (un único efecto sobre la orden ante entregas duplicadas, tardías o fuera de orden).
- ALWAYS escanear secretos antes del commit (hook de pre-commit) y en CI, como red de seguridad automatizada de la prohibición de secretos en el repo.
- ALWAYS escanear dependencias e infraestructura como código (Terraform) en CI.
- ALWAYS cifrar los datos en tránsito (TLS 1.2+) y en reposo (KMS/AES-256) para todo dato de cliente y comprobante de pago.

## Forbidden

- NEVER ejecutar `terraform apply` o `terraform destroy` desde el agente (solo `terraform fmt` y `terraform validate`; el owner ejecuta `apply`).
- NEVER guardar credenciales, llaves ni secretos en el repositorio (usar AWS Secrets Manager o SSM Parameter Store).
- NEVER procesar, registrar ni persistir datos de tarjeta (PAN/CVV) en nuestros sistemas ni en logs (se mantiene el alcance PCI reducido mediante checkout alojado/tokenización).
- NEVER registrar PII ni tokens en los logs.
