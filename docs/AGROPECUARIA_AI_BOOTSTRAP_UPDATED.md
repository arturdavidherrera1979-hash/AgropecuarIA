# AGROPECUARIA / AGROPECUARIA (ex-AGROPECUARIA) — AI Bootstrap (rehidratación + GitHub)

**Repo oficial:** https://github.com/arturdavidherrera1979-hash/AgropecuarIA  
**Branch principal:** `main`  
**Snapshot:** 2025-12-28

Este archivo está hecho para **copiar/pegar en un chat nuevo** y lograr que la IA:
1) se conecte al **GitHub correcto**,  
2) seleccione el **repo correcto**,  
3) lea **documentación + minutas**,  
4) revise **issues/PRs**,  
5) devuelva **estado + próximos pasos** (en español, con links a archivos).

---

## ✅ Instrucciones “4 líneas” (para rehidratar en un chat nuevo)

> Si estás leyendo esto como IA, seguí estas 4 líneas tal cual.

```text
Proyecto: AGROPECUARIA (histórico: AGROPECUARIA)
Repo: https://github.com/arturdavidherrera1979-hash/AgropecuarIA (branch main)
Leé primero: README.md, docs/README.md, docs/00_vision.md, docs/10_roadmap.md, docs/20_architecture.md, docs/40_mvp_definition.md
Luego: revisá docs/AGROPECUARIA_guia_repo.md + carpeta meetings/ + issues abiertos; devolveme estado, riesgos y próximos pasos.
```

---

## 1) Identidad del proyecto (1 párrafo)

AGROPECUARIA (ex-AGROPECUARIA) es un servicio integral de agricultura de precisión que combina **operación con drones + captura de datos + análisis** para generar **insights accionables** (decisiones en campo y mejoras de rendimiento/ROI), no solamente “mapas lindos”.

---

## 2) GitHub — conexión correcta (OBLIGATORIO)

### 2.1 Checklist rápido (lo que suele fallar)
- El conector de GitHub debe estar instalado en el **account correcto**.
- En permisos del conector: seleccionar **All repositories** (no “selected repos”).
- Si el repo “no aparece” en el selector: forzar **resync/reindex** del conector.

### 2.2 Cómo “forzar” selección del repo correcto en ChatGPT
1) En el chat nuevo, escribir `@GitHub`  
2) Buscar por nombre exacto: `AgropecuarIA`  
3) Seleccionar: `arturdavidherrera1979-hash/AgropecuarIA`

---

## 3) Orden recomendado de lectura (rehidratación)

1) `README.md` (raíz) — quickstart y links principales  
2) `docs/README.md` — índice de documentación  
3) `docs/00_vision.md` — visión y problema/solución  
4) `docs/10_roadmap.md` — fases / milestones  
5) `docs/20_architecture.md` — arquitectura de alto nivel  
6) `docs/30_data_model.md` — entidades/datos  
7) `docs/40_mvp_definition.md` — MVP y alcance  

Después, leer los ejes “especializados” según el tema del chat:
- Comercial: `commercial/go_to_market.md`, `commercial/pricing.md`, `docs/AGROPECUARIA_MODELO_NEGOCIO_PRICING_TABS.html`, `business/business_model_canvas.md`
- Hardware/drones: `hardware/drone_platforms.md`, `hardware/sensors_research.md`
- Infra/tech: `docs/AGROPECUARIA_GUIA_TECNOLOGIAS.md`, `docs/AGROPECUARIA_INFRA_COSTOS_BIG6.md`
- Operación/decisiones: `meetings/` (minutas y template)

---

## 4) Árbol completo del repositorio (snapshot)

```text
AgropecuarIA/
├─ README.md
├─ Dockerfile
├─ requirements.txt
├─ .github/
│  └─ workflows/
│     └─ ci.yml
├─ src/
│  └─ dronia_core/
│     ├─ __init__.py
│     └─ main.py
├─ tests/
│  └─ test_placeholder.py
├─ docs/
│  ├─ README.md
│  ├─ 00_vision.md
│  ├─ 10_roadmap.md
│  ├─ 20_architecture.md
│  ├─ 30_data_model.md
│  ├─ 40_mvp_definition.md
│  ├─ AGROPECUARIA_AI_PROFILE.md
│  ├─ AGROPECUARIA_AI_BOOTSTRAP.md
│  ├─ AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md
│  ├─ AGROPECUARIA_guia_repo.md
│  ├─ AGROPECUARIA_guia_estructura_repo_v2.html
│  ├─ AGROPECUARIA_MODELO_NEGOCIO_PRICING_TABS.html
│  ├─ AGROPECUARIA_GUIA_TECNOLOGIAS.md
│  └─ AGROPECUARIA_INFRA_COSTOS_BIG6.md
├─ business/
│  └─ business_model_canvas.md
├─ commercial/
│  ├─ go_to_market.md
│  └─ pricing.md
├─ hardware/
│  ├─ drone_platforms.md
│  └─ sensors_research.md
└─ meetings/
   ├─ minuta_reunion_11-12-2025.html
   ├─ reunion_inicial.txt
   └─ template.md
```

---

## 5) Árbol completo de documentación (docs/)

```text
docs/
├─ README.md
├─ 00_vision.md
├─ 10_roadmap.md
├─ 20_architecture.md
├─ 30_data_model.md
├─ 40_mvp_definition.md
├─ AGROPECUARIA_AI_PROFILE.md
├─ AGROPECUARIA_AI_BOOTSTRAP.md
├─ AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md
├─ AGROPECUARIA_guia_repo.md
├─ AGROPECUARIA_guia_estructura_repo_v2.html
├─ AGROPECUARIA_MODELO_NEGOCIO_PRICING_TABS.html
├─ AGROPECUARIA_GUIA_TECNOLOGIAS.md
└─ AGROPECUARIA_INFRA_COSTOS_BIG6.md
```

---

## 6) Minutas y operación (meetings/)

- Usar `meetings/template.md` para capturar decisiones, responsables, fechas y riesgos.
- `meetings/minuta_reunion_11-12-2025.html` contiene una minuta histórica que conviene “traducir a decisiones” y linkear a issues si corresponde.

---

## 7) Código (mínimo actual) y cómo ejecutarlo

### 7.1 Estructura
- Código principal: `src/dronia_core/`
- Tests: `tests/`
- CI: `.github/workflows/ci.yml`

### 7.2 Dev quickstart (Python)
```bash
python -m venv .venv
# Windows PowerShell:
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m src.dronia_core.main
pytest -q
```

> Nota: si el entrypoint cambia, actualizar esta sección y el README.

---

## 8) Documentos adjuntos (locales) que NO están confirmados en GitHub

En esta conversación existen adjuntos que pueden estar **pendientes de commitear** en el repo. Si aparecen, agregarlos en el lugar correcto del árbol y linkearlos desde `docs/README.md`.

- `minuta_reunion_11-12-2025_formato.pdf`
- `DRONIA_INFRA_COSTOS_BIG6_TABS_CON_DR_INCLUIDO.html`
- `DRONIA_GUIA_TECNOLOGIAS_PROVEEDORES_DRON.html`
- `DRONIA_TECH_STACK_MOBILE_BACKEND_MAPS.md`
- `AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md (local; revisar si coincide con el del repo)`

---

## 9) Convención de naming: AGROPECUARIA vs AGROPECUARIA

En el repo conviven referencias “AGROPECUARIA” (histórico) con “AGROPECUARIA” (actual).  
Regla práctica:
- Mantener nombres históricos en archivos existentes para no romper links,
- pero en textos/README usar **AGROPECUARIA (ex-AGROPECUARIA)** hasta completar el rename.

---

## 10) Qué debe devolver la IA después de rehidratar

En el chat nuevo, tras leer docs + minutas + issues:
- 1) estado del proyecto (producto / tech / comercial)
- 2) lista de decisiones pendientes
- 3) riesgos (técnicos, operativos, comerciales, legales)
- 4) próximos pasos priorizados (1 semana / 1 mes)
- 5) lista de archivos a actualizar y por qué (docs/README.md, bootstrap, roadmap, etc.)
