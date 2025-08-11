---
title: "Comandos básicos de Linux: Tutorial práctico paso a paso"
date: 2025-05-16
categories: [Linux, Terminal]
tags: [linux, comandos, terminal, bash, principiantes, navegacion]
---

## 🎯 Aprende navegando: Tu primera sesión de terminal

En este tutorial vas a aprender los comandos fundamentales de Linux de forma **práctica y gradual**. Vamos a simular una sesión real donde creás directorios, navegás entre ellos, creás archivos y explorás el sistema.

> 💡 **Tip inicial:** En todos los ejemplos, el símbolo `$` representa el prompt de la terminal. ¡No lo escribas!
---

## 📍 Paso 1: ¿Dónde estoy? - Comando `pwd`

Antes que nada, necesitás saber dónde estás ubicado en el sistema:

```console
$ pwd
/home/usuario
```

**¿Qué pasó aquí?**
- `pwd` significa "Print Working Directory" (mostrar directorio actual)
- Te muestra la **ruta completa** donde te encontrás
- En este caso, estás en tu directorio home `/home/usuario`

## 📂 Paso 2: ¿Qué hay aquí? - Comando `ls`

Ahora veamos qué archivos y carpetas tenés en tu ubicación actual:

```console
$ ls
Documentos  Descargas  Escritorio  Imágenes  Música  Videos
```

**Variaciones útiles de `ls`:**

```console
$ ls -l
total 24
drwxr-xr-x 2 usuario usuario 4096 ago 11 10:30 Documentos
drwxr-xr-x 2 usuario usuario 4096 ago 11 10:25 Descargas
drwxr-xr-x 2 usuario usuario 4096 ago 11 10:20 Escritorio
```

```console
$ ls -la
total 32
drwxr-xr-x 7 usuario usuario 4096 ago 11 10:35 .
drwxr-xr-x 3 root    root    4096 ago 10 15:20 ..
drwxr-xr-x 2 usuario usuario 4096 ago 11 10:30 Documentos
-rw-r--r-- 1 usuario usuario  220 ago 10 15:20 .bash_logout
```

**¿Qué significan estas opciones?**
- `ls -l`: Lista detallada (permisos, propietario, tamaño, fecha)
- `ls -la`: Lista detallada **incluyendo archivos ocultos** (que empiezan con .)
- `ls -lh`: Lista con tamaños **legibles** (1.2K, 5.5M, 2.1G)

## 📁 Paso 3: Crear tu primer directorio - Comando `mkdir`

Vamos a crear una carpeta para organizar tu trabajo:

```console
$ mkdir mi_proyecto
$ ls
Documentos  Descargas  Escritorio  Imágenes  mi_proyecto  Música  Videos
```

**¿Qué pasó?**
- `mkdir mi_proyecto` creó una nueva carpeta llamada "mi_proyecto"
- Al hacer `ls` otra vez, ahora ves que apareció en la lista
- ✅ **¡Tu primera carpeta creada exitosamente!**

**Crear múltiples directorios:**
```console
$ mkdir carpeta1 carpeta2 carpeta3
$ ls
carpeta1  carpeta2  carpeta3  mi_proyecto  Documentos  ...
```

**Crear estructura anidada:**
```console
$ mkdir -p proyectos/web/css
$ ls proyectos/web/
css
```

> 💡 **Tip:** La opción `-p` crea todos los directorios padre que no existan

## 🚶 Paso 4: Navegar entre directorios - Comando `cd`

Ahora entremos a la carpeta que acabamos de crear:

```console
$ cd mi_proyecto
$ pwd
/home/usuario/mi_proyecto
```

**¿Qué pasó?**
- `cd mi_proyecto` cambió tu ubicación al directorio "mi_proyecto"
- `pwd` confirma que ahora estás dentro de esa carpeta
- ✅ **¡Estás navegando como un pro!**

**Verificá que la carpeta está vacía:**
```console
$ ls
(sin salida - la carpeta está vacía)
```

**Movimientos útiles con `cd`:**
```console
$ cd ..              # Subir un nivel (volver al directorio padre)
$ pwd
/home/usuario

$ cd mi_proyecto     # Entrar nuevamente
$ cd ~               # Ir directo al home
$ pwd
/home/usuario

$ cd -               # Volver al directorio anterior
$ pwd
/home/usuario/mi_proyecto
```

## 📄 Paso 5: Crear archivos - Comando `touch`

Creemos algunos archivos dentro de nuestro proyecto:

```console
$ touch readme.txt
$ touch app.py
$ touch config.conf
$ ls
app.py  config.conf  readme.txt
```

**¿Qué hace `touch`?**
- Crea archivos **vacíos** instantáneamente
- Si el archivo ya existe, actualiza su fecha de modificación
- Es perfecto para crear plantillas rápidas

**Crear múltiples archivos:**
```console
$ touch archivo1.txt archivo2.txt archivo3.txt
$ ls
app.py  archivo1.txt  archivo2.txt  archivo3.txt  config.conf  readme.txt
```

## 📝 Paso 6: Trabajando con contenido - Comandos `echo` y `cat`

Vamos a agregar contenido a nuestros archivos:

```console
$ echo "Este es mi primer proyecto en Linux" > readme.txt
$ echo "print('¡Hola Linux!')" > app.py
$ cat readme.txt
Este es mi primer proyecto en Linux
```

```console
$ cat app.py
print('¡Hola Linux!')
```

**¿Qué aprendiste?**
- `echo "texto" > archivo`: Escribe texto en un archivo (lo sobrescribe)
- `echo "texto" >> archivo`: Agrega texto al final del archivo
- `cat archivo`: Muestra el contenido completo del archivo

## 🔄 Paso 7: Copiar y mover archivos - Comandos `cp` y `mv`

**Copiar archivos:**
```console
$ cp readme.txt readme_backup.txt
$ ls
app.py  config.conf  readme.txt  readme_backup.txt  archivo1.txt  archivo2.txt  archivo3.txt
```

**Renombrar archivos:**
```console
$ mv app.py aplicacion.py
$ ls
aplicacion.py  config.conf  readme.txt  readme_backup.txt  archivo1.txt  archivo2.txt  archivo3.txt
```

**Mover archivos a otra carpeta:**
```console
$ mkdir respaldos
$ mv readme_backup.txt respaldos/
$ ls respaldos/
readme_backup.txt
```

## 🧹 Paso 8: Limpiar - Comando `rm`

Eliminemos algunos archivos de prueba:

```console
$ rm archivo1.txt archivo2.txt archivo3.txt
$ ls
aplicacion.py  config.conf  readme.txt  respaldos
```

**⚠️ Cuidado con `rm`:**
```console
$ rm config.conf    # Elimina un archivo
$ rm -r respaldos/   # Elimina directorio y todo su contenido
$ ls
aplicacion.py  readme.txt
```

> 🚨 **Importante:** `rm` elimina permanentemente. No hay "papelera" en Linux.

## 🎯 Ejercicio práctico: ¡Ponelo en práctica!

Ahora que conocés los comandos básicos, hacé este ejercicio completo:

```console
# 1. Ir al directorio home
$ cd ~

# 2. Crear estructura para un proyecto web
$ mkdir -p sitio_web/css sitio_web/js sitio_web/images

# 3. Navegar al proyecto
$ cd sitio_web

# 4. Crear archivos básicos
$ touch index.html css/style.css js/script.js

# 5. Agregar contenido básico
$ echo "<h1>Mi primer sitio web</h1>" > index.html
$ echo "body { font-family: Arial; }" > css/style.css
$ echo "console.log('¡Hola mundo!');" > js/script.js

# 6. Verificar estructura
$ ls -la
$ cat index.html

# 7. Ver estructura completa del proyecto
$ find . -type f
./index.html
./css/style.css
./js/script.js
```

## 📋 Resumen de comandos aprendidos

| Comando | Función | Ejemplo |
|---------|---------|---------|
| `pwd` | Mostrar ubicación actual | `pwd` |
| `ls` | Listar archivos/carpetas | `ls -la` |
| `mkdir` | Crear directorios | `mkdir mi_carpeta` |
| `cd` | Cambiar directorio | `cd mi_carpeta` |
| `touch` | Crear archivos vacíos | `touch archivo.txt` |
| `echo` | Mostrar/escribir texto | `echo "hola" > archivo.txt` |
| `cat` | Mostrar contenido | `cat archivo.txt` |
| `cp` | Copiar archivos | `cp origen destino` |
| `mv` | Mover/renombrar | `mv viejo nuevo` |
| `rm` | Eliminar archivos | `rm archivo.txt` |

---

💡 **Consejo final:** Practicá estos comandos hasta que se vuelvan automáticos. ¡Son la base de todo lo que harás en Linux!

🎯 **En el próximo tutorial veremos comandos más avanzados para administración del sistema, búsqueda de archivos y gestión de procesos.**

**Andrés Nuñez**
