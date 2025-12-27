# DRONIA – AI Bootstrap (contexto para rehidratar en un chat nuevo)

**Repo:** https://github.com/arturdavidherrera1979-hash/dronia  
**Branch principal:** `main`  
**Actualizado:** 2025-12-26

## ✅ Instrucciones “4 líneas” (para rehidratar en un chat nuevo)

> Si estás leyendo este archivo como IA, seguí estas 4 líneas tal cual.

```text
Proyecto: DRONIA
Repo: https://github.com/arturdavidherrera1979-hash/dronia (branch main)
Leé: docs/AI_BOOTSTRAP.md, docs/guia_repo.md, docs/00_vision.md, docs/10_roadmap.md, docs/40_mvp_definition.md
Luego: revisá issues abiertos y resumime estado + próximos pasos.
```

---

## Objetivo de este archivo
Este documento existe para que cualquier asistente (humano o IA) pueda “cargar contexto” rápido:
- qué es DRONIA
- dónde está cada cosa
- cuál es el estado actual y los próximos pasos
- convenciones del repo

> Regla: si esto queda desactualizado, el equipo pierde tiempo. Actualizalo cada vez que cambie algo importante.

---

## 1) Qué es DRONIA (1 párrafo)
Servicio integral de agricultura de precisión que combina operación con drones/sensores + analítica/IA para aumentar la rentabilidad por hectárea (reducir costos y/o aumentar productividad).

---

## 2) Dónde leer primero (orden recomendado)
1. `README.md` (raíz del repo)
2. `docs/guia_repo.md` (guía de acceso + estructura)
3. `docs/00_vision.md`
4. `docs/10_roadmap.md`
5. `docs/40_mvp_definition.md`
6. Issues en GitHub (filtrar por labels: `business`, `tech`, `mvp`, `legal`)

---

## 3) Estructura (para qué sirve cada carpeta)
- `.github/`: workflows (CI) + templates (issues/PRs)
- `docs/`: documentación viva (visión, roadmap, definiciones)
- `business/`: Business Model Canvas y material de negocio
- `commercial/`: GTM y pricing
- `hardware/`: investigación drones/sensores
- `meetings/`: minutas (template + minutas por fecha)
- `src/`: código
- `tests/`: pruebas

---

## 4) Estado actual (llenar y mantener)
**Salud general:** 🟢 / 🟡 / 🔴 (elegir)

**Qué está funcionando hoy**
- (ej) repo inicial creado, CI activo, issues base creados, docs iniciales subidos.

**Qué está en progreso**
- (ej) BM-001, TECH-001, COMM-001, MVP-001…

**Riesgos / bloqueos**
- (ej) acceso a campos/pilotos, permisos, disponibilidad de sensores, etc.

**Próximos 7 días (prioridades)**
1. …
2. …
3. …

---

## 5) Convenciones operativas
- Decisiones relevantes → se reflejan en `docs/` (y se vinculan a un Issue).
- Cambios a código → en branch + PR (mantener CI verde).
- Reuniones → minuta en `meetings/` y acciones como Issues.

---

## 6) Mensaje de “arranque” para un chat nuevo (copiar/pegar)
Pegar esto cuando abras un chat nuevo para que la IA se auto-ubique:

```text
Proyecto: DRONIA
Repo: https://github.com/arturdavidherrera1979-hash/dronia (branch main)
Leé: docs/AI_BOOTSTRAP.md, docs/guia_repo.md, docs/00_vision.md, docs/10_roadmap.md, docs/40_mvp_definition.md
Luego: revisá issues abiertos y resumime estado + próximos pasos.
```

---

## 7) Mantenimiento (checklist)
- [ ] Actualizar sección “Estado actual” al menos 1 vez por semana.
- [ ] Cuando se cierre un hito, registrar qué cambió y enlazar PR/Issue.
- [ ] Si cambia la estructura del repo, actualizar sección 3.
