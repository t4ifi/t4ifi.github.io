---
title: "Git Avanzado: Workflows Profesionales y Mejores Prácticas"
date: 2025-05-25 16:45:00 +0000
categories: [Control de Versiones, Git]
tags: [git, version-control, workflows, branching, collaboration, devops]
---

Git es mucho más que `add`, `commit` y `push`. Para trabajar profesionalmente en equipos de desarrollo, necesitas dominar workflows avanzados, estrategias de branching y técnicas de colaboración que marquen la diferencia entre un desarrollador junior y senior.

> 🎯 **Objetivo:** Dominar Git a nivel profesional, desde workflows complejos hasta técnicas avanzadas de debugging y colaboración en equipos grandes.

## 🧠 Fundamentos internos: ¿Cómo funciona Git realmente?

### El modelo de datos de Git: DAG (Directed Acyclic Graph)

Git no almacena archivos como "versiones", sino como un **grafo dirigido acíclico** de objetos:

```
Commit C ──→ Commit B ──→ Commit A
    │            │            │
    ↓            ↓            ↓
  Tree C      Tree B      Tree A
    │            │            │
    ↓            ↓            ↓
  Blobs       Blobs       Blobs
```

**Tipos de objetos en Git:**

1. **Blob**: Contenido de archivos (inmutable)
2. **Tree**: Estructura de directorios
3. **Commit**: Snapshot + metadata
4. **Tag**: Referencia a commits específicos

### Anatomía de un commit

```bash
# Ver estructura interna de un commit
git cat-file -p HEAD

# Resultado típico:
tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904
parent 1234567890abcdef1234567890abcdef12345678
author Juan Pérez <juan@ejemplo.com> 1683901234 +0200
committer Juan Pérez <juan@ejemplo.com> 1683901234 +0200

Mensaje del commit
```

**Hash SHA-1:** Cada objeto tiene un hash único basado en su contenido:
```bash
# Calcular hash de contenido
echo "contenido" | git hash-object --stdin
# Resultado: d95f3ad14dee633a758d2e331151e950dd13e4ed
```

### Referencias y el directorio .git

```
.git/
├── HEAD                    # Puntero a branch actual
├── refs/
│   ├── heads/             # Branches locales
│   │   ├── main
│   │   └── feature-x
│   ├── remotes/           # Branches remotos
│   │   └── origin/
│   └── tags/              # Tags
├── objects/               # Base de datos de objetos
│   ├── 12/
│   │   └── 34567890abcdef...
│   └── info/
├── config                 # Configuración del repo
├── index                  # Staging area
└── logs/                  # RefLog
```

## 🌿 Estrategias de Branching Profesionales

### Git Flow: El estándar de la industria

**Estructura de branches:**
```
main/master     ←── Producción (solo releases)
    ↑
develop         ←── Integración (base para features)
    ↑
feature/*       ←── Nuevas funcionalidades
    ↑
release/*       ←── Preparación de releases
    ↑
hotfix/*        ←── Fixes críticos para producción
```

**Implementación práctica:**

```bash
# Inicializar Git Flow
git flow init

# Feature branch
git flow feature start nueva-funcionalidad
# ... desarrollo ...
git flow feature finish nueva-funcionalidad

# Release branch
git flow release start 1.2.0
# ... preparación release (tests, docs) ...
git flow release finish 1.2.0

# Hotfix crítico
git flow hotfix start fix-critico
# ... fix ...
git flow hotfix finish fix-critico
```

### GitHub Flow: Simplicidad para CI/CD

**Workflow:**
```
main ←── Branch protegido con CI/CD
  ↑
feature-branch ←── Pull Request → Review → Merge
```

**Ventajas:**
- Más simple que Git Flow
- Ideal para deployment continuo
- Menos overhead de gestión

**Implementación:**
```bash
# Crear feature branch
git checkout -b feature/nueva-funcionalidad

# Desarrollo y commits
git add .
git commit -m "feat: implementar nueva funcionalidad"

# Push y crear Pull Request
git push origin feature/nueva-funcionalidad
# → Crear PR en GitHub
```

### GitLab Flow: Híbrido con environment branches

```
main ────→ staging ────→ production
  ↑           ↑             ↑
feature   pre-release   release-1.0
```

**Casos de uso:**
- Múltiples entornos (dev, staging, prod)
- Releases programados
- Testing extensivo entre stages

## 🔄 Workflows de Colaboración Avanzados

### Conventional Commits: Estandarización de mensajes

**Formato:**
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Tipos estándar:**
```bash
feat:     # Nueva funcionalidad
fix:      # Bug fix
docs:     # Documentación
style:    # Formato (no afecta lógica)
refactor: # Refactoring
test:     # Tests
chore:    # Mantenimiento
```

**Ejemplos:**
```bash
git commit -m "feat(auth): implementar login con OAuth2"
git commit -m "fix(api): corregir validación de email en registro"
git commit -m "docs: actualizar README con instrucciones de deployment"
```

**Herramientas de apoyo:**
```bash
# Commitizen: ayuda interactiva
npm install -g commitizen cz-conventional-changelog
git cz

# Commitlint: validación automática
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

### Semantic Versioning automatizado

```bash
# Conventional Changelog
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -s

# Standard Version
npm install -g standard-version
standard-version  # Bump version, genera changelog, crea tag
```

### Manejo de Pull Requests/Merge Requests

**Estrategias de merge:**

**1. Merge Commit (preserva historia):**
```bash
git checkout main
git merge --no-ff feature-branch
```

**2. Squash and Merge (historia limpia):**
```bash
git checkout main
git merge --squash feature-branch
git commit -m "feat: nueva funcionalidad completa"
```

**3. Rebase and Merge (historia lineal):**
```bash
git checkout feature-branch
git rebase main
git checkout main
git merge feature-branch  # Fast-forward
```

### Code Review: Mejores prácticas

**Configuración de protección de branches:**
```yaml
# .github/branch-protection.yml
protection_rules:
  main:
    required_status_checks:
      strict: true
      contexts: ["ci/tests", "ci/lint"]
    enforce_admins: true
    required_pull_request_reviews:
      required_approving_review_count: 2
      dismiss_stale_reviews: true
      require_code_owner_reviews: true
```

**CODEOWNERS file:**
```bash
# .github/CODEOWNERS
# Global owners
* @team-leads

# Frontend
/frontend/ @frontend-team

# Backend API
/api/ @backend-team

# Infrastructure
/docker/ @devops-team
/.github/ @devops-team

# Documentación
/docs/ @tech-writers @team-leads
```

## 🔧 Comandos Avanzados y Técnicas

### Interactive Rebase: Reescribir historia

```bash
# Rebase interactivo de últimos 3 commits
git rebase -i HEAD~3

# Opciones disponibles:
pick    # Usar commit tal como está
reword  # Cambiar mensaje de commit
edit    # Parar para hacer cambios
squash  # Combinar con commit anterior
fixup   # Como squash pero descarta mensaje
drop    # Eliminar commit
```

**Ejemplo práctico:**
```bash
# Historia inicial
c3 feat: finalizar funcionalidad
c2 fix: typo en variable
c1 feat: implementar base de funcionalidad

# Después de squash c2 en c1:
c2 feat: implementar funcionalidad completa
c1 feat: finalizar funcionalidad
```

### Git Bisect: Debugging automático

```bash
# Encontrar commit que introdujo bug
git bisect start
git bisect bad                # Commit actual tiene bug
git bisect good v1.0.0       # Versión conocida sin bug

# Git automáticamente checkoutea commits para probar
# Marcar cada commit:
git bisect good   # o git bisect bad

# Al final muestra el commit culpable
git bisect reset  # Volver al estado original
```

**Automatización con script:**
```bash
# test-script.sh
#!/bin/bash
make test
exit $?

# Bisect automático
git bisect run ./test-script.sh
```

### Git Hooks: Automatización de workflows

**Pre-commit hook (client-side):**
```bash
# .git/hooks/pre-commit
#!/bin/bash

# Ejecutar linting
npm run lint
if [ $? -ne 0 ]; then
    echo "Lint failed. Commit aborted."
    exit 1
fi

# Ejecutar tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi
```

**Pre-push hook:**
```bash
# .git/hooks/pre-push
#!/bin/bash

protected_branch='main'
current_branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')

if [ $protected_branch = $current_branch ]; then
    echo "Direct push to main branch is not allowed."
    exit 1
fi
```

**Server-side hooks (en repositorio bare):**
```bash
# hooks/post-receive
#!/bin/bash
# Deployment automático
cd /var/www/app
git --git-dir=/path/to/repo.git --work-tree=/var/www/app checkout -f main
npm install
npm run build
sudo systemctl restart app-service
```

### Git Reflog: Recuperar trabajo perdido

```bash
# Ver historial de referencias
git reflog

# Salida típica:
a1b2c3d HEAD@{0}: commit: nueva funcionalidad
d4e5f6g HEAD@{1}: rebase finished: returning to refs/heads/feature
h7i8j9k HEAD@{2}: rebase: fix bug

# Recuperar commit "perdido"
git checkout a1b2c3d
git checkout -b recover-work

# O resetear a punto anterior
git reset --hard HEAD@{1}
```

### Cherry-pick: Aplicar commits específicos

```bash
# Aplicar commit específico a branch actual
git cherry-pick abc123

# Aplicar múltiples commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick con conflictos
git cherry-pick abc123
# Resolver conflictos
git add .
git cherry-pick --continue

# Range de commits
git cherry-pick main~4..main~2
```

## 🛠️ Configuración Avanzada

### Configuración global optimizada

```bash
# Información personal
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Editor preferido
git config --global core.editor "vim"

# Merge tool
git config --global merge.tool vimdiff

# Configuraciones de push
git config --global push.default simple
git config --global push.followTags true

# Configuraciones de pull
git config --global pull.rebase true

# Colores
git config --global color.ui auto
git config --global color.branch auto
git config --global color.diff auto
git config --global color.status auto

# Aliases útiles
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg 'log --oneline --graph --decorate --all'
git config --global alias.amend 'commit --amend --no-edit'
```

### Aliases avanzados

```bash
# Log visual mejorado
git config --global alias.tree 'log --graph --pretty=format:"%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset" --abbrev-commit'

# Mostrar archivos modificados
git config --global alias.changed 'diff --name-only'

# Stash con mensaje
git config --global alias.save 'stash save'

# Branch cleanup
git config --global alias.cleanup '!git branch --merged | grep -v "\*\|main\|develop" | xargs -n 1 git branch -d'

# Find commits by message
git config --global alias.find 'log --all --grep'

# Show contributors
git config --global alias.contributors 'shortlog -sn'
```

### .gitignore patterns avanzados

```bash
# .gitignore

# Directorios
node_modules/
dist/
build/
.cache/

# Archivos del sistema
.DS_Store
Thumbs.db
*.swp
*.swo
*~

# Logs
*.log
logs/

# Configuración local
.env
.env.local
config.local.js

# IDE
.vscode/
.idea/
*.sublime-project
*.sublime-workspace

# Temporal
tmp/
temp/

# Archivos de backup
*.bak
*.orig

# Patrones específicos de lenguaje
__pycache__/
*.pyc
*.pyo
*.jar
*.class

# Patrones con excepciones
build/
!build/essential-file.txt

# Ignorar en subdirectorios pero no en raíz
/**/node_modules/
!/scripts/node_modules/
```

### Git Attributes: Control fino de archivos

```bash
# .gitattributes

# Normalización de line endings
* text=auto
*.sh text eol=lf
*.bat text eol=crlf

# Archivos binarios
*.png binary
*.jpg binary
*.gif binary
*.pdf binary

# Lenguajes específicos
*.js text
*.css text
*.html text
*.json text

# Diff drivers personalizados
*.md diff=markdown
*.json diff=json

# Merge strategies
*.generated merge=ours

# Export-ignore (no incluir en archives)
.gitignore export-ignore
.gitattributes export-ignore
tests/ export-ignore
docs/ export-ignore
```

## 🔍 Debugging y Análisis

### Git Log avanzado

```bash
# Log con estadísticas
git log --stat

# Log de archivos específicos
git log --follow -- archivo.txt

# Log con grep en commits
git log --grep="bug fix" --since="2 weeks ago"

# Log por autor
git log --author="Juan" --since="1 month ago"

# Log con formato personalizado
git log --pretty=format:"%h - %an, %ar : %s"

# Ver cambios introducidos por merge
git log --merges --first-parent

# Log de un range específico
git log main..feature-branch

# Commits que tocan líneas específicas
git log -L 15,20:archivo.txt
```

### Git Blame: Rastrear cambios línea por línea

```bash
# Blame básico
git blame archivo.txt

# Blame con rango de líneas
git blame -L 10,20 archivo.txt

# Ignorar commits de whitespace
git blame -w archivo.txt

# Mostrar email en lugar de nombre
git blame -e archivo.txt

# Blame de archivo en commit específico
git blame abc123 -- archivo.txt
```

### Git Show: Inspeccionar objetos

```bash
# Mostrar commit completo
git show abc123

# Solo estadísticas del commit
git show --stat abc123

# Solo nombres de archivos
git show --name-only abc123

# Archivo específico en commit
git show abc123:path/to/file.txt

# Tree de un commit
git show --pretty="" --name-only abc123
```

## 🚀 Técnicas Avanzadas de Colaboración

### Submódulos: Gestión de dependencias

```bash
# Añadir submódulo
git submodule add https://github.com/user/repo.git modules/dependency

# Inicializar submódulos después de clone
git submodule init
git submodule update

# O en un solo comando
git submodule update --init --recursive

# Actualizar submódulos
git submodule update --remote

# Actualizar submódulo específico
git submodule update --remote modules/dependency

# Remover submódulo
git submodule deinit modules/dependency
git rm modules/dependency
```

### Subtrees: Alternativa a submódulos

```bash
# Añadir subtree
git subtree add --prefix=lib/framework \
  https://github.com/user/framework.git main --squash

# Pull de cambios del subtree
git subtree pull --prefix=lib/framework \
  https://github.com/user/framework.git main --squash

# Push de cambios al subtree
git subtree push --prefix=lib/framework \
  https://github.com/user/framework.git feature-branch
```

### Worktrees: Múltiples branches simultáneos

```bash
# Crear worktree
git worktree add ../feature-workspace feature-branch

# Listar worktrees
git worktree list

# Remover worktree
git worktree remove ../feature-workspace

# Podar worktrees eliminados manualmente
git worktree prune
```

## 📊 Análisis y Estadísticas

### Análisis de contribuciones

```bash
# Estadísticas por autor
git shortlog -sn

# Líneas añadidas/eliminadas por autor
git log --format='%aN' | sort -u | while read name; do
  echo -en "$name\t"
  git log --author="$name" --pretty=tformat: --numstat | awk '
    { add += $1; subs += $2; loc += $1 - $2 }
    END { printf "added: %s, removed: %s, total: %s\n", add, subs, loc }'
done

# Commits por día de la semana
git log --date=format:'%w' --format='%ad' | sort | uniq -c

# Files más modificados
git log --name-only --pretty=format: | sort | uniq -c | sort -rg
```

### Herramientas de análisis

```bash
# git-quick-stats (externa)
git quick-stats

# git-fame (externa)
git-fame

# Análisis con awk
git log --pretty=format:"%an %ad" --date=short | \
  awk '{author[$1]++} END {for(i in author) print author[i], i}' | \
  sort -rn
```

## 🔒 Seguridad y Signing

### GPG Signing

```bash
# Generar clave GPG
gpg --gen-key

# Listar claves
gpg --list-secret-keys --keyid-format LONG

# Configurar Git
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# Firmar commit específico
git commit -S -m "commit firmado"

# Verificar firmas
git log --show-signature
```

### SSH Keys para autenticación

```bash
# Generar clave SSH
ssh-keygen -t ed25519 -C "tu@email.com"

# Añadir al ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Configurar múltiples claves
# ~/.ssh/config
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
```

## 🚨 Recuperación de Desastres

### Escenarios comunes y soluciones

**1. Commit en branch equivocado:**
```bash
# Mover commit a otro branch
git checkout correct-branch
git cherry-pick wrong-commit-hash
git checkout wrong-branch
git reset --hard HEAD~1
```

**2. Reset accidental:**
```bash
# Recuperar usando reflog
git reflog
git reset --hard HEAD@{n}
```

**3. Merge conflictivo:**
```bash
# Abortar merge
git merge --abort

# O resolver step by step
git status  # Ver archivos en conflicto
# Editar archivos manualmente
git add .
git commit
```

**4. Rebase interrumpido:**
```bash
# Continuar después de resolver
git add .
git rebase --continue

# Saltar commit problemático
git rebase --skip

# Abortar rebase
git rebase --abort
```

**5. Push force accidental:**
```bash
# Si tienes backup local
git push --force-with-lease origin main

# Recuperar desde otro clone
git fetch other-remote
git reset --hard other-remote/main
```

## 🎯 Casos de Uso Reales

### CI/CD con Git

**GitHub Actions example:**
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Run linting
      run: npm run lint
    
    - name: Build
      run: npm run build
    
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        # Deploy logic here
        echo "Deploying to staging..."
```

### Monorepo management

```bash
# Sparse checkout para monorepos grandes
git config core.sparseCheckout true
echo "frontend/*" > .git/info/sparse-checkout
echo "shared/*" >> .git/info/sparse-checkout
git read-tree -m -u HEAD

# Partial clone (Git 2.19+)
git clone --filter=blob:none <url>
git clone --filter=tree:0 <url>
```

---

## 🎓 Teoría Avanzada: Ciencias de la Computación en Git

### Teoría de Grafos Aplicada

Git implementa conceptos fundamentales de la teoría de grafos:

**Propiedades del DAG (Directed Acyclic Graph):**
- **Dirigido**: Las aristas tienen dirección (parent → child)
- **Acíclico**: No hay ciclos (evita paradojas temporales)
- **Conectado**: Todos los commits son alcanzables desde HEAD

**Algoritmos de Traversal:**
```
Depth-First Search (DFS):
git log --graph --oneline

Breadth-First Search (BFS):
git rev-list --ancestry-path A..B

Topological Sort:
git log --topo-order
```

**Merge vs Rebase en términos de grafos:**
```
# Merge - preserva estructura del grafo
    A---B---C feature
   /         \
  D---E---F---G main

# Rebase - lineariza el grafo  
  D---E---F---A'---B'---C' main
```

### Algoritmos de Diff y Merge

**Myers Diff Algorithm:**
Git usa el algoritmo de Myers para calcular diferencias entre archivos:

```python
# Concepto simplificado del algoritmo Myers
def myers_diff(old_text, new_text):
    # Crea una matriz de edición dinámica
    # Encuentra el camino más corto (Shortest Edit Script)
    # Complejidad: O((M+N)D) donde D = número de diferencias
    pass
```

**Three-Way Merge:**
```
Common Ancestor (Base)
        |
    ┌───┴───┐
  Ours    Theirs
    └───┬───┘
     Merged
```

**Estrategias de Merge:**
- **Recursive**: Default para dos branches
- **Octopus**: Para múltiples branches
- **Ours/Theirs**: Resolución automática de conflictos

### Distributed Version Control Theory

**Teorema CAP aplicado a Git:**
- **Consistency**: Eventual consistency a través de merge/rebase
- **Availability**: Cada clone es independiente y disponible
- **Partition Tolerance**: Funciona sin conectividad de red

**Vector Clocks en Git:**
```bash
# Cada commit tiene un "timestamp" lógico
git log --pretty=format:"%h %ai %s"

# Git usa Lamport timestamps implícitamente
# donde cada commit incrementa el clock lógico
```

### Cryptographic Hashing y Integridad

**SHA-1 → SHA-256 Migration:**
```bash
# Git está migrando de SHA-1 a SHA-256
git config --global init.defaultBranch main
git config --global extensions.objectFormat sha256

# Propiedades criptográficas:
# - Determinístico: mismo input = mismo hash
# - Avalanche effect: pequeño cambio = hash completamente diferente  
# - Collision resistance: computacionalmente imposible encontrar colisiones
```

**Content-Addressable Storage:**
```bash
# Git es una base de datos key-value
# Key = SHA hash, Value = contenido del objeto

echo "Hello World" | git hash-object --stdin
# 557db03de997c86a4a028e1ebd3a1ceb225be238

# Verificar integridad
git fsck --full
```

### Concurrency Control en Git

**Optimistic Concurrency Control:**
```bash
# Git asume que los conflictos son raros
# No hay locks - todos trabajan en paralelo
# Conflictos se resuelven en merge time

# Compare con databases:
# Pessimistic: Lock antes de modificar
# Optimistic: Verificar al commit
```

**Conflict Resolution Algorithms:**
```bash
# Conflict markers explican la resolución necesaria
<<<<<<< HEAD (current change)
int x = 1;
=======
int x = 2;
>>>>>>> branch-name (incoming change)

# Git no puede resolver automáticamente porque:
# 1. Ambos cambios tocan las mismas líneas
# 2. No hay información semántica suficiente
# 3. Requiere decisión humana
```

### Performance y Scaling

**Pack Files y Delta Compression:**
```bash
# Git comprime objetos similares usando deltas
git gc --aggressive

# Estructura interna:
# - Loose objects: archivos individuales en .git/objects/
# - Pack files: múltiples objetos comprimidos
# - Delta chains: objetos expresados como diffs de otros
```

**Shallow Clones y Partial Clones:**
```bash
# Shallow: limita historia
git clone --depth 1 <repo>

# Partial: excluye objetos grandes
git clone --filter=blob:limit=100M <repo>

# Sparse checkout: excluye directorios
git sparse-checkout init --cone
git sparse-checkout set src/
```

## 🎓 Conclusión

Dominar Git a nivel profesional requiere entender no solo comandos, sino workflows, estrategias de colaboración y herramientas de debugging. La teoría subyacente de ciencias de la computación - grafos, algoritmos de diff, hashing criptográfico, y control de concurrencia - es lo que permite a Git escalar desde proyectos personales hasta repositorios como el kernel de Linux con millones de commits.

Los puntos clave son:

1. **Entender la estructura interna** de Git (DAG, objetos, referencias)
2. **Elegir el workflow correcto** para tu equipo y proyecto
3. **Automatizar con hooks y CI/CD** para prevenir errores
4. **Usar convenciones estándar** (Conventional Commits, Semantic Versioning)
5. **Saber recuperarse** de situaciones problemáticas
6. **Comprender la teoría** para optimización y troubleshooting avanzado

Git no es solo una herramienta - es una implementación brillante de múltiples áreas de ciencias de la computación trabajando en conjunto para resolver el problema fundamental de la colaboración en software.

### Recursos adicionales recomendados

- **Pro Git Book** (gratuito): https://git-scm.com/book
- **Git Flow AVH**: https://github.com/petervanderdoes/gitflow-avh
- **Conventional Commits**: https://conventionalcommits.org/
- **GitKraken** o **Sourcetree**: GUIs para visualizar mejor
- **Git Internals** (Scott Chacon): Para entender la implementación
- **Think Like (a) Git**: Para comprender el modelo mental

**Escrito por Andrés Núñez**  