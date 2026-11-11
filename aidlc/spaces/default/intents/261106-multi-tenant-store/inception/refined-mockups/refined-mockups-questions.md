# Refined Mockups — Preguntas de diseño

## Sources

- [desc] Initial description: "Frontend web responsivo; mobile-first para tienda pública y checkout; paneles usables en escritorio y móvil; objetivo WCAG AA; personalización por tienda (colores, logo, tipografía)."
- [scope] Workflow-selected scope: `feature`.

---

> Plan: refinar los 12 wireframes aprobados (P1–P14) a mockups de media/alta fidelidad por historia/pantalla, con una especificación de interacción, un mapeo al sistema de diseño, la especificación de comportamiento responsive y una lista de verificación de accesibilidad. Estas preguntas fijan las decisiones de diseño antes de generar los artefactos; luego los generas, los revisas y los apruebas en la puerta. No se tomarán decisiones técnicas de implementación (framework/stack) aquí: eso es de la fase de diseño técnico.
>
> Responde cada pregunta escribiendo tu elección en la línea `[Answer]:` (puedes elegir una opción y matizarla con tus palabras). Si una no aplica o prefieres que decida yo, dilo ahí.

## Q1. Sistema de diseño / librería de componentes base

Los wireframes asumen un sistema de diseño open source (sujeto a investigación de licencias) con personalización por tienda. ¿Qué enfoque quieres para los mockups refinados?

- A. Diseñar sobre un sistema de diseño open source genérico y agnóstico del framework (tokens de diseño: color, tipografía, espaciado, radios), sin comprometer todavía una librería concreta; la elección de librería/framework se cierra en diseño técnico.
- B. Ya tengo en mente una librería concreta (p. ej. Material, Chakra, shadcn/ui, Tailwind UI) y quiero que los mockups la reflejen.
- C. Diseño propio desde cero (sin apoyarse en un sistema existente).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q2. Identidad visual y personalización por tienda (tokens)

La tienda pública se personaliza por comercio (color principal, logo, tipografía) sobre una plantilla base (US3.4 Must). ¿Cómo modelamos esa personalización en los mockups?

- A. Definir tokens de marca por tienda (color primario, color de acento, logo, familia tipográfica) con valores por defecto neutrales en la plantilla base; los mockups muestran el estado por defecto y un ejemplo personalizado.
- B. Solo color principal y logo en el MVP (tipografía fija en la plantilla base), para acotar el alcance de personalización.
- C. Personalización amplia (colores múltiples, tipografías, espaciados) desde el MVP.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q3. Patrones de interacción predominantes

¿Qué patrones de interacción prefieres como estándar en los mockups?

- A. Mobile-first en tienda pública y checkout con patrones simples (navegación por pila, carrito como panel lateral/hoja, checkout en una sola página con secciones); paneles de comercio con tablas + formularios en página (no wizards), y diálogos de confirmación para acciones sensibles (aprobar/rechazar pago, suspender tienda).
- B. Checkout multipaso (wizard) en lugar de una sola página.
- C. Edición en línea (inline) extensiva en los paneles en vez de formularios en página.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q4. Estados obligatorios por pantalla

Los wireframes listan estados (vacío, cargando, con datos, error, borde). ¿Qué cobertura de estados exiges en los mockups refinados?

- A. Cada pantalla documenta los cinco estados cuando apliquen: cargando (skeleton), vacío (con CTA/guía), con datos, error (mensaje accionable), y los estados de borde propios (p. ej. stock perdido en checkout, orden vencida, pasarela no conectada, tienda suspendida).
- B. Solo camino feliz (con datos) en el refinamiento; los demás estados se detallan en construcción.
- C. Estados completos solo en las pantallas del walking skeleton; el resto a nivel conceptual.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q5. Representación en UI de estados de pago `failed` vs `rejected` (refinamiento de historias)

Las historias distinguen pago `rejected` (rechazo manual del comprobante) de `failed` (fallo del intento en pasarela), y el pago NO cancela la orden automáticamente (permanece `pending_payment` mientras la reserva siga vigente). ¿Cómo lo reflejamos al comprador?

- A. Mensajes y acciones distintos por caso: `rejected` muestra el motivo del staff y un CTA "volver a subir comprobante"; `failed` muestra "el pago no se completó" y un CTA "reintentar pago"; ambos indican que la orden sigue vigente hasta `expiresAt`. Nunca se muestra la orden como "cancelada" por un pago no exitoso.
- B. Un único mensaje genérico de "pago no completado" para ambos casos, con un CTA de reintento.
- C. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q6. Consulta de orden del comprador invitado (pregunta abierta OQ-US-1)

Quedó abierta la decisión funcional del mecanismo con que un invitado consulta/retoma su orden sin cuenta (AC2.2.4). Para los mockups, ¿cómo lo representamos sin cerrar la decisión técnica?

- A. Mostrar un punto de reentrada genérico ("ver estado de mi orden" desde la confirmación P6 y, conceptualmente, un acceso por enlace enviado al comprador), dejando el mecanismo exacto como nota de diseño pendiente (OQ-US-1) sin comprometer la implementación.
- B. Asumir un enlace firmado de un solo uso enviado por correo y diseñarlo así.
- C. Asumir código de orden + dato de contacto (p. ej. email) y diseñarlo así.
- D. No lo sé / usa A (mantener OQ-US-1 abierta).
- X. Other (please specify)

[Answer]:

## Q7. Nivel y alcance de accesibilidad (WCAG)

El objetivo declarado es WCAG 2.1/2.2 AA en tienda pública, carrito y checkout; AA con mejora continua en paneles. ¿Confirmas este alcance operativo para la lista de verificación?

- A. Sí: AA como meta en todas las superficies, con foco estricto en tienda pública + carrito + checkout (contraste, foco visible, navegación por teclado, etiquetas de formulario, estado por texto además de color, texto alternativo de imágenes y comprobantes, anuncios de estado); los paneles apuntan a AA con mejora continua. La conformidad real requiere pruebas con tecnologías de asistencia y revisión experta (fuera de esta etapa).
- B. Elevar a AAA en las superficies públicas.
- C. Reducir a A en el MVP y subir a AA después.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q8. Breakpoints responsive

¿Qué breakpoints responsive fijamos como referencia en la especificación?

- A. Tres referencias: móvil (base, ~360–430 px), tablet (~768 px) y escritorio (~1024–1280+ px); tienda pública y checkout diseñados mobile-first y los paneles priorizando tablet/escritorio pero usables en móvil.
- B. Solo móvil y escritorio (dos breakpoints).
- C. Un set más granular (móvil pequeño, móvil grande, tablet, escritorio, wide).
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Q9. Fidelidad y formato de entrega de los mockups

¿Qué nivel de fidelidad y formato esperas en `mockups.md` (recordando que trabajamos en artefactos Markdown versionados, no en una herramienta de diseño)?

- A. Media fidelidad en Markdown: por pantalla/historia, un layout ASCII/estructura refinada + especificación detallada (jerarquía, componentes, estados, interacción, accesibilidad, responsive) que sirva de contrato para construcción; alta fidelidad visual real se produce en la herramienta de diseño/el frontend después.
- B. Incluir además diagramas Mermaid de flujo entre pantallas.
- C. Solo especificación textual sin layouts.
- D. No lo sé / usa A.
- X. Other (please specify)

[Answer]:

## Assumptions & Open Questions

- [assumption] La elección de librería de componentes y framework frontend NO se decide en esta etapa; los mockups se expresan con tokens/estructura agnósticos y la decisión técnica se cierra en diseño técnico. [scope-document]
- [assumption] El objetivo WCAG AA es una meta de diseño; la conformidad real exige pruebas con tecnologías de asistencia y revisión experta, fuera de esta etapa. [requirements NFR8.2]
- OQ-US-1 — Mecanismo de consulta segura de órdenes del comprador invitado: permanece abierto; los mockups lo representan sin cerrar la decisión (ver Q6). [stories]
