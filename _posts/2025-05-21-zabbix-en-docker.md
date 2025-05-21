---
title: "Desplegar Zabbix en Docker con PostgreSQL"
date: 2025-05-21 19:03:42 
categories: [DevOps, Monitoring]
tags: [Zabbix, Docker, Compose, PostgreSQL]
---

Este post muestra cómo desplegar una infraestructura básica de Zabbix usando Docker y Docker Compose. Incluye servidor, frontend web, agente, soporte SNMP y base de datos PostgreSQL, todo contenido y personalizado.

## 🎯 Objetivo

Tener corriendo un entorno de monitoreo completo con:

- Zabbix Server
- Zabbix Web frontend (Nginx + PHP)
- Zabbix Agent
- SNMP Trap handler
- Base de datos PostgreSQL

## 🧩 Requisitos

- Docker
- Docker Compose
- Acceso root o permisos sudo

## ⚙️ Pasos

1. **Descargar el archivo `docker-compose.yml`:**

```bash
wget https://raw.githubusercontent.com/t4ifi/zabbix-docker/refs/heads/main/docker-compose.yml
```

> ⚠️ También podés crear el archivo manualmente con el contenido que dejo abajo.

2. **Levantar los servicios:**

```bash
sudo docker-compose up -d
```

3. **Verificar contenedores:**

```bash
sudo docker ps -a
```

4. **Acceder a la interfaz web:**

Ir a [http://localhost:8080](http://localhost:8080) (o la IP de tu host).  
Usuario por defecto: `Admin`  
Contraseña: `zabbix`

---

## 🧱 docker-compose.yml personalizado

```yaml
version: '3.5'

services:
  server:
    image: zabbix/zabbix-server-pgsql:alpine-5.4-latest
    ports:
      - "10051:10051"
    volumes:
      - ./zbx_env/usr/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts:ro
      - ./zbx_env/usr/lib/zabbix/externalscripts:/usr/lib/zabbix/externalscripts:ro
      - ./zbx_env/var/lib/zabbix/export:/var/lib/zabbix/export:rw
      - ./zbx_env/var/lib/zabbix/modules:/var/lib/zabbix/modules:ro
      - ./zbx_env/var/lib/zabbix/enc:/var/lib/zabbix/enc:ro
      - ./zbx_env/var/lib/zabbix/ssh_keys:/var/lib/zabbix/ssh_keys:ro
      - ./zbx_env/var/lib/zabbix/mibs:/var/lib/zabbix/mibs:ro
      - ./zbx_env/var/lib/zabbix/snmptraps:/var/lib/zabbix/snmptraps:ro
    restart: always
    depends_on:
      - postgres-server
    environment:
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=zabbix
      - POSTGRES_DB=zabbixNew
      - ZBX_HISTORYSTORAGETYPES=log,text
      - ZBX_DEBUGLEVEL=1
      - ZBX_HOUSEKEEPINGFREQUENCY=1
      - ZBX_MAXHOUSEKEEPERDELETE=5000
      - ZBX_PROXYCONFIGFREQUENCY=3600

  web-nginx-pgsql:
    image: zabbix/zabbix-web-nginx-pgsql:alpine-5.4-latest
    ports:
      - "8080:8080"
      - "8443:8443"
    volumes:
      - ./zbx_env/etc/ssl/nginx:/etc/ssl/nginx:ro
      - ./zbx_env/usr/share/zabbix/modules/:/usr/share/zabbix/modules/:ro
    restart: always
    depends_on:
      - server
      - postgres-server
    environment:
      - POSTGRES_USER=zabbix
      - POSTGRES_PASSWORD=zabbix
      - POSTGRES_DB=zabbixNew
      - ZBX_SERVER_HOST=server
      - ZBX_POSTMAXSIZE=64M
      - PHP_TZ=America/Montevideo
      - ZBX_MAXEXECUTIONTIME=500

  agent:
    image: zabbix/zabbix-agent:alpine-5.4-latest
    ports:
      - "10050:10050"
    volumes:
      - ./zbx_env/etc/zabbix/zabbix_agentd.d:/etc/zabbix/zabbix_agentd.d:ro
      - ./zbx_env/var/lib/zabbix/modules:/var/lib/zabbix/modules:ro
      - ./zbx_env/var/lib/zabbix/enc:/var/lib/zabbix/enc:ro
      - ./zbx_env/var/lib/zabbix/ssh_keys:/var/lib/zabbix/ssh_keys:ro
    privileged: true
    pid: "host"
    restart: always
    depends_on:
      - server
    environment:
      - ZBX_SERVER_HOST=server

  snmptraps:
    image: zabbix/zabbix-snmptraps:alpine-5.4-latest
    ports:
      - "162:1162/udp"
    volumes:
      - ./snmptraps:/var/lib/zabbix/snmptraps:rw
    restart: always
    depends_on:
      - server
    environment:
      - ZBX_SERVER_HOST=server

  postgres-server:
    image: postgres:13-alpine
    volumes:
      - ./zbx_env/var/lib/postgresql/data:/var/lib/postgresql/data:rw
    environment:
      - POSTGRES_PASSWORD=zabbix
      - POSTGRES_USER=zabbix
      - POSTGRES_DB=zabbixNew
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---
