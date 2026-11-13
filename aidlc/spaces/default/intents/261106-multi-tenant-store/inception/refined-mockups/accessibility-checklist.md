# Lista de verificación de accesibilidad — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Refined Mockups. Fecha: 2026-11-11. Objetivo: **WCAG 2.1 AA** (Q7) en tienda pública, carrito y checkout con foco estricto; paneles administrativos apuntan a AA con mejora continua.
> Entradas: `mockups.md`, `interaction-spec.md`, `design-system-mapping.md`, `requirements.md` (NFR8.1, NFR8.2).
>
> **Importante:** esta lista es una **meta de diseño** y un contrato de verificación para construcción. La conformidad WCAG real requiere pruebas con tecnologías de asistencia (lectores de pantalla, navegación solo-teclado, zoom) y revisión experta, que se ejecutan en construcción/QA, no en esta etapa (NFR8.2).

## 1. Alcance y niveles

| Superficie | Nivel objetivo | Rigor |
|---|---|---|
| Tienda pública (P1, P2) | WCAG 2.1 AA | estricto |
| Carrito y checkout (P3, P4, P5, P6) | WCAG 2.1 AA | estricto |
| Autenticación comprador / registro comercio (P7, P12) | WCAG 2.1 AA | estricto |
| Paneles comercio (P8, P9, P11, P13, P14) | WCAG 2.1 AA | AA con mejora continua |
| Panel super-admin (P10) | WCAG 2.1 AA | AA con mejora continua |

## 2. Criterios transversales (aplican a todas las pantallas)

- [ ] **1.1.1 Contenido no textual (A):** toda imagen de producto, logo, comprobante y miniatura tiene texto alternativo significativo (el logo = nombre de la tienda).
- [ ] **1.3.1 Información y relaciones (A):** encabezados, listas, tablas y formularios usan estructura semántica; tablas (P8/P9/P10) con encabezados asociados a celdas.
- [ ] **1.4.1 Uso del color (A):** el estado (orden/pago, stock, conexión de pasarela) se comunica por **texto + color**, nunca solo color (StatusTag).
- [ ] **1.4.3 Contraste mínimo (AA):** texto normal ≥4.5:1, texto grande/elementos UI ≥3:1; validado por los tokens de `design-system-mapping §7`; la personalización por tienda no puede romperlo.
- [ ] **1.4.11 Contraste de elementos no textuales (AA):** bordes de input, íconos informativos y anillo de foco ≥3:1.
- [ ] **1.4.4 Redimensionar texto (AA):** el contenido es legible y operable al 200% de zoom sin pérdida de contenido o función.
- [ ] **1.4.10 Reflow (AA):** sin scroll horizontal en 320px de ancho; layouts `bp.mobile` reflúyen a una columna.
- [ ] **2.1.1 Teclado (A):** toda función operable por teclado (navegación, grid de productos, variantes, carrito, checkout, aprobar/rechazar pago, diálogos).
- [ ] **2.1.2 Sin trampa de teclado (A):** el foco entra y sale de SidePanel/BottomSheet (carrito) y ConfirmDialog sin quedar atrapado salvo el atrapado intencional del modal (con Esc para salir).
- [ ] **2.4.1 Evitar bloques (A):** enlace "saltar al contenido" en páginas públicas.
- [ ] **2.4.2 Título de página (A):** cada pantalla tiene título descriptivo.
- [ ] **2.4.3 Orden del foco (A):** el orden de tabulación sigue el orden visual/lógico.
- [ ] **2.4.6 Encabezados y etiquetas (AA):** un único `h1` por pantalla; secciones con encabezados descriptivos (checkout usa secciones, no múltiples `h1`).
- [ ] **2.4.7 Foco visible (AA):** anillo de foco visible (`color.focus.ring`) en todo control, con contraste ≥3:1.
- [ ] **3.1.1 Idioma de la página (A):** idioma declarado (contenido en español).
- [ ] **3.2.4 Identificación consistente (AA):** componentes con la misma función (botón primario, StatusTag, acciones de tabla) se nombran y comportan igual en todas las pantallas.
- [ ] **3.3.1 Identificación de errores (A):** errores de formulario descritos en texto, asociados al campo.
- [ ] **3.3.2 Etiquetas o instrucciones (A):** todo input con label visible y asociado; instrucciones donde haga falta (formatos de archivo, subdominio).
- [ ] **3.3.3 Sugerencia ante error (AA):** mensajes de error proponen cómo corregir (p. ej. subdominio en uso → sugerir otro).
- [ ] **4.1.2 Nombre, rol, valor (A):** controles personalizados (RadioGroup de pago, StatusTag como `status`, ConfirmDialog como `dialog`) exponen nombre/rol/valor.
- [ ] **4.1.3 Mensajes de estado (AA):** cambios de estado (comprobante enviado, pago aprobado/rechazado/fallido, subdominio disponible) se anuncian con `aria-live` sin robar el foco.
- [ ] **Objetivo táctil:** controles interactivos ≥44×44px en móvil (`size.touchTarget.min`).

## 3. Puntos específicos por pantalla

| Pantalla | Puntos de accesibilidad clave |
|---|---|
| P1 Landing | landmarks header/nav/main/footer; tarjetas enfocables con nombre+precio accesibles; buscador con label |
| P2 Detalle | `h1`=producto; galería con alt; selección de variante por teclado; stock por texto |
| P3 Carrito | panel con foco atrapado mientras abierto + Esc; totales legibles; cantidades con label |
| P4 Checkout | secciones con encabezado; RadioGroup de pago con nombre accesible; errores en texto; aviso Ley 1581 antes de confirmar |
| P5 Comprobante | FileUpload con label (formatos/tamaño), alternativa cámara; estado "pendiente" anunciado (`aria-live`) |
| P6 Confirmación | dos StatusTag (orden/pago) por texto; resultado anunciado; CTA de recuperación accesible; nunca "cancelada" solo por color |
| P7 Registro | disponibilidad de subdominio por texto (`aria-live`); checkbox términos con label; aviso Ley 1581 |
| P8 Productos | DataTable con `th` asociados; stock 0 por texto "Agotado"; acciones con nombre accesible |
| P9 Órdenes/pago | estado por texto+icono; comprobante con alt; ConfirmDialog accesible; recordatorio banco leído por lector |
| P10 Super-admin | tablas con encabezados; modo soporte declara "solo lectura" por texto; confirmaciones |
| P11 Personalización | selector de color con **valor hex textual**; aviso de contraste como texto; vista previa con descripción |
| P12 Login comprador | formulario etiquetado; opción invitado clara; términos con label |
| P13 Formulario producto | labels; variantes como grupo etiquetado; carga de imágenes con alt |
| P14 Configuración | estado de conexión de pasarela por texto; credenciales nunca mostradas en claro tras guardar; un botón primario por sección |

## 4. Cobertura de requisitos de accesibilidad

| Requisito | Cómo lo cubre esta lista |
|---|---|
| NFR8.1 (responsive mobile-first) | §2 (reflow 1.4.10, objetivo táctil), breakpoints en `mockups.md` |
| NFR8.2 (WCAG 2.1/2.2 AA; conformidad real requiere pruebas con AT y revisión experta) | §1 alcance + §2 criterios AA + nota de verificación en construcción/QA |

## 5. Verificación (se ejecuta en construcción/QA, no aquí)

- [ ] Prueba solo-teclado de los flujos del walking skeleton (registro → producto → compra → comprobante → aprobación).
- [ ] Prueba con lector de pantalla de checkout, carga de comprobante y resultado de pago.
- [ ] Verificación de contraste automatizada sobre los tokens finales y sobre al menos una personalización de tienda.
- [ ] Zoom 200% y ancho 320px sin pérdida de función.
- [ ] Revisión experta de accesibilidad antes de la validación de preparación para producción.

## Sources

- [requirements] NFR8.1, NFR8.2.
- [wireframes] notas de accesibilidad por pantalla (encabezado, landmarks, entrada por teclado, estado por texto).
- [mockups] / [interaction-spec] estados y componentes; `design-system-mapping §7` (contraste y personalización).
- Q7 de `refined-mockups-questions.md`.

## Assumptions & Open Questions

- [assumption] WCAG 2.1 AA es la meta; la conformidad real exige pruebas con tecnologías de asistencia y revisión experta, fuera de esta etapa (NFR8.2).
- [assumption] Si se adopta una librería de componentes en diseño técnico, debe soportar estos criterios (foco, roles ARIA, contraste); se verifica al elegirla.
