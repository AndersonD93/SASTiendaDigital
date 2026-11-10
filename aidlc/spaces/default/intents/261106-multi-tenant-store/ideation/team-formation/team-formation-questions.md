# Team Formation & Mob Planning — Preguntas de aclaración

## Sources

- [desc] Initial description: "Cómo quiero trabajar: avanzar por fases esperando mi aprobación explícita en cada puerta; dividir el MVP en unidades independientes."
- [scope] Workflow-selected scope: `feature`.

---

> Nota: en etapas previas ya quedó claro que el equipo eres tú (owner con experiencia suficiente en AWS, ~8 h/semana) con apoyo del agente AI-DLC, sin equipos humanos separados ni contratistas. No re-pregunto eso. Estas pocas preguntas confirman la forma de trabajo para Construcción. Marca una opción por pregunta en el `[Answer]:`.

## Q1. Confirmación de composición del equipo para este proyecto

- A. Correcto: owner (yo) como único ejecutor humano y decisor, con el agente AI-DLC como ejecutor/asesor. Sin equipos humanos separados, sin contratistas en el MVP.
- B. Igual que A, pero contemplo sumar ayuda externa puntual más adelante (indícalo en Otro).
- C. Otra composición — descríbela en Otro.
- X. Other (please specify)

[Answer]: A.

## Q2. Modo de trabajo en la fase de Construcción (cómo quieres que el agente construya las unidades)

- A. Una unidad a la vez (unit-major): completar y aprobar cada unidad antes de pasar a la siguiente. Más control, encaja con dedicación parcial.
- B. Etapa por etapa a través de las unidades (stage-major).
- C. No lo sé / recomiéndame (recomendación: A, una unidad a la vez, dada tu dedicación parcial y el enfoque walking-skeleton).
- X. Other (please specify)

[Answer]: A. Una unidad a la vez, completando y aprobando cada una antes de pasar a la siguiente, empezando por el walking skeleton (el corte mínimo de punta a punta).

## Q3. Puntos de control en Construcción: ¿quieres aprobar en cada checkpoint, o que el agente avance de forma autónoma entre puntos clave?

- A. Revisar cada checkpoint (aprobación explícita en cada paso clave de construcción). Máximo control.
- B. Avance autónomo entre checkpoints, deteniéndose solo en fallos, aprobación de plan y confirmaciones clave.
- C. Empezar revisando cada checkpoint (A) y, si gano confianza, cambiar a autónomo más adelante.
- X. Other (please specify)

[Answer]:C. Revisar cada checkpoint hasta que el walking skeleton esté verificado y aprobado; en ese momento decido si paso a un modo más autónomo. Aun en modo autónomo, quiero detenerme siempre ante cambios en autenticación, aislamiento multi-tenant, pagos e IAM, ante cualquier fallo, y antes de cualquier aplicación de infraestructura (terraform apply lo ejecuto yo).

## Q4. Roles de decisión (RACI simplificado) para las fases restantes

- A. Owner (yo) = responsable y quien aprueba todo; agente = ejecuta y recomienda. Simple y directo.
- B. Igual que A, pero quiero marcar explícitamente algún punto donde consultar a un tercero (contador, abogado de datos) — indícalo en Otro.
- C. Otra distribución — descríbela en Otro.
- X. Other (please specify)

[Answer]: B. Yo soy el responsable y quien aprueba todo; el agente ejecuta y recomienda. Dejo marcados dos puntos de consulta con terceros antes de operar con comercios reales: un contador (impuestos y facturación de las suscripciones) y un abogado de protección de datos (términos de servicio, política de tratamiento y transferencia internacional de datos). El agente no sustituye esas consultas.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
