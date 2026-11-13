# Mockups refinados (media fidelidad) — Plataforma de tienda virtual multi-negocio

> Fase: Inception / Refined Mockups. Fecha: 2026-11-11. Media fidelidad en Markdown (Q9): por pantalla, estructura/layout + especificación (jerarquía, componentes, estados, interacción, responsive, accesibilidad). Cubre las 14 pantallas P1–P14. Sirve de **contrato de diseño** para construcción; la alta fidelidad visual se produce luego en la herramienta de diseño / el frontend.
> Entradas upstream: `wireframes.md` (P1–P14), `user-flow.md`, `stories.md`, `requirements.md`, `team-practices.md`, y `design-system-mapping.md` (tokens/componentes).
>
> Convenciones: componentes y tokens referencian `design-system-mapping.md`. Los estados siguen Q4 (cargando / vacío / con datos / error / borde, cuando apliquen). Accesibilidad objetivo WCAG 2.1 AA (Q7); detalle transversal en `accessibility-checklist.md`. Breakpoints `bp.mobile/tablet/desktop` (Q8). El estado se comunica por **texto + color**, nunca solo color.
> 🟢 = pantalla del walking skeleton.

---

## P1 🟢 — Landing / tienda pública (storefront) — mobile-first
Historias: US3.2, US3.3, US3.4 · Requisitos: FR3.2, FR3.3, FR1.5, NFR8.1

```
bp.mobile                               bp.desktop (grid 3–4 col)
+------------------------------+        +--------------------------------------------+
| [≡] LogoTienda  [🔍] [🛒 2]  |        | LogoTienda   [buscar........]   [🛒 2]      |
+------------------------------+        +--------------------------------------------+
| Categorías ▸ Todas·CatA·CatB |        | Todas | CatA | CatB | CatC                  |
+------------------------------+        +--------------------------------------------+
| [img]  Producto 1            |        | [img]    [img]    [img]    [img]           |
| $precio           [Ver]      |        | Prod1    Prod2    Prod3    Prod4           |
| [img]  Producto 2            |        | $..[Ver] $..[Ver] $..[Ver] $..[Ver]       |
| $precio           [Ver]      |        +--------------------------------------------+
+------------------------------+        | Footer: contacto · info de la tienda       |
| Footer: contacto · info      |        +--------------------------------------------+
+------------------------------+
```

- **Jerarquía:** `h1` = nombre de la tienda (en el header/marca). Grid de `Card` de producto (Thumbnail + nombre + precio + Button "Ver").
- **Componentes:** AppHeader (marca personalizada por tokens de tienda), SearchField, CartButton (con contador), CategoryFilter (chips/lista), Grid de Card, Footer.
- **Interacción:** buscar por texto simple; filtrar por categoría; "Ver" → P2. Carrito → P3 (SidePanel en desktop, BottomSheet en móvil).
- **Estados:** cargando = `SkeletonLoader` de tarjetas; **vacío** = EmptyState "Esta tienda aún no tiene productos" (sin CTA de compra); con datos = grid; error = ErrorState "No se pudo cargar el catálogo" + reintentar; **borde** = tienda suspendida/plan vencido → InlineAlert `info/warning` coherente con la suspensión, sin exponer datos internos (traza US1.3/FR1.6).
- **Responsive:** 1 col (móvil) → 2 col (tablet) → 3–4 col (desktop); header colapsa a `≡`.
- **Accesibilidad:** landmarks header/nav/main/footer; tarjetas enfocables por teclado con nombre y precio en el nombre accesible; buscador con label.

## P2 — Detalle de producto — mobile-first
Historias: US3.3 · Requisitos: FR3.2, FR7.1, FR7.2

```
+------------------------------+
| ‹ Volver   LogoTienda   [🛒] |
+------------------------------+
|        [ imagen principal ]  |
|        ○ ○ ○  (miniaturas)   |
+------------------------------+
| h1 Nombre del producto       |
| $precio (imp. incluido/added)|
| Variante: [Talla ▾][Color ▾] |
| Stock: Disponible / Agotado  |
| [    Añadir al carrito    ]  |
| Descripción…                 |
+------------------------------+
```

- **Jerarquía:** `h1` = nombre del producto; precio con tratamiento de impuesto según configuración de la tienda (AC3.3.2); selector de variante; Button primario único.
- **Componentes:** Back, ImageGallery (alt text), DescriptionList (precio/variante/stock), Select (variantes), Button primario.
- **Estados:** con datos; **agotado** = Button deshabilitado + InlineAlert "Sin stock"; cargando = skeleton de imagen + líneas; error; **borde** = variante seleccionada sin stock (deshabilita añadir para esa variante).
- **Responsive:** imagen full-width en móvil; dos columnas (imagen | info) en desktop.
- **Accesibilidad:** imagen con alt; selección de variante por teclado; stock por texto (no solo color).

## P3 🟢 — Carrito
Historias: US5.1, US5.2 · Requisitos: FR5.1, FR5.2, FR7.4

```
+------------------------------+
| h1 Carrito (2)      [cerrar] |
+------------------------------+
| Producto1  x1  $..      [🗑] |
| Producto2  x2  $..      [🗑] |
+------------------------------+
| Subtotal           $...      |
| Impuestos          $...      |
| Descuento          -$..  (si)|
| Total              $...      |
| [     Ir al checkout     ]   |
+------------------------------+
```

- **Jerarquía:** `h1` = "Carrito"; DescriptionList de totales; Button primario único.
- **Interacción:** editar cantidad, eliminar ítem; "Ir al checkout" → P4. **Descuento opcional (Q5/AC5.2.1):** la línea "Descuento" solo aparece si hay descuento aplicable; su ausencia no bloquea nada.
- **Estados:** **vacío** = EmptyState "Tu carrito está vacío" + CTA "Seguir comprando" → P1; con datos; **borde** = un ítem quedó sin stock → InlineAlert por ítem y ajuste; error.
- **Responsive:** SidePanel (desktop) / BottomSheet (móvil); también vista de página en móvil pequeño.
- **Accesibilidad:** cantidades con label; totales legibles por lector; foco atrapado en el panel mientras está abierto; cierre por teclado.

## P4 🟢 — Checkout (datos + método de pago) — una sola página, mobile-first
Historias: US2.2, US5.2 · Requisitos: FR5.2, FR2.4, NFR3.1

```
+------------------------------+
| h1 Checkout                  |
+------------------------------+
| § Datos del comprador        |
|  Nombre [__________]         |
|  Email  [__________]         |
|  Tel    [__________]         |
|  (si autenticado: precargado)|
+------------------------------+
| § Método de pago (según cfg) |
|  ( ) Pasarela (tarjeta/PSE)  |
|  (•) Transferencia+comprobante|
+------------------------------+
| [ ] Acepto tratamiento de    |
|     datos y términos (enlace) |
+------------------------------+
| Resumen: Total $...          |
| [     Confirmar pedido     ] |
+------------------------------+
```

- **Jerarquía:** `h1` = "Checkout"; secciones (`§` = encabezado de sección, no múltiples `h1`); un Button primario.
- **Interacción:** invitado ingresa datos / autenticado precargado (US2.2); RadioGroup de método **solo con los métodos configurados** por la tienda (P14); checkbox de términos obligatorio antes de confirmar; "Confirmar pedido" crea la orden en `pending_payment` con stock reservado (AC5.2.2) → si transferencia: P5; si pasarela: flujo P-pasarela → P6.
- **Estados:** con datos; validación (errores en texto junto al campo); cargando (procesando); error (red/pasarela) con mensaje accionable; **borde** = stock perdido durante el checkout, total o **parcial** → InlineAlert `danger` que indica qué ítem/cantidad ya no está disponible y devuelve al carrito (P3) para ajustar; no se crea la orden hasta resolverlo.
- **Responsive:** una columna (móvil) → resumen fijo lateral (desktop).
- **Accesibilidad:** labels asociadas; RadioGroup con nombre accesible; errores anunciados; aviso Ley 1581 visible antes de confirmar.

## P5 🟢 — Carga de comprobante de transferencia (comprador) — mobile-first
Historias: US6.1 · Requisitos: FR6.2, NFR3.2, FR6.3

```
+------------------------------+
| h1 Pago por transferencia    |
+------------------------------+
| Instrucciones de la tienda:  |
|  Banco / cuenta / titular /  |
|  valor a transferir          |
+------------------------------+
| Sube tu comprobante:         |
|  [📎 Archivo / 📷 Cámara]    |
|  [ vista previa ]            |
| [    Enviar comprobante    ] |
+------------------------------+
| ⓘ Tu orden queda PENDIENTE   |
|   de verificación.           |
|   Reserva vigente hasta:     |
|   <fecha/hora expiresAt>     |
+------------------------------+
```

- **Jerarquía:** `h1` = "Pago por transferencia"; FileUpload con alternativa cámara en móvil; Button primario único.
- **Interacción:** subir comprobante → pago `pending_review` + notificación al staff (AC6.1.1). Muestra `expiresAt` (duración configurable por tienda/método, AC4.1.3/.4). Reintento posible incluso para invitado mientras la orden/reserva sigan vigentes (FR6.3) vía P6/OQ-US-1.
- **Estados:** con datos; cargando (subiendo, con progreso); error (archivo inválido/grande, formato); **borde** = orden vencida → mensaje "La reserva venció, el stock se liberó" + ruta a reintentar la compra.
- **Accesibilidad:** FileUpload con label (formatos, tamaño), vista previa con alt; estado "pendiente" anunciado; aviso Ley 1581 al enviar.

## P6 🟢 — Confirmación / estado de la orden
Historias: US5.3, US6.1, US6.2, US6.3, US2.2 · Requisitos: FR5.3, FR5.4, FR6.3

```
+------------------------------+
| h1 ¡Gracias por tu compra!   |
+------------------------------+
| Orden #1234                  |
| Estado de la orden: <tag>    |
| Estado del pago:    <tag>    |
|  · Pasarela OK → PAGADA      |
|  · Transferencia → PENDIENTE |
|  · rejected/failed → ver abajo|
+------------------------------+
| Detalle de ítems y total     |
| [ Ver estado de mi orden ]   |
| (si rejected):               |
|  Motivo: <texto del staff>   |
|  [ Volver a subir comprobante]|
| (si failed):                 |
|  El pago no se completó.     |
|  [ Reintentar pago ]         |
| Orden vigente hasta <expiresAt> (si aplica)|
+------------------------------+
```

- **Jerarquía:** `h1` = confirmación; **dos StatusTag separados** (orden y pago) que reflejan máquinas distintas (AC5.3.3). El estado se muestra por texto + color.
- **Interacción (Q5, clave):**
  - `rejected` (rechazo manual) → muestra **motivo del staff** + CTA "Volver a subir comprobante" (AC6.2.3/AC6.1/FR6.3).
  - `failed` (fallo de pasarela) → "El pago no se completó" + CTA "Reintentar pago" (AC6.3.3).
  - En ambos, la **orden NO aparece como cancelada**; permanece `pending_payment` mientras la reserva siga vigente, y se muestra `expiresAt` (AC5.3.5).
  - Reentrada del invitado por "Ver estado de mi orden" (OQ-US-1, representación genérica — Q6).
- **Estados:** `paid` (éxito), `pending_review` (pendiente), `rejected` (motivo + reintento comprobante), `failed` (reintento pago), `expired` (reserva venció), error de carga.
- **Accesibilidad:** estado como texto; CTA de recuperación accesible; sin depender de color.

## P7 🟢 — Registro / alta de comercio (crea tienda)
Historias: US1.1 · Requisitos: FR1.1, FR1.2, FR1.3, NFR3.1

```
+------------------------------+
| h1 Crea tu tienda en minutos |
+------------------------------+
| Nombre del comercio [______] |
| Email               [______] |
| Contraseña          [______] |
| Subdominio [mitienda].plat.. |
|   ✓ disponible / ✗ en uso    |
| [ ] Acepto tratamiento de    |
|     datos y términos (enlace) |
| [     Crear mi tienda      ] |
+------------------------------+
```

- **Jerarquía:** `h1` = título; validación de subdominio en línea; Button primario único.
- **Interacción:** crear → se crea tienda aislada + **identidad Owner/Admin autenticable**; la **cuenta del tenant queda NO activada** (pendiente de super-admin), distinta de la identidad (AC1.1.1/AC1.1.4). Subdominio único (AC1.1.2/AC1.1.3).
- **Estados:** con datos; validación (subdominio ocupado, email inválido); cargando (creando); error. Tras crear: mensaje "Tu tienda está creada; un administrador la activará para publicar" (diferencia identidad vs activación del tenant).
- **Accesibilidad:** disponibilidad de subdominio anunciada por texto; checkbox de términos con label; aviso Ley 1581.

## P8 — Panel del comercio: productos e inventario (escritorio y móvil)
Historias: US3.1, US4.3, US7.1 · Requisitos: FR3.1, FR4.1, FR8.1

```
+----------------------------------------------+
| AdminNav: Productos|Inventario|Órdenes|Config |
+----------------------------------------------+
| h1 Productos               [+ Nuevo producto] |
| Nombre   SKU   Stock  Precio   Acciones       |
| Prod A   A-01   12    $..      [Editar]        |
| Prod B   B-02    0    $..      [Editar]        |
+----------------------------------------------+
| Movimientos de inventario (ent/sal/ajuste)    |
+----------------------------------------------+
```

- **Jerarquía:** `h1` = "Productos"; DataTable con encabezados asociados; acción primaria "Nuevo producto" → P13.
- **Interacción/roles (AC7.1.1):** visibilidad por rol — **staff de catálogo/inventario** opera P8; **staff de órdenes** NO ve P8; Owner/Admin ve todo. Config sensible (pagos/impuestos) NO está aquí (va en P14, restringida a Owner/Admin — AC7.1.3).
- **Estados:** **vacío** = EmptyState "Aún no tienes productos" + CTA crear; con datos; cargando; error; **borde** = stock 0 resaltado con texto "Agotado" (no solo color).
- **Responsive:** tabla → tarjetas apiladas en móvil (cada fila como Card con acciones).
- **Accesibilidad:** tabla con `th` asociados; acciones con nombre accesible; stock por texto.

## P9 🟢 — Panel del comercio: órdenes y aprobación de pago manual (foco móvil)
Historias: US6.2, US5.3, US8.1 · Requisitos: FR6.2, FR2.3, FR6.5

```
+----------------------------------------------+
| h1 Órdenes      [filtro: Pendientes de pago ▾]|
+----------------------------------------------+
| #1234 cliente $.. PENDIENTE   [Revisar]       |
| #1233 cliente $.. PAGADA      [Ver]           |
+----------------------------------------------+
| Al "Revisar":                                 |
|  [ comprobante adjunto (preview) ]            |
|  ⚠ Verifica el dinero en tu app bancaria      |
|  ( ) Aprobar    ( ) Rechazar (motivo [____]) |
|  [ Confirmar ] → ConfirmDialog                |
|  Historial: <acción> por <quién> <cuándo>     |
+----------------------------------------------+
```

- **Jerarquía:** `h1` = "Órdenes"; lista filtrable; panel/página de revisión con comprobante; acción con ConfirmDialog (Q3).
- **Interacción/roles (AC6.2.1/.2/.5):** **Owner/Admin o staff de órdenes** aprueban/rechazan; el staff de catálogo/inventario NO ve P9. Aprobar → pago `approved`, orden `paid`. Rechazar con motivo → pago `rejected` (la orden NO se cancela; el comprador puede recargar — P6). Recordatorio de verificar el ingreso en el banco (la plataforma no valida el movimiento). Registro visible de quién/cuándo.
- **Estados:** vacío; con datos; `pending_review`; `rejected`; error; **borde** = orden/reserva vencida → aprobar se rechaza por consistencia (AC6.2.4).
- **Accesibilidad:** estado por texto + icono; comprobante con alt; aprobar/rechazar por teclado; ConfirmDialog accesible (foco atrapado, Esc).

## P10 — Panel super-admin (tiendas, plan/estado, métricas, soporte)
Historias: US1.3, US1.4, US2.4, US7.2 · Requisitos: FR1.4, FR1.6, FR2.6, FR8.2, NFR1.2

```
+----------------------------------------------+
| h1 Super-admin  [Tiendas] [Métricas]          |
+----------------------------------------------+
| Métricas: Tiendas activas: N · Órdenes: N     |
+----------------------------------------------+
| Tienda  Plan  Estado pago  Activación  Acción |
| TiendaX Básico Al día      Activada  [Suspender]|
| TiendaY Básico Vencido     No activa [Activar][Reactivar]|
| [ Soporte: ver tienda (solo lectura, auditado)]|
+----------------------------------------------+
```

- **Jerarquía:** `h1` = "Super-admin"; DataTable de tiendas; acciones con ConfirmDialog.
- **Interacción:** crear/suspender/reactivar tiendas; **activar la cuenta del tenant** (distinta de la identidad del Owner — AC1.4.2); registrar plan/estado de pago (visible solo al super-admin); **soporte solo lectura auditado** (AC2.4.1 / regla de equipo M2): al entrar en soporte, banner persistente "Solo lectura — acceso auditado".
- **Estados:** con datos; vacío (sin tiendas); error. Operaciones globales autorizadas y auditadas (AC2.1.5).
- **Accesibilidad:** tablas con encabezados; confirmaciones; el modo soporte declara "solo lectura" por texto.

## P11 — Personalización de plantilla de la tienda (comerciante)
Historias: US3.4 (Must), US3.5 (Could) · Requisitos: FR8.3, FR1.2

```
+----------------------------------------------+
| h1 Personaliza tu tienda                      |
+----------------------------------------------+
| Plantilla:  (• Base)  ( Plantilla 2 — próx.) |
| Color principal [■ #___]  (contraste ✓/✗)    |
| Color de acento [■ #___]                      |
| Logo [subir]   Tipografía [▾]                 |
| Vista previa: [ mini-render de la tienda ]    |
| [ Guardar ]  [ Guardar y publicar ]           |
+----------------------------------------------+
```

- **Jerarquía:** `h1` = título; controles de tokens de marca (Q2: color primario+acento, logo, tipografía); vista previa.
- **Interacción:** ajustar tokens → vista previa; guardar. **US3.4 (Must):** una plantilla base funcional + personalización básica. **US3.5 (Could):** selección de plantillas adicionales → aquí "Plantilla 2 — próximamente" (deshabilitada), marcando que es diferible y fuera del skeleton.
- **Validación de contraste (design-system §7):** si el color elegido no cumple AA, se muestra aviso "✗ contraste insuficiente" y se sugiere/deriva un tono accesible; no se guarda un combo que rompa AA.
- **Estados:** con datos; cargando (guardando); error; **borde** = una sola plantilla disponible en el MVP (adicionales marcadas "próximamente").
- **Accesibilidad:** selector de color con **valor textual** (hex) además de la muestra; vista previa con descripción; selección de plantilla por teclado.

## P12 — Inicio de sesión / registro del comprador
Historias: US2.2 · Requisitos: FR2.4, FR5.1, NFR3.1

```
+------------------------------+
| h1 Entrar o crear cuenta     |
+------------------------------+
| Email      [__________]      |
| Contraseña [__________]      |
| [ Entrar ]   [ Crear cuenta ]|
| — o continuar como invitado —|
| [ ] Al crear cuenta, acepto  |
|     tratamiento de datos y T. |
+------------------------------+
```

- **Jerarquía:** `h1` = título; dos acciones (entrar / crear) + opción invitado coherente con carrito anónimo.
- **Interacción:** autenticado → ve solo sus propias órdenes (AC2.2.1/AC2.2.3); invitado → compra sin cuenta (AC2.2.2) y consulta su orden por el mecanismo abierto OQ-US-1 (representación genérica, Q6).
- **Estados:** con datos; validación; cargando; error.
- **Accesibilidad:** formulario etiquetado; errores en texto; checkbox de términos al crear cuenta; aviso Ley 1581.

## P13 🟢 — Formulario de producto (alta/edición)
Historias: US3.1 · Requisitos: FR3.1, FR7.1

```
+----------------------------------------------+
| h1 Nuevo / Editar producto                    |
+----------------------------------------------+
| Nombre      [__________________]              |
| Descripción [__________________]              |
| Categoría   [▾]                               |
| Imágenes    [📎 subir]  [prev][prev]          |
| Variantes   [+ añadir]  (SKU, atributo, stock)|
| Precio      [____]  Impuesto [▾ hereda/propio]|
| Stock inicial [____]                          |
| [ Guardar ]  [ Guardar y publicar ]           |
+----------------------------------------------+
```

- **Jerarquía:** `h1` = "Nuevo / Editar producto"; formulario con validación; un Button primario (Guardar) + secundario (Guardar y publicar).
- **Interacción:** crear producto con nombre, precio, categoría, imagen, stock (AC3.1.1); variantes con SKU y su propio stock (AC3.1.2). **Impuesto:** selector "hereda de categoría" vs "propio del producto" reflejando la precedencia producto-sobre-categoría y la exención 0% (AC5.2.3/.4 / FR7.1). La opera staff de catálogo/inventario y Owner/Admin.
- **Estados:** con datos; validación (requeridos); cargando (guardando/subiendo imágenes); error.
- **Accesibilidad:** labels; carga de imágenes con alt; variantes como grupo etiquetado.

## P14 — Configuración de la tienda (comerciante) — restringida a Owner/Admin
Historias: US7.1 · Requisitos: FR8.1, FR6.2, FR7.1, FR7.2, NFR1.2, NFR4.2

```
+----------------------------------------------+
| h1 Configuración de la tienda                 |
+----------------------------------------------+
| § Datos de transferencia                      |
|   Banco / cuenta / titular / instrucciones    |
| § Conexión de pasarela                        |
|   [ Conectar ]  estado: no conectada/conectada|
|   (credenciales propias del comercio)         |
| § Impuestos                                   |
|   Tasa por producto/categoría; precio con     |
|   impuesto incluido ( ) o agregado ( )        |
| [ Guardar ]                                   |
+----------------------------------------------+
```

- **Jerarquía:** `h1` = "Configuración de la tienda"; secciones con un Button primario por sección.
- **Interacción/restricción (AC7.1.3, clave):** **solo Owner/Admin** ve y edita pasarela, credenciales, datos bancarios e impuestos; el staff de órdenes y de catálogo NO acceden a esta pantalla. Lo configurado aquí determina los métodos de pago visibles en P4. Credenciales del comercio; la plataforma no retiene fondos; tarifas de pasarela se verifican con fuentes oficiales en diseño (regla de equipo M1).
- **Estados:** con datos; validación; cargando; error; **borde** = pasarela no conectada → en checkout (P4) solo aparece transferencia.
- **Accesibilidad:** formularios etiquetados; estado de conexión por texto (no solo color); secreto/credenciales nunca mostrados en claro tras guardar.

---

## Trazabilidad pantalla → historia → requisito (resumen)

| Pantalla | Historias | Requisitos clave |
|---|---|---|
| P1 | US3.2, US3.3, US3.4 | FR3.2, FR3.3, FR1.5 |
| P2 | US3.3 | FR3.2, FR7.1/7.2 |
| P3 | US5.1, US5.2 | FR5.1, FR5.2, FR7.4 |
| P4 | US2.2, US5.2 | FR5.2, FR2.4, NFR3.1 |
| P5 | US6.1 | FR6.2, NFR3.2, FR6.3 |
| P6 | US5.3, US6.1/6.2/6.3, US2.2 | FR5.3, FR5.4, FR6.3 |
| P7 | US1.1 | FR1.1/1.2/1.3, NFR3.1 |
| P8 | US3.1, US4.3, US7.1 | FR3.1, FR4.1, FR8.1 |
| P9 | US6.2, US5.3, US8.1 | FR6.2, FR2.3, FR6.5 |
| P10 | US1.3, US1.4, US2.4, US7.2 | FR1.4, FR1.6, FR2.6, FR8.2 |
| P11 | US3.4, US3.5 | FR8.3, FR1.2 |
| P12 | US2.2 | FR2.4, FR5.1 |
| P13 | US3.1 | FR3.1, FR7.1 |
| P14 | US7.1 | FR8.1, FR6.2, FR7.1/7.2, NFR1.2 |

## Sources

- [wireframes] P1–P14 y arquitectura de información de `wireframes.md`.
- [stories] US1.1–US8.1 con la ronda de refinamiento (estados de pago separados, identidad vs activación, config restringida, invitado OQ-US-1, etc.).
- [requirements] FR1–FR9, NFR1–NFR9.
- Q1–Q9 de `refined-mockups-questions.md`; tokens/componentes de `design-system-mapping.md`.

## Assumptions & Open Questions

- [assumption] Layouts ASCII = estructura de media fidelidad (Q9); la alta fidelidad visual se produce luego en herramienta de diseño/frontend.
- [assumption] La librería de componentes y el framework no se deciden aquí (Q1); los mockups son agnósticos.
- OQ-US-1 — Mecanismo de consulta segura de la orden del comprador invitado: representado genéricamente en P6/P12, sin cerrar la decisión (Q6).
