# Mapeo al sistema de diseño (tokens) — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Refined Mockups. Fecha: 2026-11-11.
> Entradas upstream: `wireframes.md`, `user-flow.md`, `stories.md`, `requirements.md`, `team-practices.md`.
>
> Decisión Q1/Q2: sistema de diseño **agnóstico del framework**, expresado como **tokens de diseño**. No se elige librería ni framework de componentes aquí; esa decisión es de la fase de diseño técnico (previa revisión de licencias, mantenibilidad y costo). Los tokens son el contrato de diseño: la implementación los mapea a su librería real sin cambiar las decisiones visuales.

## 1. Principios

- **Tokens primero:** todo valor visual (color, tipografía, espaciado, radio, sombra, z-index, breakpoint) es un token con nombre semántico, no un literal disperso en los mockups.
- **Dos capas de color:** tokens *base* (paleta cruda) y tokens *semánticos* (rol: `color.text.default`, `color.bg.surface`, `color.border.default`). Los mockups y componentes referencian solo los semánticos.
- **Personalización por tienda controlada (Q2):** solo cuatro ejes de marca son personalizables por tienda — color primario, color de acento, logo y familia tipográfica — con valores por defecto neutrales en la plantilla base. Espaciado, radios, estructura y componentes NO se personalizan en el MVP.
- **Accesibilidad incorporada (Q7):** los pares de color definidos deben cumplir contraste WCAG 2.1 AA (4.5:1 texto normal, 3:1 texto grande y elementos de Ui/foco). La personalización no puede romper ese contraste (ver §7).

## 2. Tokens de color

### 2.1 Tokens base (plantilla por defecto, neutrales)

| Token base | Valor ref. (ejemplo) | Nota |
|---|---|---|
| `base.neutral.0` | #FFFFFF | fondo claro |
| `base.neutral.50` | #F7F8FA | superficie sutil |
| `base.neutral.100` | #EDEFF2 | bordes suaves / relleno |
| `base.neutral.300` | #C9CED6 | borde por defecto |
| `base.neutral.600` | #5B6372 | texto secundario |
| `base.neutral.900` | #1A1D23 | texto principal |
| `base.primary.500` | #2563EB | **personalizable por tienda** (color primario por defecto) |
| `base.primary.600` | #1D4ED8 | estado hover del primario (derivado) |
| `base.accent.500` | #7C3AED | **personalizable por tienda** (color de acento por defecto) |
| `base.success.500` | #15803D | estado positivo (pagado/aprobado) |
| `base.warning.500` | #B45309 | atención (pendiente/por vencer) |
| `base.danger.500` | #B91C1C | error/rechazo/fallo |
| `base.info.500` | #0E7490 | informativo |

> Los valores hex son **referencias de partida**, no una marca definitiva; se ajustan en diseño técnico siempre respetando el contraste de §7.

### 2.2 Tokens semánticos (los que usan los mockups)

| Token semántico | Mapea a | Uso |
|---|---|---|
| `color.text.default` | `base.neutral.900` | texto principal |
| `color.text.muted` | `base.neutral.600` | texto secundario, ayudas |
| `color.text.onPrimary` | `base.neutral.0` | texto sobre botón primario |
| `color.bg.page` | `base.neutral.0` | fondo de página |
| `color.bg.surface` | `base.neutral.50` | tarjetas, paneles |
| `color.border.default` | `base.neutral.300` | bordes de inputs, tablas |
| `color.action.primary` | `base.primary.500` | botón primario, enlaces de acción |
| `color.action.primaryHover` | `base.primary.600` | hover/active del primario |
| `color.action.accent` | `base.accent.500` | realces secundarios |
| `color.status.success` | `base.success.500` | orden `paid`, pago `approved` |
| `color.status.warning` | `base.warning.500` | `pending_payment`, `pending_review`, por vencer |
| `color.status.danger` | `base.danger.500` | `error`, pago `rejected`/`failed` |
| `color.status.info` | `base.info.500` | avisos neutros (p. ej. soporte solo lectura) |
| `color.focus.ring` | `base.primary.500` (≥3:1 vs fondo) | anillo de foco visible (Q7) |

## 3. Tipografía

| Token | Valor ref. | Uso |
|---|---|---|
| `font.family.base` | familia sans del sistema de diseño | **personalizable por tienda** (default neutral) |
| `font.family.mono` | monospace del sistema | montos/IDs cuando ayude a lectura |
| `font.size.xs` | 12 px | metadatos, notas de ayuda |
| `font.size.sm` | 14 px | texto secundario, tablas |
| `font.size.md` | 16 px | cuerpo base (mínimo legible en móvil) |
| `font.size.lg` | 20 px | subtítulos |
| `font.size.xl` | 24 px | h2 |
| `font.size.2xl` | 30 px | h1 |
| `font.weight.regular` | 400 | cuerpo |
| `font.weight.medium` | 500 | énfasis, labels |
| `font.weight.bold` | 700 | títulos, botones |
| `line.height.tight` | 1.25 | títulos |
| `line.height.base` | 1.5 | cuerpo (legibilidad) |

Regla: tamaño de cuerpo mínimo 16 px en superficies públicas para legibilidad móvil; la jerarquía de encabezados sigue un único `h1` por pantalla (ver accessibility-checklist).

## 4. Espaciado, radios, sombras (NO personalizables)

| Token | Valor ref. |
|---|---|
| `space.1` … `space.8` | 4, 8, 12, 16, 24, 32, 48, 64 px (escala base 4) |
| `radius.sm` / `radius.md` / `radius.lg` / `radius.full` | 4 / 8 / 12 / 9999 px |
| `shadow.sm` / `shadow.md` | elevación de tarjeta / de diálogo |
| `size.touchTarget.min` | 44×44 px (objetivo táctil mínimo, Q7/Q3 mobile-first) |

## 5. Breakpoints (Q8)

| Token | Rango ref. | Diseño |
|---|---|---|
| `bp.mobile` | base, ~360–430 px | mobile-first (tienda pública y checkout) |
| `bp.tablet` | ~768 px | reflow a 2 columnas donde aplique |
| `bp.desktop` | ~1024–1280+ px | paneles priorizan esta vista |

Son referencias de diseño, no límites rígidos.

## 6. Componentes base (inventario, agnóstico de librería)

Catálogo mínimo que la implementación mapeará a su librería. Cada uno se especifica a nivel de componente en `interaction-spec.md`.

- **Navegación:** AppHeader (público), AdminNav (panel), Breadcrumb/Back.
- **Acción:** Button (`primary` único por pantalla, `secondary`, `ghost`, `danger`), IconButton, Link.
- **Entrada:** TextField, PasswordField, Select, RadioGroup (método de pago), Checkbox (aceptación de términos), FileUpload (comprobante/imágenes, con alternativa cámara en móvil), MoneyInput.
- **Presentación:** Card (producto), DataTable (productos/órdenes/tiendas, con encabezados asociados), Badge/StatusTag (estados de orden/pago por texto+color), Thumbnail/ImageGallery, DescriptionList (totales, snapshot).
- **Feedback:** Toast/InlineAlert (success/warning/danger/info), SkeletonLoader (estado cargando), EmptyState (vacío con CTA), ErrorState (accionable), ConfirmDialog (acciones sensibles: aprobar/rechazar pago, suspender tienda).
- **Layout:** Container, Stack, Grid (grid de productos), SidePanel/BottomSheet (carrito, Q3).

## 7. Reglas de personalización y accesibilidad (Q2 + Q7)

- Solo `base.primary.*`, `base.accent.*`, `font.family.base` y el logo son personalizables por tienda.
- El color primario personalizado debe mantener contraste **≥4.5:1** para texto `color.text.onPrimary` sobre él y **≥3:1** para el anillo de foco contra el fondo; si el color elegido no cumple, el sistema deriva un tono accesible (oscurece/aclara) y lo documenta, en vez de romper el contraste.
- El estado nunca se comunica solo por color: todo `StatusTag` lleva etiqueta de texto (`PENDIENTE`, `PAGADA`, `RECHAZADO`, etc.) además del color semántico.
- El logo personalizado requiere texto alternativo = nombre de la tienda.

## Sources

- [wireframes] Nota de diseño apoyado en sistema open source con personalización por tienda (colores, logo, tipografía).
- [requirements] NFR8.1 (responsive mobile-first), NFR8.2 (WCAG 2.1/2.2 AA), FR8.3 (personalización de plantilla), FR1.2 (configuración de marca).
- [stories] US3.4 (plantilla predeterminada funcional + personalización básica, Must), US1.2 (configuración básica de la tienda).
- Q1, Q2, Q7, Q8 de `refined-mockups-questions.md`.

## Assumptions & Open Questions

- [assumption] Los valores hex/px son referencias de partida; la paleta y la escala finales se confirman en diseño técnico sin alterar los roles semánticos ni el contraste AA.
- [assumption] La librería de componentes concreta no se decide aquí; el inventario de §6 es agnóstico y sirve de contrato.
