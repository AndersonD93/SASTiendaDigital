# Registro de Restricciones — Plataforma de tienda virtual multi-negocio

> Nivel de problema/ideación. Fecha: 2026-11-09.
>
> Entradas upstream: `intent-statement` (Intent Capture) y `build-vs-buy` / `market-trends` (Market Research).

## Restricciones técnicas

| ID | Restricción | Origen | Tipo |
|---|---|---|---|
| C-T1 | AWS serverless: Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge, Step Functions (donde aplique). | [desc] | Dura |
| C-T2 | Infraestructura como código con Terraform; estructura `modules/` y `envs/dev`, `envs/prd`; backend S3 por ambiente; provider con `profile` y `allowed_account_ids`. | [desc] | Dura |
| C-T3 | Frontend web responsivo (framework a proponer en diseño). | [desc] | Dura |
| C-T4 | Pruebas automatizadas desde el inicio y pipeline de CI/CD. | [desc] | Dura |
| C-T5 | Aislamiento multi-tenant lógico fuerte por `tenant_id` en el MVP, con diseño que permita migrar tiendas grandes a recursos/cuenta separada. | [intent-statement] | Dura |
| C-T6 | Pagos: ningún dato de tarjeta en nuestros sistemas (checkout alojado/tokenización); adaptador genérico de pasarelas; la plataforma no retiene fondos. Convive el pago manual por transferencia con comprobante. | [intent-statement] [Q2] | Dura |
| C-T7 | Techo de gasto AWS en dev: 10 USD/mes. Diseño serverless-first, sin recursos siempre encendidos. | [Q5] | Dura |
| C-T8 | AWS Budgets con alerta al 80% del presupuesto en cada cuenta (dev y prd), definidas por Terraform; el owner ejecuta el apply. | [Q7] | Dura |
| C-T9 | Región principal `us-east-1`. | [desc] | Dura |

## Restricciones organizacionales / de ejecución

| ID | Restricción | Origen | Tipo |
|---|---|---|---|
| C-O1 | Owner como principal ejecutor (experiencia suficiente en AWS) con apoyo del agente; dedicación parcial ~8 h/semana. | [Q4] | Dura |
| C-O2 | Avance por fases con aprobación explícita del owner en cada puerta; resumen al cerrar cada fase. | [desc] [Q13-intent] | Dura |
| C-O3 | NO ejecutar `terraform apply` ni `destroy` desde el agente; solo `fmt` y `validate`. El plan y el apply los corre el owner. Empezar solo por dev; prd después de validar dev. | [desc] | Dura |
| C-O4 | No guardar credenciales, llaves ni secretos en el repositorio; usar Secrets Manager o SSM Parameter Store. | [desc] | Dura |
| C-O5 | Documentar en español; código, identificadores y nombres de recursos en inglés. | [desc] | Dura |
| C-O6 | Mínimo privilegio en IAM; trazabilidad de operaciones sensibles; buenas prácticas OWASP; observabilidad básica (logs estructurados, métricas, alarmas). | [desc] | Dura |
| C-O7 | Tags comunes en todos los recursos: `project`, `env`, `owner`; nombres de recursos con prefijo por ambiente. | [desc] | Dura |
| C-O8 | Acceso de soporte a datos de una tienda/cliente: solo lectura y con registro de auditoría. | [memory:M1-team] | Dura |

## Restricciones regulatorias

| ID | Restricción | Origen | Tipo |
|---|---|---|---|
| C-R1 | Protección de datos personales (Ley 1581 de 2012 / habeas data): consentimiento previo, expreso e informado; derechos de los titulares; SIC como autoridad. Aplica a datos de clientes de cada tienda y a los comprobantes de pago. | [market-trends] [Q3] | Dura |
| C-R2 | Transferencia internacional de datos por usar infraestructura en `us-east-1`: analizar frente a la Ley 1581 en diseño/compliance. | [Q3] | A evaluar |
| C-R3 | Relación plataforma–comercio sobre los datos de los clientes del comercio (términos y política de tratamiento; roles de responsable/encargado). | [Q3] | A evaluar |
| C-R4 | Alcance PCI reducido por no manejar datos de tarjeta; documentar en diseño cómo se evidencia. | [Q2] | A evaluar |
| C-R5 | Facturación electrónica (DIAN) fuera del MVP, tanto para las tiendas como para el cobro de suscripciones; validar con contador. | [Q3] | Fuera de alcance (MVP) |

## Assumptions & Open Questions

- [assumption] Las restricciones marcadas "A evaluar" se detallan y resuelven en diseño/compliance; aquí solo se registran como condicionantes. [Q3] [Q2]
- [assumption] La verificación completa de la configuración del entorno AWS (más allá de cuentas, SSO y buckets ya verificados por el owner) la hará el diseño antes de generar módulos. [Q7]
