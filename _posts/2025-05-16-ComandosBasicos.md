---
title: Comandos básicos de Linux; mkdir, cd, ls y touch
date: 2025-05-16
categories: [Linux, Terminal]
tags: [linux, comandos, terminal, bash]
---

## Comandos básicos para manejar archivos y directorios en Linux

Aquí aprenderás a crear carpetas, moverte entre directorios, listar archivos, crear archivos vacíos y conocer el tipo de archivo con comandos esenciales.

---

En estos ejemplos, el logo de `$` va a simular ser la shell.
### mkdir — Crear directorios

```console
$ mkdir mi_carpeta
$ ls
mi_carpeta
```
En este ejemplo:
- Usamos `mkdir mi_carpeta` para crear una nueva carpeta llamada **mi_carpeta**.
- Luego con `ls` listamos los archivos y carpetas del directorio actual.
- Como resultado, vemos que **mi_carpeta** aparece en la lista, confirmando que la carpeta fue creada exitosamente.


### cd — Cambiar de directorio

```console
$ ls
mi_carpeta
$ cd mi_carpeta
$ pwd
/home/usuario/mi_carpeta
```
En este ejemplo:
- Usamos `cd mi_carpeta` para movernos al directorio mi_carpeta que creamos antes.
- Luego con `pwd` (print working directory) verificamos en qué carpeta estamos actualmente.
- El resultado muestra la ruta completa de la carpeta actual, confirmando que estamos dentro de `mi_carpeta`.

### cd .. — Volver al directorio anterior

```console
$ cd ..
$ ls
mi_carpeta
$ pwd
/home/usuario
```
En este ejemplo:
- `cd ..` nos lleva un nivel arriba en la jerarquía de directorios.
- Con pwd y ls confirmamos que estamos en la carpeta padre de la anterior.


### touch — Crear archivos vacíos

```console
$ touch archivo.txt
$ ls
archivo.txt
```
En este ejemplo:
- Usamos `touch archivo.txt` para crear un archivo vacío llamado `archivo.txt`.
- Luego con `ls` verificamos que el archivo fue creado correctamente.

---

💡  *Consejo: Usá siempre pwd para saber en qué directorio estás trabajando y evitá perderte en la terminal.*

**Andrés Nuñez**
