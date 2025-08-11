---
title: "Logs y Análisis Forense en Linux: Investigación y Seguridad de Sistemas"
date: 2025-06-30 18:30:00 +0100
categories: [Linux, Seguridad]
tags: [logs, forensics, security, monitoring, syslog, linux, analysis]
---

# Logs y Análisis Forense en Linux: Investigación y Seguridad de Sistemas

Los **logs del sistema** son la caja negra de cualquier servidor Linux. Contienen una riqueza de información sobre todo lo que ocurre en el sistema: conexiones, errores, actividad de usuarios, procesos y mucho más. Dominar el análisis de logs es fundamental para la administración de sistemas, troubleshooting y, especialmente, para la **investigación forense** y detección de incidentes de seguridad.

## Fundamentos del Sistema de Logging

### Arquitectura del Logging en Linux

En Linux, el sistema de logging tradicionalmente se basa en **syslog**, aunque sistemas modernos también utilizan **systemd-journald**. Entender ambos es crucial para un análisis efectivo.

```bash
# Ver servicios de logging activos
systemctl status rsyslog
systemctl status systemd-journald

# Verificar configuración de rsyslog
cat /etc/rsyslog.conf
ls -la /etc/rsyslog.d/
```

### Facilidades y Prioridades de Syslog

```bash
# Facilidades (qué tipo de programa genera el log)
# kern, user, mail, daemon, auth, syslog, lpr, news, uucp, cron, authpriv, ftp
# local0-local7

# Prioridades/Severidades (importancia del mensaje)
# emerg(0), alert(1), crit(2), err(3), warning(4), notice(5), info(6), debug(7)

# Ejemplo de configuración en /etc/rsyslog.conf
# mail.info          /var/log/mail.log
# auth,authpriv.*    /var/log/auth.log
# *.emerg           :omusrmsg:*
```

## Ubicaciones Principales de Logs

### Logs del Sistema

```bash
# Logs principales en /var/log/
/var/log/syslog          # Log general del sistema (Debian/Ubuntu)
/var/log/messages        # Log general del sistema (CentOS/RHEL)
/var/log/auth.log        # Autenticación (Debian/Ubuntu)
/var/log/secure          # Autenticación (CentOS/RHEL)
/var/log/kern.log        # Kernel
/var/log/dmesg           # Mensajes del kernel al arranque
/var/log/boot.log        # Proceso de arranque
/var/log/cron.log        # Tareas cron
/var/log/mail.log        # Servidor de correo
/var/log/apache2/        # Apache web server
/var/log/nginx/          # Nginx web server
/var/log/mysql/          # MySQL/MariaDB
/var/log/postgresql/     # PostgreSQL

# Ver todos los logs disponibles
ls -la /var/log/
find /var/log -name "*.log" -type f
```

### Journald (systemd)

```bash
# Ver logs del journal
journalctl

# Logs desde el último arranque
journalctl -b

# Logs en tiempo real
journalctl -f

# Logs de un servicio específico
journalctl -u nginx
journalctl -u ssh

# Logs por prioridad
journalctl -p err        # Solo errores y más críticos
journalctl -p warning    # Warnings y más críticos

# Logs por tiempo
journalctl --since "2025-06-30 10:00:00"
journalctl --since "1 hour ago"
journalctl --until "2025-06-30 18:00:00"

# Logs de usuario específico
journalctl _UID=1000
```

## Herramientas de Análisis de Logs

### Comandos Básicos de Análisis

```bash
# Ver últimas líneas de un log
tail -f /var/log/syslog
tail -n 100 /var/log/auth.log

# Buscar patrones específicos
grep "Failed password" /var/log/auth.log
grep -i "error" /var/log/syslog
grep -E "(failed|error|critical)" /var/log/syslog

# Buscar con contexto
grep -A 5 -B 5 "kernel panic" /var/log/syslog

# Buscar en múltiples archivos
grep -r "suspicious activity" /var/log/

# Contar ocurrencias
grep -c "Failed password" /var/log/auth.log
grep "Failed password" /var/log/auth.log | wc -l
```

### Análisis Estadístico con awk y sed

```bash
# Extraer IPs de intentos fallidos de SSH
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr

# Top 10 IPs con más intentos fallidos
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head -10

# Análisis de conexiones SSH exitosas por usuario
grep "Accepted password" /var/log/auth.log | awk '{print $9}' | sort | uniq -c | sort -nr

# Análisis de errores de Apache por código de estado
awk '($9 >= 400)' /var/log/apache2/access.log | awk '{print $9}' | sort | uniq -c | sort -nr

# Análisis de tráfico por hora
awk '{print $4}' /var/log/apache2/access.log | cut -d: -f2 | sort | uniq -c

# IPs que más recursos consumen
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -20
```

### Scripts de Análisis Automatizado

#### Script de Análisis de Seguridad

```bash
#!/bin/bash
# security_log_analysis.sh - Análisis automatizado de logs de seguridad

set -e

LOG_FILE="/var/log/auth.log"
REPORT_FILE="/tmp/security_report_$(date +%Y%m%d).txt"
ALERT_THRESHOLD=50

# Función de logging
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$REPORT_FILE"
}

# Análisis de intentos de login fallidos
analyze_failed_logins() {
    log "=== ANÁLISIS DE INTENTOS DE LOGIN FALLIDOS ==="
    
    local failed_attempts=$(grep "Failed password" "$LOG_FILE" | wc -l)
    log "Total de intentos fallidos: $failed_attempts"
    
    if [ "$failed_attempts" -gt "$ALERT_THRESHOLD" ]; then
        log "⚠️  ALERTA: Número elevado de intentos fallidos detectado!"
    fi
    
    log "Top 10 IPs con más intentos fallidos:"
    grep "Failed password" "$LOG_FILE" | \
    awk '{print $11}' | \
    sort | uniq -c | sort -nr | head -10 | \
    while read count ip; do
        log "  $ip: $count intentos"
        
        # Verificar si la IP debe ser bloqueada
        if [ "$count" -gt 20 ]; then
            log "    ⚠️  IP $ip supera 20 intentos - considerar bloqueo"
        fi
    done
    
    log ""
}

# Análisis de logins exitosos
analyze_successful_logins() {
    log "=== ANÁLISIS DE LOGINS EXITOSOS ==="
    
    local successful_logins=$(grep "Accepted password\|Accepted publickey" "$LOG_FILE" | wc -l)
    log "Total de logins exitosos: $successful_logins"
    
    log "Usuarios con logins exitosos:"
    grep "Accepted password\|Accepted publickey" "$LOG_FILE" | \
    awk '{print $9}' | \
    sort | uniq -c | sort -nr | \
    while read count user; do
        log "  $user: $count logins"
    done
    
    log "IPs de logins exitosos:"
    grep "Accepted password\|Accepted publickey" "$LOG_FILE" | \
    awk '{print $11}' | \
    sort | uniq -c | sort -nr | head -10 | \
    while read count ip; do
        log "  $ip: $count logins"
    done
    
    log ""
}

# Análisis de actividad de sudo
analyze_sudo_activity() {
    log "=== ANÁLISIS DE ACTIVIDAD SUDO ==="
    
    if grep -q "sudo" "$LOG_FILE"; then
        local sudo_attempts=$(grep "sudo" "$LOG_FILE" | wc -l)
        log "Total de comandos sudo: $sudo_attempts"
        
        log "Usuarios que usaron sudo:"
        grep "sudo" "$LOG_FILE" | \
        awk '{print $5}' | \
        sed 's/://' | \
        sort | uniq -c | sort -nr | \
        while read count user; do
            log "  $user: $count comandos"
        done
        
        # Comandos sudo fallidos
        local failed_sudo=$(grep "sudo.*authentication failure" "$LOG_FILE" | wc -l)
        if [ "$failed_sudo" -gt 0 ]; then
            log "⚠️  Intentos sudo fallidos: $failed_sudo"
        fi
    else
        log "No se encontró actividad sudo en el log."
    fi
    
    log ""
}

# Análisis de nuevas conexiones
analyze_new_connections() {
    log "=== ANÁLISIS DE NUEVAS CONEXIONES ==="
    
    local new_connections=$(grep "New session" "$LOG_FILE" | wc -l)
    log "Total de nuevas sesiones: $new_connections"
    
    # Conexiones por hora
    log "Distribución de conexiones por hora:"
    grep "New session" "$LOG_FILE" | \
    awk '{print $3}' | \
    cut -d: -f1 | \
    sort | uniq -c | \
    sort -k2 -n | \
    while read count hour; do
        log "  ${hour}:00 - $count conexiones"
    done
    
    log ""
}

# Detección de patrones sospechosos
detect_suspicious_patterns() {
    log "=== DETECCIÓN DE PATRONES SOSPECHOSOS ==="
    
    # Intentos de login fuera de horario normal
    local night_attempts=$(grep "Failed password" "$LOG_FILE" | \
    awk '$3 ~ /^(2[0-3]|0[0-6]):/' | wc -l)
    
    if [ "$night_attempts" -gt 10 ]; then
        log "⚠️  Intentos de login sospechosos fuera de horario: $night_attempts"
    fi
    
    # Múltiples usuarios desde misma IP
    log "IPs que intentaron múltiples usuarios:"
    grep "Failed password" "$LOG_FILE" | \
    awk '{print $11 " " $9}' | \
    sort | uniq | \
    awk '{print $1}' | \
    sort | uniq -c | \
    awk '$1 > 3 {print $2 ": " $1 " usuarios diferentes"}' | \
    while read line; do
        log "  ⚠️  $line"
    done
    
    # Verificar intentos de escalación de privilegios
    local su_attempts=$(grep "su.*authentication failure" "$LOG_FILE" | wc -l)
    if [ "$su_attempts" -gt 0 ]; then
        log "⚠️  Intentos fallidos de su: $su_attempts"
    fi
    
    log ""
}

# Generar recomendaciones
generate_recommendations() {
    log "=== RECOMENDACIONES DE SEGURIDAD ==="
    
    # Analizar configuración SSH
    if [ -f "/etc/ssh/sshd_config" ]; then
        if grep -q "PermitRootLogin yes" /etc/ssh/sshd_config; then
            log "⚠️  RECOMENDACIÓN: Deshabilitar login directo de root"
        fi
        
        if ! grep -q "MaxAuthTries" /etc/ssh/sshd_config; then
            log "⚠️  RECOMENDACIÓN: Configurar MaxAuthTries en SSH"
        fi
        
        if ! grep -q "AllowUsers\|AllowGroups" /etc/ssh/sshd_config; then
            log "⚠️  RECOMENDACIÓN: Limitar usuarios SSH con AllowUsers"
        fi
    fi
    
    # Verificar fail2ban
    if ! command -v fail2ban-client > /dev/null; then
        log "⚠️  RECOMENDACIÓN: Instalar fail2ban para protección automática"
    fi
    
    log ""
}

# Función principal
main() {
    log "Iniciando análisis de seguridad de logs"
    log "Log analizado: $LOG_FILE"
    log "Período: $(head -1 "$LOG_FILE" | awk '{print $1, $2, $3}') - $(tail -1 "$LOG_FILE" | awk '{print $1, $2, $3}')"
    log ""
    
    analyze_failed_logins
    analyze_successful_logins
    analyze_sudo_activity
    analyze_new_connections
    detect_suspicious_patterns
    generate_recommendations
    
    log "Análisis completado. Reporte guardado en: $REPORT_FILE"
}

# Verificar que el log existe
if [ ! -f "$LOG_FILE" ]; then
    echo "Error: Log file $LOG_FILE no encontrado"
    exit 1
fi

main "$@"
```

## Análisis Forense Avanzado

### Preservación de Evidencia

```bash
# Crear imagen forense de logs antes de análisis
sudo dd if=/var/log/auth.log of=/evidence/auth.log.img bs=1M conv=noerror,sync
sudo sha256sum /evidence/auth.log.img > /evidence/auth.log.img.sha256

# Montar como solo lectura para análisis
sudo mount -o loop,ro /evidence/auth.log.img /mnt/evidence/

# Crear copia de trabajo
sudo cp /evidence/auth.log.img /working/auth.log.work
```

### Timeline Analysis

```bash
#!/bin/bash
# timeline_analysis.sh - Crear timeline de eventos de seguridad

LOG_FILE="/var/log/auth.log"
TIMELINE_FILE="/tmp/security_timeline.txt"

create_timeline() {
    echo "Creando timeline de eventos de seguridad..."
    
    # Extraer eventos relevantes con timestamp
    {
        # Logins exitosos
        grep "Accepted password\|Accepted publickey" "$LOG_FILE" | \
        sed 's/^/LOGIN_SUCCESS: /'
        
        # Logins fallidos
        grep "Failed password" "$LOG_FILE" | \
        sed 's/^/LOGIN_FAILED: /'
        
        # Actividad sudo
        grep "sudo.*COMMAND" "$LOG_FILE" | \
        sed 's/^/SUDO_COMMAND: /'
        
        # Nuevas sesiones
        grep "New session" "$LOG_FILE" | \
        sed 's/^/NEW_SESSION: /'
        
        # Desconexiones
        grep "session closed" "$LOG_FILE" | \
        sed 's/^/SESSION_CLOSED: /'
        
    } | sort -k2,3 > "$TIMELINE_FILE"
    
    echo "Timeline creado en: $TIMELINE_FILE"
}

# Análisis de timeline por períodos
analyze_timeline_patterns() {
    echo "=== ANÁLISIS DE PATRONES TEMPORALES ==="
    
    # Actividad por día de la semana
    echo "Actividad por día de la semana:"
    awk '{print $2}' "$TIMELINE_FILE" | \
    xargs -I {} date -d {} +%A 2>/dev/null | \
    sort | uniq -c | sort -nr
    
    # Actividad por hora del día
    echo -e "\nActividad por hora:"
    awk '{print $3}' "$TIMELINE_FILE" | \
    cut -d: -f1 | \
    sort | uniq -c | sort -k2 -n
    
    # Detectar actividad fuera de horario normal
    echo -e "\nActividad fuera de horario (22:00-06:00):"
    awk '$3 ~ /^(2[2-3]|0[0-5]):/ {print}' "$TIMELINE_FILE" | wc -l
}

create_timeline
analyze_timeline_patterns
```

### Correlación de Eventos

```bash
#!/bin/bash
# event_correlation.sh - Correlacionar eventos de múltiples logs

correlate_web_auth_events() {
    local auth_log="/var/log/auth.log"
    local access_log="/var/log/apache2/access.log"
    local output_file="/tmp/correlation_report.txt"
    
    echo "=== CORRELACIÓN DE EVENTOS WEB Y AUTENTICACIÓN ===" > "$output_file"
    
    # Extraer IPs de ambos logs en el mismo período
    echo "Analizando correlación entre intentos SSH y tráfico web..." >> "$output_file"
    
    # IPs con intentos SSH fallidos
    grep "Failed password" "$auth_log" | \
    awk '{print $11}' | sort | uniq > /tmp/ssh_failed_ips.txt
    
    # IPs en logs de Apache
    awk '{print $1}' "$access_log" | sort | uniq > /tmp/web_ips.txt
    
    # IPs que aparecen en ambos logs
    echo -e "\nIPs con actividad SSH fallida y tráfico web:" >> "$output_file"
    comm -12 /tmp/ssh_failed_ips.txt /tmp/web_ips.txt | \
    while read ip; do
        ssh_attempts=$(grep "Failed password.*$ip" "$auth_log" | wc -l)
        web_requests=$(grep "^$ip " "$access_log" | wc -l)
        echo "  $ip: $ssh_attempts intentos SSH, $web_requests requests web" >> "$output_file"
    done
    
    # Limpiar archivos temporales
    rm -f /tmp/ssh_failed_ips.txt /tmp/web_ips.txt
    
    echo "Reporte de correlación guardado en: $output_file"
}

correlate_web_auth_events
```

## Herramientas Especializadas

### GoAccess para Logs Web

```bash
# Instalar GoAccess
sudo apt install goaccess

# Análisis en tiempo real de logs Apache/Nginx
goaccess /var/log/apache2/access.log -c

# Generar reporte HTML
goaccess /var/log/apache2/access.log -o /tmp/report.html --log-format=COMBINED

# Análisis en tiempo real con HTML
goaccess /var/log/apache2/access.log -o /var/www/html/stats.html --log-format=COMBINED --real-time-html

# Filtrar por fecha específica
grep "30/Jun/2025" /var/log/apache2/access.log | goaccess -a -o /tmp/daily_report.html
```

### ELK Stack Básico

```bash
# Configuración básica de Logstash
# /etc/logstash/conf.d/syslog.conf
input {
  file {
    path => "/var/log/syslog"
    type => "syslog"
  }
  file {
    path => "/var/log/auth.log"
    type => "auth"
  }
}

filter {
  if [type] == "auth" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{WORD:hostname} %{WORD:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}" }
    }
    
    if "Failed password" in [message] {
      grok {
        match => { "message" => "Failed password for %{USERNAME:username} from %{IP:src_ip}" }
      }
      mutate {
        add_tag => ["auth_failure"]
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### Splunk Alternative - rsyslog con database

```bash
# Configurar rsyslog para enviar a MySQL
# /etc/rsyslog.conf
$ModLoad ommysql
auth,authpriv.*     :ommysql:localhost,syslog,rsyslog,password

# Crear tabla en MySQL
CREATE TABLE SystemEvents (
    ID int unsigned not null auto_increment primary key,
    CustomerID bigint,
    ReceivedAt datetime NULL,
    DeviceReportedTime datetime NULL,
    Facility smallint NULL,
    Priority smallint NULL,
    FromHost varchar(60) NULL,
    Message text,
    NTSeverity int NULL,
    Importance int NULL,
    EventSource varchar(60),
    EventUser varchar(60) NULL,
    EventCategory int NULL,
    EventID int NULL,
    EventBinaryData text NULL,
    MaxAvailable int NULL,
    CurrUsage int NULL,
    MinUsage int NULL,
    MaxUsage int NULL,
    InfoUnitID int NULL,
    SysLogTag varchar(60),
    EventLogType varchar(60),
    GenericFileName VarChar(60),
    SystemID int NULL
);
```

## Automatización y Alertas

### Script de Monitoreo en Tiempo Real

```bash
#!/bin/bash
# real_time_monitor.sh - Monitoreo en tiempo real de logs críticos

LOG_FILES=("/var/log/auth.log" "/var/log/syslog" "/var/log/apache2/error.log")
ALERT_EMAIL="admin@empresa.com"
PATTERNS_FILE="/etc/log_monitor/alert_patterns.txt"

# Patrones que generan alertas (uno por línea)
cat > "$PATTERNS_FILE" << 'EOF'
Failed password.*root
authentication failure
kernel panic
Out of memory
segfault
sudo.*authentication failure
Invalid user
Connection closed by.*\[preauth\]
EOF

# Función para enviar alerta
send_alert() {
    local message="$1"
    local log_file="$2"
    
    echo "ALERTA: $message" | \
    mail -s "Alerta de Seguridad - $(hostname)" "$ALERT_EMAIL"
    
    logger "SECURITY_ALERT: $message in $log_file"
}

# Monitor en tiempo real
monitor_logs() {
    echo "Iniciando monitoreo en tiempo real..."
    
    # Usar tail -F para seguir múltiples archivos
    tail -F "${LOG_FILES[@]}" 2>/dev/null | \
    while read line; do
        # Verificar cada patrón
        while read pattern; do
            if echo "$line" | grep -q "$pattern"; then
                send_alert "Patrón detectado: $pattern" "$(echo $line | cut -d: -f1)"
                echo "$(date): ALERTA - $line"
            fi
        done < "$PATTERNS_FILE"
    done
}

# Verificar dependencias
if ! command -v mail > /dev/null; then
    echo "Advertencia: comando 'mail' no encontrado. Instalar mailutils"
fi

# Crear directorio para patrones si no existe
mkdir -p "$(dirname "$PATTERNS_FILE")"

# Iniciar monitoreo
monitor_logs
```

### Rotación y Compresión Automatizada

```bash
# /etc/logrotate.d/security-logs
/var/log/security-analysis/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
    postrotate
        /usr/bin/systemctl reload rsyslog > /dev/null 2>&1 || true
    endscript
}

# Verificar configuración de logrotate
sudo logrotate -d /etc/logrotate.d/security-logs

# Forzar rotación manual
sudo logrotate -f /etc/logrotate.d/security-logs
```

## Investigación de Incidentes

### Checklist de Investigación Forense

```bash
#!/bin/bash
# incident_investigation.sh - Checklist automatizado para investigación

INCIDENT_DATE="$1"
INVESTIGATION_DIR="/var/investigations/$(date +%Y%m%d_%H%M%S)"

if [ -z "$INCIDENT_DATE" ]; then
    echo "Uso: $0 YYYY-MM-DD"
    exit 1
fi

# Crear directorio de investigación
mkdir -p "$INVESTIGATION_DIR"
cd "$INVESTIGATION_DIR"

echo "=== INVESTIGACIÓN DE INCIDENTE ===" > investigation_report.txt
echo "Fecha del incidente: $INCIDENT_DATE" >> investigation_report.txt
echo "Investigación iniciada: $(date)" >> investigation_report.txt
echo "" >> investigation_report.txt

# 1. Preservar logs del día del incidente
echo "1. Preservando logs del incidente..." | tee -a investigation_report.txt
grep "$INCIDENT_DATE" /var/log/auth.log > auth_incident.log
grep "$INCIDENT_DATE" /var/log/syslog > syslog_incident.log
grep "$INCIDENT_DATE" /var/log/apache2/access.log > web_access_incident.log 2>/dev/null

# 2. Análisis de conexiones activas durante el incidente
echo "2. Analizando conexiones de red..." | tee -a investigation_report.txt
if [ -f "/var/log/netstat_$INCIDENT_DATE.log" ]; then
    cp "/var/log/netstat_$INCIDENT_DATE.log" netstat_incident.log
else
    echo "  - Log de netstat no disponible para $INCIDENT_DATE" >> investigation_report.txt
fi

# 3. Procesos ejecutándose durante el incidente
echo "3. Analizando procesos..." | tee -a investigation_report.txt
grep "$INCIDENT_DATE" /var/log/syslog | grep -E "(started|executed|launched)" > processes_incident.log

# 4. Cambios en archivos del sistema
echo "4. Verificando integridad del sistema..." | tee -a investigation_report.txt
if command -v aide > /dev/null; then
    aide --check > aide_check.txt 2>&1
else
    echo "  - AIDE no instalado. Recomendado para verificación de integridad" >> investigation_report.txt
fi

# 5. Análisis de usuarios y privilegios
echo "5. Analizando actividad de usuarios..." | tee -a investigation_report.txt
grep "$INCIDENT_DATE" /var/log/auth.log | grep -E "(sudo|su|login|session)" > user_activity_incident.log

# 6. Búsqueda de malware básica
echo "6. Búsqueda básica de malware..." | tee -a investigation_report.txt
find /tmp /var/tmp /dev/shm -type f -executable -newer /var/log/auth.log > suspicious_executables.txt 2>/dev/null

# 7. Análisis de red
echo "7. Analizando tráfico de red..." | tee -a investigation_report.txt
if [ -f "/var/log/tcpdump_$INCIDENT_DATE.pcap" ]; then
    tcpdump -r "/var/log/tcpdump_$INCIDENT_DATE.pcap" -n > network_analysis.txt 2>&1
else
    echo "  - Captura de red no disponible para $INCIDENT_DATE" >> investigation_report.txt
fi

# 8. Generar resumen
echo "8. Generando resumen..." | tee -a investigation_report.txt
{
    echo ""
    echo "=== RESUMEN DE HALLAZGOS ==="
    echo "Intentos de login fallidos: $(grep -c "Failed password" auth_incident.log 2>/dev/null || echo "0")"
    echo "Logins exitosos: $(grep -c "Accepted" auth_incident.log 2>/dev/null || echo "0")"
    echo "Comandos sudo ejecutados: $(grep -c "sudo.*COMMAND" auth_incident.log 2>/dev/null || echo "0")"
    echo "Archivos sospechosos encontrados: $(wc -l < suspicious_executables.txt 2>/dev/null || echo "0")"
    echo ""
    echo "Archivos generados en: $INVESTIGATION_DIR"
    echo "- auth_incident.log: Logs de autenticación"
    echo "- syslog_incident.log: Logs del sistema"
    echo "- user_activity_incident.log: Actividad de usuarios"
    echo "- suspicious_executables.txt: Ejecutables sospechosos"
    echo "- investigation_report.txt: Este reporte"
} >> investigation_report.txt

echo "Investigación completada. Ver: $INVESTIGATION_DIR/investigation_report.txt"
```

## Mejores Prácticas

### Configuración de Logging Seguro

```bash
# /etc/rsyslog.conf - Configuración segura
# Enviar logs críticos a servidor remoto
auth,authpriv.*     @@log-server.empresa.com:514
*.emerg             @@log-server.empresa.com:514

# Separar logs por facilidad
auth,authpriv.*     /var/log/auth.log
mail.*              /var/log/mail.log
cron.*              /var/log/cron.log
kern.*              /var/log/kern.log

# Proteger permisos de logs
$FileOwner root
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
```

### Monitoreo Proactivo

```bash
# Cron jobs para análisis automatizado
# /etc/cron.d/log-analysis

# Análisis de seguridad cada hora
0 * * * * root /usr/local/bin/security_log_analysis.sh > /dev/null 2>&1

# Reporte diario de seguridad
0 6 * * * root /usr/local/bin/daily_security_report.sh

# Limpieza de logs antiguos
0 2 * * 0 root find /var/log -name "*.log" -mtime +30 -exec gzip {} \;
```

### Hardening del Sistema de Logs

```bash
# Configurar inmutabilidad de logs críticos (requiere ext2/3/4)
sudo chattr +a /var/log/auth.log      # Solo append
sudo chattr +i /var/log/secure.log    # Inmutable

# Verificar atributos
lsattr /var/log/auth.log

# Configurar logrotate para preservar evidencia
# /etc/logrotate.d/forensic-logs
/var/log/auth.log {
    daily
    rotate 365
    compress
    delaycompress
    missingok
    notifempty
    sharedscripts
    postrotate
        # Firmar digitalmente logs rotados
        gpg --detach-sign /var/log/auth.log.1
    endscript
}
```

## Conclusión

El análisis de logs y la investigación forense son habilidades cruciales para cualquier administrador de sistemas. Los logs contienen la historia completa de lo que ocurre en un sistema, y saber interpretarlos correctamente puede significar la diferencia entre detectar un ataque a tiempo o sufrir una brecha de seguridad.

**Puntos clave para recordar:**

1. **Centralizar logs** para análisis correlacionado
2. **Automatizar análisis** con scripts y herramientas
3. **Preservar evidencia** con técnicas forenses apropiadas
4. **Monitorear proactivamente** patrones sospechosos
5. **Documentar procedimientos** de investigación
6. **Mantener chains of custody** para evidencia legal

La práctica constante con estas herramientas y técnicas te convertirá en un investigador forense más efectivo y un administrador de sistemas más seguro.

---

**Andrés Núñez**  