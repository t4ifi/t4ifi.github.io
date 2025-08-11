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

## 📋 Resumen de comandos aprendidos

| Comando | Función | Ejemplo |
|---------|---------|---------|
| `pwd` | Mostrar ubicación actual | `pwd` |
| `ls` | Listar archivos/carpetas | `ls -la` |
| `mkdir` | Crear directorios | `mkdir -p ruta/completa` |
| `cd` | Cambiar directorio | `cd mi_carpeta` |
| `touch` | Crear archivos vacíos | `touch archivo.txt` |
| `echo` | Mostrar/escribir texto | `echo "hola" > archivo.txt` |
| `cat` | Mostrar contenido | `cat archivo.txt` |
| `cp` | Copiar archivos | `cp origen destino` |
| `mv` | Mover/renombrar | `mv viejo nuevo` |
| `rm` | Eliminar archivos | `rm -r directorio` |
| `find` | Buscar archivos | `find . -name "*.txt"` |
| `grep` | Buscar en contenido | `grep "texto" archivo` |
| `chmod` | Cambiar permisos | `chmod +x script.py` |
| `tar` | Comprimir/extraer | `tar -czf archivo.tar.gz *` |
| `ps` | Ver procesos | `ps aux` |
| `kill` | Terminar proceso | `kill 1234` |
| `history` | Ver historial | `history \| grep comando` |

## 🔧 Comandos de información del sistema

| Comando | Función | Ejemplo |
|---------|---------|---------|
| `df -h` | Uso de disco | Ver espacio disponible |
| `free -h` | Uso de memoria | Ver RAM disponible |
| `top` | Procesos en tiempo real | Monitor del sistema |
| `uname -a` | Información del sistema | Kernel y arquitectura |
| `who` | Usuarios conectados | Ver sesiones activas |
| `date` | Fecha y hora | `date +"%Y-%m-%d %H:%M"` |
| `uptime` | Tiempo de actividad | Cuánto lleva encendido |

## 🌐 Comandos de red esenciales

| Comando | Función | Ejemplo |
|---------|---------|---------|
| `ping` | Probar conectividad | `ping -c 4 google.com` |
| `wget` | Descargar archivos | `wget https://example.com/file` |
| `curl` | Cliente HTTP | `curl -O https://example.com/file` |
| `ip addr` | Ver interfaces de red | Reemplaza `ifconfig` |
| `netstat` | Conexiones de red | `netstat -tuln` |
| `ss` | Conexiones (moderno) | `ss -tuln` |

## 💡 Tips y trucos profesionales

### Atajos de productividad:
```bash
# Navegación rápida
$ cd -              # Ir al directorio anterior
$ cd ~              # Ir al home
$ pushd /tmp        # Ir a /tmp y recordar ubicación anterior
$ popd              # Volver a la ubicación recordada

# Autocompletado avanzado
$ ls Doc<Tab>       # Se autocompleta a "Documentos"
$ cd /usr/sh<Tab>   # Se autocompleta a "/usr/share"

# Comodines poderosos
$ ls *.{txt,py}     # Archivos .txt y .py
$ cp archivo.{txt,bak}  # Copia archivo.txt a archivo.bak
$ rm *.[0-9]        # Eliminar archivos que terminen en número
```

### Comandos combinados útiles:
```bash
# Buscar y eliminar archivos vacíos
$ find . -type f -empty -delete

# Contar líneas de código en un proyecto
$ find . -name "*.py" -exec wc -l {} + | tail -1

# Buscar archivos grandes
$ find / -type f -size +100M 2>/dev/null

# Ver los 10 archivos más grandes
$ du -ah . | sort -hr | head -10

# Backup rápido con fecha
$ cp importante.txt importante_$(date +%Y%m%d).txt

# Monitorear cambios en un archivo
$ tail -f /var/log/syslog

# Ejecutar comando cada 2 segundos
$ watch -n 2 'ps aux | grep python'
```

### Variables de entorno útiles:
```bash
# Ver todas las variables
$ env

# Variables importantes
$ echo $HOME        # Tu directorio home
$ echo $PATH        # Rutas donde busca ejecutables
$ echo $USER        # Tu nombre de usuario
$ echo $PWD         # Directorio actual (como pwd)

# Agregar directorio al PATH temporalmente
$ export PATH=$PATH:/mi/nuevo/directorio
```

---

## 🚀 Desafío final: Script automatizado

Creá un script que automatice la creación de proyectos:

```bash
# Crear el script
$ touch crear_proyecto.sh
$ chmod +x crear_proyecto.sh

# Contenido del script (usar nano o vim)
#!/bin/bash

echo "🚀 Creador automático de proyectos"
echo "Ingresa el nombre del proyecto:"
read proyecto

echo "📁 Creando estructura para: $proyecto"
mkdir -p "$proyecto"/{src,docs,tests,config}

echo "📄 Creando archivos iniciales..."
touch "$proyecto"/README.md
touch "$proyecto"/src/main.py
touch "$proyecto"/docs/installation.md
touch "$proyecto"/tests/test_main.py

echo "✅ Proyecto '$proyecto' creado exitosamente!"
echo "📍 Ubicación: $(pwd)/$proyecto"

# Ejecutar el script
$ ./crear_proyecto.sh
```

---

💡 **Consejo final:** Estos comandos son como el alfabeto del sistema Linux. Practicá todos los días y pronto los usarás de forma natural.

🎯 **En el próximo tutorial veremos comandos más avanzados: pipes, redirecciones, expresiones regulares y scripting básico.**

**¡Felicitaciones! Ya manejás los comandos esenciales de Linux** 🎉

**Andrés Nuñez**

---

## � Paso 9: Búsqueda de archivos - Comandos `find` y `grep`

### Comando `find` - Buscar archivos por nombre:

```console
# Buscar archivos por nombre
$ find . -name "*.txt"
./readme.txt

# Buscar por tipo
$ find . -type f          # Solo archivos
$ find . -type d          # Solo directorios

# Buscar por tamaño
$ find . -size +1M        # Archivos mayores a 1MB
$ find . -size -100k      # Archivos menores a 100KB

# Buscar por fecha
$ find . -mtime -7        # Modificados en los últimos 7 días
```

### Comando `grep` - Buscar contenido dentro de archivos:

```console
# Buscar texto en un archivo
$ grep "Hola" aplicacion.py
print('¡Hola Linux!')

# Buscar en múltiples archivos
$ grep -r "web" .         # Buscar "web" recursivamente
$ grep -i "linux" *       # Buscar sin distinguir mayús/minús
$ grep -n "print" *.py    # Mostrar número de línea
```

## 📊 Paso 10: Información del sistema - Comandos de monitoreo

```console
# Ver uso de disco
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  5.2G   14G  28% /

# Ver procesos en ejecución
$ ps aux | head -5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169064 11108 ?        Ss   08:00   0:01 /sbin/init

# Ver uso de memoria
$ free -h
              total        used        free      shared  buff/cache   available
Mem:          7.7Gi       2.1Gi       3.8Gi       156Mi       1.8Gi       5.3Gi

# Ver información del sistema
$ uname -a
Linux ubuntu 5.15.0-56-generic #62-Ubuntu SMP x86_64 GNU/Linux

# Ver usuarios conectados
$ who
usuario   pts/0        2025-05-16 10:30
```

## 🔧 Paso 11: Permisos de archivos - Comando `chmod`

**Ver permisos actuales:**
```console
$ ls -l aplicacion.py
-rw-r--r-- 1 usuario usuario 21 may 16 10:45 aplicacion.py
```

**¿Qué significan estos símbolos?**
- `rw-` (propietario): lectura y escritura
- `r--` (grupo): solo lectura  
- `r--` (otros): solo lectura

**Cambiar permisos:**
```console
# Hacer ejecutable un script
$ chmod +x aplicacion.py
$ ls -l aplicacion.py
-rwxr-xr-x 1 usuario usuario 21 may 16 10:45 aplicacion.py

# Ahora puedes ejecutarlo
$ ./aplicacion.py
¡Hola Linux!

# Otros ejemplos de permisos
$ chmod 755 aplicacion.py    # rwxr-xr-x
$ chmod 644 readme.txt       # rw-r--r--
```

## 📦 Paso 12: Compresión y archivos - Comandos `tar` y `zip`

**Crear un archivo comprimido:**
```console
# Crear tar.gz
$ tar -czf mi_proyecto.tar.gz *
$ ls -lh mi_proyecto.tar.gz
-rw-r--r-- 1 usuario usuario 1.2K may 16 11:00 mi_proyecto.tar.gz

# Crear zip
$ zip -r mi_proyecto.zip *
$ ls -lh mi_proyecto.zip
-rw-r--r-- 1 usuario usuario 945 may 16 11:01 mi_proyecto.zip
```

**Extraer archivos:**
```console
# Extraer tar.gz
$ tar -xzf mi_proyecto.tar.gz

# Extraer zip
$ unzip mi_proyecto.zip

# Solo listar contenido sin extraer
$ tar -tzf mi_proyecto.tar.gz
$ unzip -l mi_proyecto.zip
```

## 🌐 Paso 13: Comandos de red básicos

```console
# Ver configuración de red
$ ip addr show
$ ifconfig          # En sistemas más antiguos

# Hacer ping a un servidor
$ ping -c 4 google.com
PING google.com (172.217.10.78) 56(84) bytes of data.
64 bytes from mad06s27-in-f14.1e100.net (172.217.10.78): icmp_seq=1 ttl=117 time=20.1 ms

# Descargar archivos de internet
$ wget https://example.com/archivo.txt
$ curl -O https://example.com/archivo.txt

# Ver conexiones de red activas
$ netstat -tuln
$ ss -tuln          # Comando moderno
```

## 🏃‍♂️ Paso 14: Gestión de procesos

```console
# Ver procesos en tiempo real
$ top
$ htop              # Versión mejorada (si está instalada)

# Ejecutar comando en segundo plano
$ python3 -m http.server 8000 &
[1] 12345

# Ver trabajos en segundo plano
$ jobs
[1]+  Running    python3 -m http.server 8000 &

# Traer proceso al primer plano
$ fg %1

# Terminar un proceso
$ kill 12345
$ killall python3   # Terminar todos los procesos python3
```

## 📚 Paso 15: Historial y atajos útiles

```console
# Ver historial de comandos
$ history
  501  ls -la
  502  cd mi_proyecto
  503  touch readme.txt

# Ejecutar comando del historial
$ !502              # Ejecuta el comando número 502
$ !!                # Ejecuta el último comando
$ !grep             # Ejecuta el último comando que empezaba con "grep"

# Buscar en el historial
$ history | grep "mkdir"
```

**Atajos de teclado esenciales:**
- `Ctrl + C`: Terminar comando actual
- `Ctrl + Z`: Suspender comando actual
- `Ctrl + L`: Limpiar pantalla (equivale a `clear`)
- `Ctrl + R`: Buscar en historial
- `Tab`: Autocompletar comandos y rutas
- `Ctrl + A`: Ir al inicio de la línea
- `Ctrl + E`: Ir al final de la línea

## 🎯 Ejercicio avanzado: ¡Proyecto completo!

Creá un proyecto completo usando todos los comandos aprendidos:

```console
# 1. Crear estructura de proyecto
$ mkdir -p mi_app/{src,docs,tests,data}
$ cd mi_app

# 2. Crear archivos del proyecto
$ touch src/main.py src/utils.py
$ touch docs/README.md docs/INSTALL.md
$ touch tests/test_main.py
$ touch data/config.json

# 3. Agregar contenido a los archivos
$ echo "#!/usr/bin/env python3" > src/main.py
$ echo "def main():" >> src/main.py
$ echo "    print('Mi aplicación funciona!')" >> src/main.py

$ echo "# Mi Aplicación" > docs/README.md
$ echo "Una aplicación de ejemplo" >> docs/README.md

$ echo '{"debug": true, "port": 8080}' > data/config.json

# 4. Hacer ejecutable el script principal
$ chmod +x src/main.py

# 5. Probar que funciona
$ python3 src/main.py
Mi aplicación funciona!

# 6. Ver estructura completa
$ find . -type f | sort
./data/config.json
./docs/INSTALL.md
./docs/README.md
./src/main.py
./src/utils.py
./tests/test_main.py

# 7. Crear backup del proyecto
$ cd ..
$ tar -czf mi_app_backup.tar.gz mi_app/
$ ls -lh mi_app_backup.tar.gz

# 8. Ver información del backup
$ file mi_app_backup.tar.gz
mi_app_backup.tar.gz: gzip compressed data
```

## 📋 Resumen de comandos aprendidos
