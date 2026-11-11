# Personas — Plataforma de tienda virtual multi-negocio

> Fase: Inception. Fecha: 2026-11-11. Derivadas de los roles de FR2 y de los wireframes de Ideación.
> Entradas upstream: `requirements.md` (FR2 roles) y `team-practices.md`.

## P-OWNER — Owner/Admin de tienda
- **Rol:** dueño o administrador del comercio; usuario principal del MVP.
- **Objetivos:** crear y publicar su tienda rápido, cargar productos con stock, cobrar (manual y/o pasarela), operar órdenes y gestionar a su staff.
- **Dolores:** no tiene equipo técnico; presupuesto ajustado; quiere vender en línea sin complicaciones.
- **Contexto:** trabaja sobre todo desde móvil y escritorio; sin capacitación previa.
- **Prioridad:** máxima (es quien valida el MVP piloto).

## P-STAFF-CAT — Staff de catálogo/inventario
- **Rol:** colaborador del comercio con permisos acotados a productos e inventario.
- **Objetivos:** mantener el catálogo y el stock al día.
- **Dolores:** no debe ver ni tocar órdenes ni configuración de pagos.
- **Prioridad:** media (puede ser el mismo Owner en pilotos pequeños).

## P-STAFF-ORD — Staff de órdenes
- **Rol:** colaborador del comercio enfocado en órdenes y verificación de pagos manuales.
- **Objetivos:** revisar comprobantes, aprobar/rechazar pagos tras confirmar el ingreso en el banco, dar seguimiento al estado de las órdenes.
- **Dolores:** necesita claridad sobre qué pago verificar y evidencia de quién aprobó/rechazó y cuándo.
- **Prioridad:** media-alta (clave en el flujo de pago manual).

## P-CLIENTE — Cliente comprador
- **Rol:** comprador final de una tienda.
- **Objetivos:** encontrar productos, comprar con carrito y checkout, pagar (pasarela o transferencia con comprobante), ver el estado de sus órdenes.
- **Dolores:** proceso de compra confuso; no saber si su pago fue recibido.
- **Contexto:** mayoritariamente móvil.
- **Prioridad:** alta.
- **Variante P-CLIENTE-INV — Comprador invitado:** mismo recorrido de compra sin crear cuenta (carrito anónimo FR5.1); puede volver a cargar comprobante tras un rechazo (FR6.3). No es un rol de autorización independiente, sino una modalidad del checkout.

## P-SUPERADMIN — Super-admin de plataforma
- **Rol:** operador de la plataforma (hoy el propio owner del producto).
- **Objetivos:** crear/suspender/reactivar tiendas, activar cuentas, registrar plan y estado de pago manual, consultar métricas básicas y dar soporte.
- **Dolores:** necesita operar sin violar el aislamiento entre tiendas; el soporte debe ser de solo lectura y auditado.
- **Prioridad:** media (administración de la plataforma y de los pilotos).

## Relaciones y ranking de prioridad

1. P-OWNER (publica y opera su tienda) y P-CLIENTE (compra) — el eje del valor del MVP.
2. P-STAFF-ORD (verificación de pago manual) — habilita la venta con transferencia.
3. P-SUPERADMIN (opera la plataforma) y P-STAFF-CAT (mantiene el catálogo).

## Assumptions & Open Questions

- [assumption] En pilotos pequeños, una misma persona puede ejercer Owner/Admin y ambos roles de staff; el modelo de permisos lo soporta sin requerir usuarios separados. [Q2]
