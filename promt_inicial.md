una plataforma de tienda virtual multi-negocio (multi-tenant, SaaS) que cualquier comercio, sin importar su rubro, pueda usar para vender en línea.

## Visión
Un comerciante se registra, crea su tienda, carga productos, publica una landing con su catálogo y empieza a vender con carrito y pagos en línea. Debe ser genérica (no atada a un tipo de negocio) y configurable por tienda.

## Alcance del MVP
1. Gestión de tiendas (tenants): registro, configuración básica (nombre, logo, colores, moneda, contacto) y subdominio propio por tienda.
2. Catálogo y landing pública: páginas de muestra de productos por tienda, con categorías, búsqueda simple, detalle de producto e imágenes.
3. Inventario: productos, variantes, SKU, stock, movimientos auditables (entradas, salidas, ajustes) y reserva de stock durante el checkout para evitar sobreventa.
4. Carrito y checkout: carrito persistente (anónimo y autenticado), totales con impuestos y descuentos configurables, creación de órdenes y estados de orden.
5. Pagos: integración vía API con proveedores externos confiables, usando checkout alojado o tokenización para que ningún dato de tarjeta pase por nuestros sistemas. Capa de adaptadores para cambiar o agregar pasarelas sin tocar el dominio. Webhooks idempotentes y con verificación de firma.
6. Panel de administración por negocio: productos, inventario, órdenes y configuración de pagos.
7. Autenticación y autorización: staff por tienda y clientes compradores, con aislamiento estricto de datos entre tiendas.

## Restricciones técnicas
- AWS serverless: Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, EventBridge y Step Functions donde aplique.
- Infraestructura como código con Terraform. Frontend web responsivo (propón el framework en el diseño).
- Pruebas automatizadas desde el inicio y pipeline de CI/CD.
- Seguridad: buenas prácticas OWASP, mínimo privilegio en IAM, secretos en Secrets Manager o SSM Parameter Store (nunca en el repositorio) y trazabilidad de operaciones sensibles.
- Observabilidad básica: logs estructurados, métricas y alarmas.

## Entorno AWS (ya preparado, no lo recrees)
- AWS Organizations con una OU llamada "tiendaComercial" que contiene dos cuentas: tienda-dev (ID 547887911593) y tienda-prd (ID 790152345727). La cuenta de gestión NO se usa para desplegar nada de la aplicación.
- Región principal: us-east-1.
- Acceso por IAM Identity Center (SSO) con perfiles locales "tienda-dev" y "tienda-prd".
- Cada cuenta ya tiene su bucket de estado de Terraform: tfstate-tienda-<ID_CUENTA>, con versionado y bloqueo de acceso público. No lo gestiones con Terraform.
- Estructura esperada: modules/ con módulos reutilizables por unidad (tenants, catalog, inventory, orders, payments, admin, frontend) y envs/dev y envs/prd, cada uno con backend S3 (use_lockfile = true, un key por ambiente) y un provider que fije profile y allowed_account_ids.
- Tags comunes en todos los recursos: project, env, owner. Nombres de recursos con prefijo por ambiente.

## Reglas de ejecución
- NO ejecutes terraform apply, terraform destroy ni comandos que modifiquen recursos en AWS. Puedes ejecutar terraform fmt y terraform validate. El plan y el apply los correré yo manualmente.
- Tras generar cada módulo, dime exactamente qué comandos correr, en qué directorio y qué recursos esperar en el plan.
- Empieza por desplegar solo dev. Producción se trabaja después de validar dev.
- No guardes credenciales, llaves ni secretos en el repositorio.

## Fuera de alcance por ahora
Envíos y logística, facturación electrónica, marketplace con varios vendedores en un mismo carrito, apps móviles nativas, multi-idioma.

## Cómo quiero trabajar
- Responde y documenta en español; código, identificadores y nombres de recursos en inglés.
- Avanza por fases y espera mi aprobación explícita en cada puerta.
- Antes de diseñar, hazme las preguntas de requisitos que necesites, en especial sobre: modelo multi-tenant (aislamiento de datos), dominios/subdominios, proveedores de pago iniciales y países/monedas, impuestos y roles de usuario.
- En decisiones de arquitectura importantes, preséntame opciones con pros, contras y tu recomendación.
- Divide el MVP en unidades independientes y propón el orden de construcción.