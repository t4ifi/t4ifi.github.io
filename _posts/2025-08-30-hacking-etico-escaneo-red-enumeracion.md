---
title: "Hacking Ético: Escaneo de Red y Enumeración de Servicios"
date: 2025-09-05 10:00:00 +0100
categories: [Ciberseguridad, Hacking Ético]
tags: [hacking-etico, pentesting, nmap, enumeracion, escaneo, vulnerabilidades, kali-linux, reconocimiento]
image:
  path: /assets/img/hacking-etico/escaneo-red.jpg
  alt: "Escaneo de Red y Enumeración en Hacking Ético"
---

## 🎯 Introducción

Bienvenidos a la **cuarta parte** de nuestra serie "Hacking Ético desde 0". Después de haber configurado nuestro laboratorio y dominado las técnicas de OSINT, es momento de adentrarnos en el **escaneo de red y enumeración**, fases cruciales en cualquier evaluación de seguridad.

En esta entrega aprenderemos a identificar servicios activos, puertos abiertos, versiones de software y posibles vectores de ataque de manera profesional y ética.

## 📚 ¿Qué es el Escaneo de Red?

El **escaneo de red** es el proceso de identificar hosts activos, puertos abiertos, servicios en ejecución y sus versiones en una red objetivo. Es como hacer un "censo" de la infraestructura digital.

### Objetivos del Escaneo:
- **Descubrimiento de hosts** activos
- **Identificación de puertos** abiertos/cerrados/filtrados
- **Detección de servicios** y versiones
- **Identificación del OS** (Operating System)
- **Enumeración de recursos** compartidos
- **Mapeo de la topología** de red

## 🛠️ Herramientas Fundamentales

### 1. Nmap - El Rey del Escaneo

```bash
# Instalación en Kali Linux (viene preinstalado)
sudo apt update && sudo apt install nmap

# Verificar versión
nmap --version
```

#### Tipos de Escaneo con Nmap:

```bash
# Escaneo básico de ping (descubrimiento de hosts)
nmap -sn 192.168.1.0/24

# Escaneo TCP SYN (stealth scan)
nmap -sS target.com

# Escaneo TCP Connect
nmap -sT target.com

# Escaneo UDP
nmap -sU target.com

# Escaneo completo con detección de servicios y OS
nmap -sS -sV -O -A target.com

# Escaneo sigiloso y detallado
nmap -sS -sV -sC -O -T4 --script=default target.com
```

### 2. Masscan - Escaneo Ultra Rápido

```bash
# Instalación
sudo apt install masscan

# Escaneo rápido de puertos comunes
sudo masscan -p80,443,22,21,25,53,110,993,995 192.168.1.0/24 --rate=1000

# Escaneo de todos los puertos TCP
sudo masscan -p1-65535 192.168.1.1 --rate=1000 --banners
```

### 3. Rustscan - Scanner Moderno

```bash
# Instalación
wget https://github.com/RustScan/RustScan/releases/download/2.0.1/rustscan_2.0.1_amd64.deb
sudo dpkg -i rustscan_2.0.1_amd64.deb

# Uso básico
rustscan -a 192.168.1.1 -- -sV -sC

# Escaneo rápido con nmap integration
rustscan -a 192.168.1.1 -p 1-65535 -- -A
```

## 🔍 Metodología de Escaneo Profesional

### Fase 1: Descubrimiento de Hosts

```bash
# Ping sweep para descubrir hosts activos
nmap -sn 192.168.1.0/24

# Usando fping para verificación rápida
fping -g 192.168.1.0/24

# ARP scan en red local
nmap -PR 192.168.1.0/24

# Script de descubrimiento personalizado
#!/bin/bash
echo "=== DESCUBRIMIENTO DE HOSTS ==="
for ip in 192.168.1.{1..254}; do
    if ping -c 1 -W 1 $ip &> /dev/null; then
        echo "[+] Host activo: $ip"
    fi
done
```

### Fase 2: Escaneo de Puertos

```bash
# Escaneo rápido de puertos comunes
nmap -F target.com

# Top 1000 puertos
nmap --top-ports 1000 target.com

# Escaneo completo (todos los puertos)
nmap -p- target.com

# Escaneo específico de puertos
nmap -p 22,80,443,8080,8443 target.com

# Escaneo con timing aggressivo
nmap -T4 -p- target.com
```

### Fase 3: Detección de Servicios

```bash
# Detección de versiones de servicios
nmap -sV target.com

# Scripts NSE para enumeración
nmap -sC target.com

# Combinación completa
nmap -sS -sV -sC -O -T4 target.com

# Banner grabbing manual
nc -nv target.com 80
telnet target.com 25
```

### Fase 4: Detección de OS

```bash
# Fingerprinting del sistema operativo
nmap -O target.com

# Detección agresiva
nmap -A target.com

# TTL analysis para OS detection
ping target.com
# Linux: TTL=64, Windows: TTL=128, Cisco: TTL=255
```

## 🎯 Enumeración de Servicios Específicos

### SSH (Puerto 22)

```bash
# Banner grabbing SSH
nc target.com 22

# Enumeración con nmap scripts
nmap --script ssh-auth-methods target.com -p 22
nmap --script ssh-brute target.com -p 22
nmap --script ssh-hostkey target.com -p 22

# Verificar configuración SSH
ssh-audit target.com
```

### HTTP/HTTPS (Puerto 80/443)

```bash
# Banner y headers HTTP
curl -I http://target.com
wget --server-response --spider http://target.com

# Enumeración web con nmap
nmap --script http-enum target.com -p 80
nmap --script http-headers target.com -p 80
nmap --script http-methods target.com -p 80

# Tecnologías web
whatweb target.com
```

### FTP (Puerto 21)

```bash
# Banner FTP
nc target.com 21

# Scripts nmap para FTP
nmap --script ftp-anon target.com -p 21
nmap --script ftp-bounce target.com -p 21
nmap --script ftp-brute target.com -p 21
```

### SMB (Puerto 445)

```bash
# Enumeración SMB
smbclient -L //target.com
enum4linux target.com

# Scripts nmap SMB
nmap --script smb-enum-shares target.com -p 445
nmap --script smb-os-discovery target.com -p 445
nmap --script smb-protocols target.com -p 445
```

### DNS (Puerto 53)

```bash
# Enumeración DNS
dig @target.com axfr domain.com
dnsrecon -d domain.com
dnsenum domain.com

# Scripts nmap DNS
nmap --script dns-zone-transfer target.com -p 53
```

## 🔬 Scripts de Automatización

### Script de Escaneo Completo

```bash
#!/bin/bash
# full_scan.sh - Script de escaneo completo

TARGET=$1
OUTPUT_DIR="scans_$(date +%Y%m%d_%H%M%S)"

if [ -z "$TARGET" ]; then
    echo "Uso: $0 <target>"
    exit 1
fi

mkdir -p $OUTPUT_DIR

echo "[+] Iniciando escaneo completo de: $TARGET"

# Fase 1: Ping scan
echo "[+] Descubrimiento de hosts..."
nmap -sn $TARGET > $OUTPUT_DIR/ping_scan.txt

# Fase 2: Port scan
echo "[+] Escaneo de puertos..."
nmap -sS -T4 --top-ports 1000 $TARGET -oN $OUTPUT_DIR/port_scan.txt

# Fase 3: Service detection
echo "[+] Detección de servicios..."
nmap -sS -sV -sC -O -T4 $TARGET -oN $OUTPUT_DIR/service_scan.txt

# Fase 4: Vulnerability scan
echo "[+] Escaneo de vulnerabilidades..."
nmap --script vuln $TARGET -oN $OUTPUT_DIR/vuln_scan.txt

echo "[+] Escaneo completado. Resultados en: $OUTPUT_DIR"
```

### Script de Enumeración por Puertos

```bash
#!/bin/bash
# port_enum.sh - Enumeración específica por puerto

TARGET=$1
PORT=$2

case $PORT in
    22)
        echo "[+] Enumerando SSH..."
        nmap --script ssh-auth-methods,ssh-hostkey $TARGET -p 22
        ;;
    80|443)
        echo "[+] Enumerando HTTP/HTTPS..."
        nmap --script http-enum,http-headers,http-methods $TARGET -p $PORT
        whatweb $TARGET:$PORT
        ;;
    445)
        echo "[+] Enumerando SMB..."
        enum4linux $TARGET
        smbclient -L //$TARGET
        ;;
    *)
        echo "[+] Puerto no reconocido, ejecutando scripts genéricos..."
        nmap --script default $TARGET -p $PORT
        ;;
esac
```

## 📊 Interpretación de Resultados

### Estados de Puertos

- **Open**: Puerto abierto y servicio respondiendo
- **Closed**: Puerto cerrado (RST recibido)
- **Filtered**: Puerto filtrado por firewall
- **Open|Filtered**: No se puede determinar estado
- **Closed|Filtered**: No se puede determinar estado

### Ejemplo de Análisis:

```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5
80/tcp  open  http    Apache httpd 2.4.41
443/tcp open  https   Apache httpd 2.4.41 (SSL)
3306/tcp filtered mysql
```

**Análisis**:
- SSH habilitado (posible vector de ataque)
- Servidor web Apache (verificar vulnerabilidades)
- MySQL filtrado (firewall configurado)
- Sistema Ubuntu (buscar exploits específicos)

## 🛡️ Técnicas de Evasión

### Evasión de Firewalls

```bash
# Fragmentación de paquetes
nmap -f target.com

# Decoy scanning
nmap -D decoy1,decoy2,ME target.com

# Source port spoofing
nmap --source-port 53 target.com

# Timing evasion
nmap -T0 target.com  # Paranoiac
nmap -T1 target.com  # Sneaky
```

### Randomización

```bash
# Orden aleatorio de puertos
nmap -r target.com

# IPs aleatorias como decoys
nmap -D RND:10 target.com
```

## 🎪 Laboratorio Práctico

### Ejercicio 1: Escaneo de Red Local

```bash
# 1. Descubrir hosts en tu red local
nmap -sn 192.168.1.0/24

# 2. Escanear puertos del router
nmap -sS 192.168.1.1

# 3. Identificar servicios
nmap -sV 192.168.1.1
```

### Ejercicio 2: Máquina Vulnerable (Metasploitable)

```bash
# Descargar Metasploitable 2
wget https://sourceforge.net/projects/metasploitable/files/Metasploitable2/metasploitable-linux-2.0.0.zip

# Escaneo completo
nmap -sS -sV -sC -O -A <IP_METASPLOITABLE>

# Análisis de resultados y documentación
```

## 📈 Mejores Prácticas

### 1. Documentación

- **Registrar** todos los comandos utilizados
- **Capturar** screenshots de resultados importantes
- **Organizar** resultados por categorías
- **Crear** timelines de actividades

### 2. Sigilo y Ética

- **Timing**: Usar velocidades apropiadas (-T2, -T3)
- **Permisos**: Solo escanear sistemas autorizados
- **Impacto**: Evitar DoS o saturación de servicios
- **Logs**: Ser consciente de que las actividades se registran

### 3. Verificación

- **Confirmar** resultados con múltiples herramientas
- **Validar** servicios manualmente
- **Crosscheck** con información OSINT
- **Documentar** falsos positivos

## 🔧 Herramientas Complementarias

### Zmap - Internet-wide Scanning

```bash
# Instalación
sudo apt install zmap

# Escaneo masivo (CUIDADO - solo en entornos controlados)
zmap -p 80 192.168.1.0/24
```

### Unicornscan

```bash
# Instalación
sudo apt install unicornscan

# Uso básico
unicornscan target.com:1-1000
```

### Angry IP Scanner (GUI)

```bash
# Para entornos gráficos
sudo apt install ipscan
```

## 🎯 Próximos Pasos

En el **próximo tutorial** de la serie cubriremos:

1. **Análisis de Vulnerabilidades**
2. **Búsqueda de Exploits**
3. **Uso de herramientas como OpenVAS/Nessus**
4. **Verificación manual de vulnerabilidades**
5. **Preparación para la explotación**

## 📚 Recursos Adicionales

- [Nmap Official Documentation](https://nmap.org/docs.html)
- [NSE Script Documentation](https://nmap.org/nsedoc/)
- [Port Numbers Database](https://www.iana.org/assignments/service-names-port-numbers/)
- [CVE Database](https://cve.mitre.org/)

## 🎓 Conclusión

El escaneo de red y enumeración son habilidades fundamentales que separan a un principiante de un pentester profesional. La clave está en la **metodología sistemática**, el **conocimiento profundo de las herramientas** y la **interpretación correcta de los resultados**.

Recuerda: un buen escaneo es la base de una evaluación exitosa. **Tómate tu tiempo**, **documenta todo** y **siempre actúa de manera ética**.

En la próxima entrega profundizaremos en el análisis de vulnerabilidades y comenzaremos a preparar nuestros vectores de ataque.

---

**Andrés Nuñez - t4ifi**
