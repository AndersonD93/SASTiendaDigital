# Análisis Competitivo — Plataforma de tienda virtual multi-negocio

> Alcance: nivel de problema/oportunidad. Las afirmaciones de mercado citan fuente o se marcan como hipótesis. Los **precios cambian con frecuencia y dependen de país/promociones**: se registran con fecha de consulta y se tratan como hipótesis a verificar con fuentes oficiales vigentes en la etapa de diseño (regla de equipo). Fecha de consulta: 2026-11-09. [Q8]
>
> Entrada upstream: este análisis parte del `intent-statement` de la etapa Intent Capture (`aidlc/spaces/default/intents/261106-multi-tenant-store/ideation/intent-capture/intent-statement.md`), del que toma el problema, el cliente objetivo (nicho de bajo presupuesto), la señal de alcance (Colombia como país piloto, no retención de fondos, pago por pasarela y manual, monetización por suscripción) para enmarcar la comparación competitiva.

## Competidores de referencia

El usuario define como competidores directos principales las **plataformas regionales de LATAM** donde un comercio crea su propia tienda en línea. [Q1]

- **Tiendanube / Nuvemshop** — plataforma de tiendas en línea presente en LATAM, con operación y planes en Colombia. [Q1] (Fuente: sitio oficial Tiendanube Colombia, planes y precios — https://www.tiendanube.com/co/planes-y-precios)
- **Jumpseller** — plataforma de tienda en línea con presencia en Colombia (dominio `jumpseller.co`), integra múltiples pasarelas locales. [Q1] (Fuente: sitio oficial Jumpseller Colombia — https://jumpseller.co/ y https://jumpseller.co/payment-gateways/)
- **Empretienda** — mencionada por el usuario como referente regional. [Q1] (Hipótesis: no verificada en esta ronda; validar presencia y planes en Colombia en diseño.)

Competidores indirectos (mismo problema, distinta forma), relevantes por el nicho de bajo presupuesto en Colombia:
- Soluciones de "link de pago" y venta por redes/WhatsApp, y pasarelas locales (Wompi, Bold, PayU, PSE, Mercado Pago, ePayco, Addi) que algunos comercios usan sin una tienda formal. [Q1] (Fuente: integraciones de pago listadas por Jumpseller Colombia — https://jumpseller.co/payment-gateways/)

## Matriz de comparación de capacidades (nivel de problema)

| Capacidad | Nuestra plataforma (MVP) | Tiendanube/Nuvemshop | Jumpseller |
|---|---|---|---|
| Crear tienda propia con subdominio | Sí (subdominio de plataforma automático) [Q2] | Sí (Fuente: sitio oficial) | Sí (Fuente: sitio oficial) |
| Catálogo, carrito, checkout | Sí [desc] | Sí | Sí |
| Inventario con reserva de stock | Sí (reserva para evitar sobreventa) [desc] | Hipótesis: presente en planes (validar) | Hipótesis: presente (validar) |
| Pagos por pasarela | Sí, con credenciales propias del comercio; la plataforma no retiene fondos [Q2] | Sí, con pasarelas integradas | Sí, 30+ integraciones, 0% cargo por transacción del propio Jumpseller [Q2] (Fuente: jumpseller.co/payment-gateways) |
| Pago manual por transferencia con comprobante | Sí (comprador sube comprobante; admin aprueba/rechaza) [Q2] | Hipótesis: no estándar (validar) | Hipótesis: vía métodos manuales/transferencia (validar) |
| Modelo de monetización al comercio | Suscripción por tienda; sin comisión por venta [Q2][desc] | Suscripción mensual por planes (Fuente: sitio oficial) | Suscripción mensual por planes (Fuente: sitio oficial) |
| Retención de fondos por la plataforma | No | No aplica/varía por pasarela | No (0% del propio Jumpseller) |

Leyenda de confianza: "Fuente" = dato de fuente oficial citada; "Hipótesis" = por verificar en diseño.

## Precios de referencia (hipótesis con fecha — verificar en diseño)

- **Tiendanube Colombia**: 3 planes desde **$33.900 COP/mes** (consulta 2026-11-09). [Q3] (Fuente: Tiendanube blog oficial "planes" — https://www.tiendanube.com/blog/planes-tiendanube/ ; ver también https://www.tiendanube.com/co/planes-y-precios). Nota oficial: además del precio del plan hay costo por transacción asociado a la pasarela que se use (https://www.tiendanube.com/blog/tiendanube-costos/).
- **Jumpseller**: planes mensuales/anuales con 0% de cargos por transacción propios; precios exactos según tipo de contrato y país (consulta 2026-11-09). [Q3] (Fuente: Jumpseller — https://jumpseller.co/sign-up/ y https://es.jumpseller.com/support/subscription/). Referencia de otra región (España): planes desde ~€35,20/mes (solo como orden de magnitud, no es el precio de Colombia). [Q3]
- **Empretienda**: precio no verificado en esta ronda (hipótesis a validar). [Q3]

> Implicación de posicionamiento: tu intención de **suscripción mensual baja fija por tienda** [Q3] es coherente con el modelo dominante de los competidores regionales (plan mensual). El punto de entrada de Tiendanube en Colombia ($33.900 COP/mes, hipótesis con fecha) sirve como ancla para fijar un plan de entrada competitivo; la cifra exacta de tu precio se decide después. [Q3]

## Diferenciación

Tu diferenciador declarado es una **combinación** de: practicidad extrema (publicar en minutos sin conocimientos técnicos), que el comercio use sus propias credenciales de pasarela con no retención de fondos, y el pago manual por transferencia con comprobante (muy usado en Colombia). [Q2]

- La **no retención de fondos + credenciales propias** reduce fricción de alta y evita que la plataforma maneje dinero de terceros; es un ángulo de confianza y simplicidad frente a modelos que intermedian el pago. [Q2]
- El **pago manual por transferencia con comprobante** atiende un comportamiento real del mercado colombiano (transferencias/PSE) y da una vía de venta sin depender de aprobación de pasarela. [Q2]
- La **practicidad** (meta de publicar en <15 min) es el eje de experiencia frente a plataformas más completas pero con mayor curva de configuración. [Q2]

## Assumptions & Open Questions

- [assumption] La lista de competidores regionales (Tiendanube/Nuvemshop, Empretienda, Jumpseller) parte de la indicación del usuario; Empretienda no se verificó en esta ronda. Validar presencia y planes en Colombia en diseño. [Q1]
- [assumption] Todos los precios citados son hipótesis con fecha de consulta (2026-11-09) y deben re-verificarse con fuentes oficiales vigentes en la etapa de diseño; cambian por país y promociones. [Q3]
- [assumption] La paridad de capacidades (inventario, pago manual) de los competidores se marcó como hipótesis donde no hubo verificación directa. [Q1]
