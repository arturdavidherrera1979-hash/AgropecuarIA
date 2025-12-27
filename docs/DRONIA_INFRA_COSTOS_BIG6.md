# Infraestructura robusta + costos (Big Six) para procesamiento con drones

En DRONIA, el “enemigo” suele ser el costo y complejidad de mover/servir datos (80–100 GB por vuelo). Este documento te baja una recomendación clara: arquitectura, estrategia de costos, comparativa Big Six y decisión mononube vs multinube.

## ✅ Resumen ejecutivo

## ✅ Resumen ejecutivo (lo más conveniente)

### Decisión recomendada

**Mononube para MVP** (90–120 días) + **diseño portable** (Docker + Terraform/OpenTofu + Postgres estándar + Object Storage “genérico”).

✔️ Menos complejidad operativa
✔️ Menos costo por “movimiento de datos”
⚠️ Evitar lock-in “por accidente”
### Por qué

Con vuelos de **80–100 GB**, la multinube suele significar: **duplicar almacenamiento**, pagar **egress** al mover datasets, y sumar IAM/red/observabilidad “x2”. En geodata, la complejidad crece más rápido que el valor.

📉 Riesgo #1: egress
📦 Riesgo #2: datos duplicados
🧰 Riesgo #3: tooling/skills
### La regla de oro DRONIA

**No servís RAW a la app.** Servís *productos* (tiles/MBTiles/COG + insights). Cache + CDN, y la app con modo offline. Ese combo es el mayor “hack” de costos.

### Checklist rápido para estimar tu factura

- **Vuelos/mes** × **GB/vuelo** → ingesta y almacenamiento raw.
- **Retención raw** (30/90/180 días) vs derivados (más tiempo).
- **Usuarios activos** × **GB de mapas por usuario por día** → egress/CDN.
- **Horas de batch** (fotogrametría/IA) → costo de cómputo.


---
## 🧱 Arquitectura

## 🧱 Arquitectura recomendada (offline-first + asincrónica)

### Flujo de datos (campo → nube → app)

1. **Captura** (sin depender de conectividad).
2. **Upload diferido** al volver (multipart/resumible) a Object Storage.
3. Trigger a **workflow** (cola/orquestación) para batch.
4. **Batch geoprocesamiento**: ortomosaico/índices/clasificación por zonas.
5. **Generación de “productos servibles”**: tiles/MBTiles/COG + resumen por lote.
6. **App Mobile** consume tiles + insights; cache local para offline.

### Componentes mínimos “pro”

- **Object Storage**: raw + derivados.
- **Postgres**: metadatos (campos, lotes, vuelos, capas, usuarios).
- **Queue/Workflow**: jobs asincrónicos y reintentos.
- **Compute batch**: contenedores; preemptible/spot donde se pueda.
- **CDN**: para tiles/mapas y bajar egress del origen.
- **Observabilidad**: logs/metrics/traces; alertas por costos.

### Diseño “portable” (anti-lock-in pragmático)

- Docker + imágenes versionadas (pipeline reproducible).
- Infra as Code: Terraform/OpenTofu.
- Postgres estándar (evitar features muy propietarias al comienzo).
- Object storage “S3-compatible” es un plus, pero no lo fuerces si te encarece.
- Formato de resultados: MBTiles/COG/tiles (evita depender de un servicio 100% propietario).


---
## 💸 Cost drivers

## 💸 Cost drivers (dónde se va la plata y cómo domarla)

### Los 3 dominantes

- **Storage**: raw crece rápido (80–100 GB por vuelo).
- **Egress**: servir mapas/tiles a usuarios es un “agujero” si no cacheás.
- **Batch compute**: fotogrametría/IA (CPU/GPU) según pipeline.

📦 Storage: lineal
🌐 Egress: puede explotar
🧠 Batch: picos
### Estrategias de reducción (las que más rinden)

- **Separar raw vs derivados**: raw retención corta (30–90 días), derivados más largos.
- **CDN** delante de tiles (y cache en mobile).
- Formato geodata eficiente: **COG**, tiles, compresión.
- **Spot/Preemptible** para batch (reintentos automáticos).
- **Evitar multinube** en data plane (mover datasets cuesta y complica).

### Ejemplo de “cuenta rápida” (orden de magnitud)

**Supuesto:** 50 vuelos/mes × 90 GB = 4.5 TB de raw/mes. Retención raw 60 días ⇒ ~9 TB raw “vivos”.

- **Storage**: ~9–12 TB (sumando derivados) → del orden de US$ 180–300/mes en object storage estándar (depende nube/region/clase).
- **Egress**: 1 TB/mes sirviendo tiles sin optimizar puede ser US$ 70–180/mes según proveedor y región; con CDN/cache puede bajar fuerte.
- **Batch**: el costo varía muchísimo por pipeline (CPU/GPU, tiempo por vuelo). En MVP, el truco es medir: *minutos de compute por GB*.

Nota: estos rangos son deliberadamente conservadores y se deben recalcular con precios de la región final (p.ej. Sudamérica suele ser más cara en egress en varias nubes).


---
## 🌩️ Big Six

## 🌩️ Big Six: costos “indispensables” (Storage + Internet egress)

Los valores exactos dependen de región y servicio. Acá está lo que más te importa para DRONIA: **almacenamiento** y **salida a internet**. Debajo tenés links oficiales para validar en tu región.

| Proveedor | Storage (objeto) — orden de magnitud | Internet egress — orden de magnitud | Notas para DRONIA |
| --- | --- | --- | --- |
| **AWS** | ≈ US$0.02–0.03/GB-mes Varía por región y clase (Standard/IA/Glacier). | Tier típico ≈ US$0.09/GB (ejemplos en pricing) Incluye mención de 100 GB gratis/mes agregados y ejemplos con $0.09/GB. | Muy maduro en tooling y ecosistema. Usar CDN/cache para no sufrir egress. |
| **Google Cloud** | Standard: $0.000027397/GiB-h (≈ $0.02/GiB-mes) Ver tabla oficial Cloud Storage. | Worldwide: $0.12/GiB (0–10 TiB) Inbound gratis; egress por destino. | Pipeline elegante; ojo con egress si no hay CDN/cache. Cloud Storage + CDN puede mejorar costos de “origen”. |
| **Microsoft Azure** | ≈ US$0.02–0.03/GB-mes Depende de Blob tier y región. | Sudamérica: $0.181/GB (Premium Global Network) Hay opciones de “routing preference” con otros valores. | Buen ecosistema enterprise. Si alojás en SA, egress puede ser caro: diseñar offline/cache. |
| **Oracle Cloud (OCI)** | ≈ US$0.02–0.03/GB-mes Object Storage por región/clase. | 10 TB/mes egress gratis Luego egress a menor costo (ver pricing de networking). | Para DRONIA, el “10 TB free” puede cambiar el TCO si servís mapas. Revisar región y servicios disponibles. |
| **IBM Cloud** | Object Storage + variantes “One-Rate” One-Rate simplifica: storage + egress dentro de allowance (según condiciones). | One-Rate: egress incluido hasta un % (según plan) Ver condiciones y regiones. | Interesante si querés previsibilidad de costo. Validar disponibilidad/latencia según región. |
| **Alibaba Cloud** | Storage Plan 1TB ≈ US$14.34/mes Tabla oficial “Storage Plans” (depende región/plan). | Outbound Traffic Plan 1TB (según región) — ejemplo También publica pay-as-you-go por GB según región. | Los “plans” facilitan presupuestar storage/egress. Validar región más cercana y soporte local. |

### Clave DRONIA

Si tu app empieza a servir mosaicos a muchos usuarios, **egress manda**. El mejor stack no es el que “tiene más features”, sino el que te deja escalar sin que cada zoom de mapa sea un golpe al hígado.


---
## 🧭 Mono vs Multi

## 🧭 Mononube vs Multinube (fortalezas, debilidades y sugerida)

### Mononube (recomendado para MVP)

✔️ Ops simple
✔️ Menos costo de movimiento
✔️ Time-to-market
- **Menos piezas**: IAM, redes, observabilidad, CI/CD en un solo lugar.
- **Datos quietos**: evitás costos de egress inter-nube.
- **Más fácil negociar** descuentos a medida que crece consumo.

Riesgo: lock-in si usás demasiados servicios “especiales” al inicio. Se mitiga con portabilidad (Docker+IaC+formatos abiertos).

### Multinube (solo si hay razón fuerte)

⚠️ Complejidad x2
💸 Egress x2
🧩 Skills x2
- **Data plane multinube** es caro: replicación, sincronización, egress.
- Requiere “plataforma” interna (SRE/DevOps) antes de tiempo.
- Puede servir para **compliance**, resiliencia extrema o negociación; rara vez para MVP.

Versión sana: **mononube** + “paracaídas” (portable), y multinube solo para “control plane” liviano si alguna vez hace falta.

### La sugerida para DRONIA

**Mononube** (dato pesado + batch + tiles) + arquitectura portable. Re-evaluar multinube recién cuando:

- Tenés **volúmenes** que ameritan descuentos significativos o contratos.
- Tenés un requerimiento legal/cliente (p.ej. sector público) que lo exija.
- Tu “data gravity” ya está dominada (tiles optimizados, cache offline, CDN) y el costo de mover datos es marginal.


---
## 🛟 Backup & DR

## 🛟 Backup & Disaster Recovery (DR): “repositorio de backup” para sobrevivir a caída del productivo

### Idea clave (sin humo)

**Replicación ≠ Backup.** Para DR real necesitás al menos 2 cosas:

- **Replica (para continuidad)**: copia casi en tiempo real en *otra región* (misma nube) para hacer failover rápido.
- **Backup aislado (para “sobrevivir”)**: copia “air-gapped” o en *otra cuenta / otra nube* para protegerte de borrados, ransomware o caída/lock de proveedor.

✔️ DR: otra región
⚠️ Backup: otra cuenta / otra nube
💸 Costo: egress + doble storage
### Modelo recomendado para DRONIA (pragmático)

1. **Productivo (nube A)**: Object Storage + Postgres + batch.
2. **Replica regional (nube A, región B)**:
  - Object Storage con **replicación cross-region**.
  - DB con **replica/standby** cross-region (o WAL shipping + restore).
3. **Repositorio de backup “de escape” (nube B)**:
  - Bucket “cold” con **snapshots cifrados** (diario/semanal).
  - IaC + imágenes de contenedor para recrear el stack si la nube A queda inaccesible.

Traducción: mononube para operar, pero con un “paracaídas” real. No es multi-cloud activo-activo (caro e innecesario en MVP).

### Qué respaldar (y qué NO)

- **Siempre**: Postgres (metadatos, usuarios, auditoría) + configuración + modelos/versionado + outputs (tiles/insights).
- **Raw de vuelos**: depende del costo. En MVP, podés:
  - retener raw corto en nube A (30–90 días) +
  - export semanal/mensual al repositorio de backup (nube B) +
  - derivados/tiles retenidos más tiempo (son los que consume la app).
- **Evitar**: replicar raw 1:1 a otra nube en tiempo real, salvo que tengas presupuesto (egress + storage se duplican).

### Nivel de DR por “tier” (para decidir costo)

| Tier | Objetivo típico | Diseño | Costo / complejidad |
| --- | --- | --- | --- |
| **Bronce** | RTO 24–72h · RPO 24h | Backups diarios (DB + outputs) a bucket aislado (otra cuenta/nube). Restore “manual”. | Bajo |
| **Plata** | RTO 4–24h · RPO 1–6h | Replica cross-region (misma nube) + backups aislados. Infra “warm” opcional. | Medio |
| **Oro** | RTO < 1–2h · RPO < 15–60 min | Arquitectura multi-región activa (o hot-standby), automatización de failover, pruebas regulares. | Alto |

### Opciones típicas por proveedor (en 1 línea)

- **AWS**: S3 Replication (CRR/SRR) para buckets; backups de DB + restauración en otra región.
- **Google Cloud**: buckets dual/multi-región + cross-bucket replication/transfer; turbo replication opcional.
- **Azure**: GRS/GZRS + failover; RA-* para lectura en secundario.
- **OCI**: Object Storage Replication (política de replicación bucket→bucket en otra región).
- **IBM**: replicación automática y asincrónica entre buckets/regiones.
- **Alibaba**: OSS Cross-region replication (CRR) automática y asincrónica; cuenta o cross-account.

Nota: “repositorio de backup” recomendado = **bucket en otra cuenta** (y/o otra nube) con versioning + retención/immutability si está disponible.

### Fuentes rápidas (replicación / redundancia)

- AWS S3 Replication (CRR/SRR): [docs.aws.amazon.com/.../replication.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- Google Cloud Storage (dual/multi-region, cross-bucket replication, turbo replication): [docs.cloud.google.com/.../availability-durability](https://docs.cloud.google.com/storage/docs/availability-durability)
- Azure Storage redundancy (GRS/GZRS/RA-*): [learn.microsoft.com/.../storage-redundancy](https://learn.microsoft.com/azure/storage/common/storage-redundancy)
- OCI Object Storage Replication: [docs.oracle.com/.../usingreplication.htm](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/usingreplication.htm)
- IBM Cloud Object Storage replication: [ibm.com/.../replication](https://www.ibm.com/products/cloud-object-storage/replication)
- Alibaba OSS Cross-region replication: [alibabacloud.com/.../cross-region-replication-overview](https://www.alibabacloud.com/help/en/oss/cross-region-replication-overview/)


---
## 🎯 Recomendación

## 🎯 Recomendación (robusta, precisa y accionable)

### Stack MVP (12–16 semanas)

- **Object Storage** (raw + derivados) + lifecycle policies.
- **Postgres managed** (metadatos).
- **Workflow/Queue** para batch (reintentos + idempotencia).
- **Batch containers** (spot/preemptible) para pipeline geo/IA.
- **CDN** para tiles + app con cache offline.
- **Cost guardrails**: budgets + alertas por egress y storage.

### ¿Qué nube elegir?

Sin datos finos de región/usuarios todavía, la decisión la baso en el costo de egress y facilidad de operación:

- **Si esperás mucho consumo de mapas**: OCI es muy atractiva por “10 TB/mes egress gratis”.
- **Si querés mercado/contratación fácil**: AWS es la apuesta “segura” (pero diseñar para egress).
- **Si querés pipelines muy integrados**: GCP, con una arquitectura de cache/CDN bien pensada.
- **Si tu región base es Sudamérica**: prestar atención extra a egress (Azure publica números altos en SA en el pricing de bandwidth).

### Próximos pasos (para cerrar con números)

1. Definir región objetivo (¿Brasil/Sao Paulo? ¿Chile? ¿US?) para storage/compute/egress.
2. Estimar: vuelos/mes, GB/vuelo, retención raw, tamaño derivados.
3. Estimar: usuarios activos, sesiones/día, GB de tiles por sesión, % cache hit (CDN + mobile).
4. Modelar batch: minutos CPU/GPU por vuelo (medir con un piloto real).
5. Comparar 2–3 nubes con calculadoras oficiales y elegir 1 para el MVP.

Si querés, el siguiente entregable “de consultoría” es una planilla de TCO con escenarios (Conservador / Base / Agresivo) para elegir nube con datos, no con fe.


---
## 🤝 Aeroterra

## 🤝 Aeroterra como socio (límites claros para que DronIA siga siendo DronIA)

### Resumen claro (no “muy resumido”)

**Lo que pretendemos:** DronIA es el *producto* (app, pipeline, IA, roadmap, datos y modelo de negocio). Aeroterra puede ser un **socio** para habilitar adopción, integración GIS y delivery enterprise cuando convenga.

**La regla de oro:** Aeroterra/ArcGIS se usa como **capa de publicación/consumo** (cuando el cliente lo necesita), pero **no** como “cocina” ni como “SSOT” (single source of truth).

✔️ DronIA = SSOT + IA + app
⚠️ Aeroterra = integración / publicación / soporte GIS
⛔ No: white-label / lock-in / dependencia total
### Qué podemos necesitar de Aeroterra (lo “sí”)

- **Entrada a organizaciones** que ya operan con ArcGIS (especialmente grandes) y piden integración “corporativa”.
- **Implementación GIS**: usuarios/roles, publicación de capas, portales y gobierno de datos geoespaciales.
- **Infra/arquitectura híbrida** (cloud/on-prem) cuando el cliente lo exige: Aeroterra declara modalidades cloud/aislada/híbrida y uso mobile/desktop/corporativo.
- **Capacitación y soporte** en ArcGIS para el cliente final.
- **Proyecto “enterprise”**: SSO/seguridad, auditoría, estándares internos del cliente.

### El límite (lo “no negociable”)

- **Propiedad del producto/IP**: código, modelos IA, pipeline, UX, marca y roadmap son de DronIA.
- **SSOT de datos**: el dato “canónico” (metadatos + outputs) vive en DronIA; ArcGIS consume/publica.
- **Anti lock-in**: formatos abiertos (COG/GeoTIFF, GeoJSON/GeoParquet, MBTiles/tiles) como contrato técnico.
- **No white-label** sin acuerdo explícito (y costoso) + cláusulas de marca.
- **Exit plan**: si la relación termina, el cliente sigue con DronIA y puede exportar/continuar operando.

### Riesgos principales (y cómo los mitigamos)

| Riesgo | Cómo se ve en la práctica | Mitigación “DronIA-first” |
| --- | --- | --- |
| **Lock-in** | Todo queda “encerrado” en ArcGIS: formatos, workflows, autenticación, UI. Migrar se vuelve carísimo. | **SSOT propio** + **adapter** de publicación ArcGIS (enchufe, no dependencia) + formatos abiertos. |
| **La marca se diluye** | El cliente cree que “esto es Aeroterra/ArcGIS” y DronIA queda invisible. | Modelo comercial **Referral + Implementation Partner** (DronIA vende el producto; Aeroterra servicios/ArcGIS). Branding contractual. |
| **Scope creep** | El proyecto se transforma en “una implementación GIS” y no en un producto replicable para pequeños/medianos. | Oferta por segmentos: **SaaS DronIA** para pequeños/medianos; ArcGIS solo como conector opcional o capa enterprise. |
| **Dependencia operativa** | Si Aeroterra no responde, quedás bloqueado en soporte/operación del cliente. | RACI + límites de soporte: DronIA (pipeline/modelos/app) / Aeroterra (ArcGIS y entorno GIS). SLA escrito. |

### Cómo lo encajamos con tu estrategia (pequeños · medianos · grandes)

### Pequeños productores

- **DronIA SaaS directo** (mobile-first + web liviana).
- Cero dependencia ArcGIS.
- Valor: recomendaciones + mapas livianos + offline.

### Medianos productores

- DronIA SaaS + **conector opcional** a ArcGIS (si ya lo tienen).
- ArcGIS como “destino de publicación” (capas/servicios), no como núcleo.

### Grandes productores / corporativos

- Co-venta posible + **Aeroterra como partner de implementación** (seguridad, SSO, roles, portales GIS).
- DronIA mantiene **producto**, **IA**, **pipeline** y **SSOT**.

### Información y contactos de Aeroterra (para iniciar conversación)

### Contacto (oficial)

- **Email general:** info@aeroterra.com
- **Teléfono:** +54 11 5272 0900
- **Soporte técnico:** soporte@aeroterra.com · +54 11 5272 0911
- **Servicio al cliente:** cs@aeroterra.com
- **Capacitación:** capacitacion@aeroterra.com
- **Dirección (CABA):** Carlos M. Della Paolera 218, C1001ADB

### Referencias (links)

- Sitio: [Sobre Aeroterra](https://www.aeroterra.com/es-ar/sobre-aeroterra/introduccion)
- Contacto: [Página de contacto](https://www.aeroterra.com/es-ar/contacto)
- Productos ArcGIS (modalidades cloud/aislada/híbrida): [Productos y soluciones](https://www.aeroterra.com/es-ar/productos/index)
- LinkedIn: [Perfil de Aeroterra](https://www.linkedin.com/company/aeroterra-s-a-)

Tip: el “primer contacto” debería pedir una reunión de 30–45 min para alinear modelo de partnership: referral + implementación (no white-label), y una arquitectura de conector (publicación de capas) sin lock-in.

### Preguntas de kickoff (para poner límites desde el minuto 1)

1. ¿Qué parte quieren hacer ustedes (Aeroterra) y qué parte hacemos nosotros (DronIA)? (RACI)
2. ¿ArcGIS sería “visor/publicación” o “plataforma core”? (Nosotros: visor/publicación)
3. ¿Qué formatos/APIs de salida recomiendan para publicar capas sin lock-in?
4. ¿Cómo se maneja el soporte al cliente? (Quién atiende incidentes del producto vs ArcGIS)
5. ¿Cómo se define el modelo comercial por segmento (pequeño/mediano/grande)?


---
## 🔗 Fuentes

## 🔗 Fuentes oficiales (para validar precios por región)

Estos links te sirven para chequear precios reales por región/uso y armar un cálculo serio.

### Pricing pages

- AWS S3 Pricing (Storage + Data transfer): [aws.amazon.com/s3/pricing](https://aws.amazon.com/s3/pricing/)
- Google Cloud Storage Pricing (storage + egress): [cloud.google.com/storage/pricing](https://cloud.google.com/storage/pricing)
- Azure Bandwidth Pricing: [azure.microsoft.com/.../bandwidth](https://azure.microsoft.com/en-gb/pricing/details/bandwidth/)
- Oracle OCI Networking (egress 10 TB free): [oracle.com/.../virtual-cloud-network/pricing](https://www.oracle.com/cloud/networking/virtual-cloud-network/pricing/)
- IBM Cloud Object Storage (One-Rate info): [ibm.com/.../one-rate-plan](https://www.ibm.com/new/announcements/get-predictable-low-cost-cloud-object-storage-for-active-workloads-with-the-one-rate-plan)
- Alibaba Cloud OSS Pricing: [alibabacloud.com/product/oss/pricing](https://www.alibabacloud.com/product/oss/pricing)

### Calculadoras

- AWS Pricing Calculator: [calculator.aws](https://calculator.aws/)
- Google Cloud Pricing Calculator: [cloud.google.com/products/calculator](https://cloud.google.com/products/calculator)
- Azure Pricing Calculator: [azure.microsoft.com/pricing/calculator](https://azure.microsoft.com/pricing/calculator/)
- Oracle Cloud Cost Estimator: [oracle.com/cloud/costestimator](https://www.oracle.com/cloud/costestimator.html)
- IBM Cloud Pricing: [ibm.com/cloud/pricing](https://www.ibm.com/cloud/pricing)
- Alibaba Cloud Pricing Calculator: [alibabacloud.com/pricing](https://www.alibabacloud.com/pricing)

Sugerencia práctica: definí 1 región objetivo (p. ej. São Paulo / Santiago / US) y corré **el mismo escenario** en todas las calculadoras: storage (TB), egress (TB), batch compute (horas), DB (size). Ahí sale el ganador de forma objetiva.


---
