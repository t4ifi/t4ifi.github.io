---
title: "Git desde Cero: Guía Completa para Principiantes"
date: 2025-05-15 10:30:00 +0000
categories: [Control de Versiones, Git]
tags: [git, principiantes, tutorial, version-control, github, comandos-basicos]
---

¿Nunca has usado Git? ¿Te da miedo romper algo? Esta guía te enseñará Git desde los conceptos más básicos hasta que puedas trabajar con confianza en cualquier proyecto. No necesitas conocimientos previos.

> 🎯 **Objetivo:** Aprender Git desde cero de forma práctica y sin miedo, desde la instalación hasta tu primer proyecto colaborativo.

## 🤔 ¿Qué es Git y por qué lo necesito?

### El problema que resuelve Git

Imagina que estás escribiendo un ensayo importante. Sin Git, probablemente hagas esto:

```
ensayo.docx
ensayo_v2.docx
ensayo_v3.docx
ensayo_final.docx
ensayo_final_final.docx
ensayo_final_definitivo.docx
```

**Problemas de este enfoque:**
- ❌ No sabes qué cambió entre versiones
- ❌ Ocupa mucho espacio
- ❌ Es difícil colaborar con otras personas
- ❌ Puedes perder trabajo si algo se corrompe

### La solución: Control de versiones

Git es como una **máquina del tiempo** para tus archivos:

- ✅ **Historial completo**: Ve todos los cambios que has hecho
- ✅ **Colaboración**: Múltiples personas pueden trabajar sin conflictos
- ✅ **Seguridad**: Nunca pierdes trabajo
- ✅ **Ramas**: Experimenta sin miedo a romper nada
- ✅ **Respaldo**: Tu código está seguro en la nube

### Git vs GitHub: ¿Cuál es la diferencia?

**Git** = La herramienta (como Microsoft Word)
**GitHub** = El servicio en la nube (como Google Drive)

```
Tu computadora (Git) ←→ Internet ←→ GitHub (almacenamiento)
```

## 📦 Instalación de Git

### Windows

**Opción 1: Descarga oficial**
1. Ve a https://git-scm.com/download/win
2. Descarga e instala
3. En las opciones, acepta las predeterminadas

**Opción 2: Windows Terminal/PowerShell**
```powershell
winget install Git.Git
```

### macOS

**Opción 1: Homebrew (recomendado)**
```bash
# Instalar Homebrew primero
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instalar Git
brew install git
```

**Opción 2: Descarga oficial**
1. Ve a https://git-scm.com/download/mac
2. Descarga e instala

### Linux

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install git
```

**CentOS/RHEL/Fedora:**
```bash
sudo dnf install git
```

### Verificar instalación

```bash
git --version
# Debería mostrar algo como: git version 2.39.0
```

## ⚙️ Configuración inicial

Antes de usar Git, necesitas decirle quién eres:

```bash
# Tu nombre (aparecerá en el historial)
git config --global user.name "Tu Nombre"

# Tu email (debe coincidir con GitHub)
git config --global user.email "tu@email.com"

# Editor de texto preferido (opcional)
git config --global core.editor "code"  # Para VS Code
# git config --global core.editor "vim"   # Para Vim
# git config --global core.editor "nano"  # Para Nano

# Ver tu configuración
git config --list
```

**¿Por qué es importante?**
- Git necesita saber quién hace cada cambio
- GitHub usa el email para asociar commits con tu cuenta
- Aparece en el historial para siempre

## 📁 Conceptos fundamentales

### Repository (Repositorio)

Un **repositorio** es una carpeta que Git está "vigilando". Contiene:
- Tus archivos del proyecto
- El historial de todos los cambios
- Configuración de Git

### Working Directory, Staging Area y Repository

Git tiene tres "áreas" principales:

```
Working Directory  →  Staging Area  →  Repository
    (tus archivos)     (preparación)     (historial)
        ↓                   ↓                ↓
     git add            git commit
```

**Analogía con fotografía:**
1. **Working Directory**: El mundo real
2. **Staging Area**: Posicionas personas para la foto
3. **Repository**: Tomas la foto (queda guardada para siempre)

### Commit

Un **commit** es como una "foto" de tu proyecto en un momento específico:
- Contiene todos los cambios que hiciste
- Tiene un mensaje describiendo qué cambió
- Tiene un ID único (hash)
- Se guarda para siempre

## 🚀 Tu primer repositorio

### Crear un repositorio nuevo

```bash
# Crear una carpeta para tu proyecto
mkdir mi-primer-proyecto
cd mi-primer-proyecto

# Inicializar Git en esta carpeta
git init

# Verificar que funcionó
ls -la
# Deberías ver una carpeta .git (está oculta)
```

### Crear tu primer archivo

```bash
# Crear un archivo README
echo "# Mi Primer Proyecto" > README.md

# Ver el estado de Git
git status
```

**Explicación del output de `git status`:**
```
On branch main
No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        README.md

nothing added to commit but untracked files present
```

- **Untracked files**: Archivos que Git no está vigilando aún
- Git te sugiere usar `git add` para empezar a vigilarlos

### Tu primer commit

```bash
# Paso 1: Agregar archivo al staging area
git add README.md

# Ver el estado ahora
git status

# Paso 2: Hacer el commit
git commit -m "Agregar archivo README inicial"

# Ver el historial
git log
```

**¡Felicidades! Acabas de hacer tu primer commit.**

## 📝 Comandos básicos esenciales

### git add - Preparar cambios

```bash
# Agregar un archivo específico
git add archivo.txt

# Agregar múltiples archivos
git add archivo1.txt archivo2.txt

# Agregar todos los archivos modificados
git add .

# Agregar archivos por extensión
git add *.txt
```

### git commit - Guardar cambios

```bash
# Commit con mensaje
git commit -m "Descripción de los cambios"

# Commit más detallado (abre editor)
git commit

# Commit agregando archivos automáticamente
git commit -am "Mensaje"  # Solo para archivos ya tracked
```

**Consejos para buenos mensajes de commit:**
```bash
# ✅ Buenos mensajes
git commit -m "Agregar función de login"
git commit -m "Corregir bug en validación de email"
git commit -m "Actualizar documentación de API"

# ❌ Malos mensajes
git commit -m "cambios"
git commit -m "fix"
git commit -m "asdf"
```

### git status - Ver estado actual

```bash
git status

# Versión corta
git status -s
```

**Interpretando git status:**
- **Rojo**: Archivos modificados no agregados al staging
- **Verde**: Archivos en staging listos para commit
- **Untracked**: Archivos nuevos que Git no vigila

### git log - Ver historial

```bash
# Historial completo
git log

# Historial resumido
git log --oneline

# Historial con gráfico
git log --graph --oneline

# Últimos 5 commits
git log -5
```

### git diff - Ver diferencias

```bash
# Ver cambios no agregados al staging
git diff

# Ver cambios en staging
git diff --staged

# Comparar con commit anterior
git diff HEAD~1
```

## 🌿 Trabajando con ramas (branches)

### ¿Qué es una rama?

Una **rama** es como una línea de tiempo paralela de tu proyecto:

```
main:     A---B---C---F---G
               \       /
feature:        D---E
```

- **main**: La rama principal (tu línea de tiempo principal)
- **feature**: Una rama para experimentar
- Puedes trabajar en **feature** sin afectar **main**
- Cuando termines, puedes unir (**merge**) los cambios

### Comandos básicos de ramas

```bash
# Ver todas las ramas
git branch

# Crear una nueva rama
git branch nueva-funcionalidad

# Cambiar a una rama
git checkout nueva-funcionalidad

# Crear y cambiar en un comando
git checkout -b otra-rama

# Cambiar a main
git checkout main

# Eliminar una rama
git branch -d nombre-rama
```

### Ejemplo práctico con ramas

```bash
# Estás en main, crear rama para nueva funcionalidad
git checkout -b agregar-contacto

# Crear un nuevo archivo
echo "Página de contacto" > contacto.html

# Agregar y hacer commit
git add contacto.html
git commit -m "Agregar página de contacto"

# Volver a main
git checkout main

# ¡El archivo contacto.html no está aquí!
ls

# Volver a la rama
git checkout agregar-contacto

# ¡Ahí está el archivo!
ls
```

### Unir ramas (merge)

```bash
# Estar en la rama donde quieres traer los cambios (normalmente main)
git checkout main

# Traer los cambios de la otra rama
git merge agregar-contacto

# Eliminar la rama ya merged
git branch -d agregar-contacto
```

## 🌐 Trabajando con GitHub

### Crear cuenta en GitHub

1. Ve a https://github.com
2. Crea una cuenta gratuita
3. Verifica tu email

### Crear tu primer repositorio en GitHub

1. Haz clic en el botón verde "New"
2. Nombre: `mi-primer-repositorio`
3. Descripción: "Mi primer proyecto con Git"
4. Público o privado (elige lo que prefieras)
5. **NO** marques "Initialize with README" (ya tenemos uno)
6. Clic en "Create repository"

### Conectar tu repositorio local con GitHub

GitHub te dará comandos como estos:

```bash
# Agregar el repositorio remoto
git remote add origin https://github.com/tu-usuario/mi-primer-repositorio.git

# Subir tu código por primera vez
git push -u origin main

# En futuros push, solo necesitas:
git push
```

### Comandos para trabajar con remotos

```bash
# Ver repositorios remotos configurados
git remote -v

# Descargar cambios del remoto
git pull

# Subir cambios al remoto
git push

# Clonar un repositorio existente
git clone https://github.com/usuario/repositorio.git
```

## 🤝 Colaboración básica

### Fork y Pull Request

**Fork**: Hacer una copia de un repositorio de otra persona a tu cuenta

**Pull Request**: Pedirle al dueño original que incluya tus cambios

**Flujo típico:**
1. **Fork** el repositorio
2. **Clone** tu fork a tu computadora
3. **Crear una rama** para tu feature
4. **Hacer cambios** y commits
5. **Push** a tu fork
6. **Crear Pull Request** desde GitHub

### Ejemplo de colaboración

```bash
# 1. Clonar un repositorio
git clone https://github.com/usuario/proyecto-interesante.git
cd proyecto-interesante

# 2. Crear rama para tu contribución
git checkout -b mejorar-documentacion

# 3. Hacer cambios
echo "Documentación mejorada" >> README.md

# 4. Commit y push
git add README.md
git commit -m "Mejorar documentación del README"
git push origin mejorar-documentacion

# 5. Ir a GitHub y crear Pull Request
```

## 🔧 Herramientas útiles

### .gitignore - Ignorar archivos

Algunos archivos no deberían estar en Git:
- Archivos temporales
- Contraseñas
- Archivos muy grandes
- Archivos del sistema

```bash
# Crear archivo .gitignore
touch .gitignore

# Editarlo con tu editor favorito
code .gitignore
```

**Ejemplo de .gitignore:**
```
# Archivos del sistema
.DS_Store
Thumbs.db

# Editores
.vscode/
.idea/

# Dependencias
node_modules/
venv/

# Archivos de configuración local
.env
config.local.js

# Archivos temporales
*.log
*.tmp
temp/

# Archivos compilados
*.class
*.o
dist/
build/
```

### Aliases útiles

Los aliases te permiten crear comandos más cortos:

```bash
# Configurar aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.lg "log --oneline --graph --decorate"

# Ahora puedes usar:
git st      # en lugar de git status
git co main # en lugar de git checkout main
git lg      # log bonito
```

### Interfaces gráficas

Si prefieres interfaces visuales:

**Gratuitas:**
- **GitHub Desktop**: https://desktop.github.com/
- **SourceTree**: https://www.sourcetreeapp.com/
- **GitKraken** (versión gratuita)

**En editores:**
- **VS Code**: Extensión GitLens
- **IntelliJ/PyCharm**: Git integrado
- **Vim**: Extensión fugitive

## 🚨 Situaciones comunes y soluciones

### "¡Hice algo mal! ¿Cómo lo deshago?"

```bash
# Deshacer cambios no guardados
git checkout -- archivo.txt

# Deshacer git add (quitar del staging)
git reset archivo.txt

# Deshacer último commit (mantener cambios)
git reset --soft HEAD~1

# Deshacer último commit (borrar cambios)
git reset --hard HEAD~1

# Ver historial detallado para recovery
git reflog
```

### Conflictos de merge

Los conflictos pasan cuando dos personas modifican las mismas líneas:

```
<<<<<<< HEAD
Tu código
=======
El código de otra persona
>>>>>>> rama-que-estas-mergeando
```

**Cómo resolverlos:**
1. Abrir el archivo en conflicto
2. Decidir qué código mantener
3. Eliminar las marcas `<<<<<<<`, `=======`, `>>>>>>>`
4. Hacer `git add` y `git commit`

### "Git me pide usuario y contraseña todo el tiempo"

```bash
# Configurar credential helper
git config --global credential.helper store

# O mejor aún, usar SSH keys
# Generar clave SSH
ssh-keygen -t ed25519 -C "tu@email.com"

# Agregar a GitHub: Settings > SSH and GPG keys
```

## 🎯 Proyectos de práctica

### Proyecto 1: Página web personal

```bash
# Crear proyecto
mkdir mi-pagina-web
cd mi-pagina-web
git init

# Crear archivos básicos
echo "<h1>Mi Página Web</h1>" > index.html
echo "h1 { color: blue; }" > style.css

# Primer commit
git add .
git commit -m "Crear página web básica"

# Crear rama para mejoras
git checkout -b agregar-estilo

# Mejorar CSS
echo "body { font-family: Arial; }" >> style.css

# Commit y merge
git add style.css
git commit -m "Mejorar estilos CSS"
git checkout main
git merge agregar-estilo
```

### Proyecto 2: Lista de tareas

```bash
# Nuevo proyecto
mkdir lista-tareas
cd lista-tareas
git init

# Crear archivo de tareas
cat > tareas.md << EOF
# Mis Tareas

## Por hacer
- [ ] Aprender Git
- [ ] Crear repositorio
- [ ] Hacer primer commit

## Completadas
EOF

git add tareas.md
git commit -m "Crear lista de tareas inicial"

# Marcar tarea como completada
sed -i 's/- \[ \] Aprender Git/- [x] Aprender Git/' tareas.md

git add tareas.md
git commit -m "Marcar 'Aprender Git' como completada"
```

## 📚 Recursos para seguir aprendiendo

### Documentación oficial
- **Git Handbook**: https://git-scm.com/docs
- **GitHub Guides**: https://guides.github.com/
- **Interactive Git Tutorial**: https://learngitbranching.js.org/

### Comandos de referencia rápida

```bash
# Setup inicial
git config --global user.name "Nombre"
git config --global user.email "email"

# Repositorio
git init                    # Crear repositorio
git clone <url>            # Clonar repositorio

# Cambios
git status                 # Ver estado
git add <archivo>          # Agregar al staging
git add .                  # Agregar todo
git commit -m "mensaje"    # Hacer commit
git diff                   # Ver diferencias

# Ramas
git branch                 # Ver ramas
git branch <nombre>        # Crear rama
git checkout <nombre>      # Cambiar rama
git checkout -b <nombre>   # Crear y cambiar
git merge <nombre>         # Unir rama

# Remoto
git remote add origin <url> # Agregar remoto
git push                   # Subir cambios
git pull                   # Descargar cambios

# Historial
git log                    # Ver historial
git log --oneline         # Historial resumido
```

## 🎓 Teoría básica: ¿Cómo funciona Git internamente?

### El modelo de snapshots

Git no guarda diferencias, guarda **snapshots completos**:

```
Commit 1: [archivo1_v1, archivo2_v1, archivo3_v1]
Commit 2: [archivo1_v2, archivo2_v1, archivo3_v2]
Commit 3: [archivo1_v2, archivo2_v2, archivo3_v2]
```

**Ventajas:**
- ✅ Muy rápido para ver cualquier versión
- ✅ Integridad de datos garantizada
- ✅ Fácil de hacer branches y merges

### Los tres estados de Git

Todo archivo en Git está en uno de estos estados:

1. **Modified**: Cambiado pero no agregado al staging
2. **Staged**: Agregado al staging, listo para commit
3. **Committed**: Guardado en el repositorio

### Integridad y checksums

Git usa **SHA-1 hashes** para garantizar integridad:
- Cada commit tiene un hash único
- Si cambias algo, el hash cambia
- Es imposible corromper datos sin que Git se dé cuenta

```bash
# Ejemplo de hash de commit
git log --oneline
# a1b2c3d Agregar nueva funcionalidad
# e4f5g6h Corregir bug en login
```

---

## 🎓 Conclusión

¡Felicidades! Ahora tienes los conocimientos fundamentales de Git. Has aprendido:

✅ **Conceptos básicos**: Repository, commits, staging area  
✅ **Comandos esenciales**: add, commit, status, log, diff  
✅ **Trabajo con ramas**: Crear, cambiar, y merge branches  
✅ **Colaboración**: GitHub, clone, push, pull  
✅ **Resolución de problemas**: Deshacer cambios, conflictos  
✅ **Mejores prácticas**: .gitignore, buenos mensajes de commit

### ¿Qué sigue?

1. **Practica regularmente**: Usa Git en todos tus proyectos
2. **Contribuye a proyectos**: Busca repositorios en GitHub con la etiqueta "good first issue"
3. **Aprende Git avanzado**: Rebase, cherry-pick, hooks, workflows complejos
4. **Automatización**: GitHub Actions, CI/CD
5. **Git en equipo**: Estrategias de branching, code review

### Consejos finales

💡 **No tengas miedo de experimentar**: Git está diseñado para ser seguro  
💡 **Haz commits frecuentes**: Es mejor muchos commits pequeños que pocos grandes  
💡 **Escribe buenos mensajes**: Tu futuro yo te lo agradecerá  
💡 **Usa ramas liberalmente**: Son baratas y fáciles de crear  
💡 **Practica con proyectos reales**: La mejor forma de aprender es haciendo

Git puede parecer intimidante al principio, pero con práctica se vuelve segunda naturaleza. ¡Cada desarrollador profesional lo usa diariamente!

**Escrito por Andrés Núñez**  