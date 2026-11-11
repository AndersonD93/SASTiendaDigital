# Historias de Usuario — Plataforma de tienda virtual multi-negocio

> Fase: Inception. Fecha: 2026-11-11. Formato INVEST; criterios de aceptación en Given/When/Then; prioridad MoSCoW; IDs estables `US{grupo}.{seq}` y `AC{g}.{s}.{seq}`. Trazan a los requisitos FR/NFR de `requirements.md`.
> Entradas upstream: `requirements.md` y `team-practices.md`.
>
> 🟢 = historia del **walking skeleton** (corte end-to-end mínimo; sus pruebas R1-R3 son compuerta de salida).

## Grupo US1 — Tenants (gestión de tiendas)

### US1.1 🟢 — Registro y creación automática de tienda
Como **Owner/Admin**, quiero registrarme y que se cree automáticamente mi tienda, para empezar a configurarla sin pasos técnicos. (Must) — traza FR1.1, FR1.2
- AC1.1.1 — Given un visitante sin cuenta, When completa el registro con datos válidos, Then se crea su tienda aislada y su **identidad Owner/Admin** queda creada y autenticable para configurar la tienda.
- AC1.1.2 — Given un registro, When se crea la tienda, Then se le asigna un subdominio de plataforma único. (traza FR1.3)
- AC1.1.3 — Given un subdominio ya en uso, When se intenta registrar con ese subdominio, Then el sistema lo rechaza con un mensaje claro.
- AC1.1.4 — Given un registro recién completado, When se crea la tienda, Then la **cuenta del tenant queda en estado no activado** (pendiente de activación por el super-admin): la identidad Owner/Admin puede autenticarse y configurar, pero la activación del tenant (US1.4) es un estado independiente de la identidad y no se infiere del registro. (traza FR1.6)

### US1.2 — Configuración básica de la tienda
Como **Owner/Admin**, quiero configurar nombre, logo, colores, moneda y contacto, para personalizar mi tienda. (Must) — traza FR1.2
- AC1.2.1 — Given una tienda, When el Owner guarda su configuración básica, Then los datos quedan persistidos y visibles en la tienda pública.
- AC1.2.2 — Given la configuración, When el Owner elige la moneda, Then se usa esa moneda en precios y totales. (traza C5)

### US1.3 — Plan y estado de pago de la suscripción (manual)
Como **Super-admin**, quiero registrar el plan y el estado de pago de cada tienda manualmente, para controlar la suscripción sin cobro automático. (Must) — traza FR1.4
- AC1.3.1 — Given una tienda, When el super-admin fija su plan y estado de pago, Then queda registrado y visible solo para el super-admin.
- AC1.3.2 — Given una tienda con estado de pago vencido, When un comprador visita su subdominio, Then ve un aviso coherente con la suspensión (sin exponer datos internos). (traza FR1.6)

### US1.4 🟢 — Activación de cuenta de la tienda por el super-admin
Como **Super-admin**, quiero activar la cuenta del comercio, para habilitar el inicio del cronómetro de publicación y el uso de la tienda. (Must) — traza FR1.6, NFR9.1
- AC1.4.1 — Given una tienda registrada, When el super-admin la activa, Then el Owner puede operar y el criterio de "publicar en <15 min" cuenta desde ese momento.
- AC1.4.2 — Given el estado de activación del **tenant** (distinto de la existencia de la identidad Owner/Admin de US1.1), When el tenant aún no está activado, Then el Owner/Admin puede autenticarse y preparar la configuración pero no puede publicar; y When el super-admin cambia el estado del tenant a activado, Then se habilita la operación sin crear ni modificar la identidad Owner/Admin. (traza FR1.6)

## Grupo US2 — Auth (autenticación y autorización)

### US2.1 🟢 — Aislamiento estricto entre tiendas
Como **plataforma**, quiero que cada usuario solo acceda a los datos de su propia tienda, para garantizar el aislamiento multi-tenant. (Must) — traza FR2.5, NFR1.1, R3
- AC2.1.1 — Given un usuario autenticado del tenant A, When intenta leer o escribir recursos del tenant B por cualquier ruta, Then la operación es denegada sistemáticamente.
- AC2.1.2 — Given cualquier consulta de datos, When se ejecuta, Then incluye el filtro de `tenant_id` resuelto en el punto único transversal; ninguna consulta lo omite.
- AC2.1.3 — Given una **operación privada de tenant** (datos de administración, inventario, órdenes, configuración), When un usuario autenticado la ejecuta, Then el `tenant_id` se resuelve desde su identidad/sesión y solo opera sobre su propio tenant, nunca desde un valor suministrado por el cliente.
- AC2.1.4 — Given el **catálogo público** de una tienda, When un visitante no autenticado accede por el subdominio, Then el `tenant_id` se resuelve a partir del subdominio y solo expone datos públicos publicados de ese tenant (sin datos privados ni de otros tenants), sin requerir sesión.
- AC2.1.5 — Given una **operación global autorizada del super-admin** (crear/suspender/reactivar tiendas, registrar plan/estado de pago, métricas agregadas), When el super-admin la ejecuta, Then se permite de forma controlada y auditada como excepción explícita al filtro de tenant; el soporte del super-admin dentro de una tienda sigue siendo solo lectura (US2.4) y ningún otro rol obtiene alcance cross-tenant. (traza FR2.6, FR8.2)

### US2.2 — Inicio de sesión y registro del comprador
Como **Cliente comprador**, quiero iniciar sesión o comprar como invitado, para completar mi compra con o sin cuenta. (Must) — traza FR2.4, FR5.1
- AC2.2.1 — Given un comprador con cuenta, When inicia sesión, Then accede solo a sus propias órdenes.
- AC2.2.2 — Given un comprador invitado, When procede al checkout, Then puede comprar sin crear cuenta (carrito anónimo).
- AC2.2.3 — Given un comprador que se registra o inicia sesión con cuenta, When consulta sus órdenes, Then el listado se resuelve por su identidad autenticada dentro del tenant y nunca expone órdenes de otros compradores ni de otros tenants. (traza FR2.4)
- AC2.2.4 — Given un comprador invitado que completó una compra, When regresa a consultar el estado de su orden, Then el acceso a esa orden se realiza por un mecanismo de consulta segura que no depende de una cuenta y no permite enumerar ni acceder a órdenes ajenas; el mecanismo concreto es una decisión funcional pendiente (ver Open Questions OQ-US-1). (traza FR2.4, FR6.3)

### US2.3 — Gestión de staff por el Owner/Admin
Como **Owner/Admin**, quiero crear y asignar roles a mi staff (catálogo/inventario, órdenes), para delegar tareas con permisos acotados. (Must — gestión básica) — traza FR2.1, FR2.2, FR2.3
> Prioridad ajustada a **Must** porque el modelo de cinco roles y la gestión de staff por el Owner/Admin son parte de Auth, capacidad Must del alcance aprobado. El MVP incluye solo la **gestión básica** (alta de usuario staff y asignación de uno de los roles acotados definidos); la administración avanzada de usuarios (roles personalizados, jerarquías, permisos granulares más allá de los roles fijos) queda fuera del MVP. [scope-document]
- AC2.3.1 — Given un Owner/Admin, When crea un usuario staff con rol de catálogo/inventario, Then ese usuario accede a productos e inventario pero no a órdenes ni a configuración de pagos.
- AC2.3.2 — Given un usuario staff de órdenes, When accede, Then puede gestionar órdenes y aprobar/rechazar pagos manuales, pero no editar el catálogo ni la configuración de pagos.
- AC2.3.3 — Given cualquier permiso, When se evalúa, Then se valida en backend bajo mínimo privilegio.
- AC2.3.4 — Given la gestión de staff del MVP, When el Owner/Admin administra usuarios, Then solo dispone de la gestión básica (alta y asignación de uno de los roles fijos); no se ofrecen roles personalizados ni permisos granulares adicionales en el MVP.

### US2.4 — Soporte de solo lectura del super-admin, auditado
Como **Super-admin**, quiero acceder a una tienda en modo solo lectura para dar soporte, para ayudar sin alterar datos y dejando rastro. (Should) — traza FR2.6, NFR1.2
- AC2.4.1 — Given el super-admin en soporte, When accede a una tienda, Then solo puede leer, y el acceso queda registrado en auditoría con quién y cuándo.

## Grupo US3 — Catálogo y storefront

### US3.1 🟢 — Gestión de productos y categorías
Como **Owner/Admin o staff de catálogo**, quiero crear productos con variantes, SKU, categorías e imágenes, para armar mi catálogo. (Must) — traza FR3.1
- AC3.1.1 — Given una tienda, When se crea un producto con nombre, precio, categoría, imagen y stock inicial, Then queda disponible para publicarse.
- AC3.1.2 — Given un producto, When se definen variantes con SKU, Then cada variante tiene su propio stock.

### US3.2 🟢 — Publicar la tienda
Como **Owner/Admin**, quiero publicar mi tienda cuando esté lista, para que los compradores la vean. (Must) — traza FR1.5, NFR9.1
- AC3.2.1 — Given una tienda activa con datos básicos, ≥1 categoría y ≥1 producto con stock, When el Owner ejecuta "publicar", Then la tienda queda pública en su subdominio.
- AC3.2.2 — Given una tienda sin los requisitos mínimos, When el Owner intenta publicar, Then el sistema indica qué falta y no publica.

### US3.3 — Landing pública, navegación y búsqueda
Como **Cliente comprador**, quiero ver el catálogo, navegar categorías, buscar y ver el detalle del producto, para decidir mi compra. (Must) — traza FR3.2, FR3.3
- AC3.3.1 — Given una tienda publicada, When el comprador la visita, Then ve el catálogo con categorías y puede buscar por texto simple.
- AC3.3.2 — Given un producto, When el comprador abre su detalle, Then ve imágenes, precio (con el tratamiento de impuesto configurado) y disponibilidad.

### US3.4 — Plantilla predeterminada funcional y personalización básica (storefront)
Como **Owner/Admin**, quiero que mi tienda tenga una plantilla predeterminada funcional y poder ajustar colores, logo y tipografía, para que mi tienda se vea presentable sin trabajo técnico. (Must — una plantilla) — traza FR8.3
> Alcance aprobado: **una plantilla base bien diseñada y personalizable (colores/logo/tipografía) es Must**; la selección de plantillas adicionales es **Could**. [scope-document]
- AC3.4.1 — Given una plantilla base, When el Owner ajusta colores/logo/tipografía, Then la tienda pública refleja esos ajustes.
- AC3.4.2 — Given una tienda recién creada que no ha personalizado nada, When se publica, Then se renderiza con una **plantilla predeterminada funcional** (catálogo, detalle de producto, carrito y checkout usables) sin requerir configuración de diseño previa.

### US3.5 — Selección de plantillas adicionales (storefront)
Como **Owner/Admin**, quiero poder elegir entre plantillas adicionales además de la predeterminada, para diferenciar más mi tienda. (Could) — traza FR8.3
> Capacidad diferible: las **plantillas adicionales** son Could en el alcance aprobado y no forman parte del mínimo viable ni del walking skeleton. El número de plantillas depende de la investigación de licencias open source (A4, scope-document). [scope-document]
- AC3.5.1 — Given que existe más de una plantilla disponible, When el Owner selecciona una plantilla distinta de la predeterminada, Then la tienda pública adopta esa plantilla conservando sus ajustes de colores/logo/tipografía.

## Grupo US4 — Inventario y reserva de stock

### US4.1 🟢 — Reserva de stock al crear la orden
Como **plataforma**, quiero reservar las unidades al crear una orden, para evitar sobreventa. (Must) — traza FR4.2, FR4.3, NFR2.1, R1
- AC4.1.1 — Given un producto con stock M, When se crean N reservas concurrentes (N>M), Then exactamente M tienen éxito y el resto se rechaza, sin stock negativo.
- AC4.1.2 — Given una orden con pago confirmado dentro del plazo, When se confirma, Then la reserva se convierte en salida definitiva de inventario.
- AC4.1.3 — Given la configuración de la tienda, When se crea una reserva, Then su `expiresAt` se calcula a partir de una **duración configurable por tienda y diferenciada por método de pago**, no de una constante fija del sistema; cada tienda puede fijar su propia duración por método dentro de los límites de la plataforma.
- AC4.1.4 — Given una tienda que no personaliza la duración, When se crea una reserva, Then se aplican los **valores por defecto propuestos** (30 minutos para pasarela y 24 horas para transferencia manual), entendidos como defaults pendientes de aprobación y no como constantes inmutables del producto.

### US4.2 🟢 — Liberación por vencimiento (sin TTL)
Como **plataforma**, quiero liberar explícitamente las reservas vencidas, para devolver el stock sin depender del TTL de DynamoDB. (Must) — traza FR4.4, FR4.5, R1
- AC4.2.1 — Given una reserva con `expiresAt` pasado y sin pago, When se consulta o corre el barrido de reclamo, Then la unidad se trata como liberada y vuelve a estar disponible.
- AC4.2.2 — Given una orden vencida, When se intenta confirmar el pago, Then se rechaza; y una reserva ya liberada no se libera dos veces.
- AC4.2.3 — Given una reserva vencida, When se ejecuta su liberación, Then la liberación es **atómica** (el stock se devuelve en una única operación consistente, sin estados intermedios observables que permitan doble conteo) e **idempotente**: reintentos, barridos concurrentes o una consulta y un barrido que coincidan producen **una sola** devolución de stock para esa reserva, con el mismo efecto neto que una sola ejecución.
- AC4.2.4 — Given la misma reserva vencida procesada por más de un disparador (consulta bajo demanda y barrido programado), When ambos intentan liberarla, Then exactamente una liberación surte efecto y las demás son no-ops verificables; nunca se devuelve stock dos veces ni queda stock negativo. (traza FR4.5, NFR2.1)

### US4.3 — Movimientos de inventario auditables
Como **Owner/Admin o staff de inventario**, quiero ver entradas, salidas y ajustes de stock, para controlar mi inventario. (Should) — traza FR4.1
- AC4.3.1 — Given operaciones de stock, When ocurren, Then quedan registradas como movimientos auditables con tipo, cantidad y fecha.

## Grupo US5 — Carrito, checkout y órdenes

### US5.1 🟢 — Carrito persistente
Como **Cliente comprador (invitado o autenticado)**, quiero un carrito que conserve mis ítems, para completar la compra sin perder la selección. (Must) — traza FR5.1
- AC5.1.1 — Given un comprador que añade ítems, When regresa (misma sesión o autenticado), Then su carrito conserva los ítems.

### US5.2 🟢 — Checkout con cálculo de totales y creación de orden
Como **Cliente comprador**, quiero ver totales con impuestos (y descuento cuando aplique) y crear mi orden, para pagar con claridad. (Must) — traza FR5.2, FR7.1, FR7.2, FR7.3
- AC5.2.1 — Given un carrito, When el comprador va al checkout, Then ve subtotal, impuestos (según configuración de la tienda) y un total correcto; el **descuento es opcional**: solo se muestra y se resta cuando hay un descuento aplicable (la capacidad de descuento es US5.4, Could), y su ausencia no bloquea el checkout.
- AC5.2.2 — Given el checkout confirmado, When se crea la orden, Then queda en estado `pending_payment` con stock reservado y una fotografía (snapshot) de la orden. (traza FR5.3, FR7.3)
- AC5.2.5 — Given la creación de la orden, When se toma su fotografía histórica, Then preserva de forma inmutable: precios unitarios, cantidades por ítem, moneda, el descuento aplicado (si lo hubo), la configuración tributaria vigente y los impuestos calculados, y los totales resultantes; cambios posteriores en catálogo, impuestos o descuentos no alteran la orden ya creada. (traza FR7.3)
- AC5.2.3 — Given un producto con **configuración tributaria propia**, When se calcula el impuesto de su línea, Then prevalece esa configuración específica del producto sobre la heredada de su categoría; las tasas **no se acumulan** (no se suma la del producto con la de la categoría), se aplica una sola. (traza FR7.1, FR7.2)
- AC5.2.4 — Given un producto **sin configuración tributaria propia**, When se calcula el impuesto de su línea, Then hereda la configuración de su categoría; y Given un producto configurado como exento (0%), When se calcula, Then su impuesto es 0 y así queda reflejado en el desglose.

### US5.3 🟢 — Máquina de estados de la orden
Como **plataforma**, quiero estados de orden claros y consistentes, para que ninguna orden quede sin estado final. (Must) — traza FR5.3, FR5.5, FR5.6, NFR2.2, R2
- AC5.3.1 — Given una orden `pending_payment`, When el pago se aprueba, Then pasa a `paid`; y When su reserva vence sin pago aprobado, Then pasa a `expired` y libera el stock. (Las únicas transiciones de la orden dirigidas por pago/tiempo son estas dos; traza FR5.5)
- AC5.3.2 — Given cualquier transición, When ocurre, Then queda trazada y auditada, y un estado terminal no retrocede.
- AC5.3.5 — Given una orden `pending_payment`, When el comprobante manual es `rejected` o un intento de pasarela queda en `failed`, Then la orden **NO** pasa automáticamente a `cancelled`: permanece en `pending_payment` mientras su reserva siga vigente, permitiendo reintento o recarga de comprobante (FR6.3); la orden solo llega a `cancelled` por una cancelación explícita de la regla de negocio, nunca como efecto directo de un pago `rejected`/`failed`. (traza FR5.3, FR5.4, FR6.3)
- AC5.3.3 — Given que los estados de **pago** (`pending_review`, `approved`, `rejected`, `failed`) y los estados de **orden** (`pending_payment`, `paid`, `expired`, `cancelled`) son máquinas separadas, When un pago entra en un estado no exitoso (`rejected` por rechazo manual del comprobante, o `failed` por fallo del intento en la pasarela), Then la orden NO pasa a `paid` y permanece en `pending_payment` mientras su reserva siga vigente, o pasa a `expired`/`cancelled` solo por su propia regla (vencimiento o cancelación), nunca por el resultado del pago por sí mismo.
- AC5.3.4 — Given un evento de pago duplicado, tardío o fuera de orden (p. ej. una aprobación que llega después de que la orden ya está `expired`), When se procesa, Then se aplica idempotentemente: no reactiva un estado terminal de la orden ni produce una transición inconsistente (una orden `expired`/`cancelled` no vuelve a `paid`), y el efecto neto es único.

### US5.4 — Descuento simple a nivel de orden
Como **Owner/Admin**, quiero aplicar un descuento simple (% o monto fijo) a la orden, para hacer promociones básicas. (Could) — traza FR7.4
> Resolución de la contradicción con US5.2: el **checkout es obligatorio (Must)** y debe calcular correctamente el total; el **descuento es una capacidad opcional (Could)**. El checkout no exige que exista un descuento: cuando no hay descuento configurado, el total es subtotal + impuestos; cuando sí lo hay, US5.2 lo muestra y lo resta. Así la prioridad Could de US5.4 no obliga a nada en el checkout.
- AC5.4.1 — Given un descuento configurado, When se aplica a una orden, Then no supera el subtotal elegible y su valor aplicado queda registrado.
- AC5.4.2 — Given una tienda sin descuento configurado, When el comprador llega al checkout, Then el flujo se completa sin descuento (total = subtotal + impuestos) y no se exige configurar ninguno. (traza FR5.2)

## Grupo US6 — Pagos

### US6.1 🟢 — Pago manual por transferencia con comprobante
Como **Cliente comprador**, quiero pagar por transferencia subiendo el comprobante, para comprar sin pasarela. (Must) — traza FR6.2, NFR3.2
- AC6.1.1 — Given una orden `pending_payment` con método transferencia, When el comprador sube un comprobante válido, Then el pago pasa a `pending_review` y se notifica al staff de la tienda. (traza FR6.5)
- AC6.1.2 — Given un comprobante, When se almacena, Then queda cifrado y accesible solo para el **Owner/Admin y el staff de órdenes** de la tienda dueña, y el super-admin en soporte solo-lectura auditado; el comprador solo accede a comprobantes de sus propias órdenes; sin acceso público permanente. (traza NFR3.2)

### US6.2 🟢 — Aprobación/rechazo del pago manual por el Owner/Admin o staff de órdenes
Como **Owner/Admin o staff de órdenes**, quiero aprobar o rechazar el comprobante tras verificar el ingreso en mi banco, para confirmar o anular la venta. (Must) — traza FR6.2, FR2.3
- AC6.2.1 — Given un pago `pending_review`, When el **Owner/Admin o el staff de órdenes** aprueba, Then el pago pasa a `approved` y la orden a `paid`; y queda registro de quién y cuándo.
- AC6.2.2 — Given un pago `pending_review`, When el **Owner/Admin o el staff de órdenes** rechaza con motivo, Then el pago pasa a `rejected` y el comprador puede volver a cargar comprobante mientras la orden y su reserva sigan vigentes. (traza FR6.3)
- AC6.2.3 — Given el estado de pago `rejected` (resultado del **rechazo manual del comprobante**), When ocurre, Then es verificablemente distinto de `failed`: la orden NO pasa a `paid`, permanece en `pending_payment` mientras la reserva siga vigente, y el motivo del rechazo queda registrado para el comprador y en auditoría.
- AC6.2.5 — Given la matriz de permisos aprobada, When se evalúa quién puede consultar, aprobar o rechazar comprobantes, Then están autorizados el **Owner/Admin y el staff de órdenes** de la tienda dueña (el super-admin solo en soporte solo-lectura auditado); ningún otro rol ni tenant puede hacerlo. (traza FR6.2, FR2.3, NFR3.2)
- AC6.2.4 — Given una orden `pending_review` cuya reserva ya venció (`expired`), When el staff intenta aprobar el comprobante, Then la aprobación se rechaza por consistencia (la orden no vuelve a `paid`) y el sistema indica que la orden ya no está vigente.

### US6.3 — Pago por pasarela con webhooks firmados e idempotentes
Como **Cliente comprador**, quiero pagar por pasarela de forma segura, para completar la compra con tarjeta/PSE sin exponer mis datos. (Must) — traza FR6.1, FR6.4, NFR2.2, NFR4.1, R2
- AC6.3.1 — Given un pago por pasarela, When se usa, Then se realiza por checkout alojado/tokenización y ningún dato de tarjeta pasa por nuestros sistemas.
- AC6.3.2 — Given un webhook de pasarela, When llega, Then se verifica su autenticidad mediante el **mecanismo oficial de verificación de firma definido por el proveedor de pasarela** (por ejemplo HMAC u otro que el proveedor establezca; el mecanismo concreto se fija en diseño, OQ3) y se procesa idempotentemente; entregas duplicadas, tardías o fuera de orden producen un único efecto y no retroceden estados terminales.
- AC6.3.3 — Given el estado de pago `failed` (resultado de un **intento de pago fallido en la pasarela**: declinación, timeout o error del proveedor), When ocurre, Then es verificablemente distinto de `rejected`: la orden NO pasa a `paid`, permanece en `pending_payment` mientras la reserva siga vigente para permitir reintento, y el fallo queda registrado con su causa técnica en auditoría sin exponer datos sensibles.
- AC6.3.4 — Given un webhook de resultado de pasarela (éxito o `failed`), When la orden asociada ya está en un estado terminal (`expired`/`cancelled`), Then el webhook se procesa idempotentemente sin forzar `paid` ni reabrir la orden, y el efecto neto es único aunque el webhook se reenvíe.
- AC6.3.5 — Given que una orden puede originar **varios intentos de pago** (p. ej. un intento `failed` seguido de un reintento), When se registran, Then cada **intento** es una entidad propia con su resultado (`failed`/`approved`) y su timestamp, y se **preserva el historial completo de intentos** sin sobrescribirlos; el **estado consolidado del pago de la orden** se deriva de esos intentos (por ejemplo, queda `approved` solo si existe un intento aprobado). (traza FR5.4, NFR2.2)
- AC6.3.6 — Given el estado consolidado del pago y el historial de intentos, When llega un resultado de un intento (incluso duplicado, tardío o fuera de orden), Then la actualización es idempotente y consistente: un intento `failed` posterior no revierte un pago ya `approved`, un intento aprobado no se aplica dos veces, y el estado consolidado nunca queda en una transición incoherente con el historial. (traza NFR2.2, R2)

## Grupo US7 — Administración

### US7.1 — Panel del comercio (operación)
Como **Owner/Admin o staff**, quiero un panel para gestionar productos, inventario, órdenes y configuración de pagos/impuestos, para operar mi tienda. (Must) — traza FR8.1
- AC7.1.1 — Given un panel, When el usuario accede según su rol, Then ve y opera solo lo que su rol permite (catálogo/inventario, órdenes, configuración).
- AC7.1.2 — Given la configuración de la tienda, When el Owner define datos de transferencia, conexión de pasarela e impuestos, Then el checkout muestra solo los métodos de pago configurados.
- AC7.1.3 — Given la **configuración sensible de pagos y tributaria** (conexión/credenciales de pasarela, datos bancarios de transferencia, parámetros de impuestos), When un usuario intenta verla o editarla, Then solo el **Owner/Admin** está autorizado; el staff de catálogo/inventario y el staff de órdenes no pueden acceder a esta configuración (el staff de órdenes aprueba/rechaza comprobantes pero no configura credenciales ni datos bancarios). (traza FR8.1, FR2.3, NFR1.2)

### US7.2 — Panel de super-admin (plataforma)
Como **Super-admin**, quiero administrar tiendas y ver métricas básicas, para operar la plataforma. (Must) — traza FR8.2
- AC7.2.1 — Given el panel de super-admin, When accede, Then puede crear/suspender/reactivar tiendas, registrar plan/estado de pago y ver tiendas activas, órdenes totales y por tienda.

## Grupo US8 — Notificaciones

### US8.1 — Notificaciones transaccionales básicas
Como **Cliente comprador y staff**, quiero recibir confirmación de orden y el resultado de la verificación de pago, para saber en qué va mi compra. (Should) — traza FR9.1, FR6.5
> Prioridad: el conjunto general de notificaciones transaccionales es **Should** (alcance aprobado). La **notificación al staff cuando un comprobante queda `pending_review`** (AC6.1.1 / FR6.5) es parte intrínseca del flujo de pago manual (Must) y se cubre allí; US8.1 la complementa con la confirmación de orden y el resultado al comprador. No se eleva la prioridad global de US8.1 para no ampliar el MVP. [scope-document]
- AC8.1.1 — Given una orden creada o un pago aprobado/rechazado, When ocurre, Then se envía una notificación transaccional por el canal de menor costo, sin incluir datos sensibles.
- AC8.1.2 — Given un comprobante manual que pasa a `pending_review` (AC6.1.1), When ocurre, Then la notificación al Owner/Admin o staff de órdenes de la tienda es coherente con el flujo de pago manual (mismo evento que dispara FR6.5), sin duplicar ni contradecir lo definido en US6.1. (traza FR6.5)

## Resumen del walking skeleton (🟢)

El corte end-to-end mínimo lo forman: US1.1, US1.4, US2.1, US3.1, US3.2, US4.1, US4.2, US5.1, US5.2, US5.3, US6.1, US6.2. Debe validar R1 (US4.1, US4.2), R2 y R3 (US2.1) con escenarios exitosos, negativos, concurrentes y de reintentos; sus pruebas en verde son la compuerta de salida del skeleton. [team-practices]

Para R2, el walking skeleton valida la **consistencia orden↔pago y la idempotencia del flujo de pago MANUAL** (US5.3 + US6.1/US6.2): que un pago manual `rejected` o un evento duplicado/tardío no deje la orden en un estado inconsistente ni la marque `paid` indebidamente. **US6.3 (pasarela) y la idempotencia de sus webhooks firmados NO forman parte del walking skeleton**: la pasarela se entrega después del corte mínimo (coherente con el MoSCoW de Scope Definition) y su validación de idempotencia se aborda fuera de la compuerta del skeleton.

## Dependencias (resumen)

- US1.x + US2.1 son base de todo. US3.1/US3.2/US3.3 dependen de US1/US2; US3.4 (plantilla predeterminada, Must) es base de la tienda pública; US3.5 (plantillas adicionales, Could) depende de US3.4 y queda fuera del walking skeleton. US4.x depende de US3.1. US5.x depende de US3/US4. US6.x depende de US5. US7/US8 dependen de las anteriores.

## Assumptions & Open Questions

- [assumption] El comprador invitado (US2.2/US5.1) es una modalidad de checkout respaldada por FR5.1 y FR6.3, no un rol de autorización. [Q2]
- [assumption] US6.3 (pasarela) puede entregarse después de US6.1/US6.2 (pago manual) sin bloquear el walking skeleton, coherente con el MoSCoW de Scope Definition (la pasarela es lo primero que pasaría a Should si hay recorte). [scope-document]
- [assumption] La plantilla predeterminada funcional (US3.4, Must) satisface el requisito de "≥1 plantilla base" del alcance; las plantillas adicionales (US3.5, Could) dependen de la investigación de licencias open source (A4) y no amplían el MVP. [scope-document]
- OQ-US-1 — Mecanismo de consulta segura de órdenes para el **comprador invitado** (AC2.2.4): está pendiente decidir cómo un invitado consulta el estado de su orden sin cuenta (p. ej. enlace firmado de un solo uso, código de orden + dato de contacto, u otro) garantizando que no se puedan enumerar ni acceder órdenes ajenas. Es una decisión funcional abierta; **no se resuelve aquí** y se arrastra a Functional Design / refinamiento, sin convertirla en decisión aprobada sin autorización. [Q2]
