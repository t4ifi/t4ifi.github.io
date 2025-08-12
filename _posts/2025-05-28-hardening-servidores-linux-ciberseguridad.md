---
title: "Hardening de Servidores Linux: Guía Completa de Ciberseguridad"
date: 2025-05-28 18:20:00 +0000
categories: [Ciberseguridad, Linux]
tags: [linux, ciberseguridad, hardening, security, servidor, firewall, ssh, audit]
---

La seguridad de servidores Linux no es opcional en 2025. Con ataques cada vez más sofisticados, el hardening (endurecimiento) del sistema se ha vuelto crítico. Esta guía te enseñará técnicas profesionales para proteger servidores en entornos de producción.

> 🛡️ **Objetivo:** Implementar una estrategia completa de hardening que proteja tu servidor Linux contra amenazas modernas, desde la configuración básica hasta técnicas avanzadas de monitoreo y respuesta ante incidentes.

## 🎯 Fundamentos de Seguridad en Linux

### Modelo de seguridad Linux: DAC vs MAC

**DAC (Discretionary Access Control)** - Modelo tradicional:
```bash
# Permisos estándar Unix
-rwxr-xr-x  # Owner: rwx, Group: r-x, Others: r-x
chmod 755 archivo
chown user:group archivo
```

**MAC (Mandatory Access Control)** - Seguridad avanzada:
- **SELinux** (Security-Enhanced Linux)
- **AppArmor** (Application Armor)
- **grsecurity** (Kernel hardening)

### Principios fundamentales de hardening

1. **Principio del menor privilegio**: Solo los permisos mínimos necesarios
2. **Defensa en profundidad**: Múltiples capas de seguridad
3. **Fail-safe defaults**: Configuración segura por defecto
4. **Superficie de ataque mínima**: Menos servicios = menos vulnerabilidades
5. **Separation of duties**: Diferentes roles para diferentes tareas

## 🔐 Configuración Inicial Segura

### Instalación mínima del sistema

**Principios para instalación:**
```bash
# Solo paquetes esenciales
apt-get install --no-install-recommends package

# Verificar servicios innecesarios
systemctl list-unit-files --type=service --state=enabled

# Deshabilitar servicios no requeridos
systemctl disable bluetooth
systemctl disable cups
systemctl disable avahi-daemon
```

### Configuración de usuarios y grupos

**Política de usuarios:**
```bash
# Crear usuario para aplicaciones (sin login)
useradd -r -s /bin/false -d /opt/app appuser

# Usuario administrador (no root directo)
useradd -m -G sudo,adm admin
passwd admin

# Bloquear cuenta root para SSH
passwd -l root

# Configurar sudoers de forma granular
visudo
# admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
# admin ALL=(ALL) NOPASSWD: /usr/bin/tail -f /var/log/*
```

**Configuración de contraseñas:**
```bash
# /etc/login.defs
PASS_MAX_DAYS   90      # Expiración de contraseñas
PASS_MIN_DAYS   1       # Mínimo días entre cambios
PASS_MIN_LEN    12      # Longitud mínima
PASS_WARN_AGE   7       # Días de aviso antes de expirar

# Política de complejidad con PAM
# /etc/pam.d/common-password
password requisite pam_pwquality.so retry=3 minlen=12 maxrepeat=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1
```

### SSH Hardening: Primera línea de defensa

**Configuración segura de SSH:**
```bash
# /etc/ssh/sshd_config

# Cambiar puerto por defecto (security through obscurity)
Port 2222

# Solo protocolo 2
Protocol 2

# Deshabilitar login root
PermitRootLogin no

# Solo usuarios específicos
AllowUsers admin developer

# Autenticación por clave únicamente
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Configuraciones de timeout
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 60

# Deshabilitar features peligrosos
X11Forwarding no
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no

# Logging detallado
LogLevel VERBOSE

# Limitar intentos de autenticación
MaxAuthTries 3
MaxStartups 3:30:10

# Banner de advertencia
Banner /etc/ssh/banner.txt
```

**Configuración de claves SSH:**
```bash
# Generar clave con algoritmo moderno
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519 -C "admin@servidor"

# O RSA 4096 bits si ed25519 no está disponible
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -C "admin@servidor"

# Configurar authorized_keys con restricciones
# ~/.ssh/authorized_keys
command="rsync --server -vlogDtpre.iLsfxC . /backup/",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAA...

# Permisos correctos
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

## 🔥 Firewall y Control de Red

### iptables: Configuración avanzada

**Política por defecto DROP:**
```bash
#!/bin/bash
# firewall-rules.sh

# Limpiar reglas existentes
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Política por defecto: denegar todo
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

# Permitir loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Permitir conexiones establecidas
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT

# SSH (puerto personalizado)
iptables -A INPUT -p tcp --dport 2222 -m state --state NEW -j ACCEPT
iptables -A OUTPUT -p tcp --sport 2222 -j ACCEPT

# HTTP/HTTPS salientes (para updates)
iptables -A OUTPUT -p tcp --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT

# DNS
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 53 -j ACCEPT

# NTP
iptables -A OUTPUT -p udp --dport 123 -j ACCEPT

# Web server (si aplicable)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Rate limiting para SSH
iptables -A INPUT -p tcp --dport 2222 -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 2222 -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# Logging de packets DROP
iptables -A INPUT -j LOG --log-prefix "IPTABLES DROP: " --log-level 4
iptables -A FORWARD -j LOG --log-prefix "IPTABLES FORWARD DROP: " --log-level 4

# Persistir reglas
iptables-save > /etc/iptables/rules.v4
```

### UFW: Firewall simplificado

**Configuración básica:**
```bash
# Resetear UFW
ufw --force reset

# Política por defecto
ufw default deny incoming
ufw default deny outgoing
ufw default deny forward

# Permitir servicios esenciales salientes
ufw allow out 53        # DNS
ufw allow out 123       # NTP
ufw allow out 80        # HTTP
ufw allow out 443       # HTTPS

# SSH con rate limiting
ufw limit 2222/tcp

# Web server
ufw allow 80/tcp
ufw allow 443/tcp

# Logging
ufw logging on

# Habilitar UFW
ufw enable
```

### fail2ban: Protección contra brute force

**Configuración básica:**
```bash
# Instalar fail2ban
apt-get install fail2ban

# /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
backend = systemd

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 6

[nginx-noscript]
enabled = true
filter = nginx-noscript
logpath = /var/log/nginx/access.log
maxretry = 6

[nginx-badbots]
enabled = true
filter = nginx-badbots
logpath = /var/log/nginx/access.log
maxretry = 2
```

**Filtros personalizados:**
```bash
# /etc/fail2ban/filter.d/nginx-noscript.conf
[Definition]
failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\.scgi)
ignoreregex =

# /etc/fail2ban/filter.d/nginx-badbots.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*HTTP.*".*".*bot.*"$
ignoreregex =
```

## 🔒 Control de Acceso y Auditoría

### SELinux: Mandatory Access Control

**Conceptos básicos:**
```bash
# Verificar estado de SELinux
getenforce

# Modos de SELinux
# Enforcing: Políticas activas
# Permissive: Solo logging
# Disabled: Deshabilitado

# Configurar modo
setenforce 1  # Enforcing
# /etc/selinux/config
SELINUX=enforcing
SELINUXTYPE=targeted
```

**Gestión de contextos:**
```bash
# Ver contextos de archivos
ls -Z /var/www/html/

# Restaurar contextos por defecto
restorecon -R /var/www/html/

# Cambiar contexto manualmente
chcon -t httpd_exec_t /usr/local/bin/webapp

# Hacer cambio permanente
semanage fcontext -a -t httpd_exec_t "/usr/local/bin/webapp"
restorecon /usr/local/bin/webapp

# Ver políticas activas
getsebool -a | grep httpd

# Cambiar boolean
setsebool -P httpd_can_network_connect on
```

### AppArmor: Alternativa a SELinux

**Configuración básica:**
```bash
# Instalar AppArmor
apt-get install apparmor apparmor-utils

# Ver estados de perfiles
aa-status

# Modo complain (solo logging)
aa-complain /etc/apparmor.d/usr.bin.nginx

# Modo enforce (activo)
aa-enforce /etc/apparmor.d/usr.bin.nginx

# Generar perfil automáticamente
aa-genprof /usr/bin/aplicacion
```

**Perfil personalizado:**
```bash
# /etc/apparmor.d/usr.bin.webapp
#include <tunables/global>

/usr/bin/webapp {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  
  capability setuid,
  capability setgid,
  
  /usr/bin/webapp mr,
  /etc/webapp/ r,
  /var/lib/webapp/ rw,
  /var/log/webapp/ w,
  /tmp/ rw,
  
  deny /etc/shadow r,
  deny /etc/passwd w,
  deny /proc/sys/kernel/** w,
}
```

### Auditoría del sistema con auditd

**Configuración de auditd:**
```bash
# Instalar auditd
apt-get install auditd audispd-plugins

# /etc/audit/auditd.conf
log_file = /var/log/audit/audit.log
log_format = RAW
log_group = adm
priority_boost = 4
flush = INCREMENTAL
freq = 20
num_logs = 5
disp_qos = lossy
dispatcher = /sbin/audispd
name_format = HOSTNAME
max_log_file = 6
max_log_file_action = ROTATE
space_left = 75
space_left_action = SYSLOG
admin_space_left = 50
admin_space_left_action = SUSPEND
disk_full_action = SUSPEND
disk_error_action = SUSPEND
```

**Reglas de auditoría:**
```bash
# /etc/audit/rules.d/audit.rules

# Limpiar reglas anteriores
-D

# Buffer para reglas
-b 1024

# Failure mode
-f 1

# Auditar cambios en archivos críticos
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity

# Auditar configuración de red
-w /etc/network/ -p wa -k network_changes
-w /etc/hosts -p wa -k network_changes

# Auditar cambios en configuración del sistema
-w /etc/ssh/sshd_config -p wa -k sshd_config
-w /etc/crontab -p wa -k cron
-w /etc/cron.allow -p wa -k cron
-w /etc/cron.deny -p wa -k cron

# Auditar intentos de escalada de privilegios
-a always,exit -F arch=b64 -S setuid -S setgid -S setreuid -S setregid -k privilege_escalation
-a always,exit -F arch=b32 -S setuid -S setgid -S setreuid -S setregid -k privilege_escalation

# Auditar acceso a archivos sensibles
-a always,exit -F arch=b64 -S openat -F dir=/etc -F success=0 -k unautorized_access_attempts
-a always,exit -F arch=b32 -S openat -F dir=/etc -F success=0 -k unautorized_access_attempts

# Auditar modificaciones del kernel
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules

# Immutable (debe ir al final)
-e 2
```

**Análisis de logs de auditoría:**
```bash
# Buscar eventos específicos
ausearch -k identity
ausearch -k privilege_escalation
ausearch -ts today -te now

# Resumen de eventos
aureport --summary
aureport --auth
aureport --failed
aureport --login

# Análisis en tiempo real
ausearch -ts recent | tail -f
```

## 🛡️ Hardening del Kernel

### Parámetros del kernel (sysctl)

```bash
# /etc/sysctl.d/99-security.conf

# Network security
net.ipv4.ip_forward = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# IPv6 security
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Memory protection
kernel.randomize_va_space = 2
kernel.exec-shield = 1
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 1

# Process restrictions
fs.suid_dumpable = 0
kernel.core_uses_pid = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1

# Network DoS protection
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Apply changes
sysctl -p /etc/sysctl.d/99-security.conf
```

### Configuración de límites de recursos

```bash
# /etc/security/limits.conf

# Límites para usuarios normales
*               soft    core            0
*               hard    core            0
*               soft    nproc           1000
*               hard    nproc           1500
*               soft    nofile          4096
*               hard    nofile          8192

# Límites para usuario web server
www-data        soft    nproc           2000
www-data        hard    nproc           2000
www-data        soft    nofile          8192
www-data        hard    nofile          8192

# Prohibir login para usuarios de sistema
bin             -       nologin
daemon          -       nologin
sys             -       nologin
```

## 🔍 Monitoreo y Detección de Intrusiones

### AIDE: Advanced Intrusion Detection Environment

**Configuración inicial:**
```bash
# Instalar AIDE
apt-get install aide

# Configurar
# /etc/aide/aide.conf
database=file:/var/lib/aide/aide.db
database_out=file:/var/lib/aide/aide.db.new
gzip_dbout=yes

# Configurar qué monitorear
/bin            p+i+n+u+g+s+b+m+c+md5+sha1
/sbin           p+i+n+u+g+s+b+m+c+md5+sha1
/lib            p+i+n+u+g+s+b+m+c+md5+sha1
/lib64          p+i+n+u+g+s+b+m+c+md5+sha1
/opt            p+i+n+u+g+s+b+m+c+md5+sha1
/usr            p+i+n+u+g+s+b+m+c+md5+sha1
/root           p+i+n+u+g+s+b+m+c+md5+sha1

# Directorios que cambian frecuentemente
!/tmp
!/var/tmp
!/var/cache
!/var/log
!/var/spool
!/var/run

# Inicializar base de datos
aide --init
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Verificar integridad
aide --check

# Automatizar con cron
echo "0 6 * * * root /usr/bin/aide --check | mail -s 'AIDE Report' admin@domain.com" >> /etc/crontab
```

### Detección de rootkits

**rkhunter:**
```bash
# Instalar rkhunter
apt-get install rkhunter

# Configurar
# /etc/rkhunter.conf
UPDATE_MIRRORS=1
MIRRORS_MODE=0
WEB_CMD=""

# Actualizar definiciones
rkhunter --update

# Escaneo completo
rkhunter --check --sk

# Automatizar
echo "0 2 * * * root /usr/bin/rkhunter --check --sk --quiet" >> /etc/crontab
```

**chkrootkit:**
```bash
# Instalar chkrootkit
apt-get install chkrootkit

# Escaneo manual
chkrootkit

# Automatizar
echo "0 3 * * * root /usr/sbin/chkrootkit | grep -v 'nothing found' | mail -s 'Chkrootkit Report' admin@domain.com" >> /etc/crontab
```

### Monitoreo de logs con logwatch

```bash
# Instalar logwatch
apt-get install logwatch

# Configurar
# /etc/logwatch/conf/logwatch.conf
LogDir = /var/log
TmpDir = /var/cache/logwatch
MailTo = admin@domain.com
MailFrom = logwatch@servidor.com
Detail = High
Service = All
Range = yesterday
Format = html

# Ejecutar manualmente
logwatch --detail High --mailto admin@domain.com --range yesterday

# Automatizar
echo "0 7 * * * root /usr/sbin/logwatch" >> /etc/crontab
```

## 🔐 Cifrado y Protección de Datos

### Cifrado de disco con LUKS

**Configurar partición cifrada:**
```bash
# Preparar partición
cryptsetup luksFormat /dev/sdb1

# Abrir partición cifrada
cryptsetup luksOpen /dev/sdb1 encrypted_data

# Crear filesystem
mkfs.ext4 /dev/mapper/encrypted_data

# Montar
mkdir /mnt/secure_data
mount /dev/mapper/encrypted_data /mnt/secure_data

# Configurar montaje automático
# /etc/crypttab
encrypted_data /dev/sdb1 none luks

# /etc/fstab
/dev/mapper/encrypted_data /mnt/secure_data ext4 defaults 0 2
```

### GPG para cifrado de archivos

```bash
# Generar par de claves
gpg --gen-key

# Cifrar archivo
gpg --encrypt --recipient user@domain.com archivo.txt

# Descifrar archivo
gpg --decrypt archivo.txt.gpg > archivo.txt

# Firmar archivo
gpg --sign archivo.txt

# Verificar firma
gpg --verify archivo.txt.gpg
```

### Configuración de certificados SSL/TLS

```bash
# Generar clave privada
openssl genrsa -out server.key 4096

# Generar CSR
openssl req -new -key server.key -out server.csr

# Auto-firmar para testing (no para producción)
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# Configurar permisos seguros
chmod 600 server.key
chmod 644 server.crt
chown root:root server.key server.crt
```

## 🚨 Respuesta ante Incidentes

### Forense básico

**Preservar evidencia:**
```bash
# Crear imagen forense
dd if=/dev/sda of=/mnt/external/evidence.img bs=64K conv=noerror,sync

# Calcular hash para integridad
sha256sum /mnt/external/evidence.img > evidence.sha256

# Montar imagen para análisis
mkdir /mnt/analysis
mount -o ro,loop /mnt/external/evidence.img /mnt/analysis

# Buscar indicadores de compromiso
find /mnt/analysis -type f -name "*.php" -exec grep -l "eval\|base64_decode" {} \;
find /mnt/analysis -type f -newer /mnt/analysis/etc/passwd
```

**Análisis de conexiones de red:**
```bash
# Conexiones activas
netstat -tulpn
ss -tulpn

# Conexiones establecidas
netstat -an | grep ESTABLISHED

# Procesos escuchando
lsof -i

# Historial de conexiones (si está habilitado)
last -i
lastlog
```

### Plan de respuesta ante incidentes

**1. Detección:**
- Monitoreo automático de logs
- Alertas de IDS/IPS
- Comportamiento anómalo

**2. Contención:**
```bash
# Aislar sistema comprometido
iptables -A INPUT -j DROP
iptables -A OUTPUT -j DROP

# Preservar memoria RAM
dd if=/dev/mem of=/tmp/memory.dump

# Mantener logs
rsync -av /var/log/ backup-server:/incident-logs/
```

**3. Erradicación:**
```bash
# Eliminar malware identificado
rm -f /tmp/suspicious_file
killall -9 malicious_process

# Cerrar vectores de ataque
passwd -l compromised_user
```

**4. Recuperación:**
```bash
# Restaurar desde backup limpio
restore-from-backup.sh

# Aplicar parches de seguridad
apt-get update && apt-get upgrade

# Verificar integridad del sistema
aide --check
rkhunter --check
```

## 📋 Checklist de Hardening

### ✅ Configuración inicial
- [ ] Instalación mínima del sistema
- [ ] Configuración de usuarios y grupos
- [ ] Políticas de contraseñas
- [ ] Configuración segura de SSH
- [ ] Deshabilitación de servicios innecesarios

### ✅ Red y firewall
- [ ] Configuración de iptables/UFW
- [ ] Instalación y configuración de fail2ban
- [ ] Configuración de parámetros de red del kernel
- [ ] Deshabilitación de servicios de red innecesarios

### ✅ Control de acceso
- [ ] Configuración de SELinux/AppArmor
- [ ] Configuración de sudoers
- [ ] Límites de recursos del sistema
- [ ] Configuración de PAM

### ✅ Auditoría y monitoreo
- [ ] Configuración de auditd
- [ ] Instalación de AIDE
- [ ] Configuración de logwatch
- [ ] Instalación de herramientas anti-rootkit

### ✅ Cifrado y protección de datos
- [ ] Configuración de cifrado de disco
- [ ] Certificados SSL/TLS
- [ ] Backup cifrado
- [ ] Configuración de GPG

### ✅ Mantenimiento continuo
- [ ] Actualizaciones automáticas de seguridad
- [ ] Monitoreo de vulnerabilidades
- [ ] Revisiones regulares de configuración
- [ ] Pruebas de penetración periódicas

---

## 🎓 Conclusión

El hardening de servidores Linux es un proceso continuo que requiere:

1. **Configuración inicial segura** desde la instalación
2. **Múltiples capas de defensa** (firewall, IDS, access control)
3. **Monitoreo constante** y detección temprana
4. **Planes de respuesta** ante incidentes
5. **Actualizaciones regulares** y mejora continua

### Herramientas recomendadas

- **Lynis**: Auditoría de seguridad automatizada
- **OpenSCAP**: Compliance y vulnerability assessment
- **OSSEC**: Host-based intrusion detection
- **Tripwire**: File integrity monitoring
- **Nmap**: Network discovery y security auditing

## 🎓 Teoría Avanzada: Fundamentos de la Ciberseguridad

### El modelo CIA (Confidentiality, Integrity, Availability)

La ciberseguridad moderna se basa en tres pilares fundamentales que toda implementación de hardening debe proteger:

**Confidencialidad (Confidentiality):**
- **Definición**: Garantizar que la información solo sea accesible por entidades autorizadas
- **Implementación en Linux**: Cifrado (LUKS, GPG), control de acceso (DAC/MAC), permisos de archivos
- **Amenazas**: Interceptación de comunicaciones, acceso no autorizado, lateral movement

**Integridad (Integrity):**
- **Definición**: Asegurar que la información no sea modificada de manera no autorizada
- **Implementación en Linux**: Checksums (AIDE), firmas digitales (GPG), logs de auditoría (auditd)
- **Amenazas**: Modificación de archivos críticos, rootkits, tampering de logs

**Disponibilidad (Availability):**
- **Definición**: Garantizar que los recursos estén disponibles cuando se necesiten
- **Implementación en Linux**: Rate limiting, fail2ban, monitoreo proactivo, backups
- **Amenazas**: DoS/DDoS, resource exhaustion, ransomware

### Defensa en Profundidad: Teoría de Capas

El hardening efectivo implementa **múltiples capas de seguridad**, donde cada capa compensa las debilidades de las otras:

**Capa 1 - Perímetro de Red:**
```
Internet → Firewall → DMZ → Firewall Interno → Red Interna
```
- **Tecnologías**: iptables/nftables, fail2ban, IDS/IPS
- **Principio**: Filtrar tráfico malicioso antes de que llegue al servidor

**Capa 2 - Sistema Operativo:**
```
Kernel Hardening → Service Minimization → Access Control
```
- **Tecnologías**: sysctl, systemd, SELinux/AppArmor
- **Principio**: Reducir superficie de ataque y controlar acceso a recursos

**Capa 3 - Aplicación:**
```
Secure Coding → Input Validation → Output Encoding
```
- **Tecnologías**: WAF, sandboxing, containers
- **Principio**: Proteger la lógica de negocio y datos de aplicación

**Capa 4 - Datos:**
```
Encryption at Rest → Encryption in Transit → Access Logging
```
- **Tecnologías**: LUKS, TLS, database encryption
- **Principio**: Proteger los datos incluso si otras capas son comprometidas

### Análisis de Riesgo y Modelado de Amenazas

Antes de implementar hardening, es crucial entender **qué estamos protegiendo y de quién**:

**Metodología STRIDE:**
- **S**poofing: Suplantación de identidad
- **T**ampering: Alteración de datos
- **R**epudiation: Negación de acciones
- **I**nformation Disclosure: Divulgación de información
- **D**enial of Service: Denegación de servicio
- **E**levation of Privilege: Escalada de privilegios

**Matriz de Riesgo:**
```
Impacto vs Probabilidad

Alto     | Medio  | Crítico | Crítico
Medio    | Bajo   | Medio   | Alto
Bajo     | Bajo   | Bajo    | Medio
         Baja    Media     Alta
              Probabilidad
```

### Principios Criptográficos en Hardening

**Algoritmos de Cifrado Modernos:**

**Cifrado Simétrico:**
- **AES-256**: Estándar actual para cifrado de datos
- **ChaCha20**: Alternativa moderna, especialmente en móviles
- **Uso en Linux**: LUKS utiliza AES-256-XTS por defecto

**Cifrado Asimétrico:**
- **RSA-4096**: Estándar establecido, computacionalmente intensivo
- **Ed25519**: Curva elíptica moderna, más eficiente que RSA
- **Uso en Linux**: SSH keys, GPG, certificados SSL/TLS

**Funciones Hash:**
- **SHA-256/SHA-3**: Para integridad de archivos
- **bcrypt/Argon2**: Para hashing de contraseñas
- **HMAC**: Para autenticación de mensajes

**Perfect Forward Secrecy (PFS):**
Principio donde el compromiso de una clave privada no compromete sesiones pasadas:
```bash
# Configuración TLS con PFS
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256;
ssl_prefer_server_ciphers on;
```

### Teoría de Detección de Intrusiones

**Tipos de IDS/IPS:**

**1. Network-based (NIDS):**
- **Principio**: Analiza tráfico de red en tiempo real
- **Ventajas**: Vista completa del tráfico, detección de ataques de red
- **Desventajas**: Problemas con tráfico cifrado, false positives

**2. Host-based (HIDS):**
- **Principio**: Monitorea actividad en el host específico
- **Ventajas**: Detección de ataques internos, análisis de logs detallado
- **Desventajas**: Consume recursos del host, vista limitada

**Técnicas de Detección:**

**Signature-based Detection:**
```python
# Ejemplo de regla YARA para detección de malware
rule Linux_Rootkit
{
    strings:
        $a = "/proc/self/exe"
        $b = "LD_PRELOAD"
        $c = "hook"
    condition:
        all of them
}
```

**Anomaly-based Detection:**
- **Machine Learning**: Detecta desviaciones del comportamiento normal
- **Statistical Analysis**: Usa baseline estadístico para detección
- **Behavioral Analysis**: Analiza patrones de comportamiento de usuarios/procesos

### Compliance y Frameworks de Seguridad

**ISO 27001/27002:**
Framework internacional para gestión de seguridad de la información:
- **Control A.12.6.1**: Gestión de vulnerabilidades técnicas
- **Control A.13.1.1**: Controles de red
- **Control A.14.1.3**: Protección de transacciones de servicios de aplicaciones

**NIST Cybersecurity Framework:**
```
Identify → Protect → Detect → Respond → Recover
```
- **Identify**: Asset management, risk assessment
- **Protect**: Access control, data security
- **Detect**: Anomaly detection, monitoring
- **Respond**: Incident response, communications
- **Recover**: Recovery planning, improvements

**CIS Controls:**
Los 20 controles críticos de seguridad:
1. Inventory of Authorized/Unauthorized Devices
2. Inventory of Authorized/Unauthorized Software
3. Continuous Vulnerability Management
4. Controlled Use of Administrative Privileges
5. Secure Configuration of Hardware/Software

### Hardening Basado en Threat Intelligence

**Pyramid of Pain:**
Jerarquía de indicadores que causan diferentes niveles de "dolor" a los atacantes:

```
         TTPs (Tactics, Techniques, Procedures)
              ↑ (Difícil de cambiar)
         Tools (Herramientas utilizadas)
              ↑
    Network/Host Artifacts (Comportamientos)
              ↑
      Domain Names (Dominios utilizados)
              ↑
     IP Addresses (Direcciones IP)
              ↑
    Hash Values (Hashes de malware)
              ↓ (Fácil de cambiar)
```

**Implementación en Hardening:**
- **Hash-based blocking**: fail2ban con listas de IPs maliciosas
- **Domain blocking**: DNS sinkholing con unbound/bind
- **Behavioral detection**: auditd rules para TTPs específicos

### Recursos adicionales

- **CIS Benchmarks**: https://www.cisecurity.org/cis-benchmarks/
- **NIST Cybersecurity Framework**: https://www.nist.gov/cyberframework
- **OWASP Server Side Request Forgery Prevention**: https://owasp.org/
- **SANS Critical Security Controls**: https://www.sans.org/critical-security-controls/
- **MITRE ATT&CK Framework**: https://attack.mitre.org/

---

## 📝 Reflexiones Finales

El hardening de servidores Linux no es una tarea que se hace una vez y se olvida. Es un **proceso continuo** que requiere:

1. **Entendimiento profundo** de los principios de seguridad
2. **Implementación layered** de controles de seguridad  
3. **Monitoreo constante** y mejora continua
4. **Actualización regular** de conocimientos y herramientas
5. **Testing periódico** de la efectividad de las medidas

Como administradores de sistemas, nuestra responsabilidad va más allá de mantener los servidores funcionando. Debemos proteger la información y recursos de nuestras organizaciones contra amenazas cada vez más sofisticadas.

El tiempo invertido en hardening apropiado **siempre** se compensa con creces cuando previene un incidente de seguridad que podría costar miles de veces más en recuperación, multas regulatorias y daño reputacional.

¿Tienes un servidor que necesita hardening? ¡Empieza con el checklist y adapta las configuraciones a tu entorno específico!

---
**Andrés Nuñez - t4ifi**
