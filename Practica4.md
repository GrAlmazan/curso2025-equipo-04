# Práctica guiada — Sesión 4

## “Repositorio de equipo + Flujo Git (GitHub Flow / Git Flow) + Evidencias”

### 🎯 Objetivos

-   Crear un **repo de equipo** en GitHub (vía SSH).
    
-   Configurar **protecciones de rama** y **convenciones** (nombres, commits, PRs).
    
-   Aplicar **GitHub Flow** (simple) o **Git Flow** (estructurado) en una mini-feature real.
    
-   Dejar **evidencias**: PR con checklist, captura de ramas, diagrama de flujo, README.
    

### 👥 Roles recomendados (por equipo)

-   **Líder de repo:** crea repo, protege ramas, aprueba PRs.
    
-   **Dev A:** crea `feature/…`, abre PR.
    
-   **Dev B (reviewer):** revisa PR, comenta y aprueba.
    
-   **Dev C (opcional):** mantiene CI mínimo y documentación.
    

----------

## 0) Requisitos previos (10 min)

-   Llave SSH configurada y probada: `ssh -T git@github.com` → “Hi !”
    
-   Git configurado:
    

```bash
git config --global user.name  "Nombre Apellido"
git config --global user.email "tu_correo@ejemplo.com"
git config --global init.defaultBranch main

```

----------

## 1) Crear organización/repositorio del equipo (15 min)

> _Líder de repo_

1.  En GitHub, crea **Organization** (si no existe) o usa una existente.
    
2.  Crea un repo: `curso2025-equipo-XX` (privado o público).
    
    -   **Sin** README, **sin** .gitignore, **sin** license (lo agregaremos por CLI).
        
3.  Copia la URL SSH del repo: `git@github.com:ORG/curso2025-equipo-XX.git`
    

**Clonar e inicializar:**

```bash
git clone git@github.com:ORG/curso2025-equipo-XX.git
cd curso2025-equipo-XX

# README inicial
echo "# Curso 2025 – Equipo XX" > README.md
echo "Repositorio del equipo para prácticas de Git y flujo de trabajo." >> README.md

# .gitignore básico (dotnet + node + general)
cat > .gitignore << 'EOF'
# OS
.DS_Store
Thumbs.db

# Node
node_modules/
npm-debug.log
dist/

# .NET
bin/
obj/
*.user
*.suo

# Entornos
.env
.env.*
EOF

# Licencia y contribución
echo "MIT" > LICENSE
cat > CONTRIBUTING.md << 'EOF'
# Contribución
- Convención de ramas: feature/<tema>, hotfix/<tema>, release/<versión>
- Commits: tipo(scope): mensaje en imperativo
  Ej: feat(api): añade endpoint /health
- PRs: enlazar issue, checklist, 1 reviewer mínimo
EOF

```

**Primer commit y push:**

```bash
git add .
git commit -m "chore(repo): init README, .gitignore, LICENSE y CONTRIBUTING"
git push -u origin main

```
----------

# 2) Proteger ramas y definir reglas (paso a paso)

> Objetivo: impedir pushes directos a `main` (y `develop` si usas Git Flow), exigir PRs con revisión y, si quieres, checks de CI.

### ✅ Opción A — Desde la interfaz de GitHub (recomendado)

1.  Entra al repo → **Settings** (pestaña)
    
2.  Menú lateral → **Branches** → sección **Branch protection rules** → **Add rule**
    
3.  En **Branch name pattern** escribe:
    
    -   Para GitHub Flow: `main`
        
    -   Para Git Flow: primero `main`, luego harás otra regla para `develop`
        
4.  Marca estas casillas (recomendado mínimo):
    
    -   **Require a pull request before merging**
        
    -   **Require approvals**: 1 (o 2 si quieres más rigor)
        
    -   _(Opcional)_ **Dismiss stale pull request approvals when new commits are pushed**
        
    -   _(Opcional, si ya tienes CI)_ **Require status checks to pass before merging**
        
        -   Selecciona el check, por ejemplo “CI” si ya lo configuraste.
            
    -   _(Opcional)_ **Require conversation resolution before merging**
        
    -   _(Opcional, más estricto)_ **Restrict who can push to matching branches** (elige solo admins o un equipo de release)
        
    -   _(Opcional, limpieza de historial)_ **Require linear history**
        
5.  **Create** o **Save changes**
    

> Repite los pasos para `develop` si eliges **Git Flow**.

### 💻 Opción B — Por CLI (vía GitHub CLI)

> Requiere **gh** (GitHub CLI) autenticado: `gh auth login`

```bash
# Regla para main: PR + 1 review
gh api \
  -X PUT \
  repos/ORG/curso2025-equipo-XX/branches/main/protection \
  -f required_pull_request_reviews.dismiss_stale_reviews=true \
  -f required_pull_request_reviews.required_approving_review_count=1 \
  -f enforce_admins=true \
  -F restrictions='null' \
  -F required_status_checks='null'

```

> Para **develop**, crea primero la rama y sube (ver abajo en #3), luego replica el comando cambiando `main` por `develop`.

----------

# 3) Elegir el flujo: GitHub Flow vs Git Flow (con guía de decisión)

> Objetivo: que el equipo **elija conscientemente** el flujo y lo deje documentado.

### ¿Cuál uso?

-   **GitHub Flow (simple):**
    
    -   Equipos pequeños / APIs con despliegues frecuentes.
        
    -   Regla: todo parte de `main` + ramas `feature/*` → PR → merge.
        
    -   Menos ramas, más velocidad.
        
-   **Git Flow (estructurado):**
    
    -   Equipos con releases programados, QA formal.
        
    -   Ramas principales: `main` (producción) y `develop` (integración).
        
    -   Ramas de soporte: `feature/*`, `release/*`, `hotfix/*`.
        
    -   Más ceremonia, más control.
        

### Cómo **crear y preparar** `develop` (si eliges Git Flow)

```bash
# Partiendo de main
git checkout main
git pull

# Crear y subir develop
git branch develop
git push -u origin develop

# (Opcional) que todos configuren desarrollos a partir de develop:
# En local, trabajar así:
git checkout develop
git pull
git switch -c feature/tu-feature

```

### Proteger `develop` (igual que hiciste con `main`)

-   Repite **Settings → Branches → Add rule** para `develop`.
    
-   Misma configuración: PR obligatoria, 1 aprobación, (checks si tienes CI), linear history opcional.
    

### Documenta la elección en el README

Añade un bloque:

**Si GitHub Flow:**

```
Flujo elegido: GitHub Flow
- main protegido (solo merges por PR).
- ramas: feature/<tema>
- proceso: crear rama → commit/push → PR → review → merge (squash).

```

**Si Git Flow:**

```
Flujo elegido: Git Flow (resumido)
- ramas: main (prod), develop (integración)
- soporte: feature/<tema>, release/<versión>, hotfix/<bug>
- proceso: feature → develop → release → main (+tag) y back-merge a develop

```

----------

# 4) Convenciones del equipo (con ejemplos claros)

> Objetivo: que todo el equipo hable el mismo “idioma Git” y que GitHub aplique plantillas automáticamente.

### 4.1 Nombres de ramas (copiar/pegar al README)

-   **Features nuevas:** `feature/<tema-kebab-case>`
    
    -   Ej: `feature/api-health-endpoint`, `feature/ui-login-form`
        
-   **Bugs urgentes en producción:** `hotfix/<bug-descriptivo>`
    
    -   Ej: `hotfix/fix-nullref-on-auth`
        
-   **Versiones de release (solo Git Flow):** `release/<version>`
    
    -   Ej: `release/1.2.0`
        

> Reglas rápidas: minúsculas, sin espacios, usa `-` para separar palabras.

### 4.2 Convención de commits (Conventional Commits + ejemplos)

**Formato:**

```
<type>(<scope>): <mensaje en imperativo>

```

**Tipos comunes:**

-   `feat` (nueva funcionalidad)
    
-   `fix` (arreglo de bug)
    
-   `docs` (documentación)
    
-   `chore` (mantenimiento, ajustes no funcionales)
    
-   `refactor` (cambia estructura sin cambiar comportamiento)
    
-   `test` (añade o ajusta pruebas)
    
-   `build` (cambios en build, dependencias)
    
-   `ci` (cambios en pipelines)
    

**Scope** (opcional): `api`, `db`, `ui`, `infra`, `auth`, etc.

**Ejemplos buenos:**

```text
feat(api): añade endpoint /health
fix(auth): corrige validación de token expirado
docs(readme): agrega diagrama de flujo git
refactor(db): separa repositorio de consultas
ci(workflow): agrega job de build en PRs
chore(repo): inicializa .gitignore y license

```

**Ejemplos a evitar:**

```text
update files
fix stuff
final version

```

### 4.3 Plantilla de Pull Request (auto-aplicada)

Crea **.github/pull_request_template.md** para que GitHub la inserte cada vez:

```bash
mkdir -p .github
cat > .github/pull_request_template.md << 'EOF'
## Resumen
- ¿Qué hace este PR?

## Checklist
- [ ] Compila / pasa tests
- [ ] Sigue convención de commits
- [ ] Actualiza README/Docs si aplica
- [ ] Enlaza issue (close #N)

## Notas
- Riesgos, alcance y pruebas realizadas
EOF

git add .github/pull_request_template.md
git commit -m "docs: añade PR template"
git push

```

> Tip: añade un **ISSUE_TEMPLATE** si quieres forzar un formato para issues.

### 4.4 (Opcional) Enforzar estilo de commits y hooks

-   **Commitlint + Husky** (para proyectos Node) o **pre-commit** (multi-lenguaje).
    
-   Objetivo: bloquear commits que no cumplan el formato.
    

Ejemplo rápido con **pre-commit** (Python, multi-propósito):

```bash
# Instalar herramienta (requiere python/pip)
pip install pre-commit

# Config mínima: valida EOF y fin de línea (puedes integrar linters)
cat > .pre-commit-config.yaml << 'EOF'
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.6.0
  hooks:
    - id: end-of-file-fixer
    - id: trailing-whitespace
EOF

pre-commit install
git add .pre-commit-config.yaml
git commit -m "chore(pre-commit): instala hooks básicos"
git push

```

> Para **Conventional Commits** estrictos, considera `commitlint` (Node) o reglas equivalentes en tu stack.

### 4.5 (Opcional) CODEOWNERS para revisores automáticos

```bash
cat > .github/CODEOWNERS << 'EOF'
# Cambios en /src requieren revisión del líder
/src/ @lider-repo
EOF

git add .github/CODEOWNERS
git commit -m "chore: CODEOWNERS para /src"
git push

```

Con esto, GitHub asignará automáticamente reviewers para rutas específicas.

----------

## Mini-checklist rápido (para ti y los alumnos)

-   `main` protegido (PRs + 1 aprobación + (opcional) linear history + checks)
    
-   (Git Flow) `develop` creada y protegida
    
-   README con flujo elegido + convenciones (ramas, commits)
    
-   PR template en `.github/pull_request_template.md`
    
-   (Opcional) CODEOWNERS y hooks
    

----------

## 5) **Mini-feature** guiada (≈60–80 min)

> _Dev A hace la feature; Dev B revisa; Líder aprueba/mergea._

### 5.1 Crear issue (opcional pero recomendado)

-   En **Issues**, crea: “Añadir endpoint /health (o script de ejemplo)”.
    
-   Asigna a **Dev A**, etiqueta `feature`.
    

### 5.2 Crear rama y desarrollar

```bash
# Asegúrate de estar actualizado
git checkout main
git pull

# Crea rama de feature
git switch -c feature/health-endpoint

```

**Ejemplo A (si el repo es .NET):**

```bash
mkdir -p src/Api
cd src/Api
dotnet new webapi -n Api
cd Api
# Agrega endpoint simple en Program.cs:
# app.MapGet("/health", () => Results.Ok(new { status = "ok", at = DateTime.UtcNow }));

```

**Ejemplo B (si el repo no tiene código aún):**

```bash
mkdir scripts
cat > scripts/health.sh << 'EOF'
#!/usr/bin/env bash
echo "{ \"status\": \"ok\", \"at\": \"$(date -u +%FT%TZ)\" }"
EOF
chmod +x scripts/health.sh

```

**Commit y push:**

```bash
git add .
git commit -m "feat(health): agrega endpoint/script de verificación"
git push -u origin feature/health-endpoint

```

### 5.3 Abrir Pull Request

-   En GitHub, abre PR → base: `main` (o `develop` si eligieron Git Flow) ← compare: `feature/health-endpoint`.
    
-   Usa el **template**, enlaza el issue (ej. `closes #1`).
    
-   Asigna **reviewer: Dev B** y **required reviewers** según reglas.
    

### 5.4 Revisión (Dev B)

-   Comenta sugerencias (nombres, mensajes, README).
    
-   Pide añadir test o script de verificación.
    

**Ejemplo test muy simple (opcional):**

```bash
mkdir -p tests
cat > tests/health.test.md << 'EOF'
# Prueba manual
- Ejecutar scripts/health.sh
- Esperar JSON con status "ok" y fecha UTC
EOF

git add tests/health.test.md
git commit -m "test(health): añade guía de prueba manual"
git push

```

### 5.5 Resolver conflictos (si los hay)

```bash
git fetch origin
git checkout feature/health-endpoint
git merge origin/main   # o origin/develop
# Resolver archivos con conflictos, luego:
git add <archivos>
git commit
git push

```

### 5.6 Merge

-   Cuando la PR esté aprobada, **merge** con:
    
    -   **Squash and merge** (recomendado para mantener historial limpio)  
        o
        
    -   **Rebase and merge** (si lo dominan).
        
-   **Eliminar** rama de `feature` en GitHub y local:
    

```bash
git branch -d feature/health-endpoint
git fetch -p

```

----------

## 6) (Opcional) CI mínimo (10–15 min)

> Pueden añadir un workflow muy simple para validar el repo.

```bash
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: CI
on:
  pull_request:
    branches: [ "main", "develop" ]
  push:
    branches: [ "main" ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "CI OK - $(date)"
EOF

git add .github/workflows/ci.yml
git commit -m "ci: workflow mínimo de validación"
git push

```

-   Vuelve a abrir una PR y verifica que el **status check** pase.
    

----------

## 7) Evidencias obligatorias (para entregar)

1.  **Capturas:**
    
    -   Reglas de protección de `main` (y `develop` si aplicó).
        
    -   PR abierta con checklist + reviewer asignado.
        
    -   Historial de commits con la convención (screenshot `git log --oneline --graph` o Graph en GitHub).
        
2.  **URLs:**
    
    -   Enlace al repo del equipo.
        
    -   Enlace a la PR mergeada.
        
3.  **README actualizado** con:
    
    -   Convención de ramas y commits.
        
    -   Diagrama del flujo elegido.
        
4.  **Diagrama** (ASCII o imagen) del flujo aplicado.
    

**Ejemplo diagrama (GitHub Flow):**

```
main ──●──────────●──────────●───────────▶
         \         \          \
          \         \          \── PR ── merge
           \         \── PR ── merge
            \── PR ── merge
           feature/* branches

```

**Ejemplo diagrama (Git Flow):**

```
main ────────●─────────────●────────────▶
              \             \
develop  ──────●──●──●──●───●──●──●──────▶
                 \      \
               feature/*  \__ release/*
                           \__ hotfix/*

```

----------

## 8) Rúbrica de evaluación (100 pts)

-   **Protecciones de rama (main/develop) configuradas** — 20 pts
    
-   **Ramas y convención de commits correctas** — 20 pts
    
-   **PR con template + checklist + review** — 25 pts
    
-   **Diagrama y README claros (flujo elegido)** — 20 pts
    
-   **Ejecución mini-feature (endpoint/script + test básico)** — 15 pts
    

----------

## 9) Troubleshooting (rápidos)

-   **`Permission denied (publickey)`**
    
    -   Verifica llave en GitHub (Settings → SSH keys).
        
    -   Permisos: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/id_ed25519`
        
    -   `ssh -T git@github.com` debe saludar.
        
-   **No puedo push a `main`** (protegida)
    
    -   Correcto: haz PR desde rama `feature/*`.
        
-   **Conflictos en merge**
    
    -   `git fetch`, `git merge origin/main` → resuelve, `git add`, `git commit`, `git push`.
        
-   **El CI no corre**
    
    -   Revisa nombre de rama en `on:` y que el archivo esté en `.github/workflows/`.
        

----------

## 10) Extensiones (si te da tiempo)

-   Añadir **CODEOWNERS** para que ciertas carpetas requieran reviewers específicos:
    

```bash
cat > .github/CODEOWNERS << 'EOF'
# Todos los cambios en /src requieren revisión del líder
/src/ @lider-repo
EOF
git add .github/CODEOWNERS
git commit -m "chore: CODEOWNERS para /src"
git push

```

-   Crear **tag** y **release**:
    

```bash
git tag -a v0.1.0 -m "Primera versión de práctica"
git push origin v0.1.0

```

----------

### ✅ Cierre (qué deben subir)

-   Enlace al repo del equipo y a la PR final.
    
-   Capturas solicitadas (protección ramas, PR, historial).
    
-   README con convención y diagrama.
    
-   Nota breve (3–5 puntos) de _lecciones aprendidas_ por el equipo.
    

Si quieres, te lo convierto en **PPTX + handout** (instructor & alumno) con los pasos resumidos, checklist imprimible y hojas para evidencias.