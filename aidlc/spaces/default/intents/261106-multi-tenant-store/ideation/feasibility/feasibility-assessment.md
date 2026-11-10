# Evaluación de Viabilidad — Plataforma de tienda virtual multi-negocio

> Alcance: nivel de problema/ideación. Estimaciones conservadoras; los supuestos se marcan explícitamente. El detalle técnico se aborda en diseño. Fecha: 2026-11-09.
>
> Entradas upstream: parte del `intent-statement` (Intent Capture) y de `competitive-analysis`, `market-trends` y `build-vs-buy` (Market Research), de los que toma el problema, el alcance del MVP, el cliente objetivo (Colombia, bajo presupuesto), el marco regulatorio (Ley 1581) y la postura build-vs-buy (construir núcleo, integrar pagos y servicios AWS gestionados).

## Veredicto de viabilidad

**Viable** a nivel técnico y de ejecución para un MVP, bajo restricciones de presupuesto ajustado y dedicación parcial. La arquitectura serverless en AWS ya elegida es apropiada para un modelo multi-tenant de bajo costo con tráfico inicial de pilotos. Los principales riesgos son de diseño (concurrencia de stock, consistencia de pagos, aislamiento multi-tenant), no de factibilidad fundamental. [intent-statement] [build-vs-buy]

## Viabilidad técnica

- **Arquitectura serverless (Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge, Step Functions):** adecuada para pago por uso y escalado a cero, coherente con el presupuesto ajustado. [desc] [Q5]
- **Multi-tenant con aislamiento lógico fuerte por `tenant_id`:** patrón estándar y viable en DynamoDB; requiere disciplina de diseño en claves de partición y autorización para garantizar el aislamiento (riesgo de diseño, no de factibilidad). [intent-statement] [Q8]
- **Pagos con adaptador genérico + no retención de fondos + pago manual por transferencia:** viable; el pago manual reduce la dependencia de aprobación de pasarelas durante los pilotos. El desarrollo se apoya en entornos de prueba de las pasarelas. [Q6] [competitive-analysis]
- **Subdominio automático por tienda:** viable con CloudFront + DNS; su automatización es una incertidumbre de diseño a resolver. [Q8]
- **Notificaciones transaccionales + CRM mínimo:** alcance acotado (confirmación de orden y resultado de verificación de pago; lista de clientes e historial por tienda). Puertos de integración previstos para futuro. [Q1]

## Viabilidad de ejecución

- **Equipo:** owner con experiencia suficiente en AWS, apoyado por el agente; dedicación parcial ~8 h/semana. Implica un ritmo de avance por fases, sin fecha límite dura. [Q4] [Q5]
- **Entorno AWS:** cuentas dev/prd, perfiles SSO y buckets de estado de Terraform ya verificados por el owner; el diseño validará el resto antes de generar módulos. Empezar solo por dev. [Q7] [desc]
- **Costo:** techo de gasto AWS en dev de 10 USD/mes. Alcanzable con diseño serverless-first y sin recursos siempre encendidos, pero es una restricción dura que condiciona decisiones (ver registro de restricciones). [Q5]

## Cumplimiento (nivel de problema)

- **Protección de datos (Ley 1581 / habeas data):** aplica al tratamiento de datos de clientes de cada tienda y de los comprobantes de pago; considerar transferencia internacional de datos por usar `us-east-1`, y la relación plataforma-comercio sobre los datos de los clientes. [Q3] [market-trends]
- **PCI DSS (alcance reducido):** al usar checkout alojado/tokenización, ningún dato de tarjeta pasa por nuestros sistemas; esto reduce (no elimina) el alcance PCI. El pago manual por transferencia no maneja datos de tarjeta. [Q2]
- **Facturación electrónica (DIAN):** fuera del MVP, tanto la de las tiendas como la de las suscripciones que cobre la plataforma; a validar con un contador en su momento. [Q3]

## Assumptions & Open Questions

- [assumption] El techo de 10 USD/mes en dev se cumple con diseño serverless-first; costos fijos pequeños (zona DNS, dominio, posibles mínimos de Cognito/CloudFront) deben vigilarse y podrían presionar el techo. [Q5]
- [assumption] La transferencia internacional de datos por usar `us-east-1` debe analizarse frente a la Ley 1581 en diseño/compliance; mantener la región o evaluar alternativas es una decisión posterior. [Q3]
- [assumption] La validación real de pagos con pasarela depende de que cada comercio apruebe/configure su cuenta; en pilotos se cubre con entornos de prueba y con el pago manual. [Q6]
