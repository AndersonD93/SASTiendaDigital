# Tendencias de Mercado y Marco Regulatorio — Colombia

> Alcance: nivel de problema/oportunidad. Afirmaciones con fuente citada o marcadas como hipótesis. Fecha de consulta: 2026-11-09. [Q8]
>
> Entrada upstream: parte del `intent-statement` de Intent Capture (`aidlc/spaces/default/intents/261106-multi-tenant-store/ideation/intent-capture/intent-statement.md`), del que toma el país piloto (Colombia), el cliente objetivo y el alcance para seleccionar las tendencias y el marco regulatorio relevantes.

## Tendencias de comercio electrónico en Colombia

- El comercio electrónico en Colombia mantiene crecimiento sostenido. Según la Cámara Colombiana de Comercio Electrónico (CCCE), en 2025 las ventas en línea alcanzaron su nivel más alto desde 2019, con un valor nominal del orden de **$145,4 billones COP** y un crecimiento de alrededor de **11,1 %** frente a 2024. [Q5] (Fuente: CCCE, "Informe de cierre ecommerce 2025" — https://ccce.org.co/noticias/informe-de-cierre-ecommerce-2025-version-publica/ ; cobertura de prensa: El Tiempo — https://www.eltiempo.com/economia/sectores/el-comercio-electronico-en-colombia-alcanzo-en-2025-su-nivel-mas-alto-desde-2019-684-millones-de-transacciones-y-145-4-billones-de-pesos-en-ventas-3533702)
- La base de compradores en línea sigue ampliándose: fuentes de industria reportan del orden de **19 millones** de colombianos comprando en línea en 2025, con incorporación de nuevos compradores primerizos. [Q5] (Fuente: marketing4ecommerce.co citando "Digital 2026" — https://marketing4ecommerce.co/crecimiento-del-ecommerce-en-colombia/ — medio especializado, no fuente primaria; tratar como indicativo.)
- Señal relevante para el nicho: una proporción alta de pymes que venden en línea no tiene tienda física, y para muchas la venta digital es su principal fuente de ingresos. [Q5] (Fuente: Portafolio citando a Mercado Libre — https://www.portafolio.co/negocios/comercio/asi-estan-transformando-las-pymes-colombianas-el-ecommerce-segun-mercado-libre-635721 — tratar la cifra como indicativa del medio.)
- El Observatorio E-commerce del MinTIC publica análisis del posicionamiento de Colombia en la región. [Q5] (Fuente: MinTIC Observatorio E-commerce — https://observatorioecommerce.mintic.gov.co/)

> Implicación: el entorno respalda la oportunidad del nicho objetivo (pymes y emprendedores de bajo presupuesto que migran desde ventas por redes/WhatsApp a una tienda propia). Es coherente con el problema y el cliente definidos en Intent Capture. [Q5]

## Marco regulatorio priorizado: protección de datos personales

El usuario priorizó la **protección de datos personales** como el foco regulatorio relevante para el manejo de datos de clientes. [Q5]

- **Ley 1581 de 2012** ("Régimen General de Protección de Datos Personales" / Habeas Data) es la norma principal en Colombia; aplica al tratamiento de datos personales en cualquier base de datos, pública o privada, en territorio colombiano. [Q5] (Fuente oficial: Función Pública — https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=49981 ; texto en Cancillería — https://cancilleria.gov.co/sites/default/files/Normograma/docs/ley_1581_2012.htm)
- Exige **consentimiento previo, expreso e informado** para recopilar y tratar datos personales, y reconoce el derecho de las personas a conocer, actualizar y rectificar su información. [Q5] (Fuente oficial: MinCIT — https://www.mincit.gov.co/minindustria/estrategia-transversal/regulacion/proteccion-de-datos-personales ; SIC — https://www.sic.gov.co/noticias/node/6581)
- La **Superintendencia de Industria y Comercio (SIC)** es la autoridad de vigilancia del tratamiento de datos personales. [Q5] (Fuente oficial: SIC — https://sedeelectronica.sic.gov.co/transparencia/informacion-de-la-entidad/organigrama/delegatura-de-proteccion-de-datos-personales)

> Implicación para el producto (a nivel de problema): dado que cada tienda maneja datos de sus clientes compradores con aislamiento entre tiendas, el diseño debe contemplar consentimiento informado, finalidad del tratamiento y derechos de los titulares (conocer, actualizar, rectificar, suprimir). El detalle técnico y legal se aborda en diseño/feasibility. [Q5]

## Otras señales a tener en cuenta (fuera del MVP, para fases futuras)

- Facturación electrónica y obligaciones ante la DIAN: fuera del alcance del MVP, pero a considerar cuando se aborde facturación. [Q14] (Hipótesis: requisitos DIAN vigentes a verificar en su momento.)

## Assumptions & Open Questions

- [assumption] Las cifras de crecimiento provienen de informes de la CCCE y cobertura de prensa/medios especializados; las de medios se tratan como indicativas, no como fuente primaria. Revalidar las vigentes en diseño. [Q5]
- [assumption] El cumplimiento específico de la Ley 1581 (textos de política de privacidad, registro de bases de datos ante la SIC, flujos de consentimiento) se detalla en etapas posteriores; aquí solo se identifica como requisito regulatorio relevante. [Q5]
