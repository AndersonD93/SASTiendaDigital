# Wireframes de baja fidelidad (concepto) — Plataforma de tienda virtual multi-negocio

> Bocetos conceptuales, no diseño final. Invitan a feedback. Fecha: 2026-11-09.
> Entradas upstream: `intent-statement` (Intent Capture), `scope-document` e `intent-backlog` (Scope Definition), de los que toman las pantallas, el alcance del MVP y las unidades.
>
> Enfoque: mobile-first para tienda pública y checkout (incluida carga de comprobante desde móvil); paneles usables en escritorio y móvil [Q3]. Diseño apoyado en un sistema de diseño open source (sujeto a investigación de licencias) con personalización por tienda (colores, logo, tipografía) [Q4]. Objetivo de accesibilidad WCAG 2.1 AA (o 2.2 AA) en tienda pública, carrito y checkout; AA con mejora continua en paneles [Q5].
>
> Nota de accesibilidad: el objetivo WCAG es una meta de diseño; la conformidad real exige pruebas con tecnologías de asistencia y revisión experta. [Q5]
>
> Detalle: mayor detalle en las pantallas del walking skeleton (registro de comercio, carga de producto, tienda pública, detalle, carrito, checkout con transferencia, carga de comprobante, aprobación de pago); el resto a nivel conceptual. [Q1]

Las pantallas están numeradas. Cada una incluye una nota de accesibilidad (encabezado principal, regiones landmark, punto de entrada por teclado) y los estados a considerar.

---

## P1. Landing / tienda pública (storefront con plantilla) — mobile-first [Q1]

```
+------------------------------------------+
| [=]  LogoTienda           [buscar] [🛒2] |  <- header (nav, búsqueda, carrito)
+------------------------------------------+
| Categorías: Todas | Cat A | Cat B | ...  |  <- filtro por categoría
+------------------------------------------+
|  [img]      [img]                        |
|  Producto1  Producto2                    |  <- grid de tarjetas (card layout)
|  $precio    $precio                      |
|  [Ver]      [Ver]                        |
|  [img]      [img]                        |
|  Producto3  Producto4                    |
|  $precio    $precio                      |
+------------------------------------------+
|  Footer: contacto · info de la tienda    |
+------------------------------------------+
```
- Accesibilidad: h1 = nombre de la tienda; landmarks header/nav/main/footer; entrada de teclado al buscador; tarjetas navegables por teclado con nombre y precio accesibles.
- Estados: vacío (tienda sin productos → mensaje + guía), cargando (skeleton de tarjetas), con datos, error (no carga el catálogo), borde (tienda suspendida/plan vencido → aviso al comprador). [Q6]

## P2. Detalle de producto — mobile-first [Q1]

```
+------------------------------------------+
| < Volver        LogoTienda        [🛒]   |
+------------------------------------------+
|           [  imagen producto  ]          |
|           [ o o o ] (miniaturas)         |
+------------------------------------------+
| Nombre del producto                      |
| $precio (impuesto incluido/agregado)     |
| Variante: [ Talla v ] [ Color v ]        |
| Stock: disponible / agotado              |
| [   Añadir al carrito   ]                |
| Descripción...                           |
+------------------------------------------+
```
- Accesibilidad: h1 = nombre del producto; imagen con texto alternativo; selección de variante por teclado; botón primario único "Añadir al carrito".
- Estados: con datos, agotado (botón deshabilitado + aviso), cargando, error, borde (variante sin stock). [Q6]

## P3. Carrito [Q1]

```
+------------------------------------------+
| Carrito (2)                     [cerrar] |
+------------------------------------------+
| Producto1  x1   $precio          [x]     |
| Producto2  x2   $precio          [x]     |
+------------------------------------------+
| Subtotal            $...                 |
| Impuestos           $...                 |
| Descuento           -$...                |
| Total               $...                 |
| [      Ir al checkout      ]             |
+------------------------------------------+
```
- Accesibilidad: h1 = "Carrito"; cantidades editables con etiqueta; totales legibles por lector de pantalla; botón primario único.
- Estados: vacío (carrito sin ítems → CTA a seguir comprando), con datos, borde (ítem quedó sin stock → aviso), error. [Q6]

## P4. Checkout (elección de método de pago) — mobile-first [Q1] [Q2]

```
+------------------------------------------+
| Checkout                                 |
+------------------------------------------+
| Datos del comprador                      |
|  Nombre [__________]                     |
|  Email  [__________]                     |
|  Tel    [__________]                     |
+------------------------------------------+
| Método de pago (según configuración):    |
|  ( ) Pasarela (tarjeta/PSE)              |
|  (•) Transferencia + comprobante         |
+------------------------------------------+
| [ ] Acepto el tratamiento de datos y     |
|     los términos (enlace)                |
+------------------------------------------+
| Resumen: Total $...                      |
| [        Confirmar pedido        ]       |
+------------------------------------------+
```
- Accesibilidad: h1 = "Checkout"; formulario con etiquetas asociadas y validación en línea; grupo de radios de método de pago con nombre accesible; casilla de aceptación de términos con etiqueta; un botón primario.
- Estados: con datos, validación (campos con error explicados en texto), cargando (procesando), error (fallo de red/pasarela), borde (stock perdido durante checkout). [Q6]
- Nota de concepto (R-02): el checkout distingue **comprador invitado** vs. **autenticado** — si está autenticado (ver P12), sus datos vienen precargados; si es invitado, los ingresa. El carrito anónimo persiste. [Q2]
- Nota de concepto (métodos de pago): los métodos mostrados **dependen de lo que la tienda tenga configurado** (solo transferencia, solo pasarela, o ambos). Ver P14 (configuración de la tienda). [Q2]
- Nota de concepto (datos): se muestra el aviso de tratamiento de datos y los términos antes de confirmar (Ley 1581). [Q5]

## P5. Carga de comprobante de transferencia (comprador) — mobile-first [Q1] [Q2]

```
+------------------------------------------+
| Pago por transferencia                   |
+------------------------------------------+
| Instrucciones de pago de la tienda:      |
|  Banco / cuenta / titular / valor        |
+------------------------------------------+
| Sube tu comprobante:                     |
|  [ 📎 Seleccionar archivo / cámara ]     |
|  [ vista previa del archivo ]            |
| [        Enviar comprobante        ]     |
+------------------------------------------+
| Tu orden queda PENDIENTE de verificación |
| (se reserva el stock hasta [fecha])      |
+------------------------------------------+
```
- Accesibilidad: h1 = "Pago por transferencia"; carga de archivo accesible (etiqueta, formatos, tamaño), alternativa cámara en móvil; mensaje de estado anunciado.
- Estados: con datos, cargando (subiendo), error (archivo inválido/grande), borde (orden vencida → stock liberado, mensaje claro). [Q6]
- Nota de concepto (reintento): el comprador, **incluso invitado**, puede volver a subir el comprobante desde un enlace o la página de estado de la orden (sin necesidad de cuenta). [Q2]
- Nota de concepto (datos): al enviar el comprobante se muestra el aviso de tratamiento de datos y los términos. [Q5]

## P6. Confirmación de orden (estado según método) [Q1] [Q2]

```
+------------------------------------------+
|   ✔ ¡Gracias por tu compra!              |
+------------------------------------------+
| Orden #1234                              |
| Estado:                                  |
|  - Pasarela: PAGADA                      |
|  - Transferencia: PENDIENTE de verif.    |
| Detalle de ítems y total                 |
| [ Ver estado de mi orden ]               |
| (si RECHAZADA o PENDIENTE):              |
| [ Reintentar / volver a subir comprobante]|
+------------------------------------------+
```
- Accesibilidad: h1 = confirmación; estado de la orden como texto (no solo color); enlace a seguimiento; acción de recuperación accesible.
- Estados: pagada, pendiente de verificación, rechazada (motivo + qué hacer), error. [Q6]
- Nota de concepto (R-03): en el estado "rechazada" (y en "pendiente") hay un **punto de reentrada visible** — reintentar la compra o volver a subir el comprobante — accesible desde el enlace o la página de estado de la orden, **incluso para comprador invitado**, en línea con la variante de pago rechazado del flujo (user-flow.md). [Q2]

## P7. Registro / alta de comercio (crea tienda automáticamente) [Q1] [Q2]

```
+------------------------------------------+
| Crea tu tienda en minutos                |
+------------------------------------------+
| Nombre del comercio [__________]         |
| Email               [__________]         |
| Contraseña          [__________]         |
| Subdominio: [mitienda] .plataforma.com   |
| [ ] Acepto el tratamiento de datos y     |
|     los términos (enlace)                |
| [        Crear mi tienda        ]        |
+------------------------------------------+
```
- Accesibilidad: h1 = título; formulario con etiquetas y validación; disponibilidad de subdominio anunciada; casilla de aceptación de términos con etiqueta; un botón primario.
- Estados: con datos, validación (subdominio ocupado), cargando (creando tienda), error. [Q6]
- Nota de concepto (datos): se muestra el aviso de tratamiento de datos y los términos en el registro del comercio (Ley 1581). [Q5]

## P8. Panel del comercio — productos e inventario (escritorio y móvil) [Q1]

```
+---------------------------------------------------+
| [nav] Productos | Inventario | Órdenes | Config   |
+---------------------------------------------------+
| Productos                        [+ Nuevo prod.]  |
| ------------------------------------------------- |
| Nombre     SKU     Stock   Precio     Acciones    |
| Prod A     A-01     12     $...       [editar]    |
| Prod B     B-02      0     $...       [editar]    |
+---------------------------------------------------+
| Movimientos de inventario (entradas/salidas/ajuste)|
+---------------------------------------------------+
```
- Accesibilidad: h1 = "Productos"; tabla con encabezados asociados; navegación por teclado; acciones con nombre accesible.
- Estados: vacío (sin productos → CTA crear primero), con datos, cargando, error, borde (stock 0 destacado sin depender solo del color). [Q6]
- Nota de concepto (R-01): la **visibilidad y las acciones varían por rol de staff**. El staff de catálogo/inventario ve y opera P8 (productos e inventario); el staff de órdenes ve y opera P9. El Owner/Admin ve todo. No es un panel de permiso único. [intent-statement]
- El formulario de alta/edición de producto es la pantalla P13. [Q1-corr]

## P9. Panel del comercio — órdenes y aprobación de pago manual (foco móvil) [Q1] [Q2]

```
+---------------------------------------------------+
| Órdenes            [filtro: Pendientes de pago v] |
+---------------------------------------------------+
| #1234  cliente   $...   PENDIENTE   [revisar]     |
| #1233  cliente   $...   PAGADA      [ver]         |
+---------------------------------------------------+
| Al abrir "revisar":                               |
|  [ comprobante adjunto ]                          |
|  ⚠ Verifica el dinero en tu app bancaria antes    |
|  ( ) Aprobar pago   ( ) Rechazar (motivo)         |
|  [ Confirmar ] → paso de confirmación             |
|  Historial: aprobado/rechazado por <quién> <cuándo>|
+---------------------------------------------------+
```
- Accesibilidad: h1 = "Órdenes"; estado por texto + icono (no solo color); comprobante con texto alternativo; acción de aprobar/rechazar accesible por teclado; diálogo de confirmación accesible.
- Estados: vacío (sin órdenes), con datos, pendiente de verificación, rechazada, error, borde (orden vencida/stock liberado). [Q6]
- Nota de concepto (R-01): esta pantalla la opera el staff de órdenes (y el Owner/Admin); el staff de catálogo/inventario no la ve. [intent-statement]
- Nota de concepto (nota 5): aprobar/rechazar tiene un **paso de confirmación**; queda un **registro visible de quién y cuándo** lo hizo; y un **recordatorio de verificar el dinero en la app bancaria** antes de aprobar (la plataforma no valida el movimiento). [Q2]

## P10. Panel super-admin (tiendas, plan/estado de pago, métricas) [Q1]

```
+---------------------------------------------------+
| Super-admin: Tiendas | Métricas                   |
+---------------------------------------------------+
| Métricas: Tiendas activas: N | Órdenes totales: N |
+---------------------------------------------------+
| Tienda      Plan     Estado pago   Acciones       |
| TiendaX     Básico   Al día        [suspender]    |
| TiendaY     Básico   Vencido       [reactivar]    |
| [soporte: ver tienda (solo lectura, auditado)]    |
+---------------------------------------------------+
```
- Accesibilidad: h1 = "Super-admin"; tablas con encabezados; acciones con confirmación; el acceso de soporte indica "solo lectura".
- Estados: con datos, vacío (sin tiendas), error. Acceso de soporte de solo lectura y auditado (regla de equipo). [memory:M2-team]

## P11. Selección y personalización de plantilla de la tienda (comerciante) [Q1] [Q4]

```
+---------------------------------------------------+
| Personaliza tu tienda                             |
+---------------------------------------------------+
| Plantilla: ( • Base )  ( Plantilla 2 - próx. )    |
| Color principal [■]  Logo [subir]  Tipografía [v] |
| Vista previa: [  mini-render de la tienda  ]      |
| [ Guardar y publicar ]                            |
+---------------------------------------------------+
```
- Accesibilidad: h1 = título; selector de plantilla por teclado; selector de color con valor textual (no solo muestra); vista previa con descripción.
- Estados: con datos, cargando (guardando), error, borde (una sola plantilla en el MVP; otras marcadas como próximas). [Q1] [Q6]

## P12. Inicio de sesión / registro del comprador [Q1]

```
+------------------------------------------+
| Entrar o crear cuenta                    |
+------------------------------------------+
|  Email    [__________]                   |
|  Contraseña [________]                    |
|  [ Entrar ]   [ Crear cuenta ]           |
|  (o continuar como invitado)             |
|  [ ] Al crear cuenta, acepto el          |
|      tratamiento de datos y términos     |
+------------------------------------------+
```
- Accesibilidad: h1 = título; formulario etiquetado; opción de compra como invitado (coherente con carrito anónimo); casilla de aceptación de términos al crear cuenta; errores en texto.
- Estados: con datos, validación, cargando, error. [Q6]
- Nota de concepto (datos): se muestra el aviso de tratamiento de datos y los términos al registrar al comprador (Ley 1581). [Q5]

## P13. Formulario de producto (alta/edición — parte del walking skeleton) [Q1-corr]

```
+---------------------------------------------------+
| Nuevo / Editar producto                           |
+---------------------------------------------------+
| Nombre      [________________]                    |
| Descripción [________________]                    |
| Categoría   [ v ]                                 |
| Imágenes    [ 📎 subir ]  [prev][prev]            |
| Variantes   [+ añadir]  (SKU, atributo)           |
| Precio      [____]  Impuesto [ v ]                |
| Stock inicial [____]                              |
| [ Guardar ]   [ Guardar y publicar ]              |
+---------------------------------------------------+
```
- Accesibilidad: h1 = "Nuevo / Editar producto"; formulario con etiquetas y validación; carga de imágenes accesible (texto alternativo); un botón primario.
- Estados: con datos, validación (campos requeridos), cargando (guardando/subiendo imágenes), error. [Q6]
- Nota de concepto: pantalla del walking skeleton (cargar un producto con stock). La opera el staff de catálogo/inventario y el Owner/Admin. [Q2] [intent-statement]

## P14. Configuración de la tienda (comerciante) [Q1-corr]

```
+---------------------------------------------------+
| Configuración de la tienda                        |
+---------------------------------------------------+
| Datos de transferencia:                           |
|   Banco / cuenta / titular / instrucciones        |
| Conexión de pasarela:                             |
|   [ conectar ]  estado: no conectada / conectada  |
|   (credenciales propias del comercio)             |
| Impuestos:                                        |
|   Tasa por producto/categoría; precio con         |
|   impuesto incluido ( ) o agregado ( )            |
| [ Guardar ]                                       |
+---------------------------------------------------+
```
- Accesibilidad: h1 = "Configuración de la tienda"; formularios con etiquetas; estado de conexión por texto (no solo color); un botón primario por sección.
- Estados: con datos, validación, cargando, error, borde (pasarela no conectada → solo transferencia disponible en checkout). [Q6]
- Nota de concepto: lo que se configure aquí (métodos de pago disponibles, impuestos) determina lo que ve el comprador en P4. Las credenciales de pasarela son del comercio; la plataforma no retiene fondos. Las tarifas de la pasarela se verifican con fuentes oficiales en diseño. [Q2] [memory:M1-team]

## Arquitectura de información (resumen)

- **Sitio público por tienda (subdominio):** Landing (P1) → Detalle (P2) → Carrito (P3) → Checkout (P4) → [Transferencia: Carga de comprobante (P5)] → Confirmación (P6). Login/registro del comprador (P12) opcional (permite invitado); desde P6 hay reentrada tras rechazo/pendiente.
- **Administración del comercio:** Registro/alta (P7) → Panel: Productos/Inventario (P8), Formulario de producto (P13), Órdenes/aprobación de pago (P9), Personalización de plantilla (P11), Configuración de la tienda (P14). La visibilidad del panel varía por rol de staff.
- **Plataforma:** Super-admin (P10).

## Assumptions & Open Questions

- [assumption] El objetivo WCAG 2.1/2.2 AA es una meta de diseño; la conformidad real requiere pruebas con tecnologías de asistencia y revisión experta, fuera de esta fase de concepto. [Q5]
- [assumption] La elección de sistema de diseño y plantillas open source queda sujeta a la investigación de licencias (uso comercial en SaaS) previa al frontend. [Q4]
- [assumption] El nivel de detalle mayor se concentró en las pantallas del walking skeleton; las demás son conceptuales y se refinan en la etapa de mockups refinados (Inception). [Q1]
- [assumption] Número de plantillas de storefront en el MVP: se parte de **una** plantilla base bien diseñada y personalizable por tienda; una **segunda** plantilla podría incluirse si la investigación de plantillas open source (licencia de uso comercial, mantenimiento, accesibilidad, rendimiento) lo permite sin costo de tiempo relevante. A confirmar en diseño/construcción del storefront. [Q1-scope]
