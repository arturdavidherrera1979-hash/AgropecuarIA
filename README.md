# AGROPECUARIA — Documentación (ex-AGROPECUARIA)

Esta carpeta (`/docs`) es la **fuente de verdad** para decisiones, supuestos, arquitectura, roadmap y definición del MVP.

> Nota de naming: el repositorio fue renombrado a **AGROPECUARIA**, pero varios archivos todavía conservan el prefijo **AGROPECUARIA**. Mantener consistencia en links y títulos; si se consolida el rename, renombrar archivos gradualmente.

---

## Inicio rápido (rehidratación de un chat nuevo)

1) Conectar GitHub con la cuenta correcta: **`arturdavidherrera1979-hash`**  
2) En el chat: `@github` → seleccionar repo **`arturdavidherrera1979-hash/AgropecuarIA`** (branch `main`)  
3) Leer en este orden:
- 🤖 **Bootstrap principal**: `docs/AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md`
- 🧭 Visión: `docs/00_vision.md`
- 🗺 Roadmap: `docs/10_roadmap.md`
- 🧱 Arquitectura: `docs/20_architecture.md`
- 🧬 Modelo de datos: `docs/30_data_model.md`
- ✅ MVP: `docs/40_mvp_definition.md`
- 🧠 Perfil IA: `docs/AGROPECUARIA_AI_PROFILE.md`

---

## Índice de documentación (árbol completo de `/docs`)

### Núcleo del proyecto
- `docs/README.md` (este archivo)
- `docs/00_vision.md`
- `docs/10_roadmap.md`
- `docs/20_architecture.md`
- `docs/30_data_model.md`
- `docs/40_mvp_definition.md`

### AI / Operación de chats
- `docs/AGROPECUARIA_AI_PROFILE.md`
- `docs/AGROPECUARIA_AI_BOOTSTRAP.md`
- `docs/AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md`  ← **usar este como principal**

### Guías del repo
- `docs/AGROPECUARIA_guia_repo.md`
- `docs/AGROPECUARIA_guia_estructura_repo_v2.html` (guía navegable)

### Investigación y materiales de soporte (histórico útil)
- `docs/AGROPECUARIA_GUIA_TECNOLOGIAS.md`
- `docs/AGROPECUARIA_INFRA_COSTOS_BIG6.md`
- `docs/AGROPECUARIA_MODELO_NEGOCIO_PRICING_TABS.html`

---

## Meetings (minutas y material de reuniones)

Carpeta: `/meetings`

- `meetings/template.md` (plantilla)
- `meetings/minuta_reunion_11-12-2025.html`
- `meetings/reunion_inicial.txt`

Recomendación: cuando haya una minuta nueva, subirla y **linkearla desde el issue** relacionado (ej. `OPS-001`, `MVP-001`).

---

## Estructura del repositorio (snapshot)

> Este snapshot resume el árbol actual del repo para ubicar rápido dónde está cada cosa.

```
AgropecuarIA/
├─ README.md
├─ Dockerfile
├─ requirements.txt
├─ .github/
│  └─ workflows/
│     └─ ci.yml
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
├─ meetings/
│  ├─ template.md
│  ├─ reunion_inicial.txt
│  └─ minuta_reunion_11-12-2025.html
├─ business/
│  └─ business_model_canvas.md
├─ commercial/
│  ├─ pricing.md
│  └─ go_to_market.md
├─ hardware/
│  ├─ drone_platforms.md
│  └─ sensors_research.md
├─ src/
│  └─ dronia_core/
│     ├─ __init__.py
│     └─ main.py
└─ tests/
   └─ test_placeholder.py
```

---

## Convenciones (para mantener el repo usable)

- Docs **cortos**, editables y accionables.
- Preferir **checklists** + links a Issues/PRs.
- Decisiones no obvias → agregar una mini sección “Decision log” al doc correspondiente (fecha + motivo + tradeoff).
- Evitar duplicación: si existe un “UPDATED”, el viejo queda como referencia o redirect.

---

## Pendientes recomendados (consistencia)

- Consolidar naming: decidir si se renombra gradualmente `DRONIA_*` a `AGROPECUARIA_*` o se mantiene por compatibilidad.
- Linkear desde `README.md` (raíz) hacia:
  - `docs/AGROPECUARIA_AI_BOOTSTRAP_UPDATED.md`
  - `docs/README.md`
  - `meetings/minuta_reunion_11-12-2025.html`
