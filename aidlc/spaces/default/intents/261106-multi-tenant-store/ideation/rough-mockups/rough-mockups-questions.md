# Rough Mockups & Concept Visualization — Preguntas de aclaración

## Sources

- [desc] Initial description: "Frontend web responsivo. Catálogo y landing pública: páginas de muestra de productos por tienda, con categorías, búsqueda simple, detalle de producto e imágenes. Carrito y checkout. Panel de administración por negocio. Cada tienda puede escoger entre plantillas de diseño y personalizar (storefront)."
- [scope] Workflow-selected scope: `feature`.

---

> Nota: aquí hacemos bocetos de baja fidelidad (concepto, no diseño final) de las pantallas clave. Ya tengo el alcance y las unidades; estas preguntas afinan el enfoque visual. Marca una opción por pregunta en el `[Answer]:`. Si ninguna encaja, usa `X`.

## Q1. Pantallas clave a bocetar en el MVP. Propongo estas. ¿Las apruebas?

Propuesta:
1. **Landing/tienda pública** (storefront con plantilla): catálogo con categorías, búsqueda simple, tarjetas de producto.
2. **Detalle de producto** (imágenes, variantes, precio con impuesto, botón añadir al carrito).
3. **Carrito** (líneas, totales con impuestos/descuentos, ir a checkout).
4. **Checkout** (datos del comprador, elección de método de pago: pasarela o transferencia con comprobante).
5. **Confirmación de orden** (estado según método de pago).
6. **Registro/alta de comercio** (crea tienda automáticamente).
7. **Panel del comercio — productos e inventario**.
8. **Panel del comercio — órdenes** (incluye aprobar/rechazar pago manual).
9. **Panel super-admin** (tiendas, plan/estado de pago, métricas básicas).

- A. Sí, estas 9 pantallas.
- B. Sí, pero reduce a las imprescindibles del walking skeleton (indícalo en Otro).
- C. Sí, pero añade alguna (indícalo en Otro).
- D. No, otra lista (descríbela en Otro).
- X. Other (please specify)

[Answer]: C. Aprobar las 9 pantallas y añadir: selección y personalización de plantilla de la tienda, carga de comprobante de transferencia (comprador) e inicio de sesión/registro del comprador. Dar mayor detalle a las pantallas del walking skeleton y mantener el resto a nivel conceptual.

## Q2. Flujo principal (happy path) a diagramar. ¿Cuál priorizas como "el flujo" del MVP?

- A. Flujo del comprador: entra a la tienda → ve catálogo → detalle → añade al carrito → checkout → paga (pasarela o manual) → confirmación.
- B. Flujo del comerciante: se registra → se crea su tienda → configura y elige plantilla → carga un producto con stock → publica.
- C. Ambos flujos (comprador y comerciante) como los dos principales.
- D. El flujo del walking skeleton end-to-end (comerciante crea tienda+producto y comprador compra con pago manual).
- X. Other (please specify)

[Answer]: D. Flujo: el comerciante se registra, se crea su tienda, carga un producto con stock y publica. El comprador entra a la tienda, agrega al carrito, hace checkout con transferencia, sube el comprobante, y el comerciante lo aprueba tras revisar su banco; la orden queda pagada y el stock confirmado. Variantes a diagramar: pago rechazado y orden vencida por no enviar comprobante (el stock se libera).

## Q3. Dispositivos / formatos a soportar

- A. Responsivo mobile-first (la tienda pública se usará sobre todo en móvil); paneles de admin usables en escritorio y móvil.
- B. Responsivo con prioridad en escritorio.
- C. Solo escritorio en el MVP.
- D. No lo sé / recomiéndame (recomendación: A, mobile-first para la tienda pública).
- X. Other (please specify)

[Answer]:A. Mobile-first para la tienda pública y el checkout, incluida la carga de comprobante desde el móvil. Paneles de administración usables en escritorio y móvil, con especial atención al listado de órdenes y la aprobación de pagos desde el móvil.

## Q4. ¿Hay guías de marca, sistema de diseño o patrones de UI existentes a seguir?

- A. No, partir de cero con un diseño limpio y neutro; cada tienda personaliza colores/logo sobre la plantilla base.
- B. Sí, tengo guías/colores de marca de la plataforma (indícalo en Otro).
- C. Prefiero apoyarme en un sistema de diseño/plantilla de código abierto (sujeto a la investigación de licencias del storefront).
- D. No lo sé / recomiéndame.
- X. Other (please specify)

[Answer]: C. Apoyarme en un sistema de diseño de código abierto para los paneles y el checkout, y evaluar plantillas de código abierto para el storefront, sujeto a la investigación de licencias (compatibles con uso comercial en un SaaS). Cada tienda personaliza colores, logo y tipografía sobre la plantilla elegida.

## Q5. Nivel de accesibilidad objetivo

- A. WCAG 2.1 AA como objetivo (navegación por teclado, contraste, roles ARIA, textos alternativos). Recomendado.
- B. Accesibilidad básica sin comprometerse a un nivel formal en el MVP.
- C. No lo sé / recomiéndame (recomendación: A, WCAG 2.1 AA como línea base).
- X. Other (please specify)

[Answer]: A. Objetivo WCAG 2.1 AA (o 2.2 AA si el diseño lo recomienda), exigible en la tienda pública, el carrito y el checkout. En los paneles de administración, buscar AA con un enfoque de mejora continua. Las plantillas candidatas deben evaluar también su accesibilidad.

## Q6. Estados de pantalla a reflejar en los bocetos (más allá del happy path)

- A. Reflejar los 5 estados clave donde apliquen: vacío, cargando, con datos, error y borde (p. ej. sin productos, pago pendiente de verificación, stock agotado). Recomendado.
- B. Solo el estado con datos (happy path) en esta fase de concepto.
- C. No lo sé / recomiéndame (recomendación: A, al menos vacío/error en las pantallas críticas).
- X. Other (please specify)

[Answer]:A. Reflejar vacío, cargando, con datos, error y borde donde apliquen. Casos de borde propios del producto: stock agotado, orden pendiente de verificación de pago, pago rechazado, comprobante inválido, orden vencida y stock liberado, tienda suspendida o con plan vencido (qué ve el comprador) y tienda sin productos publicados.

---

## Consolidated Summary Confirmation

- Looks correct
- Request changes

[Answer]: Looks correct
