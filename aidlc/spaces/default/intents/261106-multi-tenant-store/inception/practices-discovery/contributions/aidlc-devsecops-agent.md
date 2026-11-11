**Collaborator:** aidlc-devsecops-agent

## Contribution

Revisión ciega de seguridad del borrador del líder (DevSecOps), enfocada en lint/format, SAST/DAST, escaneo de secretos y dependencias, controles de cadena de suministro y las compuertas de seguridad del pipeline. El borrador es sólido en su columna vertebral (trunk-based + squash, skeleton-first con validación de R1/R2/R3, paradas obligatorias, owner ejecuta `apply`/`destroy`, secretos fuera del repo, mínimo privilegio IAM, trazabilidad). Mi evaluación es que esas reglas duras son **necesarias pero insuficientes**: faltan controles de seguridad automatizados que, dado el contexto (OWASP, PCI reducido con checkout alojado/tokenización, Ley 1581, multi-tenant por `tenant_id`), conviene fijar ya como práctica para no dejarlos al azar del diseño.

### 1. Lint/format y Code Style (seguridad shift-left)
- La sección `## Code Style` difiere correctamente al linter por lenguaje, pero no fija la dimensión de seguridad del linting. Recomiendo afirmar como práctica que el linter incluya su **plugin de seguridad** (p. ej. `eslint-plugin-security` para JS/TS, `bandit`/`ruff` reglas S para Python, `gosec`/golangci-lint para Go) y que **el fallo del linter de seguridad bloquee el merge igual que el de estilo**. Esto es defensa en profundidad y shift-left: detectar anti-patrones (inyección, deserialización insegura, uso de `eval`) en el commit/PR, no en runtime.
- El escaneo de IaC (Terraform) merece una regla propia: `terraform fmt` + `terraform validate` ya están, pero falta un **escáner de seguridad de IaC** (Checkov / tfsec / Trivy config) corriendo en CI antes de que el owner haga `plan`/`apply`. Esto captura S3 sin cifrado, buckets públicos, security groups abiertos y políticas IAM demasiado permisivas, que es exactamente donde el aislamiento multi-tenant y el mínimo privilegio se rompen.

### 2. SAST / DAST
- **SAST** no aparece como práctica fija. Debe afirmarse: análisis estático en cada PR (Semgrep/SonarCloud/CodeGuru según stack), con compuerta **bloquear merge ante hallazgos Critical/High** y archivo de supresiones justificado y con expiración. Es la contraparte estática del linting de seguridad.
- **DAST** (OWASP ZAP) contra el entorno dev/efímero como compuerta previa a promover a producción. Encaja con el modelo de despliegue del borrador (dev primero, prd tras validar) y cubre clases de vulnerabilidad que SAST no ve (autenticación rota en runtime, headers de seguridad ausentes, IDOR). Dado que el aislamiento multi-tenant es riesgo R3, un chequeo DAST/integración específico de **fuga entre tenants** (un usuario de tenant A no accede a datos de tenant B vía IDOR) debería ser parte de la compuerta a producción.

### 3. Escaneo de secretos
- La regla `NEVER guardar credenciales/secretos en el repo` es correcta pero es una **política sin mecanismo de enforcement**. Recomiendo fijar como práctica: **escaneo de secretos obligatorio en CI** (Gitleaks/trufflehog) como red de seguridad, y **hook de pre-commit** (git-secrets/Gitleaks) en la máquina del owner. Sin el escaneo automatizado, la prohibición depende de la disciplina humana. Añadir también el procedimiento ante fuga: revocar → rotar → limpiar historial.

### 4. Escaneo de dependencias y cadena de suministro
- No hay práctica de **escaneo de dependencias** ni de **cadena de suministro**, y es una brecha relevante (OWASP A06: componentes vulnerables). Recomiendo afirmar:
  - Escaneo de dependencias en cada build (npm audit / pip-audit / Snyk / Dependabot), con compuerta **fallar build ante CVE Critical/High con exploit conocido**.
  - Generación de **SBOM** (Syft/Trivy) por build para transparencia de cadena de suministro.
  - **Versiones de dependencias ancladas/pinneadas** (lockfiles commiteados) y revisión/actualización al menos mensual; preferir paquetes mantenidos y vigilar typosquatting.
  - Si se usan imágenes de contenedor en algún punto, escaneo de imagen antes de desplegar (ECR scanning / Trivy).

### 5. Controles específicos de pagos y PCI reducido
- El borrador trata pagos como "parada obligatoria" (bien), pero conviene fijar dos controles duros que mantienen el alcance PCI reducido real (C-T6, C-R4):
  - **Checkout alojado/tokenización**: NUNCA procesar, registrar ni persistir datos de tarjeta (PAN/CVV) en nuestros sistemas ni en logs. Esto debería ser una regla dura, no solo una parada de revisión.
  - **Verificación de firma de webhooks de pago**: todo webhook entrante de la pasarela debe verificar firma/HMAC y ser idempotente (protege R2 consistencia orden↔pago ante webhooks duplicados/tardíos, y contra spoofing/tampering STRIDE). Debe quedar como práctica/regla y como caso de prueba del skeleton.

### 6. Cifrado y datos personales (Ley 1581)
- El borrador no fija cifrado explícitamente. Recomiendo afirmar como práctica dura: **cifrado en tránsito (TLS 1.2+) y en reposo (KMS/AES-256)** para todo dato de cliente y comprobantes de pago. La Ley 1581 (datos personales de clientes de cada tienda) y la clasificación Confidencial lo exigen; dejarlo "a diseño" es arriesgado para un dato que ya sabemos sensible.
- **No registrar datos sensibles en logs** (PII, tokens, credenciales): debe ser práctica dura y caso de revisión, coherente con la observabilidad "básica" ya prevista y con OWASP A09.

### 7. Trazabilidad / auditoría (reforzar lo existente)
- La regla `ALWAYS mínimo privilegio IAM + trazabilidad` es correcta. Para que sea verificable, sugiero que la auditoría de operaciones sensibles (incluido el acceso de soporte de solo lectura ya afirmado en `team.md`) se apoye en **CloudTrail + logs de aplicación a almacenamiento resistente a manipulación**, de forma que la trazabilidad no dependa solo de la intención sino de un control técnico comprobable (non-repudiation, STRIDE R).

### Resumen de la evaluación
Las reglas duras propuestas (no `apply`/`destroy` desde el agente, no secretos en repo, mínimo privilegio IAM, trazabilidad, paradas en auth/aislamiento/pagos/IAM) son correctas y **deben conservarse**, pero por sí solas son políticas declarativas. Para cerrar la brecha entre "política" y "control efectivo" conviene fijar ya, como práctica de este proyecto, el conjunto mínimo de compuertas de seguridad de CI/CD: escaneo de secretos (pre-commit + CI), SAST + linting de seguridad, escaneo de dependencias + SBOM, escaneo de IaC, DAST a dev antes de prd, verificación de firma de webhooks de pago, prohibición de datos de tarjeta en nuestros sistemas, y cifrado en tránsito/reposo. La elección de herramienta concreta puede diferirse al diseño/stack, pero **la obligatoriedad de cada control debería afirmarse aquí** para que no quede como decisión opcional más adelante.

## Positions

- AGREE: Way of Working (trunk-based + squash, ramas cortas, feature flags) — superficie de ataque y riesgo de merge bajos; historial lineal auditable que apoya la trazabilidad.
- AGREE: Walking skeleton que valida temprano R1/R2/R3 con checkpoints gated — es correcto exigir que el aislamiento multi-tenant y la consistencia orden↔pago se prueben antes de las features.
- AGREE: Deployment con owner ejecutando `apply`/`destroy` y agente limitado a `fmt`/`validate`, dev antes de prd, rollback documentado — mínimo privilegio operativo y separación de funciones correctas.
- AGREE: Reglas duras de secretos fuera del repo y mínimo privilegio IAM + trazabilidad — son la base correcta.
- OBJECT: `discovered-rules.md` y `## Deployment` carecen de compuertas de seguridad automatizadas (escaneo de secretos en CI, SAST, escaneo de dependencias + SBOM, escaneo de IaC, DAST a dev) — sin enforcement automatizado, las prohibiciones dependen de disciplina humana; deben afirmarse como práctica de CI/CD ahora.
- OBJECT: Falta una regla dura de verificación de firma (HMAC) e idempotencia de webhooks de pago — es un control crítico para R2 (consistencia orden↔pago) y contra spoofing/tampering; no debería quedar implícito en "paradas de pagos".
- OBJECT: Falta una regla dura de no procesar/registrar/persistir datos de tarjeta (PAN/CVV) — es lo que sostiene el alcance PCI reducido (C-T6, C-R4) y debería ser explícita, no solo "parada de revisión".
- OBJECT: `## Testing Posture`/`## Deployment` no fijan cifrado en tránsito (TLS 1.2+) ni en reposo (KMS/AES-256) ni la prohibición de registrar PII/tokens en logs — exigido por Ley 1581 y OWASP A09 para datos ya conocidos como sensibles; dejarlo "a diseño" es un riesgo evitable.
- OBJECT: `## Code Style` no incluye la dimensión de seguridad del linting (plugins de seguridad con el mismo poder de bloqueo que el estilo) — es shift-left de bajo costo que debería afirmarse junto al resto del estilo.
