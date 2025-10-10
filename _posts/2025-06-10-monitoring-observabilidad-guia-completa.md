---
title: "Monitoring y Observabilidad: Implementando una Estrategia Completa de Monitoreo"
date: 2025-06-10 11:00:00 +0200
categories: [Monitoring, Observabilidad]
tags: [monitoring, observabilidad, métricas, logs, trazas, prometheus, grafana, elk]
pin: false
---

# Monitoring y Observabilidad: Implementando una Estrategia Completa de Monitoreo

En el mundo actual de sistemas distribuidos y microservicios, el monitoring y la observabilidad se han convertido en componentes críticos para mantener la salud, rendimiento y confiabilidad de nuestras aplicaciones e infraestructuras. Esta guía completa te llevará desde los conceptos fundamentales hasta la implementación práctica de una estrategia de observabilidad moderna.

## ¿Qué es la Observabilidad?

La **observabilidad** es la capacidad de comprender el estado interno de un sistema basándose en sus salidas externas. Mientras que el monitoring tradicional se enfoca en métricas predefinidas, la observabilidad proporciona una visión más profunda y contextual del comportamiento del sistema.

### Los Tres Pilares de la Observabilidad

#### 1. Métricas (Metrics)
Datos numéricos agregados a lo largo del tiempo que representan el estado del sistema:
- **Contadores**: Valores que solo incrementan (requests totales, errores)
- **Gauges**: Valores que pueden subir o bajar (CPU, memoria, conexiones activas)
- **Histogramas**: Distribución de valores observados (latencia, tamaño de respuesta)

#### 2. Logs
Registros discretos de eventos que ocurren en el sistema:
- **Structured logs**: Formato JSON/YAML con campos consistentes
- **Unstructured logs**: Texto libre tradicional
- **Application logs**: Eventos de aplicación
- **System logs**: Eventos del sistema operativo

#### 3. Trazas (Traces)
Representación del flujo de una request a través de múltiples servicios:
- **Spans**: Operaciones individuales dentro de una traza
- **Distributed tracing**: Seguimiento across servicios
- **Correlation IDs**: Identificadores únicos para correlacionar eventos

## Arquitectura de un Sistema de Observabilidad

```mermaid
graph TB
    A[Aplicaciones] --> B[Agentes de Recolección]
    C[Infraestructura] --> B
    D[Bases de Datos] --> B
    
    B --> E[Métricas - Prometheus]
    B --> F[Logs - Elasticsearch]
    B --> G[Trazas - Jaeger]
    
    E --> H[Visualización - Grafana]
    F --> I[Visualización - Kibana]
    G --> J[Visualización - Jaeger UI]
    
    H --> K[Alerting]
    I --> K
    J --> K
    
    K --> L[Notifications]
```

## Implementación Práctica con Prometheus y Grafana

### 1. Configuración de Prometheus

```yaml
# prometheus.yml - Configuración principal
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

scrape_configs:
  # Monitoreo de Prometheus mismo
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter para métricas del sistema
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']

  # Aplicaciones custom
  - job_name: 'mi-aplicacion'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
    metrics_path: '/metrics'
    scrape_interval: 10s

  # Base de datos MySQL
  - job_name: 'mysql'
    static_configs:
      - targets: ['localhost:9104']

  # Nginx
  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

### 2. Reglas de Alertas

```yaml
# alert_rules.yml - Definición de alertas
groups:
- name: system.rules
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "CPU usage is above 80%"
      {% raw %}description: "CPU usage on {{ $labels.instance }} has been above 80% for more than 5 minutes"{% endraw %}

  - alert: HighMemoryUsage
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 85
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Memory usage is above 85%"
      {% raw %}description: "Memory usage on {{ $labels.instance }} is {{ $value }}%"{% endraw %}

  - alert: DiskSpaceLow
    expr: (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"}) * 100 < 10
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Disk space is running low"
      {% raw %}description: "Disk space on {{ $labels.instance }} {{ $labels.mountpoint }} is only {{ $value }}% available"{% endraw %}

- name: application.rules
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      {% raw %}description: "Error rate is {{ $value }} errors per second"{% endraw %}

  - alert: HighLatency
    expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High latency detected"
      {% raw %}description: "95th percentile latency is {{ $value }}s"{% endraw %}
```

### 3. Script de Instalación y Configuración

```bash
#!/bin/bash

# install_monitoring.sh - Instalación completa del stack de monitoring
# Autor: Andrés Núñez

set -euo pipefail

# Configuración
PROMETHEUS_VERSION="2.40.0"
GRAFANA_VERSION="9.3.0"
NODE_EXPORTER_VERSION="1.5.0"
ALERTMANAGER_VERSION="0.25.0"

# Directorios
INSTALL_DIR="/opt/monitoring"
DATA_DIR="/var/lib/monitoring"
LOG_DIR="/var/log/monitoring"

# Función de logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

# Crear usuario para servicios
crear_usuarios() {
    log "Creando usuarios del sistema..."
    
    if ! id prometheus &>/dev/null; then
        useradd --no-create-home --shell /bin/false prometheus
    fi
    
    if ! id grafana &>/dev/null; then
        useradd --no-create-home --shell /bin/false grafana
    fi
}

# Crear directorios necesarios
crear_directorios() {
    log "Creando estructura de directorios..."
    
    mkdir -p "$INSTALL_DIR"/{prometheus,grafana,node_exporter,alertmanager}
    mkdir -p "$DATA_DIR"/{prometheus,grafana}
    mkdir -p "$LOG_DIR"
    mkdir -p /etc/{prometheus,grafana,alertmanager}
    
    chown prometheus:prometheus "$DATA_DIR/prometheus"
    chown grafana:grafana "$DATA_DIR/grafana"
}

# Instalar Prometheus
instalar_prometheus() {
    log "Instalando Prometheus..."
    
    cd /tmp
    wget "https://github.com/prometheus/prometheus/releases/download/v${PROMETHEUS_VERSION}/prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz"
    tar -xzf "prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz"
    
    cp "prometheus-${PROMETHEUS_VERSION}.linux-amd64/prometheus" "$INSTALL_DIR/prometheus/"
    cp "prometheus-${PROMETHEUS_VERSION}.linux-amd64/promtool" "$INSTALL_DIR/prometheus/"
    cp -r "prometheus-${PROMETHEUS_VERSION}.linux-amd64/consoles" "$INSTALL_DIR/prometheus/"
    cp -r "prometheus-${PROMETHEUS_VERSION}.linux-amd64/console_libraries" "$INSTALL_DIR/prometheus/"
    
    chown -R prometheus:prometheus "$INSTALL_DIR/prometheus"
    chmod +x "$INSTALL_DIR/prometheus/prometheus"
    chmod +x "$INSTALL_DIR/prometheus/promtool"
    
    # Crear archivo de configuración
    cat > /etc/prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "/etc/prometheus/alert_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093
EOF

    chown prometheus:prometheus /etc/prometheus/prometheus.yml
}

# Instalar Node Exporter
instalar_node_exporter() {
    log "Instalando Node Exporter..."
    
    cd /tmp
    wget "https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz"
    tar -xzf "node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz"
    
    cp "node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64/node_exporter" "$INSTALL_DIR/node_exporter/"
    chown prometheus:prometheus "$INSTALL_DIR/node_exporter/node_exporter"
    chmod +x "$INSTALL_DIR/node_exporter/node_exporter"
}

# Crear servicios systemd
crear_servicios() {
    log "Creando servicios systemd..."
    
    # Servicio Prometheus
    cat > /etc/systemd/system/prometheus.service << EOF
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=$INSTALL_DIR/prometheus/prometheus \\
  --config.file=/etc/prometheus/prometheus.yml \\
  --storage.tsdb.path=$DATA_DIR/prometheus \\
  --web.console.templates=$INSTALL_DIR/prometheus/consoles \\
  --web.console.libraries=$INSTALL_DIR/prometheus/console_libraries \\
  --web.listen-address=0.0.0.0:9090 \\
  --web.enable-lifecycle \\
  --log.level=info

[Install]
WantedBy=multi-user.target
EOF

    # Servicio Node Exporter
    cat > /etc/systemd/system/node_exporter.service << EOF
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=$INSTALL_DIR/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
EOF

    systemctl daemon-reload
    systemctl enable prometheus node_exporter
}

# Instalar Grafana
instalar_grafana() {
    log "Instalando Grafana..."
    
    # Añadir repositorio de Grafana
    wget -q -O - https://packages.grafana.com/gpg.key | apt-key add -
    echo "deb https://packages.grafana.com/oss/deb stable main" > /etc/apt/sources.list.d/grafana.list
    
    apt-get update
    apt-get install -y grafana
    
    # Configurar Grafana
    cat > /etc/grafana/grafana.ini << 'EOF'
[server]
http_addr = 0.0.0.0
http_port = 3000
domain = localhost

[database]
type = sqlite3
path = /var/lib/grafana/grafana.db

[session]
provider = file
provider_config = sessions

[analytics]
reporting_enabled = false

[security]
admin_user = admin
admin_password = admin123

[users]
allow_sign_up = false

[auth.anonymous]
enabled = false
EOF

    systemctl enable grafana-server
}

# Configurar dashboards de Grafana
configurar_dashboards() {
    log "Configurando dashboards de Grafana..."
    
    # Esperar a que Grafana esté disponible
    sleep 10
    
    # Añadir datasource de Prometheus
    curl -X POST \
      http://admin:admin123@localhost:3000/api/datasources \
      -H 'Content-Type: application/json' \
      -d '{
        "name": "Prometheus",
        "type": "prometheus",
        "url": "http://localhost:9090",
        "access": "proxy",
        "isDefault": true
      }'
    
    # Importar dashboard de Node Exporter
    curl -X POST \
      http://admin:admin123@localhost:3000/api/dashboards/db \
      -H 'Content-Type: application/json' \
      -d '{
        "dashboard": {
          "id": null,
          "title": "Node Exporter Full",
          "tags": ["prometheus", "node-exporter"],
          "timezone": "browser",
          "panels": [],
          "time": {"from": "now-1h", "to": "now"},
          "timepicker": {},
          "templating": {"list": []},
          "annotations": {"list": []},
          "schemaVersion": 16,
          "version": 0
        }
      }'
}

# Función principal
main() {
    log "Iniciando instalación del stack de monitoring..."
    
    # Verificar permisos de root
    if [ "$EUID" -ne 0 ]; then
        echo "Este script debe ejecutarse como root"
        exit 1
    fi
    
    # Actualizar sistema
    apt-get update
    apt-get install -y wget curl tar
    
    # Ejecutar instalación
    crear_usuarios
    crear_directorios
    instalar_prometheus
    instalar_node_exporter
    crear_servicios
    instalar_grafana
    
    # Iniciar servicios
    systemctl start prometheus node_exporter grafana-server
    
    # Configurar Grafana
    configurar_dashboards
    
    log "Instalación completada!"
    log "Prometheus: http://localhost:9090"
    log "Grafana: http://localhost:3000 (admin/admin123)"
    log "Node Exporter: http://localhost:9100"
}

# Ejecutar instalación
main "$@"
```

## Monitoreo de Aplicaciones

### Instrumentación de aplicaciones Python con Prometheus

```python
# app_metrics.py - Instrumentación de aplicación Flask
from flask import Flask, request
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
import psutil

app = Flask(__name__)

# Métricas de aplicación
REQUEST_COUNT = Counter('app_requests_total', 'Total requests', ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('app_request_duration_seconds', 'Request latency')
ACTIVE_CONNECTIONS = Gauge('app_active_connections', 'Active connections')
MEMORY_USAGE = Gauge('app_memory_usage_bytes', 'Memory usage in bytes')

# Middleware para métricas automáticas
@app.before_request
def before_request():
    request.start_time = time.time()

@app.after_request
def after_request(response):
    request_latency = time.time() - request.start_time
    REQUEST_LATENCY.observe(request_latency)
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status=response.status_code
    ).inc()
    return response

# Endpoint para métricas
@app.route('/metrics')
def metrics():
    # Actualizar métricas de sistema
    MEMORY_USAGE.set(psutil.Process().memory_info().rss)
    ACTIVE_CONNECTIONS.set(len(psutil.net_connections()))
    
    return generate_latest(), 200, {'Content-Type': 'text/plain; charset=utf-8'}

# Endpoints de aplicación
@app.route('/')
def home():
    return "Hello World!"

@app.route('/api/data')
def get_data():
    # Simular procesamiento
    time.sleep(0.1)
    return {"data": "example"}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

### Script de monitoreo de servicios

```bash
#!/bin/bash

# monitor_services.sh - Monitoreo de servicios y aplicaciones
# Autor: Andrés Núñez

set -euo pipefail

# Configuración
SERVICES=("nginx" "mysql" "redis" "postgresql")
URLS=("http://localhost" "http://localhost:8080/health")
LOG_FILE="/var/log/service_monitor.log"
ALERT_EMAIL="admin@ejemplo.com"

# Función de logging
log() {
    local level=$1
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
}

# Función para enviar alertas
send_alert() {
    local subject=$1
    local body=$2
    
    if command -v mail > /dev/null; then
        echo "$body" | mail -s "$subject" "$ALERT_EMAIL"
    fi
    
    # También enviar a syslog
    logger -p user.crit "$subject: $body"
}

# Monitorear servicios systemd
check_services() {
    log "INFO" "Verificando servicios systemd..."
    
    for service in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$service"; then
            log "INFO" "Servicio $service está activo"
        else
            local status=$(systemctl is-active "$service" || echo "failed")
            log "ERROR" "Servicio $service está $status"
            send_alert "Servicio $service caído" "El servicio $service no está funcionando correctamente. Estado: $status"
            
            # Intentar reiniciar el servicio
            log "INFO" "Intentando reiniciar $service..."
            if systemctl restart "$service"; then
                log "INFO" "Servicio $service reiniciado exitosamente"
                send_alert "Servicio $service recuperado" "El servicio $service ha sido reiniciado y está funcionando."
            else
                log "ERROR" "Falló el reinicio de $service"
                send_alert "Fallo crítico en $service" "No se pudo reiniciar el servicio $service. Intervención manual requerida."
            fi
        fi
    done
}

# Monitorear URLs/endpoints
check_urls() {
    log "INFO" "Verificando endpoints HTTP..."
    
    for url in "${URLS[@]}"; do
        response_code=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 10 "$url" || echo "000")
        response_time=$(curl -s -o /dev/null -w "%{time_total}" --connect-timeout 10 "$url" || echo "999")
        
        if [ "$response_code" -eq 200 ]; then
            log "INFO" "URL $url responde correctamente (${response_time}s)"
            
            # Alertar si el tiempo de respuesta es muy alto
            if (( $(echo "$response_time > 2.0" | bc -l) )); then
                log "WARN" "Tiempo de respuesta alto para $url: ${response_time}s"
                send_alert "Latencia alta en $url" "El endpoint $url está respondiendo lentamente: ${response_time}s"
            fi
        else
            log "ERROR" "URL $url no responde correctamente. Código: $response_code"
            send_alert "Endpoint $url no disponible" "El endpoint $url retornó código $response_code"
        fi
    done
}

# Monitorear recursos del sistema
check_system_resources() {
    log "INFO" "Verificando recursos del sistema..."
    
    # CPU
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | sed 's/%us,//')
    cpu_usage=${cpu_usage%.*}
    
    if [ "$cpu_usage" -gt 90 ]; then
        log "ERROR" "Uso de CPU crítico: ${cpu_usage}%"
        send_alert "CPU crítica" "El uso de CPU está en ${cpu_usage}%"
    elif [ "$cpu_usage" -gt 80 ]; then
        log "WARN" "Uso de CPU alto: ${cpu_usage}%"
    fi
    
    # Memoria
    mem_usage=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100.0}')
    
    if [ "$mem_usage" -gt 90 ]; then
        log "ERROR" "Uso de memoria crítico: ${mem_usage}%"
        send_alert "Memoria crítica" "El uso de memoria está en ${mem_usage}%"
    elif [ "$mem_usage" -gt 85 ]; then
        log "WARN" "Uso de memoria alto: ${mem_usage}%"
    fi
    
    # Disco
    while read line; do
        usage=$(echo $line | awk '{print $5}' | sed 's/%//')
        partition=$(echo $line | awk '{print $6}')
        
        if [ "$usage" -gt 95 ]; then
            log "ERROR" "Disco lleno: $partition al ${usage}%"
            send_alert "Disco lleno crítico" "La partición $partition está al ${usage}%"
        elif [ "$usage" -gt 90 ]; then
            log "WARN" "Disco casi lleno: $partition al ${usage}%"
        fi
    done < <(df -h | grep -E '^/dev/' | grep -v tmpfs)
}

# Verificar conectividad de red
check_network() {
    log "INFO" "Verificando conectividad de red..."
    
    # Verificar conectividad a internet
    if ping -c 3 8.8.8.8 > /dev/null 2>&1; then
        log "INFO" "Conectividad a internet OK"
    else
        log "ERROR" "Sin conectividad a internet"
        send_alert "Sin conectividad" "El servidor no tiene conectividad a internet"
    fi
    
    # Verificar DNS
    if nslookup google.com > /dev/null 2>&1; then
        log "INFO" "Resolución DNS OK"
    else
        log "ERROR" "Problemas con resolución DNS"
        send_alert "Problema DNS" "El servidor tiene problemas con la resolución DNS"
    fi
}

# Generar reporte de estado
generate_report() {
    local report_file="/tmp/system_report_$(date +%Y%m%d_%H%M%S).txt"
    
    cat > "$report_file" << EOF
=== REPORTE DE ESTADO DEL SISTEMA ===
Fecha: $(date)
Host: $(hostname)
Uptime: $(uptime -p)

=== SERVICIOS ===
$(systemctl status "${SERVICES[@]}" --no-pager -l)

=== RECURSOS ===
CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
Memoria: $(free -h)
Disco: $(df -h)

=== RED ===
Interfaces: $(ip addr show | grep -E '^[0-9]+:' | awk '{print $2}')
Rutas: $(ip route show)

=== PROCESOS TOP ===
$(ps aux --sort=-%cpu | head -10)

EOF

    log "INFO" "Reporte generado: $report_file"
    
    # Enviar reporte por email si es solicitado
    if [ "${1:-}" = "--email" ]; then
        mail -s "Reporte de Sistema - $(hostname)" "$ALERT_EMAIL" < "$report_file"
    fi
}

# Función principal
main() {
    log "INFO" "Iniciando monitoreo de servicios..."
    
    case "${1:-check}" in
        "check")
            check_services
            check_urls
            check_system_resources
            check_network
            ;;
        "report")
            generate_report "${2:-}"
            ;;
        "services")
            check_services
            ;;
        "urls")
            check_urls
            ;;
        "system")
            check_system_resources
            ;;
        "network")
            check_network
            ;;
        *)
            echo "Uso: $0 {check|report|services|urls|system|network}"
            echo "  check    - Verificación completa (por defecto)"
            echo "  report   - Generar reporte de estado"
            echo "  services - Solo verificar servicios"
            echo "  urls     - Solo verificar URLs"
            echo "  system   - Solo verificar recursos del sistema"
            echo "  network  - Solo verificar red"
            exit 1
            ;;
    esac
    
    log "INFO" "Monitoreo completado"
}

# Ejecutar script
main "$@"
```

## Stack ELK para Gestión de Logs

### Configuración de Elasticsearch

```yaml
# elasticsearch.yml
cluster.name: mi-cluster-logs
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node

# Configuración de memoria
bootstrap.memory_lock: true

# Seguridad básica
xpack.security.enabled: false
```

### Configuración de Logstash

```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
  
  file {
    path => "/var/log/apache2/access.log"
    start_position => "beginning"
    type => "apache-access"
  }
  
  file {
    path => "/var/log/syslog"
    start_position => "beginning"
    type => "syslog"
  }
}

filter {
  if [type] == "apache-access" {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    
    mutate {
      convert => { "response" => "integer" }
      convert => { "bytes" => "integer" }
    }
  }
  
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{IPORHOST:host} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:msg}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "%{type}-%{+YYYY.MM.dd}"
  }
  
  stdout {
    codec => rubydebug
  }
}
```

## Mejores Prácticas de Observabilidad

### 1. Definir SLIs, SLOs y SLAs

```bash
# slo_calculator.sh - Calculadora de SLO
#!/bin/bash

# Service Level Indicators (SLIs)
calculate_availability_sli() {
    local total_requests=$1
    local successful_requests=$2
    
    echo "scale=4; $successful_requests / $total_requests * 100" | bc
}

calculate_latency_sli() {
    local p95_latency=$1
    local target_latency=$2
    
    if (( $(echo "$p95_latency <= $target_latency" | bc -l) )); then
        echo "100"
    else
        echo "scale=4; $target_latency / $p95_latency * 100" | bc
    fi
}

# Service Level Objectives (SLOs)
check_slo_compliance() {
    local current_sli=$1
    local slo_target=$2
    local service_name=$3
    
    if (( $(echo "$current_sli >= $slo_target" | bc -l) )); then
        echo "✅ $service_name SLO cumplido: ${current_sli}% (objetivo: ${slo_target}%)"
        return 0
    else
        echo "❌ $service_name SLO no cumplido: ${current_sli}% (objetivo: ${slo_target}%)"
        return 1
    fi
}

# Error Budget
calculate_error_budget() {
    local slo_target=$1
    local time_period_hours=$2
    
    local allowed_downtime_percent=$(echo "scale=4; 100 - $slo_target" | bc)
    local allowed_downtime_hours=$(echo "scale=2; $time_period_hours * $allowed_downtime_percent / 100" | bc)
    
    echo "Error budget para ${time_period_hours}h: ${allowed_downtime_hours}h (${allowed_downtime_percent}%)"
}
```

### 2. Correlación de eventos

```python
# correlation_engine.py - Motor de correlación de eventos
import json
import time
from collections import defaultdict
from datetime import datetime, timedelta

class EventCorrelator:
    def __init__(self):
        self.events = []
        self.patterns = []
        self.correlations = defaultdict(list)
    
    def add_pattern(self, name, conditions, time_window=300):
        """Añadir patrón de correlación"""
        self.patterns.append({
            'name': name,
            'conditions': conditions,
            'time_window': time_window
        })
    
    def process_event(self, event):
        """Procesar nuevo evento"""
        event['timestamp'] = datetime.now()
        self.events.append(event)
        
        # Limpiar eventos antiguos
        cutoff = datetime.now() - timedelta(hours=1)
        self.events = [e for e in self.events if e['timestamp'] > cutoff]
        
        # Verificar patrones
        self.check_patterns(event)
    
    def check_patterns(self, new_event):
        """Verificar si el nuevo evento cumple algún patrón"""
        for pattern in self.patterns:
            window_start = new_event['timestamp'] - timedelta(seconds=pattern['time_window'])
            relevant_events = [e for e in self.events if e['timestamp'] >= window_start]
            
            if self.pattern_matches(pattern, relevant_events):
                correlation_id = f"{pattern['name']}_{int(time.time())}"
                self.correlations[correlation_id] = relevant_events
                self.trigger_correlation_alert(pattern, correlation_id, relevant_events)
    
    def pattern_matches(self, pattern, events):
        """Verificar si los eventos cumplen el patrón"""
        conditions = pattern['conditions']
        
        # Ejemplo: verificar que ocurran eventos específicos
        required_types = conditions.get('event_types', [])
        event_types_found = set(e['type'] for e in events)
        
        return all(req_type in event_types_found for req_type in required_types)
    
    def trigger_correlation_alert(self, pattern, correlation_id, events):
        """Disparar alerta de correlación"""
        alert = {
            'correlation_id': correlation_id,
            'pattern_name': pattern['name'],
            'timestamp': datetime.now().isoformat(),
            'event_count': len(events),
            'events': [e['type'] for e in events]
        }
        
        print(f"🔗 CORRELACIÓN DETECTADA: {json.dumps(alert, indent=2)}")

# Ejemplo de uso
correlator = EventCorrelator()

# Definir patrón: alta CPU seguida de reinicio de servicio
correlator.add_pattern(
    name="high_cpu_service_restart",
    conditions={
        'event_types': ['high_cpu_alert', 'service_restart']
    },
    time_window=600  # 10 minutos
)

# Simular eventos
correlator.process_event({'type': 'high_cpu_alert', 'host': 'web01', 'value': 95})
time.sleep(2)
correlator.process_event({'type': 'service_restart', 'host': 'web01', 'service': 'apache'})
```

## Conclusión

La implementación de una estrategia completa de monitoring y observabilidad es fundamental para mantener sistemas confiables y de alto rendimiento. Los puntos clave incluyen:

### Elementos esenciales:

1. **Instrumentación proactiva**: Añadir métricas y logs desde el diseño
2. **Alertas inteligentes**: Evitar fatiga de alertas con umbrales bien definidos
3. **Dashboards contextuales**: Visualizaciones que cuenten una historia
4. **Automatización**: Scripts y herramientas que respondan automáticamente
5. **Correlación**: Conectar eventos aparentemente inconexos
6. **Documentación**: Runbooks y procedimientos de respuesta

### Beneficios de una observabilidad madura:

- **Detección temprana**: Identificar problemas antes que afecten usuarios
- **Resolución rápida**: Reducir el MTTR (Mean Time To Recovery)
- **Optimización**: Identificar cuellos de botella y oportunidades de mejora
- **Planificación de capacidad**: Tomar decisiones basadas en datos
- **Cumplimiento de SLAs**: Garantizar niveles de servicio acordados

La observabilidad no es un destino, sino un viaje continuo de mejora. Comienza con lo básico y evoluciona gradualmente hacia un sistema más sofisticado que proporcione insights profundos sobre el comportamiento de tus sistemas.

---

---
**Andrés Nuñez - t4ifi**
