---
title: "Introducción completa a Docker: desde cero hasta Docker Compose"
date: 2025-05-21 19:45:08 +0000
categories: [Linux, Contenedores]
tags: [Docker, Linux, DevOps, Debian]
---


## ¿Qué es Docker?

Docker es una plataforma diseñada para desarrollar, enviar y ejecutar aplicaciones dentro de contenedores. Un **contenedor** es una unidad estándar de software que empaqueta el código y todas sus dependencias para que la aplicación se ejecute de manera rápida y confiable en diferentes entornos.

A diferencia de las máquinas virtuales, los contenedores **no necesitan un sistema operativo completo** para ejecutarse. Comparten el kernel del sistema operativo host, lo que los hace **ligeros, rápidos y portables**.

## ¿Para qué sirve Docker?

Docker se utiliza para:

- Aislar aplicaciones.
- Crear entornos de desarrollo reproducibles.
- Facilitar la integración continua y el despliegue continuo (CI/CD).
- Ejecutar múltiples servicios (por ejemplo, una API, una base de datos y una interfaz web) en una sola máquina de forma ordenada.

## ¿Por qué usar Docker y no máquinas virtuales?
```console
| Característica       | Contenedores (Docker)        | Máquinas Virtuales       |
|----------------------|------------------------------|--------------------------|
| Peso                 | Ligero                       | Pesado                   |
| Velocidad de arranque| Segundos                     | Minutos                  |
| Uso de recursos      | Eficiente                    | Alto                     |
| Portabilidad         | Alta                         | Limitada                 |
| Aislamiento          | Parcial (a nivel de proceso) | Completo (a nivel de SO) |
```

## Conceptos clave

- **Imagen**: plantilla inmutable que define cómo se construye un contenedor.
- **Contenedor**: instancia en ejecución de una imagen.
- **Dockerfile**: archivo de texto que contiene instrucciones para construir una imagen.
- **Volume**: mecanismo para persistir datos.
- **Network**: permite que contenedores se comuniquen entre sí.

## Cómo instalar Docker en Debian 12

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Agregar la clave GPG oficial
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Agregar repositorio
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian   $(lsb_release -cs) stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verificar instalación
sudo docker run hello-world
```

## Comandos básicos

```bash
# Ver versión de Docker
docker --version

# Ver imágenes locales
docker images

# Descargar imagen
docker pull ubuntu

# Ejecutar un contenedor interactivo
docker run -it ubuntu /bin/bash

# Ver contenedores en ejecución
docker ps

# Ver todos los contenedores
docker ps -a

# Eliminar contenedor
docker rm <id>

# Eliminar imagen
docker rmi <id>
```

## Crear tu propia imagen con Dockerfile

```Dockerfile
# Dockerfile
FROM ubuntu:20.04

RUN apt update && apt install -y nginx

CMD ["nginx", "-g", "daemon off;"]
```

```bash
# Construir imagen
docker build -t mi-nginx .

# Ejecutar contenedor
docker run -d -p 8080:80 mi-nginx
```

Ahora podés acceder a `http://localhost:8080` y ver el servidor web.

## ¿Qué es docker-compose?

Es una herramienta para definir y correr múltiples contenedores usando un archivo `docker-compose.yml`.

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
```

```bash
# Levantar todos los servicios
docker compose up -d

# Detener
docker compose down
```

## Buenas prácticas

- Usá `.dockerignore` para evitar copiar archivos innecesarios.
- No corras procesos como root dentro del contenedor.
- Mantené las imágenes pequeñas.
- Usá volúmenes para persistencia.
- Eliminá contenedores, redes y volúmenes no usados regularmente.

---