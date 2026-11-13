# Especificación de interacción — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Refined Mockups. Fecha: 2026-11-11. Especificaciones a nivel de componente (formato `component-spec-template.md`). Agnóstico de librería (Q1): describe comportamiento, no implementación. Objetivo WCAG 2.1 AA (Q7). Breakpoints `bp.mobile/tablet/desktop` (Q8).
> Entradas: `mockups.md`, `design-system-mapping.md`, `stories.md`, `requirements.md`.

## Resumen de patrones (Q3)

- **Mobile-first** en tienda pública y checkout; navegación por pila; carrito como SidePanel (desktop) / BottomSheet (móvil); **checkout en una sola página por secciones** (no wizard).
- **Paneles** del comercio y super-admin: tablas + filtros + formularios en página; **ConfirmDialog** obligatorio en acciones sensibles (aprobar/rechazar pago, suspender/activar tienda).
- **Estado por texto + color** siempre (nunca solo color). Un único botón primario por pantalla/sección.

---

## StatusTag (orden / pago)

| Field | Value |
|---|---|
| Component | StatusTag |
| Description | Etiqueta de estado de una orden o de un pago, por texto + color semántico |
| Category | display |

### States

| State | Description | Trigger |
|---|---|---|
| default | Render con etiqueta + color del estado | datos cargados |
| — | No tiene hover/focus (no interactivo) | — |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| kind | `order \| payment` | yes | — | de qué máquina es el estado |
| value | string | yes | — | `pending_payment\|paid\|expired\|cancelled` (order) o `pending\|pending_review\|approved\|rejected\|failed` (payment) |
| detail | string | no | — | texto corto adicional (p. ej. motivo de rechazo resumido) |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| mobile | etiqueta compacta; texto nunca se oculta |
| tablet/desktop | etiqueta con texto completo |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | `status` cuando refleja un cambio en vivo; si no, texto normal |
| Keyboard interaction | no interactivo |
| Label | el valor se escribe como texto legible (p. ej. "PENDIENTE DE VERIFICACIÓN"), no solo color |
| Contrast ratio | AA: texto del tag ≥4.5:1 contra su fondo |
| Screen reader | anuncia "Estado de la orden: …" / "Estado del pago: …" |
| Focus management | n/a |

> Regla de negocio (AC5.3.3): order y payment se muestran como **dos StatusTag separados**; un pago `rejected`/`failed` nunca cambia el StatusTag de la orden a `cancelled`.

---

## PaymentOutcomePanel (resultado de pago en P6)

| Field | Value |
|---|---|
| Component | PaymentOutcomePanel |
| Description | Muestra el resultado del pago y la acción de recuperación según el caso |
| Category | feedback |

### States

| State | Description | Trigger |
|---|---|---|
| approved | Pago aprobado → orden `paid` | pago `approved` |
| pending_review | Comprobante en revisión | pago `pending_review` |
| rejected | Comprobante rechazado por staff | pago `rejected` |
| failed | Intento de pasarela fallido | pago `failed` |
| expired | Reserva/orden vencida | orden `expired` |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| paymentState | string | yes | — | estado consolidado del pago de la orden |
| orderState | string | yes | — | estado de la orden (para no mostrar "cancelada" por pago) |
| rejectionReason | string | no | — | motivo registrado por el staff (solo `rejected`) |
| expiresAt | datetime | no | — | vencimiento de la reserva, si vigente |
| isGuest | boolean | no | false | ajusta el punto de reentrada (OQ-US-1) |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| mobile | CTA a ancho completo; mensaje y motivo apilados |
| tablet/desktop | mensaje + CTA en bloque centrado |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | `region` con `aria-live=polite` para anunciar el resultado |
| Keyboard interaction | CTA enfocable y activable por teclado |
| Label | mensaje por texto; motivo de rechazo como texto |
| Contrast ratio | AA |
| Screen reader | anuncia resultado + acción disponible |
| Focus management | al entrar, foco al encabezado del resultado |

> Regla (Q5 / AC6.2.3, AC6.3.3, AC5.3.5):
> - `rejected` → muestra **motivo** + CTA "Volver a subir comprobante".
> - `failed` → "El pago no se completó" + CTA "Reintentar pago".
> - En `rejected`/`failed`, muestra que la orden sigue **vigente hasta `expiresAt`** y **no** la presenta como cancelada.

---

## PaymentMethodRadioGroup (checkout P4)

| Field | Value |
|---|---|
| Component | PaymentMethodRadioGroup |
| Description | Selección del método de pago, limitada a los métodos configurados por la tienda |
| Category | input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Opciones disponibles según config | carga de checkout |
| focus | Opción enfocada | Tab / flechas |
| error | Ningún método seleccionado al confirmar | submit sin selección |
| empty (borde) | La tienda no tiene método configurado | config incompleta |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| availableMethods | `array<gateway\|transfer>` | yes | — | derivado de P14 (config de la tienda) |
| value | string | no | — | método seleccionado |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| mobile | opciones apiladas, objetivo táctil ≥44px |
| tablet/desktop | apiladas o en fila según espacio |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | `radiogroup` con `aria-labelledby` al encabezado de sección |
| Keyboard interaction | flechas para moverse, Space para seleccionar |
| Label | cada opción con label textual |
| Contrast ratio | AA; foco visible en la opción |
| Screen reader | anuncia nombre del grupo y opción seleccionada |
| Focus management | foco entra al primer radio |

> Regla (AC7.1.2): solo se listan los métodos que la tienda configuró en P14; si solo hay transferencia, la pasarela no aparece.

---

## ProofUpload (carga de comprobante P5)

| Field | Value |
|---|---|
| Component | ProofUpload |
| Description | Carga del comprobante de transferencia, con alternativa de cámara en móvil |
| Category | input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Sin archivo seleccionado | carga de pantalla |
| focus | Control de carga enfocado | Tab |
| loading | Subiendo archivo | envío |
| error | Archivo inválido/grande/formato | validación |
| disabled (borde) | Orden vencida → no permite subir | `expiresAt` pasado |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| acceptedFormats | array<string> | yes | — | formatos permitidos (definidos en diseño) |
| maxSize | number | yes | — | tamaño máximo |
| allowCamera | boolean | no | true | habilita captura por cámara en móvil |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| mobile | botón de cámara visible; vista previa a ancho completo |
| tablet/desktop | selección de archivo; vista previa en bloque |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | `button`/input file con label |
| Keyboard interaction | Tab + Enter/Space abre el selector |
| Label | "Sube tu comprobante" + formatos/tamaño en texto de ayuda |
| Contrast ratio | AA |
| Screen reader | anuncia archivo seleccionado y estado de subida |
| Focus management | tras subir, foco al mensaje "pendiente de verificación" |

> Reglas: al enviar → pago `pending_review` + notifica al staff (AC6.1.1) y muestra aviso Ley 1581 (NFR3.1). Comprobante se almacena cifrado, acceso solo Owner/Admin + staff de órdenes + super-admin soporte auditado (AC6.1.2 / NFR3.2).

---

## PaymentReviewAction (aprobar/rechazar pago manual, P9)

| Field | Value |
|---|---|
| Component | PaymentReviewAction |
| Description | Acción del Owner/Admin o staff de órdenes para aprobar o rechazar un comprobante |
| Category | input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Opciones Aprobar/Rechazar | abrir "Revisar" |
| focus | Control enfocado | Tab |
| loading | Confirmando la decisión | submit |
| error | Fallo al registrar | error de sistema |
| disabled (borde) | Orden/reserva vencida → aprobar bloqueado | `expired` |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| decision | `approve \| reject` | yes | — | decisión |
| reason | string | cond. | — | obligatorio si `reject` |
| actorRole | `owner_admin \| staff_orders` | yes | — | autorizados (AC6.2.5) |

### Responsive Behaviour

| Breakpoint | Behaviour |
|---|---|
| mobile | acciones a ancho completo; ConfirmDialog ocupa pantalla |
| tablet/desktop | acciones en fila; ConfirmDialog como modal |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | botones + `dialog` (ConfirmDialog) con foco atrapado |
| Keyboard interaction | Enter/Space activa; Esc cierra el diálogo |
| Label | "Aprobar pago" / "Rechazar pago"; campo motivo con label |
| Contrast ratio | AA |
| Screen reader | anuncia la confirmación y el resultado; lee el recordatorio de verificar el banco |
| Focus management | foco al diálogo al abrir; vuelve al disparador al cerrar |

> Reglas: aprobar → pago `approved`, orden `paid`; rechazar con motivo → pago `rejected`, la orden NO se cancela (AC6.2.2/.3). Si la orden está `expired`, aprobar se rechaza por consistencia (AC6.2.4). Registro visible de quién/cuándo. Recordatorio: "Verifica el dinero en tu app bancaria" (la plataforma no valida el movimiento).

---

## SubdomainField (registro de comercio P7)

| Field | Value |
|---|---|
| Component | SubdomainField |
| Description | Campo de subdominio con verificación de disponibilidad en línea |
| Category | input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Vacío/escribiendo | carga |
| focus | Enfocado | Tab |
| loading | Verificando disponibilidad | debounce tras escribir |
| error | Subdominio en uso / inválido | respuesta de verificación |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| value | string | yes | — | subdominio propuesto |
| suffix | string | yes | `.plataforma.com` | dominio base de la plataforma |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | textbox con `aria-describedby` al mensaje de disponibilidad |
| Keyboard interaction | entrada estándar |
| Label | "Subdominio"; disponibilidad anunciada por texto (✓/✗) |
| Contrast ratio | AA |
| Screen reader | anuncia "disponible"/"en uso" con `aria-live` |
| Focus management | el error no roba el foco |

> Regla: crear → identidad Owner/Admin autenticable + tenant **no activado** (AC1.1.1/.4); subdominio único (AC1.1.2/.3).

---

## BrandTokenControls (personalización P11)

| Field | Value |
|---|---|
| Component | BrandTokenControls |
| Description | Controles de los tokens de marca por tienda (color primario/acento, logo, tipografía) con validación de contraste |
| Category | input |

### States

| State | Description | Trigger |
|---|---|---|
| default | Valores actuales/por defecto | carga |
| focus | Control enfocado | Tab |
| loading | Guardando | submit |
| error | Contraste insuficiente / archivo de logo inválido | validación |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| primaryColor | string(hex) | yes | token por defecto | color primario |
| accentColor | string(hex) | yes | token por defecto | color de acento |
| logo | file | no | — | logo de la tienda (alt = nombre) |
| fontFamily | string | no | base | familia tipográfica |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | inputs con labels; selector de color con valor hex textual |
| Keyboard interaction | entrada estándar; selección por teclado |
| Label | cada token con label; aviso de contraste como texto |
| Contrast ratio | **valida** AA del par texto/primario y del anillo de foco (design-system §7) |
| Screen reader | anuncia "contraste suficiente/insuficiente" |
| Focus management | el error de contraste no bloquea el foco |

> Regla (Q2 + design-system §7): si el color no cumple AA, se avisa y se deriva un tono accesible; no se guarda un combo que rompa AA. Personalización acotada: solo estos cuatro ejes.

---

## OrderLookupEntry (reentrada del invitado, OQ-US-1)

| Field | Value |
|---|---|
| Component | OrderLookupEntry |
| Description | Punto de reentrada para consultar el estado de la orden (comprador invitado o autenticado) |
| Category | navigation |

### States

| State | Description | Trigger |
|---|---|---|
| default | Enlace/acción "Ver estado de mi orden" | confirmación P6 |
| focus | Enfocado | Tab |

### Props / Inputs

| Prop | Type | Required | Default | Description |
|---|---|---|---|---|
| isGuest | boolean | yes | — | invitado vs autenticado |
| orderRef | string | yes | — | referencia de la orden |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | link/button con nombre accesible |
| Keyboard interaction | activable por teclado |
| Label | "Ver estado de mi orden" |
| Contrast ratio | AA |
| Screen reader | anuncia destino |
| Focus management | estándar |

> Regla (Q6 / OQ-US-1): representación **genérica**. El mecanismo exacto (enlace firmado, código+contacto, u otro) queda **abierto** y NO se compromete en el diseño; se resuelve en diseño funcional. No permite enumerar ni acceder a órdenes ajenas (AC2.2.4).

---

## ConfirmDialog (acciones sensibles, transversal)

| Field | Value |
|---|---|
| Component | ConfirmDialog |
| Description | Diálogo de confirmación para acciones sensibles (aprobar/rechazar pago, suspender/activar tienda) |
| Category | feedback |

### States

| State | Description | Trigger |
|---|---|---|
| open | Visible, esperando confirmación | acción sensible |
| loading | Procesando la confirmación | confirmar |
| closed | Oculto | cancelar/confirmar/Esc |

### Accessibility

| Requirement | Implementation |
|---|---|
| ARIA role | `dialog` modal con `aria-labelledby`/`aria-describedby` |
| Keyboard interaction | Esc cierra; Tab circula dentro (foco atrapado) |
| Label | título + descripción de la consecuencia |
| Contrast ratio | AA |
| Screen reader | anuncia el diálogo y su propósito al abrir |
| Focus management | foco al abrir va al diálogo; al cerrar vuelve al disparador |

---

## Estados transversales (Q4)

Cada pantalla de `mockups.md` implementa, cuando aplique:
- **Cargando:** `SkeletonLoader` (no spinners aislados sin contexto).
- **Vacío:** `EmptyState` con mensaje + CTA/guía.
- **Con datos:** contenido real.
- **Error:** `ErrorState`/`InlineAlert` con mensaje accionable y, si aplica, reintento.
- **Borde (por pantalla):** stock perdido en checkout (P3/P4), orden vencida (P5/P6/P9), pasarela no conectada (P4/P14), tienda suspendida/plan vencido (P1), una sola plantilla en MVP (P11), aprobar sobre orden vencida (P9).

## Sources

- [mockups] P1–P14 y sus reglas de interacción.
- [stories] US refinadas (pagos separados, identidad vs activación, config restringida, OQ-US-1).
- [requirements] FR/NFR relacionados; NFR8.2 (AA), NFR3.1/3.2 (datos/comprobantes).
- `component-spec-template.md`; Q3–Q7 de `refined-mockups-questions.md`.

## Assumptions & Open Questions

- [assumption] Props/tipos son contrato de comportamiento, agnósticos de framework; su mapeo a la librería real es de diseño técnico.
- OQ-US-1 — Mecanismo de consulta de la orden del invitado: abierto; `OrderLookupEntry` lo representa sin cerrarlo.
