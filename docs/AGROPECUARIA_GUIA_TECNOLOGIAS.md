# Guía didáctica de tecnologías para AGROPECUARIA

Objetivo: elegir tecnología sin inventar problemas . AGROPECUARIA tiene captura con drone offline , subida al volver, procesamiento asíncrono en nube, y salida principal en App Mobile (Android + iOS). Volumen típico por vuelo/salida: 80–100 GB .

## 🧭 Resumen

✅ Recomendación principal (para decidir hoy)
## Stack recomendado para MVP y escalado

Esto es lo más conveniente para AGROPECUARIA:
- **Mobile:** React Native + TypeScript (Android + iOS), enfoque **offline-first**
- **API:** FastAPI (Python)
- **Pipeline (geo + IA):** Python
- **Geodatos:** PostgreSQL + PostGIS
- **Archivos:** Object Storage (S3/MinIO)
- **Ortomosaico:** COG + **tiles on-demand** (no bajar GBs al teléfono)

Idea clave (para evitar el error típico):
“Mapa pesado” en móvil casi siempre significa “quiero zoom fluido sobre un ortomosaico enorme”. Eso se resuelve con
tiles
(cuadritos) bajo demanda, no descargando el archivo completo.
Lo menos conveniente en MVP (y por qué):
- **Procesamiento en vuelo (tiempo real):** sube complejidad, costos y hardware antes de validar ROI.
- **Offline total con ortomosaico completo:** empuja GBs a teléfonos y complica iOS/Android (cache, permisos, UX).
- **Backend en dos runtimes desde día 1 (TS + Python):** más coordinación y DevOps para un MVP.

**¿Por qué “un stack por capa” es la jugada correcta?**

AGROPECUARIA es (1) hardware/captura, (2) ciencia de datos geoespacial, (3) producto. Intentar resolver todo con “un solo lenguaje” suele llevar a pelear con tooling inmaduro o duplicar esfuerzos.

## 🎯 3 decisiones rápidas

1) Mobile:
React Native + TypeScript
2) Backend:
FastAPI (Python) + PostGIS + S3/MinIO
3) Mapas:
Zonas livianas + orto por tiles on-demand
### 📌 Señales de alerta (anti-humo)

- Si alguien promete “humedad del suelo directa” solo con imágenes: pedir calibración/validación.
- Si no hay upload reanudable, 100 GB te rompen el flujo real.
- Si el MVP intenta resolver “todo” (multi-cultivo, tiempo real, offline total), se dilata.


---
## 📱 Mobile

## 📱 App Mobile (Android + iOS)

✅ Más conveniente
### React Native + TypeScript

- Un solo codebase para Android/iOS → **menos costo** y más velocidad.
- Buena oferta de devs y ecosistema maduro en Argentina.
- Encaja con apps de campo: offline-first, cámara, checklists, notas, sync.
- Permite “módulos nativos” cuando mapas/tiles lo exijan.

Recomendación de implementación:
- Arrancar con RN “bare” para no quedar limitado si necesitamos mapas/tiles avanzados.
- Diseñar desde el día 1 el patrón offline-first (cache + sync + reintentos).

⚖️ Alternativa válida
### Flutter (Dart)

- UI muy consistente y fluida, buen rendimiento.
- Conviene si el equipo ya domina Dart/Flutter o si la UI es muy exigente.

**¿Qué es “offline-first” en AGROPECUARIA, sin humo?**

La app debe funcionar aunque no haya señal: ver lotes, zonas, historial, checklist y notas. Cuando vuelve la conectividad, sincroniza sin perder datos.

- **Cache local** (SQLite/Realm): lotes, zonas, reportes ligeros, tareas.
- **Sync** por lotes: retries, idempotencia, “estado de subida”.
- **Uploads grandes**: reanudables (multipart/chunked).

**Qué NO hacer en mobile con 80–100 GB**

- No intentar subir 100 GB desde el campo por 4G. Diseñar para subir al volver (Wi‑Fi/base).
- No bajar raws al teléfono. El móvil consume resultados: zonas + tiles.

## ✅ Checklist “móvil listo para campo”

- Modo offline (sin señal) usable
- Cache local + “sync status” visible
- Descarga previa (pre-cache) por lote si hace falta
- Mapas: zonas livianas siempre; orto por tiles
- Control de permisos (ubicación/cámara) en iOS y Android
- Monitoreo de tamaño de caché

Nota iOS:
cuidado con background tasks y límites de almacenamiento de caché.

---
## 🧠 Backend & Datos

## 🧠 Backend & Datos (API + procesamiento)

✅ Más conveniente
### Python end-to-end (FastAPI + pipeline)

- En geoespacial/teledetección, el tooling más maduro está en Python.
- Unificar API y pipeline reduce fricción del MVP (menos runtimes y menos DevOps).
- Ideal para iterar rápido en heurísticas/IA y “insight accionable”.

### PostgreSQL + PostGIS (geometría)

- Polígonos de lotes, zonas recomendadas, prescripciones y resultados con geometría.
- Consultas espaciales: intersecciones, buffers, áreas, etc.

### Object Storage (S3/MinIO)

- Fotos raw, ortomosaicos, productos procesados, tiles, reportes.
- Regla: metadata en DB; archivos en buckets.

**Arquitectura mental mínima (MVP)**

```

APP (Android/iOS)
   ↓ (API)
FastAPI (auth + lotes + resultados)
   ↓
PostGIS (geometrías, entidades)
   ↘
Object Storage (raws + procesados)
   ↓
Workers (jobs) procesan datasets asíncronos
   ↓
Se publican resultados: zonas + tiles + resumen

```

**¿Por qué no Node/NestJS para API desde el día 1?**

Se puede, pero te deja con dos runtimes (TS para API + Python para pipeline), más coordinación y DevOps. Para MVP, Python end-to-end suele ser más barato y rápido.

## 📌 “Salida mínima” del pipeline

Producto liviano (para móvil):
- Zonas/polígonos + niveles de confianza
- Resumen accionable (qué, dónde, cuándo, cómo verificar)
- Preview/thumbnail

Producto pesado (para lupa):
- Ortomosaico en formato COG
- Tiles on-demand para zoom fluido


---
## 🗺️ Mapas

## 🗺️ Mapas: pesado vs liviano (sin jerga innecesaria)

| Capa | Qué es | Peso típico | Qué conviene en móvil |
| --- | --- | --- | --- |
| **Mapa base** | Satélite/calles (terceros). Es el “fondo”. | Liviano | Online cuando hay señal. No lo genera AGROPECUARIA. |
| **Zonas / prescripciones** | Polígonos: “acá sí”, “acá no”, “acá estrés”. | KB–MB | **Ideal** para móvil. Offline excelente. |
| **Ortomosaico** | Imagen cosida del lote, zoom con detalle. | GBs | Servir como **tiles** bajo demanda (no descargar completo). |

Estrategia recomendada (híbrida):
Siempre
zonas livianas + recomendación accionable
Cuando hace falta
ortomosaico por tiles on-demand + cache
Si se necesita
pre-cache por lote antes de ir al campo
**Tiles explicado en una frase**

Partimos el ortomosaico en cuadritos por nivel de zoom. El celular pide solo los cuadritos que ve (como Google Maps).

**¿Qué significa COG (Cloud Optimized GeoTIFF)?**

Es un GeoTIFF preparado para que el servidor/cliente lea partes específicas sin bajar el archivo completo. Es la pieza que hace viable “orto pesado” sin matar al móvil.

## ✅ Regla de producto

El valor del MVP vive en “zonas + acción”.
El ortomosaico es la “lupa” para auditar y explicar, pero no debería ser el único entregable.
### 🧩 ¿Cuándo necesitas orto sí o sí?

- Auditar un insight (“¿por qué me recomendás esto?”)
- Ver microdetalle (manchones finos, filas, anomalías visuales)
- Soporte a decisiones de campo en zonas específicas

PROVEEDORES
## 🚁 Proveedores y redes de servicios con drones (Argentina)

Esta pestaña resume proveedores y redes que públicamente ofrecen servicios de drones para agricultura de precisión en Argentina. Útil para **partnership** (tercerizar vuelos) o para armar una **red operativa** mientras AGROPECUARIA desarrolla software y pipeline. **Nota importante:** la oferta real (sensores disponibles, cobertura geográfica, tiempos y precios) cambia. Usar esto como shortlist y validar por contacto directo y pruebas piloto.

Lectura rápida (lo más conveniente para el MVP):
- **Si querés operar “sin tener drones propios”:** buscar redes/servicios completos (ej. mapeo + procesamiento + prescripción).
- **Si querés “tu propia capa IA” con datos controlados:** negociar acceso a raws/procesados y propiedad de datos.
- **Si te preocupa compliance/seguro:** apoyarte en cámaras/redes donde el estándar sea operar bajo normativa y con seguros.

### Shortlist (según info pública)

| Entidad | Qué ofrecen (resumen) | Tecnología / drones / sensores (mencionados) |
| --- | --- | --- |
| **Foto Aérea (CABA)** | Servicios de mapeo para agro y medio ambiente; generación de ortomosaicos e índices (NDVI/NDRE, etc.); también procesamiento de imágenes. | Sensores **MicaSense** multiespectrales (5 bandas); menciona cámaras térmicas **FLIR** radiométricas; drones ala fija y multirotor. |
| **Dronify** | Mapeo multiespectral, informes y planeamiento; aplicación variable/localizada de líquidos. | Menciona “drones Maverick” para mapeo; y drones **T40** para aplicación localizada. |
| **Sky Solutions** | Agricultura de precisión: mapeo multiespectral, índices (NDVI/NDRE), detección de estrés hídrico y otros reportes. | Drones con sensores multiespectrales; menciona índices (NDVI/NDRE) y “pilotos certificados por ANAC” en su sitio. |
| **BASF xarvio® (servicio MDM)** | Servicio de **Mapeo Digital de Malezas** que procesa imágenes de vuelos con drones y entrega mapa/prescripción para aplicaciones sectorizadas (12–48h según lote); ofrece “paquete con imágenes propias” o **servicio completo** (incluye vuelo). | Trabaja con drones propios o contratados; menciona compatibilidad con drones/cámaras “accesibles” (RGB) y red de operadores autorizados. |
| **Vistaguay (red / plataforma)** | Plataforma que conecta pilotos con productores para servicios: fotogrametría, pulverización, siembra, ortomosaicos (RGB/multi/térmico), NDVI, modelos de elevación, daños, etc. | En su descripción enumera productos típicos (RGB/multi/térmico, NDVI, ortomosaicos) y servicios (pulverización con drone, etc.). |
| **CAEDyA (cámara)** | Cámara Argentina de Empresas de Drones y Afines. Sirve como “radar” de empresas/profesionales y estándar de operación. | Enfatiza cumplimiento de normativa ANAC y seguros de responsabilidad civil como parte del compromiso de membresía. |

### Notas por proveedor (qué mirar / para qué sirve)

**Foto Aérea — cuando necesitás multiespectral serio (NDVI/NDRE) y/o térmico**

- **Servicios:** mapeo agro/medio ambiente, ortomosaicos e índices; también procesamiento.
- **Tecnología citada:** sensores MicaSense (5 bandas) para multiespectral; FLIR radiométrico para térmico; drones ala fija y multirotor.
- **Para AGROPECUARIA:** buen partner si tu modelo necesita datasets multiespectrales bien calibrados.
- **Pregunta clave:** ¿entregan COG/GeoTIFF + metadatos + protocolo de calibración?

**Dronify — cuando querés “mapeo + aplicación localizada” (spraying variable)**

- **Servicios:** mapeo multiespectral + informes; aplicación variable/localizada de líquidos.
- **Tecnología citada:** drones “Maverick” para mapeo; drones T40 para aplicación localizada.
- **Para AGROPECUARIA:** partner natural si el MVP apunta a prescripción/accionamiento (aplicar donde hace falta).
- **Pregunta clave:** ¿qué formato de prescripción entregan y con qué equipos/monitores es compatible?

**Sky Solutions — agricultura de precisión “servicio generalista”**

- **Servicios:** NDVI/NDRE, estrés hídrico, monitoreo, informes.
- **Dato público:** menciona pilotos certificados ANAC y uso de sensores multiespectrales.
- **Para AGROPECUARIA:** posible proveedor por cobertura y paquetes de servicio; validar sensores exactos y outputs.

**BASF xarvio® (MDM) — si querés “servicio completo” y prescripción rápida**

- **Qué es:** Mapeo Digital de Malezas: vuelo con drones + procesamiento con algoritmos + prescripción para aplicación sectorizada.
- **Modalidades:** (1) vos aportás imágenes; (2) servicio completo (incluye vuelo) con red de operadores.
- **Tiempos:** menciona entrega 12–48h post vuelo (según lote).
- **Para AGROPECUARIA:** gran referencia de “producto”: del vuelo a la prescripción con SLA claro.

**Vistaguay — para construir red operativa (pilotos) y estandarizar servicios**

- **Qué es:** plataforma que conecta pilotos con productores; menciona fotogrametría, pulverización, siembra.
- **Productos:** ortomosaicos RGB/multi/térmico, NDVI, modelos, daños, etc.
- **Para AGROPECUARIA:** referencia de “marketplace operativo” (si en el futuro AGROPECUARIA escala con red de pilotos).

**CAEDyA — para sourcing y filtro de profesionalismo**

- **Qué es:** cámara del sector; útil para encontrar empresas, pilotos, partners y eventos/webinars.
- **Dato público:** enfatiza cumplimiento ANAC y seguros de responsabilidad civil de sus miembros.
- **Para AGROPECUARIA:** buen canal para armar red (con checklist de compliance).

### Checklist para elegir/contratar proveedor (anti-sorpresas)

Pedí esto sí o sí:
- **Entregables:** raws + procesados (GeoTIFF/COG/orthomosaic) + metadatos.
- **Calibración (si multiespectral):** cómo hacen radiometría/paneles/condiciones.
- **Georreferenciación:** RTK/PPK, GSD, solapes, QA.
- **Propiedad de datos:** que AGROPECUARIA pueda usar datasets para mejorar modelos.
- **SLA:** tiempo de entrega (ej. 12–48h).
- **Compliance:** operación alineada a normativa ANAC y seguros.

**Fuentes (links)**

- [Foto Aérea (sitio)](https://fotoaerea.com.ar/)
- [Foto Aérea — Agro y Medio Ambiente](https://fotoaerea.com.ar/servicio-mapeo-con-drones-agricultura-medio_ambiente/)
- [Dronify (sitio)](https://dronify.com.ar/)
- [Sky Solutions — Agricultura](https://skysolutions.com.ar/servicios/agricultura/)
- [BASF — xarvio en Aapresid (MDM)](https://agriculture.basf.com/ar/es/notas-de-prensa/2024/agosto/xarvio-congreso-aapresid-2024)
- [BASF — lanzamiento MDM (servicio)](https://agriculture.basf.com/basf/agriculture/ar/es/notas-de-prensa/2023/Octubre/xarvio-lanza-el-servicio-de-mapeo-digital-de-malezas-con-drones)
- [Vistaguay Experts (App Store)](https://apps.apple.com/ar/app/vistaguay-experts/id6747045323)
- [Argentina.gob.ar — drones + Vistaguay](https://www.argentina.gob.ar/noticias/alfalfa-usan-drones-para-cuantificar-la-calidad-de-siembra)
- [CAEDyA (sitio)](https://caedya.com.ar/)
- [ANAC — marco normativo RAAC 100/101/102](https://www.argentina.gob.ar/anac/nuevo-marco-normativo-para-la-operacion-de-drones)

## 🤝 Modelos de partnership (rápidos)

Modelo A — “Servicio completo” (vuelo + procesamiento):
Te acelera MVP. Riesgo: menos control sobre datos/protocolo.
Modelo B — “Vos procesás, ellos vuelan”:
Ideal para AGROPECUARIA: asegurás raws/metadatos y tu pipeline/IA hace el diferencial.
Modelo C — “Propio desde día 1”:
Comprar/operar drones ya en MVP suele distraer. Reservar para fase 2 (si el negocio lo justifica).
### 📌 Tip operativo

- Arrancar con 1–2 proveedores para pilotos controlados.
- Estandarizar un “protocolo de vuelo + entrega de datos”.
- Repetibilidad > variedad (al principio).


---
## 🚁 Proveedores (AR)

## 🚁 Proveedores y redes de servicios con drones (Argentina)

Esta pestaña resume proveedores y redes que públicamente ofrecen servicios de drones para agricultura de precisión en Argentina. Útil para **partnership** (tercerizar vuelos) o para armar una **red operativa** mientras AGROPECUARIA desarrolla software y pipeline. **Nota importante:** la oferta real (sensores disponibles, cobertura geográfica, tiempos y precios) cambia. Usar esto como shortlist y validar por contacto directo y pruebas piloto.

Lectura rápida (lo más conveniente para el MVP):
- **Si querés operar “sin tener drones propios”:** buscar redes/servicios completos (ej. mapeo + procesamiento + prescripción).
- **Si querés “tu propia capa IA” con datos controlados:** negociar acceso a raws/procesados y propiedad de datos.
- **Si te preocupa compliance/seguro:** apoyarte en cámaras/redes donde el estándar sea operar bajo normativa y con seguros.

### Shortlist (según info pública)

| Entidad | Qué ofrecen (resumen) | Tecnología / drones / sensores (mencionados) |
| --- | --- | --- |
| **Foto Aérea (CABA)** | Servicios de mapeo para agro y medio ambiente; generación de ortomosaicos e índices (NDVI/NDRE, etc.); también procesamiento de imágenes. | Sensores **MicaSense** multiespectrales (5 bandas); menciona cámaras térmicas **FLIR** radiométricas; drones ala fija y multirotor. |
| **Dronify** | Mapeo multiespectral, informes y planeamiento; aplicación variable/localizada de líquidos. | Menciona “drones Maverick” para mapeo; y drones **T40** para aplicación localizada. |
| **Sky Solutions** | Agricultura de precisión: mapeo multiespectral, índices (NDVI/NDRE), detección de estrés hídrico y otros reportes. | Drones con sensores multiespectrales; menciona índices (NDVI/NDRE) y “pilotos certificados por ANAC” en su sitio. |
| **BASF xarvio® (servicio MDM)** | Servicio de **Mapeo Digital de Malezas** que procesa imágenes de vuelos con drones y entrega mapa/prescripción para aplicaciones sectorizadas (12–48h según lote); ofrece “paquete con imágenes propias” o **servicio completo** (incluye vuelo). | Trabaja con drones propios o contratados; menciona compatibilidad con drones/cámaras “accesibles” (RGB) y red de operadores autorizados. |
| **Vistaguay (red / plataforma)** | Plataforma que conecta pilotos con productores para servicios: fotogrametría, pulverización, siembra, ortomosaicos (RGB/multi/térmico), NDVI, modelos de elevación, daños, etc. | En su descripción enumera productos típicos (RGB/multi/térmico, NDVI, ortomosaicos) y servicios (pulverización con drone, etc.). |
| **CAEDyA (cámara)** | Cámara Argentina de Empresas de Drones y Afines. Sirve como “radar” de empresas/profesionales y estándar de operación. | Enfatiza cumplimiento de normativa ANAC y seguros de responsabilidad civil como parte del compromiso de membresía. |

### Notas por proveedor (qué mirar / para qué sirve)

**Foto Aérea — cuando necesitás multiespectral serio (NDVI/NDRE) y/o térmico**

- **Servicios:** mapeo agro/medio ambiente, ortomosaicos e índices; también procesamiento.
- **Tecnología citada:** sensores MicaSense (5 bandas) para multiespectral; FLIR radiométrico para térmico; drones ala fija y multirotor.
- **Para AGROPECUARIA:** buen partner si tu modelo necesita datasets multiespectrales bien calibrados.
- **Pregunta clave:** ¿entregan COG/GeoTIFF + metadatos + protocolo de calibración?

**Dronify — cuando querés “mapeo + aplicación localizada” (spraying variable)**

- **Servicios:** mapeo multiespectral + informes; aplicación variable/localizada de líquidos.
- **Tecnología citada:** drones “Maverick” para mapeo; drones T40 para aplicación localizada.
- **Para AGROPECUARIA:** partner natural si el MVP apunta a prescripción/accionamiento (aplicar donde hace falta).
- **Pregunta clave:** ¿qué formato de prescripción entregan y con qué equipos/monitores es compatible?

**Sky Solutions — agricultura de precisión “servicio generalista”**

- **Servicios:** NDVI/NDRE, estrés hídrico, monitoreo, informes.
- **Dato público:** menciona pilotos certificados ANAC y uso de sensores multiespectrales.
- **Para AGROPECUARIA:** posible proveedor por cobertura y paquetes de servicio; validar sensores exactos y outputs.

**BASF xarvio® (MDM) — si querés “servicio completo” y prescripción rápida**

- **Qué es:** Mapeo Digital de Malezas: vuelo con drones + procesamiento con algoritmos + prescripción para aplicación sectorizada.
- **Modalidades:** (1) vos aportás imágenes; (2) servicio completo (incluye vuelo) con red de operadores.
- **Tiempos:** menciona entrega 12–48h post vuelo (según lote).
- **Para AGROPECUARIA:** gran referencia de “producto”: del vuelo a la prescripción con SLA claro.

**Vistaguay — para construir red operativa (pilotos) y estandarizar servicios**

- **Qué es:** plataforma que conecta pilotos con productores; menciona fotogrametría, pulverización, siembra.
- **Productos:** ortomosaicos RGB/multi/térmico, NDVI, modelos, daños, etc.
- **Para AGROPECUARIA:** referencia de “marketplace operativo” (si en el futuro AGROPECUARIA escala con red de pilotos).

**CAEDyA — para sourcing y filtro de profesionalismo**

- **Qué es:** cámara del sector; útil para encontrar empresas, pilotos, partners y eventos/webinars.
- **Dato público:** enfatiza cumplimiento ANAC y seguros de responsabilidad civil de sus miembros.
- **Para AGROPECUARIA:** buen canal para armar red (con checklist de compliance).

### Checklist para elegir/contratar proveedor (anti-sorpresas)

Pedí esto sí o sí:
- **Entregables:** raws + procesados (GeoTIFF/COG/orthomosaic) + metadatos.
- **Calibración (si multiespectral):** cómo hacen radiometría/paneles/condiciones.
- **Georreferenciación:** RTK/PPK, GSD, solapes, QA.
- **Propiedad de datos:** que AGROPECUARIA pueda usar datasets para mejorar modelos.
- **SLA:** tiempo de entrega (ej. 12–48h).
- **Compliance:** operación alineada a normativa ANAC y seguros.

**Fuentes (links)**

- [Foto Aérea (sitio)](https://fotoaerea.com.ar/)
- [Foto Aérea — Agro y Medio Ambiente](https://fotoaerea.com.ar/servicio-mapeo-con-drones-agricultura-medio_ambiente/)
- [Dronify (sitio)](https://dronify.com.ar/)
- [Sky Solutions — Agricultura](https://skysolutions.com.ar/servicios/agricultura/)
- [BASF — xarvio en Aapresid (MDM)](https://agriculture.basf.com/ar/es/notas-de-prensa/2024/agosto/xarvio-congreso-aapresid-2024)
- [BASF — lanzamiento MDM (servicio)](https://agriculture.basf.com/basf/agriculture/ar/es/notas-de-prensa/2023/Octubre/xarvio-lanza-el-servicio-de-mapeo-digital-de-malezas-con-drones)
- [Vistaguay Experts (App Store)](https://apps.apple.com/ar/app/vistaguay-experts/id6747045323)
- [Argentina.gob.ar — drones + Vistaguay](https://www.argentina.gob.ar/noticias/alfalfa-usan-drones-para-cuantificar-la-calidad-de-siembra)
- [CAEDyA (sitio)](https://caedya.com.ar/)
- [ANAC — marco normativo RAAC 100/101/102](https://www.argentina.gob.ar/anac/nuevo-marco-normativo-para-la-operacion-de-drones)

## 🤝 Modelos de partnership (rápidos)

Modelo A — “Servicio completo” (vuelo + procesamiento):
Te acelera MVP. Riesgo: menos control sobre datos/protocolo.
Modelo B — “Vos procesás, ellos vuelan”:
Ideal para AGROPECUARIA: asegurás raws/metadatos y tu pipeline/IA hace el diferencial.
Modelo C — “Propio desde día 1”:
Comprar/operar drones ya en MVP suele distraer. Reservar para fase 2 (si el negocio lo justifica).
### 📌 Tip operativo

- Arrancar con 1–2 proveedores para pilotos controlados.
- Estandarizar un “protocolo de vuelo + entrega de datos”.
- Repetibilidad > variedad (al principio).


---
## 📦 80–100 GB

## 📦 80–100 GB por salida: diseño realista

✅ Necesario / conveniente
- **Upload reanudable** (multipart/chunked): si se corta, sigue.
- Subida al volver (Wi‑Fi/base). Diseño asíncrono (no “sincrónico”).
- Móvil consume resultados: zonas + tiles, no raws.
- Procesamiento en workers/jobs (colas), no en el request de API.
- Lifecycle de storage: conservar raws, versionar procesados, y limpiar caches.

⛔ Menos conveniente
- Intentar “subir 100 GB desde el campo” como flujo estándar.
- Descargar ortomosaico completo en el teléfono.
- Construir edge/tiempo real antes de validar ROI.

**Checklist de ingesta (cuando definamos infraestructura)**

- Multipart upload + reintentos
- Checksum/validación (evitar archivos corruptos)
- Metadata mínima por dataset (lote, fecha, sensor, GSD si aplica)
- Estado del job (queued/running/done/failed) visible en app

## 🔧 “Por qué se caen” proyectos con 100 GB

- Subidas no reanudables
- Procesamiento dentro de la API (timeouts)
- No hay estrategia de tiles (se intenta bajar el archivo gigante)
- No hay disciplina de metadata (datasets “sin identidad”)

Regla:
separar “archivo” (S3) de “estado/metadata” (DB).

---
## ✅ Stack final

✅ Stack final recomendado
## Lista corta (sin vueltas)

```

Mobile: React Native + TypeScript (Android + iOS) · Offline-first
API: FastAPI (Python)
Workers: jobs/colas (Celery/RQ o jobs containerizados)
DB geoespacial: PostgreSQL + PostGIS
Archivos: Object Storage (S3 / MinIO)
Ortomosaicos: COG + tiles on-demand (cache + pre-cache por lote si aplica)

```

Notas:
- Esto es compatible con un MVP “subo cuando vuelvo” y escala bien.
- Si más adelante se quiere tiempo real/edge: se evalúa ROS2 + C++/Python con hardware dedicado.

**Qué se puede posponer (para acelerar MVP)**

- Tiempo real en vuelo / edge computing
- Offline total de ortomosaicos completos
- Multi-cultivo y multi-región desde el día 1

Próximo paso natural: definir infraestructura (MVP vs escalable), costos, ingestión multipart, tile-serving y estrategia de observabilidad (logs/metrics).
## 📌 Qué validar en piloto

El usuario compra “acción”:
zonas + recomendación + verificación.
- ¿El productor usa más las **zonas** que el ortomosaico?
- ¿Qué zoom/inspección real necesita (tiles vs offline total)?
- ¿Tiempo de entrega 12–48h es suficiente?
- ¿Qué métricas ROI se pueden capturar sin fricción?


---
## ❓ FAQ / Decisiones

## ❓ FAQ / Decisiones (para alinear equipo)

**¿Necesitamos “mapas online pesados”?**

Probablemente necesitás **zoom sobre el ortomosaico**, pero no “pesado” como archivo descargado. La forma correcta es: **tiles on-demand** + cache local + pre-cache del lote si hace falta.

**¿Es obligatorio PostGIS?**

Para geometrías (lotes, zonas, prescripciones) y consultas espaciales, PostGIS es el estándar. Ahorra tiempo y evita inventar formatos propios.

**¿Python también para API? ¿No conviene TS?**

TS es válido, pero suma un runtime extra si el pipeline está en Python. Para MVP, Python end-to-end suele ser más rápido y más barato.

**¿Qué cambia por incluir iPhone?**

- Publicación: Apple Developer Program, certificados/provisioning.
- Caché/almacenamiento: cuidar límites y experiencia.
- Uploads en background: diseñar reintentos y continuidad.

**¿Qué necesitamos para definir infraestructura?**

- Frecuencia de vuelos por semana/mes
- SLA deseado (12h/24h/48h)
- Sensor (RGB/multi/termal), resolución y cantidad de fotos
- Cuántos lotes concurrentes
- Si habrá GPU (solo si modelos pesados lo justifican)

## 🧭 Guía de decisión rápida

Si el objetivo es ROI rápido:
Zonas livianas + recomendación accionable + tiles para lupa.
Si el objetivo es “wow visual”:
Sumás orto por tiles y un buen “storytelling” en app (sin meter GBs).
Si el objetivo es “tiempo real en vuelo”:
Eso es fase 2; no lo uses para definir MVP.

---
