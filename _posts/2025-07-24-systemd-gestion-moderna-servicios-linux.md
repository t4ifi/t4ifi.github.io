---
title: "Systemd: Gestión Moderna de Servicios y Sistema en Linux"
date: 2025-07-24 10:00:00 +0000
categories: [Administración, Servicios]
tags: [systemd, servicios, init, administración, linux]
---

# Systemd: Gestión Moderna de Servicios y Sistema en Linux

Systemd ha revolucionado la administración de sistemas Linux, reemplazando al tradicional System V init con un enfoque moderno, paralelo y rico en características.

## Introducción a Systemd

### ¿Qué es Systemd?

Systemd es un sistema init y gestor de servicios para Linux que proporciona un marco completo para gestionar el arranque del sistema, servicios, procesos daemon y recursos del sistema.

### Filosofía y Diseño

#### Características Principales
- **Arranque Paralelo**: Servicios se inician en paralelo cuando es posible
- **Gestión Basada en Dependencias**: Control granular de dependencias
- **On-Demand Starting**: Servicios se inician solo cuando se necesitan
- **Socket Activation**: Servicios se inician cuando reciben conexiones
- **Logging Centralizado**: Journal integrado para logs del sistema

#### Comparación con System V Init
```bash
# System V Init (tradicional)
/etc/init.d/apache2 start
service apache2 restart
chkconfig apache2 on

# Systemd (moderno)
systemctl start apache2
systemctl restart apache2
systemctl enable apache2
```

## Arquitectura de Systemd

### Componentes Principales

#### 1. systemd (PID 1)
```bash
# Proceso principal del sistema
ps aux | grep systemd
pstree -p 1

# Información del sistema
systemctl --version
```

#### 2. Units (Unidades)
```bash
# Tipos de units
systemctl list-units --type=service
systemctl list-units --type=socket
systemctl list-units --type=timer
systemctl list-units --type=mount
systemctl list-units --type=target
```

#### 3. Targets (Objetivos)
```bash
# Equivalentes a runlevels
systemctl list-units --type=target

# Target actual
systemctl get-default

# Cambiar target por defecto
systemctl set-default multi-user.target
```

### Directorios de Configuración
```bash
# Directorios principales
ls -la /etc/systemd/system/       # Configuraciones locales
ls -la /usr/lib/systemd/system/   # Configuraciones del sistema
ls -la /run/systemd/system/       # Configuraciones runtime

# Jerarquía de prioridad
# /etc/systemd/system/ (mayor prioridad)
# /run/systemd/system/ (prioridad media)
# /usr/lib/systemd/system/ (menor prioridad)
```

## Gestión de Servicios

### Comandos Básicos de Systemctl

#### Control de Servicios
```bash
# Iniciar servicio
systemctl start apache2

# Detener servicio
systemctl stop apache2

# Reiniciar servicio
systemctl restart apache2

# Recargar configuración sin reiniciar
systemctl reload apache2

# Reiniciar solo si está activo
systemctl try-restart apache2

# Recargar o reiniciar según disponibilidad
systemctl reload-or-restart apache2
```

#### Estado de Servicios
```bash
# Estado detallado
systemctl status apache2

# Estado simple
systemctl is-active apache2
systemctl is-enabled apache2
systemctl is-failed apache2

# Listar servicios fallidos
systemctl --failed
```

#### Habilitación y Deshabilitación
```bash
# Habilitar servicio (autostart)
systemctl enable apache2

# Deshabilitar servicio
systemctl disable apache2

# Habilitar e iniciar en un comando
systemctl enable --now apache2

# Enmascarar servicio (prevenir inicio)
systemctl mask apache2
systemctl unmask apache2
```

### Información y Listado

#### Listar Servicios
```bash
# Todos los servicios
systemctl list-units --type=service

# Solo servicios activos
systemctl list-units --type=service --state=active

# Solo servicios habilitados
systemctl list-unit-files --type=service --state=enabled

# Servicios que fallaron al iniciar
systemctl list-units --type=service --state=failed
```

#### Dependencias de Servicios
```bash
# Ver dependencias
systemctl list-dependencies apache2

# Dependencias recursivas
systemctl list-dependencies apache2 --all

# Qué servicios dependen de este
systemctl list-dependencies apache2 --reverse
```

## Creación de Service Units

### Estructura de Unit Files

#### Service Unit Básico
```ini
# /etc/systemd/system/mi-app.service
[Unit]
Description=Mi Aplicación Web
Documentation=https://mi-app.com/docs
After=network.target mysql.service
Wants=mysql.service
Requires=network.target

[Service]
Type=forking
User=mi-app
Group=mi-app
WorkingDirectory=/opt/mi-app
ExecStartPre=/opt/mi-app/scripts/pre-start.sh
ExecStart=/opt/mi-app/bin/start.sh
ExecReload=/opt/mi-app/bin/reload.sh
ExecStop=/opt/mi-app/bin/stop.sh
PIDFile=/var/run/mi-app/mi-app.pid
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### Tipos de Servicios
```bash
# Type=simple (por defecto)
[Service]
Type=simple
ExecStart=/usr/bin/mi-daemon

# Type=forking (daemon tradicional)
[Service]
Type=forking
ExecStart=/usr/bin/mi-daemon
PIDFile=/var/run/mi-daemon.pid

# Type=oneshot (tareas que terminan)
[Service]
Type=oneshot
ExecStart=/usr/bin/mi-script.sh
RemainAfterExit=yes

# Type=notify (notifica cuando está listo)
[Service]
Type=notify
ExecStart=/usr/bin/mi-daemon-notify

# Type=idle (espera a que otros terminen)
[Service]
Type=idle
ExecStart=/usr/bin/mi-daemon-idle
```

### Configuración Avanzada de Servicios

#### Gestión de Recursos
```ini
[Service]
# Limites de CPU
CPUQuota=50%
CPUWeight=100

# Limites de memoria
MemoryMax=1G
MemoryHigh=800M

# Limites de I/O
IOWeight=100
IOReadBandwidthMax=/dev/sda 10M

# Limites de procesos
TasksMax=100

# Control de OOM
OOMPolicy=kill
OOMScoreAdjust=-100
```

#### Seguridad y Sandboxing
```ini
[Service]
# Ejecución como usuario específico
User=mi-app
Group=mi-app
DynamicUser=yes

# Restricciones de filesystem
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/mi-app /var/log/mi-app

# Restricciones de red
PrivateNetwork=yes
# o
RestrictAddressFamilies=AF_INET AF_INET6

# Capacidades
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Namespaces
PrivateUsers=yes
ProtectHostname=yes
ProtectKernelTunables=yes
```

#### Variables de Entorno
```ini
[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk"
Environment="APP_CONFIG=/etc/mi-app/config.yml"
EnvironmentFile=/etc/mi-app/environment
```

## Socket Activation

### Crear Socket Units

#### Socket Unit
```ini
# /etc/systemd/system/mi-app.socket
[Unit]
Description=Mi App Socket
Documentation=https://mi-app.com/docs

[Socket]
ListenStream=8080
ListenStream=[::]:8080
Accept=no
SocketUser=mi-app
SocketGroup=mi-app
SocketMode=0660

[Install]
WantedBy=sockets.target
```

#### Service asociado
```ini
# /etc/systemd/system/mi-app.service
[Unit]
Description=Mi App Service
Requires=mi-app.socket

[Service]
Type=notify
ExecStart=/opt/mi-app/bin/server
StandardInput=socket
User=mi-app
Group=mi-app
```

#### Activación y Uso
```bash
# Habilitar e iniciar socket
systemctl enable --now mi-app.socket

# Verificar socket
systemctl status mi-app.socket
ss -tlnp | grep :8080

# El servicio se iniciará automáticamente
# cuando alguien se conecte al puerto 8080
```

## Timer Units: Cron Mejorado

### Crear Timer Units

#### Timer Unit
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Backup Timer
Requires=backup.service

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=3600

[Install]
WantedBy=timers.target
```

#### Service asociado
```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup
Documentation=file:///etc/backup/README

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=backup
Group=backup
```

#### Sintaxis de Tiempo
```bash
# Ejemplos de OnCalendar
OnCalendar=minutely           # Cada minuto
OnCalendar=hourly             # Cada hora
OnCalendar=daily              # Diariamente a medianoche
OnCalendar=weekly             # Semanalmente los lunes
OnCalendar=monthly            # Mensualmente el día 1
OnCalendar=yearly             # Anualmente el 1 de enero

# Sintaxis específica
OnCalendar=Mon..Fri 09:00     # Lunes a viernes a las 9:00
OnCalendar=*-*-01 00:00:00    # Primer día de cada mes
OnCalendar=2025-07-24 15:30   # Fecha específica

# Intervalos
OnBootSec=15min               # 15 min después del boot
OnUnitActiveSec=1h            # 1 hora después de la última activación
```

#### Gestionar Timers
```bash
# Listar timers
systemctl list-timers

# Habilitar timer
systemctl enable --now backup.timer

# Estado del timer
systemctl status backup.timer

# Próximas ejecuciones
systemctl list-timers backup.timer
```

## Mount Units y Automount

### Mount Units

#### Mount Unit Manual
```ini
# /etc/systemd/system/mnt-datos.mount
[Unit]
Description=Mount /mnt/datos
Documentation=file:///etc/fstab

[Mount]
What=/dev/disk/by-uuid/12345678-1234-1234-1234-123456789012
Where=/mnt/datos
Type=ext4
Options=defaults,noatime

[Install]
WantedBy=multi-user.target
```

#### Automount Unit
```ini
# /etc/systemd/system/mnt-datos.automount
[Unit]
Description=Automount /mnt/datos
Documentation=file:///etc/fstab

[Automount]
Where=/mnt/datos
TimeoutIdleSec=60

[Install]
WantedBy=multi-user.target
```

#### Generar desde fstab
```bash
# Generar units desde /etc/fstab
systemd-fstab-generator

# Ver units generadas
ls /run/systemd/generator/
```

## Targets: Sistema de Runlevels

### Targets Principales
```bash
# Ver todos los targets
systemctl list-units --type=target

# Targets comunes
systemctl list-units --type=target | grep -E "(graphical|multi-user|rescue)"
```

#### Correspondencia con Runlevels
```bash
# Runlevel 0 -> poweroff.target
# Runlevel 1 -> rescue.target
# Runlevel 2,3,4 -> multi-user.target
# Runlevel 5 -> graphical.target
# Runlevel 6 -> reboot.target
```

### Crear Target Personalizado
```ini
# /etc/systemd/system/mi-servidor.target
[Unit]
Description=Mi Servidor Web Target
Documentation=file:///etc/mi-servidor/README
Requires=multi-user.target
After=multi-user.target
AllowIsolate=yes

[Install]
WantedBy=multi-user.target
```

#### Asociar Servicios al Target
```bash
# Modificar Install section en services
[Install]
WantedBy=mi-servidor.target

# O crear enlaces manualmente
mkdir -p /etc/systemd/system/mi-servidor.target.wants
ln -s /etc/systemd/system/apache2.service /etc/systemd/system/mi-servidor.target.wants/
```

### Cambiar Targets
```bash
# Cambiar target temporalmente
systemctl isolate rescue.target

# Cambiar target por defecto
systemctl set-default graphical.target
systemctl get-default

# Arrancar en target específico (GRUB)
# Agregar al final de la línea del kernel:
# systemd.unit=rescue.target
```

## Logging con journalctl

### Consultar Logs

#### Comandos Básicos
```bash
# Ver todos los logs
journalctl

# Logs desde el último boot
journalctl -b

# Logs desde boot específico
journalctl -b -1  # Boot anterior
journalctl --list-boots

# Seguir logs en tiempo real
journalctl -f

# Logs de servicio específico
journalctl -u apache2
journalctl -u apache2 -f
```

#### Filtros por Tiempo
```bash
# Desde hace una hora
journalctl --since "1 hour ago"

# Desde ayer
journalctl --since yesterday

# Rango específico
journalctl --since "2025-07-24 10:00" --until "2025-07-24 11:00"

# Últimas líneas
journalctl -n 50
journalctl -u apache2 -n 100
```

#### Filtros por Prioridad
```bash
# Solo errores
journalctl -p err

# Errores y warnings
journalctl -p warning

# Prioridades: emerg, alert, crit, err, warning, notice, info, debug
journalctl -p err..crit
```

### Configuración del Journal

#### Configurar journald
```bash
# Editar configuración
vim /etc/systemd/journald.conf

# Configuraciones importantes
[Journal]
Storage=persistent
Compress=yes
SystemMaxUse=500M
SystemMaxFileSize=100M
MaxRetentionSec=1month
```

#### Gestión de Espacio
```bash
# Ver uso de espacio
journalctl --disk-usage

# Limpiar logs antiguos
journalctl --vacuum-time=30d
journalctl --vacuum-size=500M

# Verificar journal
journalctl --verify
```

## Análisis de Performance

### Tiempo de Arranque

#### Analizar Boot Time
```bash
# Tiempo total de arranque
systemd-analyze

# Servicios más lentos
systemd-analyze blame

# Cadena crítica de arranque
systemd-analyze critical-chain

# Gráfico de arranque (SVG)
systemd-analyze plot > boot-analysis.svg
```

#### Tiempo de Servicios
```bash
# Tiempo de un servicio específico
systemd-analyze blame | grep apache2

# Cadena crítica de un servicio
systemd-analyze critical-chain apache2.service
```

### Debugging de Servicios

#### Modo Debug
```bash
# Configurar debug level
systemctl log-level debug

# Debug de servicio específico
systemd-run --uid=1000 --gid=1000 --pty bash

# Ver configuración efectiva
systemctl cat apache2.service
systemctl show apache2.service
```

#### Verificar Configuración
```bash
# Verificar sintaxis de unit files
systemd-analyze verify /etc/systemd/system/mi-app.service

# Recargar configuración
systemctl daemon-reload

# Ver dependencias rotas
systemctl list-dependencies --failed
```

## Administración Avanzada

### Namespaces y Contenedores

#### Servicios Portables
```bash
# Crear servicio portable
mkdir -p /var/lib/portables/mi-app.raw
# ... configurar imagen ...

# Adjuntar servicio portable
portablectl attach mi-app.raw

# Listar servicios portables
portablectl list

# Habilitar servicio portable
systemctl enable mi-app.service
```

#### Máquinas y Contenedores
```bash
# Listar máquinas
machinectl list

# Servicios en máquina específica
systemctl -M mi-container status

# Logs de máquina
journalctl -M mi-container
```

### Configuración de Red

#### networkd
```bash
# Habilitar networkd
systemctl enable systemd-networkd
systemctl enable systemd-resolved

# Configurar interfaz
# /etc/systemd/network/eth0.network
[Match]
Name=eth0

[Network]
DHCP=yes
DNS=8.8.8.8
DNS=8.8.4.4

# Aplicar configuración
networkctl reload
networkctl status
```

### Gestión de Tiempo

#### timesyncd
```bash
# Configurar NTP
# /etc/systemd/timesyncd.conf
[Time]
NTP=pool.ntp.org
FallbackNTP=time.cloudflare.com

# Habilitar sincronización
systemctl enable systemd-timesyncd
timedatectl set-ntp true

# Estado del tiempo
timedatectl status
```

## Migración desde SysV Init

### Convertir Scripts Init

#### Script SysV típico
```bash
#!/bin/bash
# /etc/init.d/mi-app

case "$1" in
  start)
    echo "Starting mi-app"
    /opt/mi-app/bin/start.sh
    ;;
  stop)
    echo "Stopping mi-app"
    /opt/mi-app/bin/stop.sh
    ;;
  restart)
    $0 stop
    $0 start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
    ;;
esac
```

#### Unit File equivalente
```ini
[Unit]
Description=Mi Aplicación
After=network.target

[Service]
Type=forking
ExecStart=/opt/mi-app/bin/start.sh
ExecStop=/opt/mi-app/bin/stop.sh
PIDFile=/var/run/mi-app.pid

[Install]
WantedBy=multi-user.target
```

### Herramientas de Migración
```bash
# Convertir crontab a timers
# Usar systemd-crontab-generator o convertir manualmente

# Migrar syslog a journal
# Configurar rsyslog para reenviar a journal
# o usar solo journal
```

## Best Practices y Optimización

### Diseño de Services
```ini
# Service bien diseñado
[Unit]
Description=Servicio Web de Producción
Documentation=https://docs.ejemplo.com
After=network-online.target mysql.service redis.service
Wants=network-online.target
Requires=mysql.service

[Service]
Type=notify
User=webapp
Group=webapp
WorkingDirectory=/opt/webapp

# Preparación y limpieza
ExecStartPre=/opt/webapp/scripts/check-deps.sh
ExecStartPost=/opt/webapp/scripts/register-service.sh
ExecStopPost=/opt/webapp/scripts/cleanup.sh

# Comando principal
ExecStart=/opt/webapp/bin/server
ExecReload=/bin/kill -HUP $MAINPID

# Política de reinicio
Restart=always
RestartSec=5
StartLimitBurst=3
StartLimitIntervalSec=60

# Recursos y seguridad
MemoryMax=1G
CPUQuota=50%
PrivateTmp=yes
ProtectSystem=strict
ReadWritePaths=/var/lib/webapp /var/log/webapp

[Install]
WantedBy=multi-user.target
```

### Monitoreo y Alertas
```bash
#!/bin/bash
# monitor_systemd.sh

# Verificar servicios críticos
CRITICAL_SERVICES=(
    "apache2.service"
    "mysql.service"
    "redis.service"
)

for service in "${CRITICAL_SERVICES[@]}"; do
    if ! systemctl is-active --quiet "$service"; then
        echo "ALERT: $service is not running" | \
            mail -s "Service Down: $service" admin@ejemplo.com
        
        # Intentar reiniciar
        systemctl restart "$service"
    fi
done

# Verificar servicios fallidos
FAILED_COUNT=$(systemctl --failed --no-legend | wc -l)
if [ "$FAILED_COUNT" -gt 0 ]; then
    systemctl --failed | \
        mail -s "Failed Services: $FAILED_COUNT" admin@ejemplo.com
fi
```

### Performance Tuning
```bash
# Configurar en /etc/systemd/system.conf
[Manager]
DefaultTimeoutStartSec=90s
DefaultTimeoutStopSec=30s
DefaultRestartSec=100ms
DefaultLimitNOFILE=65536
DefaultLimitNPROC=4096

# Configurar en /etc/systemd/logind.conf
[Login]
HandlePowerKey=poweroff
HandleSuspendKey=suspend
IdleAction=ignore
```

## Conclusiones

Systemd proporciona:

1. **Gestión Moderna**: Arranque paralelo y gestión basada en dependencias
2. **Flexibilidad**: Units especializadas para diferentes necesidades
3. **Seguridad**: Sandboxing avanzado y control de recursos
4. **Observabilidad**: Logging centralizado y análisis de performance
5. **Integración**: Ecosystem unificado para administración del sistema

Dominar systemd es esencial para cualquier administrador de sistemas Linux moderno.

---

*Andrés Núñez*
