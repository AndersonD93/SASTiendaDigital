# Refined Mockups — Preguntas de diseño

## Sources

- [desc] Initial description: "Frontend web responsivo; mobile-first para tienda pública y checkout; paneles usables en escritorio y móvil; objetivo WCAG AA; personalización por tienda (colores, logo, tipografía)."
- [scope] Workflow-selected scope: `feature`.

---

> Plan: refinar los 14 wireframes aprobados (P1–P14) a mockups de media/alta fidelidad por historia/pantalla, con una especificación de interacción, un mapeo al sistema de diseño, la especificación de comportamiento responsive y una lista de verificación de accesibilidad. Estas preguntas fijan las decisiones de diseño antes de generar los artefactos; luego los generas, los revisas y los apruebas en la puerta. No se tomarán decisiones técnicas de implementación (framework/stack) aquí: eso es de la fase de diseño técnico.
>
> Responde cada pregunta escribiendo tu elección en la línea `[Answer]:` (puedes elegir una opción y matizarla con tus palabras). Si una no aplica o prefieres que decida yo, dilo ahí.

## Q1. Sistema de diseño / librería de componentes base

Los wireframes asumen un sistema de diseño open source (sujeto a investigación de licencias) con personalización por tienda. ¿Qué enfoque quieres para los mockups refinados?

- A. Diseñar sobre un sistema de diseño open source genérico y agnóstico del framework (tokens de diseño: color, tipografía, espaciado, radios), sin comprometer todavía una librería concreta; la elección de librería/framework se cierra en diseño técnico.
- B. Ya tengo en mente una librería concreta (p. ej. Material, Chakra, shadcn/ui, Tailwind UI) y quiero que los mockups la reflejen.
- C. Diseño propio desde cero (sin apoyarse en un sistema existente).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Diseñar sobre un sistema de diseño agnóstico del framework, basado en tokens de diseño (color, tipografía, espaciado, radios y demás elementos visuales). No seleccionar todavía una librería concreta. La elección de framework y componentes se realizará durante el diseño técnico, previa revisión de licencias, mantenibilidad y costos.

## Q2. Identidad visual y personalización por tienda (tokens)

La tienda pública se personaliza por comercio (color principal, logo, tipografía) sobre una plantilla base (US3.4 Must). ¿Cómo modelamos esa personalización en los mockups?

- A. Definir tokens de marca por tienda (color primario, color de acento, logo, familia tipográfica) con valores por defecto neutrales en la plantilla base; los mockups muestran el estado por defecto y un ejemplo personalizado.
- B. Solo color principal y logo en el MVP (tipografía fija en la plantilla base), para acotar el alcance de personalización.
- C. Personalización amplia (colores múltiples, tipografías, espaciados) desde el MVP.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Definir tokens de marca por tienda con un alcance controlado para el MVP: color primario, color de acento, logo y familia tipográfica, con valores predeterminados neutrales en la plantilla base. Los mockups deben mostrar tanto el estado predeterminado como un ejemplo personalizado. No incluir personalización avanzada de espaciados, estructuras o componentes. La personalización debe conservar la legibilidad, el contraste y la coherencia visual.

## Q3. Patrones de interacción predominantes

¿Qué patrones de interacción prefieres como estándar en los mockups?

- A. Mobile-first en tienda pública y checkout con patrones simples (navegación por pila, carrito como panel lateral/hoja, checkout en una sola página con secciones); paneles de comercio con tablas + formularios en página (no wizards), y diálogos de confirmación para acciones sensibles (aprobar/rechazar pago, suspender tienda).
- B. Checkout multipaso (wizard) en lugar de una sola página.
- C. Edición en línea (inline) extensiva en los paneles en vez de formularios en página.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Adoptar un enfoque mobile-first para la tienda pública y el checkout, utilizando patrones simples e intuitivos: navegación por pila, carrito como panel lateral u hoja inferior y checkout en una sola página organizada por secciones. Para los paneles administrativos, utilizar tablas, filtros y formularios en página, evitando asistentes multipaso innecesarios. Las acciones sensibles, como aprobar o rechazar pagos y suspender tiendas, deben requerir confirmación explícita.

## Q4. Estados obligatorios por pantalla

Los wireframes listan estados (vacío, cargando, con datos, error, borde). ¿Qué cobertura de estados exiges en los mockups refinados?

- A. Cada pantalla documenta los cinco estados cuando apliquen: cargando (skeleton), vacío (con CTA/guía), con datos, error (mensaje accionable), y los estados de borde propios (p. ej. stock perdido en checkout, orden vencida, pasarela no conectada, tienda suspendida).
- B. Solo camino feliz (con datos) en el refinamiento; los demás estados se detallan en construcción.
- C. Estados completos solo en las pantallas del walking skeleton; el resto a nivel conceptual.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:  A. Documentar los estados que correspondan a cada pantalla: cargando, vacío, con datos y error, además de los estados de borde específicos del proceso. Cada estado debe incluir su representación visual y el comportamiento esperado. Los estados vacíos deben orientar al usuario mediante acciones claras; los errores deben ofrecer mensajes comprensibles y recuperables cuando sea posible. Incluir, entre otros, pérdida de disponibilidad de inventario durante el checkout, orden vencida, pasarela no configurada y tienda suspendida. No es necesario forzar estados que no sean aplicables a una pantalla.

## Q5. Representación en UI de estados de pago `failed` vs `rejected` (refinamiento de historias)

Las historias distinguen pago `rejected` (rechazo manual del comprobante) de `failed` (fallo del intento en pasarela), y el pago NO cancela la orden automáticamente (permanece `pending_payment` mientras la reserva siga vigente). ¿Cómo lo reflejamos al comprador?

- A. Mensajes y acciones distintos por caso: `rejected` muestra el motivo del staff y un CTA "volver a subir comprobante"; `failed` muestra "el pago no se completó" y un CTA "reintentar pago"; ambos indican que la orden sigue vigente hasta `expiresAt`. Nunca se muestra la orden como "cancelada" por un pago no exitoso.
- B. Un único mensaje genérico de "pago no completado" para ambos casos, con un CTA de reintento.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Diferenciar claramente ambos escenarios. Cuando el comprobante sea rejected, mostrar el motivo registrado por el personal autorizado y permitir volver a cargar el comprobante mientras la orden y su reserva continúen vigentes. Cuando un intento de pago en pasarela sea failed, informar que el pago no se completó y ofrecer reintentar, siempre que la pasarela esté disponible y la reserva siga vigente. En ambos casos, mostrar que la orden permanece en pending_payment mientras corresponda y comunicar su fecha y hora de vencimiento. Un pago no exitoso no debe presentar automáticamente la orden como cancelada.

## Q6. Consulta de orden del comprador invitado (pregunta abierta OQ-US-1)

Quedó abierta la decisión funcional del mecanismo con que un invitado consulta/retoma su orden sin cuenta (AC2.2.4). Para los mockups, ¿cómo lo representamos sin cerrar la decisión técnica?

- A. Mostrar un punto de reentrada genérico ("ver estado de mi orden" desde la confirmación P6 y, conceptualmente, un acceso por enlace enviado al comprador), dejando el mecanismo exacto como nota de diseño pendiente (OQ-US-1) sin comprometer la implementación.
- B. Asumir un enlace firmado de un solo uso enviado por correo y diseñarlo así.
- C. Asumir código de orden + dato de contacto (p. ej. email) y diseñarlo así.
- D. No lo sé / usa A (mantener OQ-US-1 abierta).
- X. Other (please specify)

[Answer]: A. Representar un punto de reentrada genérico para consultar el estado de la orden desde la pantalla de confirmación y contemplar conceptualmente un acceso posterior mediante un enlace o mecanismo seguro enviado al comprador. El mecanismo exacto de autenticación y consulta permanecerá como pregunta abierta OQ-US-1. Los mockups no deben comprometer una solución técnica que todavía no ha sido aprobada.

## Q7. Nivel y alcance de accesibilidad (WCAG)

El objetivo declarado es WCAG 2.1/2.2 AA en tienda pública, carrito y checkout; AA con mejora continua en paneles. ¿Confirmas este alcance operativo para la lista de verificación?

- A. Sí: AA como meta en todas las superficies, con foco estricto en tienda pública + carrito + checkout (contraste, foco visible, navegación por teclado, etiquetas de formulario, estado por texto además de color, texto alternativo de imágenes y comprobantes, anuncios de estado); los paneles apuntan a AA con mejora continua. La conformidad real requiere pruebas con tecnologías de asistencia y revisión experta (fuera de esta etapa).
- B. Elevar a AAA en las superficies públicas.
- C. Reducir a A en el MVP y subir a AA después.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:A. Mantener WCAG 2.1 AA como objetivo de diseño en todas las superficies, con especial atención a la tienda pública, el carrito y el checkout. La lista de verificación debe contemplar contraste suficiente, foco visible, navegación por teclado, etiquetas de formularios, información no transmitida exclusivamente mediante colores, textos alternativos y comunicación accesible de los cambios de estado. Los paneles administrativos también deben apuntar a AA. La conformidad real requerirá pruebas con tecnologías de asistencia y revisión especializada, que se realizarán posteriormente.

## Q8. Breakpoints responsive

¿Qué breakpoints responsive fijamos como referencia en la especificación?

- A. Tres referencias: móvil (base, ~360–430 px), tablet (~768 px) y escritorio (~1024–1280+ px); tienda pública y checkout diseñados mobile-first y los paneles priorizando tablet/escritorio pero usables en móvil.
- B. Solo móvil y escritorio (dos breakpoints).
- C. Un set más granular (móvil pequeño, móvil grande, tablet, escritorio, wide).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Utilizar tres referencias principales: móvil (aproximadamente 360–430 px), tablet (aproximadamente 768 px) y escritorio (aproximadamente 1024–1280 px o superior). La tienda pública y el checkout se diseñarán mobile-first. Los paneles administrativos priorizarán tablet y escritorio, garantizando también su usabilidad en dispositivos móviles. Los valores son referencias de diseño y no límites rígidos.

## Q9. Fidelidad y formato de entrega de los mockups

¿Qué nivel de fidelidad y formato esperas en `mockups.md` (recordando que trabajamos en artefactos Markdown versionados, no en una herramienta de diseño)?

- A. Media fidelidad en Markdown: por pantalla/historia, un layout ASCII/estructura refinada + especificación detallada (jerarquía, componentes, estados, interacción, accesibilidad, responsive) que sirva de contrato para construcción; alta fidelidad visual real se produce en la herramienta de diseño/el frontend después.
- B. Incluir además diagramas Mermaid de flujo entre pantallas.
- C. Solo especificación textual sin layouts.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]: A. Elaborar mockups de media fidelidad en Markdown, organizados por pantalla e historia, con una estructura visual refinada o layout ASCII y una especificación suficientemente detallada para orientar la construcción. Cada pantalla debe incluir jerarquía visual, componentes, estados, interacciones, comportamiento responsive y criterios de accesibilidad. Los artefactos deben funcionar como contrato de diseño para las siguientes etapas. La alta fidelidad visual podrá desarrollarse posteriormente en la herramienta de diseño o durante la construcción del frontend.

## Assumptions & Open Questions

- [assumption] La elección de librería de componentes y framework frontend NO se decide en esta etapa; los mockups se expresan con tokens/estructura agnósticos y la decisión técnica se cierra en diseño técnico. [scope-document]
- [assumption] El objetivo WCAG AA es una meta de diseño; la conformidad real exige pruebas con tecnologías de asistencia y revisión experta, fuera de esta etapa. [requirements NFR8.2]
- OQ-US-1 — Mecanismo de consulta segura de órdenes del comprador invitado: permanece abierto; los mockups lo representan sin cerrar la decisión (ver Q6). [stories]

## Consolidated Summary Confirmation

Resumen de las decisiones de diseño tomadas a partir de tus respuestas (Q1–Q9) y de los artefactos generados en esta etapa. Confírmalo para que la etapa continúe a la revisión.

**Decisiones de diseño (de tus respuestas):**
- Q1 — Sistema de diseño **agnóstico del framework** basado en tokens; la librería/framework se decide en diseño técnico.
- Q2 — Personalización por tienda acotada: color primario, acento, logo y tipografía, con defaults neutrales y contraste AA preservado.
- Q3 — Mobile-first en tienda pública/checkout; checkout en una sola página; paneles con tablas+formularios; confirmación en acciones sensibles.
- Q4 — Estados por pantalla cuando apliquen: cargando, vacío (con guía), con datos, error (recuperable) y bordes del proceso.
- Q5 — Pago `rejected` (muestra motivo + recargar comprobante) ≠ `failed` (reintentar pago); la orden nunca se muestra "cancelada" por un pago no exitoso; se muestra vigencia hasta `expiresAt`.
- Q6 — Reentrada del comprador invitado representada de forma genérica; OQ-US-1 permanece abierta (no se decide el mecanismo).
- Q7 — WCAG 2.1 AA como objetivo; foco estricto en tienda pública, carrito y checkout; paneles AA con mejora continua; conformidad real se verifica después.
- Q8 — Tres breakpoints de referencia: móvil, tablet, escritorio.
- Q9 — Mockups de media fidelidad en Markdown como contrato de diseño; alta fidelidad visual, después.

**Artefactos generados:**
- `design-system-mapping.md` — tokens de color/tipografía/espaciado/breakpoints, inventario de componentes y reglas de personalización/contraste.
- `mockups.md` — P1–P14 con layout, jerarquía, componentes, estados, interacción, responsive y accesibilidad; trazabilidad pantalla→historia→requisito.
- `interaction-spec.md` — especificación de componentes críticos (StatusTag, resultado de pago, método de pago, carga de comprobante, aprobación de pago, subdominio, tokens de marca, reentrada invitado, diálogo de confirmación).
- `accessibility-checklist.md` — lista WCAG 2.1 AA transversal y por pantalla, con pasos de verificación para construcción/QA.

**Concern que permanece abierto:** OQ-US-1 (mecanismo de consulta de la orden del invitado), a resolver en diseño funcional.

¿Esto refleja correctamente lo que quieres? (`Looks correct` / `Request changes`)

[Answer]: Looks correct

> Nota del humano: Confirmo que el resumen consolidado refleja las decisiones acordadas para Q1–Q9 y apruebo los artefactos generados como base para la revisión. Se mantiene abierta OQ-US-1 (mecanismo seguro de consulta de órdenes del comprador invitado), sin asumir una solución técnica en esta etapa. Esta aprobación no cierra decisiones pendientes de diseño técnico, arquitectura, selección de librerías ni implementación.
