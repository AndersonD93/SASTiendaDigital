# Approval & Handoff — Preguntas de confirmación de la iniciativa

## Sources

- [desc] Initial description: "Avanzar por fases esperando mi aprobación explícita en cada puerta; dividir el MVP en unidades independientes y proponer el orden de construcción."
- [scope] Workflow-selected scope: `feature`.

---

> Nota: esta etapa compila todo lo decidido en Ideación en un brief de iniciativa y verifica el paso a Inception. No introduce decisiones nuevas; cada punto ya se aprobó en su etapa. Estas preguntas solo confirman la disposición para pasar a Inception. Marca una opción por pregunta en el `[Answer]:`.

## Q1. ¿Confirmas que el intent y el alcance aprobados reflejan lo que quieres construir, tal como quedaron tras las 6 etapas de Ideación?

- A. Sí, el intent y el alcance (MVP con 9 unidades, pago manual + pasarela, Colombia, aislamiento multi-tenant) reflejan lo que quiero.
- B. Sí, con un matiz menor (indícalo en Otro).
- C. No, quiero revisar algo antes del handoff (indícalo en Otro).
- X. Other (please specify)

[Answer]: A. El brief debe reflejar las decisiones finales de pagos (transferencia con comprobante y pasarela, sin retener fondos ni conciliación bancaria), la monetización solo por suscripción con cobro manual, el aislamiento lógico fuerte y la lista completa de exclusiones.

## Q2. ¿Los riesgos principales quedan reconocidos con su mitigación para arrancar Inception?

Riesgos clave (de feasibility): sobreventa/concurrencia, consistencia de órdenes ante webhooks y aprobaciones manuales, aislamiento multi-tenant, y el techo de costo AWS dev (10 USD/mes). Mitigación: walking skeleton primero y diseño que los aborde como prioridad.

- A. Sí, los riesgos y su mitigación están claros; podemos avanzar.
- B. Sí, pero quiero añadir/ajustar un riesgo (indícalo en Otro).
- C. No, hay un riesgo sin reconocer (indícalo en Otro).
- X. Other (please specify)

[Answer]: B. Añadir estos riesgos con su mitigación:

Comprobantes de transferencia falsificados (mitigación: el admin confirma el dinero en su banco, con paso de confirmación y registro de quién aprobó).
Tiempo limitado (~8 h/semana) frente al alcance del MVP (mitigación: walking skeleton primero y recorte del alcance si hace falta, empezando por la pasarela).
Dependencia de comercios piloto y de la aprobación de sus cuentas en pasarelas.
Cumplimiento de la Ley 1581 y transferencia internacional de datos (mitigación: consulta con abogado antes de operar con comercios reales).

## Q3. ¿Hay compromiso de recursos para avanzar a Inception (tu dedicación ~8 h/semana y el entorno AWS dev listo)?

- A. Sí, dispongo del tiempo y el entorno dev para empezar Inception.
- B. Sí, con una salvedad (indícalo en Otro).
- C. No todavía (indícalo en Otro).
- X. Other (please specify)

[Answer]: B. Dispongo del tiempo (~8 h/semana) y del entorno dev. Pendientes que no bloquean Inception: confirmar la alarma de presupuesto del techo de 10 USD en dev y adquirir el dominio de la plataforma antes de probar subdominios.

## Q4. Recomendación go/no-go para pasar de Ideación a Inception. Mi recomendación es **GO**. ¿Estás de acuerdo?

- A. GO — avanzar a la fase de Inception.
- B. GO con condiciones (indícalas en Otro).
- C. NO-GO / esperar (indica el motivo en Otro).
- X. Other (please specify)

[Answer]: B. GO. Llevar a Inception como decisiones pendientes: elección del primer proveedor de pasarela con tarifas verificadas en fuentes oficiales; evaluación del costo por tienda del aislamiento multi-tenant; investigación de licencias de plantillas y sistema de diseño; una o dos plantillas en el MVP; y la ventana de días de la métrica de pilotos.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
