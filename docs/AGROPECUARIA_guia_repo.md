# AGROPECUARIA – Guía del repositorio y estructura

**Repo:** https://github.com/arturdavidherrera1979-hash/dronia  
**Actualizado:** 2025-12-26

> Nota: GitHub Pages está deshabilitado para repos privados sin upgrade, así que la forma más simple de que *se vea bien en GitHub* es mantener esta guía en **Markdown** (este archivo).  
> El HTML puede quedar como anexo descargable.

---

## 0) Acceso al repositorio (para nuevos colaboradores)

**Link oficial:** https://github.com/arturdavidherrera1979-hash/dronia

**Paso previo:** aceptar la invitación a colaborar en GitHub.  
Si no aceptan, en repos privados suele aparecer como “no existe” / “not found”.

### ¿Necesitan instalar algo?
- **Solo para leer / revisar:** No. Con navegador alcanza (**GitHub Web**).
- **Para editar docs:** Se puede desde la web (ideal: crear Pull Request).
- **Para trabajar con código:** Recomendado instalar **GitHub Desktop** o **Git (CLI)**.

### Opción A: GitHub Web (sin instalar nada)
1. Entrar al repo.
2. Navegar carpetas, Issues y Pull Requests.
3. Para cambios chicos en docs: editar → commit → (mejor) Pull Request.

### Opción B: GitHub Desktop (recomendado para no técnicos)
1. Instalar GitHub Desktop.
2. Iniciar sesión con la cuenta de GitHub.
3. `File → Clone repository…` → elegir `dronia` → seleccionar carpeta local.
4. Editar archivos → commit → push → Pull Request.

### Opción C: Git (CLI) para devs
1. Instalar Git for Windows.
2. Clonar repo y trabajar con ramas.
3. Si usan GitHub CLI: `gh auth setup-git` ayuda a configurar credenciales.

---

## 1) ¿Para qué sirve este repositorio?

El repositorio **AGROPECUARIA** es el “centro de gravedad” del proyecto: acá viven el código, la documentación y los artefactos de gestión (issues, plantillas, minutas).  
La idea es que cualquiera pueda entrar, entender el estado y contribuir sin depender de memoria oral.

**Regla simple**
- **Todo lo que se decide** (técnico/comercial/operativo) debe quedar reflejado en `docs/` y/o en un Issue.
- **Todo lo que se ejecuta** (código/pipeline) vive en `src/` y se prueba desde `tests/`.
- **Todo lo que se repite** (plantillas, SOPs) debe quedar en el repo.

---

## 2) Mapa rápido de la estructura

```text
dronia/
├─ .github/
│  ├─ workflows/
│  │  └─ ci.yml
│  ├─ ISSUE_TEMPLATE/
│  │  ├─ business_model.md
│  │  ├─ tech_research.md
│  │  └─ meeting_note_template.md
│  └─ PULL_REQUEST_TEMPLATE.md
├─ docs/
├─ business/
├─ commercial/
├─ hardware/
├─ meetings/
├─ src/
│  └─ dronia_core/
├─ tests/
├─ Dockerfile
├─ requirements.txt
├─ README.md
├─ LICENSE
└─ .gitignore
```

---

## 3) ¿Para qué sirve cada carpeta?

| Ruta | Propósito | ¿Quién la usa más? | Contenido típico |
|---|---|---|---|
| `.github/` | Automatización y estándares (CI, plantillas, PRs). | Todo el equipo | Actions, templates de Issues/PRs |
| `.github/workflows/` | CI (integración continua) en pushes/PRs. | Dev/QA | `ci.yml` |
| `.github/ISSUE_TEMPLATE/` | Plantillas de Issues consistentes. | PM/Equipo | Formatos de negocio, research, minutas |
| `docs/` | Documentación viva: visión, roadmap, definiciones. | Todo el equipo | Visión, roadmap, data model, MVP |
| `business/` | Modelo de negocio (Canvas, hipótesis, ROI). | Negocio | BMC, segmentación, unit economics |
| `commercial/` | Go-to-market y pricing. | Comercial | GTM, precios, mensajes, objeciones |
| `hardware/` | Investigación drones/sensores (stack operativo). | Técnico/Operaciones | comparativas, criterios, costos |
| `meetings/` | Minutas y plantillas de reuniones. | Todo el equipo | template + minutas por fecha |
| `src/` | Código fuente (ejecutable). | Dev | módulos, pipeline |
| `src/dronia_core/` | Núcleo del proyecto (lógica principal). | Dev | entrypoint + utilidades |
| `tests/` | Tests automatizados. | Dev/QA | unit/integration tests |

---

## 4) Archivos importantes

| Archivo | Para qué sirve |
|---|---|
| `README.md` | Portada del repo: qué es AGROPECUARIA y cómo arrancar |
| `requirements.txt` | Dependencias Python |
| `Dockerfile` | Ejecución consistente (contenedor) |
| `.gitignore` | Evita subir basura (venv, cache, logs) |
| `LICENSE` | Términos de uso del código |
| `.github/PULL_REQUEST_TEMPLATE.md` | Checklist para PRs |
| `.github/workflows/ci.yml` | CI que valida el repo (si falla, alarma útil) |

---

## 5) Cómo contribuir sin romper nada

- **Docs:** editar/crear en `docs/`, `business/`, `commercial/`, `hardware/` y vincular a un Issue.
- **Código:** trabajar en ramas y abrir PRs. Mantener CI verde.
- **Minutas:** cada reunión deja minuta en `meetings/` y actualiza decisiones en `docs/`.
