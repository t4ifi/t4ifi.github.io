---
title: "Elasticsearch y el Stack ELK: Guía Completa para Búsqueda y Análisis de Datos"
date: 2025-08-08 08:00:00 +0100
categories: [Big Data, Search]
tags: [elasticsearch, elk, logstash, kibana, beats, search, analytics, logs, monitoring]
---

# Elasticsearch y el Stack ELK: Guía Completa para Búsqueda y Análisis de Datos

Elasticsearch es el motor de búsqueda y análisis distribuido más popular del mundo. Junto con Logstash, Kibana y Beats forma el stack ELK, una solución completa para gestión de logs, monitoreo y análisis de datos en tiempo real.

## 📋 Tabla de Contenidos

1. [Introducción al Stack ELK](#introduccion-elk)
2. [Instalación y Configuración](#instalacion-configuracion)
3. [Elasticsearch: Motor de Búsqueda](#elasticsearch-motor)
4. [Logstash: Procesamiento de Datos](#logstash-procesamiento)
5. [Kibana: Visualización y Dashboard](#kibana-visualizacion)
6. [Beats: Recolección de Datos](#beats-recoleccion)
7. [Configuraciones Avanzadas](#configuraciones-avanzadas)
8. [Monitoreo y Mantenimiento](#monitoreo-mantenimiento)
9. [Casos de Uso Prácticos](#casos-uso-practicos)

## 1. Introducción al Stack ELK {#introduccion-elk}

### ¿Qué es el Stack ELK?

El stack ELK es una colección de tres proyectos open source:

- **Elasticsearch**: Motor de búsqueda y análisis distribuido
- **Logstash**: Pipeline de procesamiento de datos
- **Kibana**: Plataforma de visualización y gestión

Más recientemente se ha añadido **Beats**, creando el stack **Elastic**.

### Arquitectura del Stack

```mermaid
graph LR
    A[Data Sources] --> B[Beats]
    A --> C[Logstash]
    B --> D[Elasticsearch]
    C --> D
    D --> E[Kibana]
    E --> F[Users/Dashboards]
```

## 2. Instalación y Configuración {#instalacion-configuracion}

### Instalación con Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=elk-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - elk-network

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    volumes:
      - ./logstash/config:/usr/share/logstash/pipeline
      - ./logstash/logstash.yml:/usr/share/logstash/config/logstash.yml
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx512m -Xms512m"
    networks:
      - elk-network
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: '["http://elasticsearch:9200"]'
    networks:
      - elk-network
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - elk-network
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
    driver: local

networks:
  elk-network:
    driver: bridge
```

### Instalación Nativa en Ubuntu

```bash
#!/bin/bash
# install_elk_stack.sh

# Agregar repositorio Elastic
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

# Actualizar repositorios
sudo apt update

# Instalar Java (requisito)
sudo apt install -y openjdk-11-jdk

# Instalar Elasticsearch
sudo apt install -y elasticsearch

# Instalar Logstash
sudo apt install -y logstash

# Instalar Kibana
sudo apt install -y kibana

# Instalar Filebeat
sudo apt install -y filebeat

# Configurar servicios
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl enable logstash
sudo systemctl enable kibana
sudo systemctl enable filebeat

echo "Stack ELK instalado exitosamente"
echo "Configurar cada componente antes de iniciar los servicios"
```

## 3. Elasticsearch: Motor de Búsqueda {#elasticsearch-motor}

### Configuración Básica

```yaml
# /etc/elasticsearch/elasticsearch.yml
cluster.name: production-cluster
node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node

# Memoria
bootstrap.memory_lock: true

# Seguridad básica
xpack.security.enabled: false
xpack.security.transport.ssl.enabled: false
```

### Operaciones Básicas con API REST

```bash
# Verificar salud del cluster
curl -X GET "localhost:9200/_cluster/health?pretty"

# Listar índices
curl -X GET "localhost:9200/_cat/indices?v"

# Crear un índice
curl -X PUT "localhost:9200/mi-indice" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "timestamp": {
        "type": "date"
      },
      "message": {
        "type": "text",
        "analyzer": "standard"
      },
      "level": {
        "type": "keyword"
      },
      "host": {
        "type": "keyword"
      }
    }
  }
}
'

# Insertar documento
curl -X POST "localhost:9200/mi-indice/_doc" -H 'Content-Type: application/json' -d'
{
  "timestamp": "2025-08-08T10:00:00Z",
  "message": "Aplicación iniciada correctamente",
  "level": "INFO",
  "host": "web-server-01"
}
'

# Buscar documentos
curl -X GET "localhost:9200/mi-indice/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "message": "iniciada"
    }
  }
}
'
```

### Consultas Avanzadas

```bash
# Búsqueda con filtros múltiples
curl -X GET "localhost:9200/logs-*/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "2025-08-01T00:00:00Z",
              "lte": "2025-08-08T23:59:59Z"
            }
          }
        }
      ],
      "filter": [
        {
          "term": {
            "level.keyword": "ERROR"
          }
        }
      ]
    }
  },
  "aggs": {
    "errors_by_host": {
      "terms": {
        "field": "host.keyword",
        "size": 10
      }
    }
  },
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}
'

# Búsqueda con agregaciones
curl -X GET "localhost:9200/metrics-*/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "cpu_stats": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h"
      },
      "aggs": {
        "avg_cpu": {
          "avg": {
            "field": "system.cpu.total.pct"
          }
        },
        "max_cpu": {
          "max": {
            "field": "system.cpu.total.pct"
          }
        }
      }
    }
  }
}
'
```

### Gestión de Índices

```bash
#!/bin/bash
# elasticsearch_index_management.sh

ES_URL="http://localhost:9200"

# Crear template de índice
curl -X PUT "$ES_URL/_index_template/logs-template" -H 'Content-Type: application/json' -d'
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "level": {
          "type": "keyword"
        },
        "message": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "host": {
          "type": "keyword"
        },
        "service": {
          "type": "keyword"
        }
      }
    }
  }
}
'

# Crear política de lifecycle
curl -X PUT "$ES_URL/_ilm/policy/logs-policy" -H 'Content-Type: application/json' -d'
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "1GB",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "1d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          }
        }
      },
      "cold": {
        "min_age": "7d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          }
        }
      },
      "delete": {
        "min_age": "30d"
      }
    }
  }
}
'

echo "Template y política de lifecycle creados"
```

## 4. Logstash: Procesamiento de Datos {#logstash-procesamiento}

### Configuración Principal

```yaml
# /etc/logstash/logstash.yml
node.name: logstash-node-1
path.data: /var/lib/logstash
pipeline.workers: 4
pipeline.batch.size: 125
pipeline.batch.delay: 50
path.config: /etc/logstash/conf.d/*.conf
path.logs: /var/log/logstash
xpack.monitoring.enabled: false
```

### Pipeline de Procesamiento de Logs

```ruby
# /etc/logstash/conf.d/syslog.conf
input {
  # Recibir logs via beats
  beats {
    port => 5044
  }
  
  # Recibir logs via syslog
  syslog {
    port => 5514
  }
  
  # Leer archivos de log
  file {
    path => "/var/log/application/*.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
}

filter {
  # Parsear timestamp
  if [fields][logtype] == "application" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:log_message}" 
      }
    }
    
    date {
      match => [ "timestamp", "yyyy-MM-dd HH:mm:ss" ]
    }
  }
  
  # Procesar logs de Apache
  if [fields][logtype] == "apache" {
    grok {
      match => { 
        "message" => "%{COMBINEDAPACHELOG}" 
      }
    }
    
    # Convertir response code a número
    mutate {
      convert => { "response" => "integer" }
    }
    
    # Categorizar responses
    if [response] >= 400 {
      mutate {
        add_field => { "response_category" => "error" }
      }
    } else if [response] >= 300 {
      mutate {
        add_field => { "response_category" => "redirect" }
      }
    } else {
      mutate {
        add_field => { "response_category" => "success" }
      }
    }
  }
  
  # Enriquecimiento con GeoIP
  if [clientip] {
    geoip {
      source => "clientip"
      target => "geoip"
    }
  }
  
  # Filtrar campos innecesarios
  mutate {
    remove_field => [ "host", "agent", "ecs", "log", "input" ]
  }
}

output {
  # Enviar a Elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{[fields][logtype]}-%{+YYYY.MM.dd}"
    template_name => "logs"
    template_pattern => "logs-*"
  }
  
  # Debug output
  if [level] == "ERROR" {
    stdout {
      codec => rubydebug
    }
  }
}
```

### Pipeline para Métricas

```ruby
# /etc/logstash/conf.d/metrics.conf
input {
  # Recibir métricas de Metricbeat
  beats {
    port => 5045
    type => "metrics"
  }
  
  # Generar métricas sintéticas para testing
  generator {
    message => '{"cpu_percent": %{random_int}, "memory_percent": %{random_int}, "timestamp": "%{+ISO8601}"}'
    count => 1000
    codec => json
  }
}

filter {
  # Procesar métricas del sistema
  if [metricset][name] == "cpu" {
    mutate {
      add_field => { "metric_type" => "system_cpu" }
    }
  }
  
  if [metricset][name] == "memory" {
    mutate {
      add_field => { "metric_type" => "system_memory" }
    }
  }
  
  # Calcular alertas
  if [system][cpu][total][pct] and [system][cpu][total][pct] > 0.8 {
    mutate {
      add_field => { "alert_level" => "warning" }
      add_field => { "alert_message" => "High CPU usage detected" }
    }
  }
  
  if [system][memory][actual][used][pct] and [system][memory][actual][used][pct] > 0.9 {
    mutate {
      add_field => { "alert_level" => "critical" }
      add_field => { "alert_message" => "Critical memory usage" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "metrics-%{+YYYY.MM.dd}"
  }
  
  # Enviar alertas críticas a webhook
  if [alert_level] == "critical" {
    http {
      url => "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
      http_method => "post"
      format => "json"
      mapping => {
        "text" => "ALERT: %{alert_message} on %{host[name]}"
        "channel" => "#alerts"
      }
    }
  }
}
```

## 5. Kibana: Visualización y Dashboard {#kibana-visualizacion}

### Configuración de Kibana

```yaml
# /etc/kibana/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
server.name: "kibana-server"
elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.requestTimeout: 30000
elasticsearch.shardTimeout: 30000

# Configuración de logging
logging.appenders:
  file:
    type: file
    fileName: /var/log/kibana/kibana.log
    layout:
      type: json

logging.root:
  appenders:
    - default
    - file
  level: info

# Configuración de seguridad
xpack.security.enabled: false
xpack.encryptedSavedObjects.encryptionKey: "a-32-character-long-encryption-key"
```

### Creación de Index Patterns

```bash
#!/bin/bash
# create_kibana_index_patterns.sh

KIBANA_URL="http://localhost:5601"

# Crear index pattern para logs
curl -X POST "$KIBANA_URL/api/saved_objects/index-pattern/logs-*" \
  -H "Content-Type: application/json" \
  -H "kbn-xsrf: true" \
  -d '{
    "attributes": {
      "title": "logs-*",
      "timeFieldName": "@timestamp"
    }
  }'

# Crear index pattern para métricas
curl -X POST "$KIBANA_URL/api/saved_objects/index-pattern/metrics-*" \
  -H "Content-Type: application/json" \
  -H "kbn-xsrf: true" \
  -d '{
    "attributes": {
      "title": "metrics-*",
      "timeFieldName": "@timestamp"
    }
  }'

echo "Index patterns creados en Kibana"
```

### Dashboard de Monitoreo de Aplicación

```json
{
  "version": "8.11.0",
  "objects": [
    {
      "id": "app-monitoring-dashboard",
      "type": "dashboard",
      "attributes": {
        "title": "Application Monitoring Dashboard",
        "hits": 0,
        "description": "Dashboard para monitoreo de aplicaciones",
        "panelsJSON": "[{\"version\":\"8.11.0\",\"gridData\":{\"x\":0,\"y\":0,\"w\":24,\"h\":15,\"i\":\"1\"},\"panelIndex\":\"1\",\"embeddableConfig\":{},\"panelRefName\":\"panel_1\"}]",
        "timeRestore": false,
        "timeTo": "now",
        "timeFrom": "now-24h",
        "refreshInterval": {
          "pause": false,
          "value": 30000
        }
      }
    },
    {
      "id": "error-logs-visualization",
      "type": "visualization",
      "attributes": {
        "title": "Error Logs Over Time",
        "visState": "{\"title\":\"Error Logs Over Time\",\"type\":\"histogram\",\"params\":{\"grid\":{\"categoryLines\":false,\"style\":{\"color\":\"#eee\"}},\"categoryAxes\":[{\"id\":\"CategoryAxis-1\",\"type\":\"category\",\"position\":\"bottom\",\"show\":true,\"style\":{},\"scale\":{\"type\":\"linear\"},\"labels\":{\"show\":true,\"truncate\":100},\"title\":{}}],\"valueAxes\":[{\"id\":\"ValueAxis-1\",\"name\":\"LeftAxis-1\",\"type\":\"value\",\"position\":\"left\",\"show\":true,\"style\":{},\"scale\":{\"type\":\"linear\",\"mode\":\"normal\"},\"labels\":{\"show\":true,\"rotate\":0,\"filter\":false,\"truncate\":100},\"title\":{\"text\":\"Count\"}}],\"seriesParams\":[{\"show\":\"true\",\"type\":\"histogram\",\"mode\":\"stacked\",\"data\":{\"label\":\"Count\",\"id\":\"1\"},\"valueAxis\":\"ValueAxis-1\",\"drawLinesBetweenPoints\":true,\"showCircles\":true}],\"addTooltip\":true,\"addLegend\":true,\"legendPosition\":\"right\",\"times\":[],\"addTimeMarker\":false},\"aggs\":[{\"id\":\"1\",\"enabled\":true,\"type\":\"count\",\"schema\":\"metric\",\"params\":{}},{\"id\":\"2\",\"enabled\":true,\"type\":\"date_histogram\",\"schema\":\"segment\",\"params\":{\"field\":\"@timestamp\",\"interval\":\"auto\",\"customInterval\":\"2h\",\"min_doc_count\":1,\"extended_bounds\":{}}}]}",
        "uiStateJSON": "{}",
        "description": "",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": "{\"index\":\"logs-*\",\"query\":{\"match\":{\"level\":\"ERROR\"}},\"filter\":[]}"
        }
      }
    }
  ]
}
```

## 6. Beats: Recolección de Datos {#beats-recoleccion}

### Configuración de Filebeat

```yaml
# /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/*.log
  fields:
    logtype: nginx
  fields_under_root: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

- type: log
  enabled: true
  paths:
    - /var/log/application/*.log
  fields:
    logtype: application
  fields_under_root: true
  
- type: docker
  containers.ids:
    - "*"
  fields:
    logtype: docker
  fields_under_root: true

# Processors para enriquecimiento
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
      
  - add_docker_metadata: ~
  
  - add_kubernetes_metadata: ~
  
  - drop_fields:
      fields: ["agent", "ecs", "log", "input"]

# Output a Logstash
output.logstash:
  hosts: ["localhost:5044"]
  
# Output directo a Elasticsearch (alternativo)
#output.elasticsearch:
#  hosts: ["localhost:9200"]
#  index: "filebeat-%{+yyyy.MM.dd}"

# Configuración de logging
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644

# Configuración de monitoreo
monitoring.enabled: true
```

### Configuración de Metricbeat

```yaml
# /etc/metricbeat/metricbeat.yml
metricbeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s

metricbeat.modules:
- module: system
  metricsets:
    - cpu
    - load
    - memory
    - network
    - process
    - process_summary
    - socket_summary
    - filesystem
    - fsstat
    - uptime
  enabled: true
  period: 10s
  processes: ['.*']
  cpu.metrics: ["percentages", "normalized_percentages"]
  core.metrics: ["percentages"]

- module: docker
  metricsets: ["container", "cpu", "diskio", "info", "memory", "network"]
  hosts: ["unix:///var/run/docker.sock"]
  period: 10s
  enabled: true

- module: nginx
  metricsets: ["stubstatus"]
  period: 10s
  hosts: ["http://127.0.0.1/nginx_status"]
  enabled: true

- module: mysql
  metricsets: ["status"]
  period: 10s
  hosts: ["tcp(127.0.0.1:3306)/"]
  username: metricbeat
  password: password
  enabled: false

processors:
  - add_host_metadata: ~
  - add_docker_metadata: ~

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "metricbeat-%{+yyyy.MM.dd}"

setup.template.settings:
  index.number_of_shards: 1
  index.codec: best_compression

setup.kibana:
  host: "localhost:5601"

logging.level: info
```

### Configuración de Heartbeat

```yaml
# /etc/heartbeat/heartbeat.yml
heartbeat.config.monitors:
  path: ${path.config}/monitors.d/*.yml
  reload.enabled: true
  reload.period: 5s

heartbeat.monitors:
- type: http
  name: "Web Application"
  schedule: '@every 30s'
  urls: ["http://localhost:8080/health"]
  check.response.status: 200
  check.response.body: "OK"
  
- type: tcp
  name: "Database Connection"
  schedule: '@every 10s'
  hosts: ["localhost:3306"]
  
- type: icmp
  name: "Gateway Ping"
  schedule: '@every 5s'
  hosts: ["192.168.1.1"]

processors:
  - add_observer_metadata:
      geo:
        name: "datacenter-1"
        location: "40.7128, -74.0060"
        continent_name: "North America"
        country_iso_code: "US"
        region_name: "New York"
        city_name: "New York"

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "heartbeat-%{+yyyy.MM.dd}"

setup.kibana:
  host: "localhost:5601"

logging.level: info
```

## 7. Configuraciones Avanzadas {#configuraciones-avanzadas}

### Cluster Elasticsearch Multi-Nodo

```yaml
# Nodo Master
# /etc/elasticsearch/elasticsearch.yml (nodo-1)
cluster.name: production-cluster
node.name: master-node-1
node.roles: [ master, data, ingest ]
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 192.168.1.10
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["192.168.1.10", "192.168.1.11", "192.168.1.12"]
cluster.initial_master_nodes: ["master-node-1"]

# Configuración de memoria
bootstrap.memory_lock: true
ES_HEAP_SIZE: 2g

# Configuración de shards
cluster.routing.allocation.node_concurrent_recoveries: 2
cluster.routing.allocation.disk.threshold.enabled: true
cluster.routing.allocation.disk.watermark.low: 85%
cluster.routing.allocation.disk.watermark.high: 90%
```

```yaml
# Nodo Data
# /etc/elasticsearch/elasticsearch.yml (nodo-2)
cluster.name: production-cluster
node.name: data-node-1
node.roles: [ data, ingest ]
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 192.168.1.11
http.port: 9200
transport.port: 9300
discovery.seed_hosts: ["192.168.1.10", "192.168.1.11", "192.168.1.12"]
cluster.initial_master_nodes: ["master-node-1"]
```

### Configuración de Seguridad

```bash
#!/bin/bash
# setup_elasticsearch_security.sh

# Generar certificados
/usr/share/elasticsearch/bin/elasticsearch-certutil ca
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

# Configurar usuarios
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto

# Configurar TLS
cat << EOF >> /etc/elasticsearch/elasticsearch.yml
# Seguridad
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

# HTTP SSL
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: elastic-certificates.p12
EOF

echo "Configuración de seguridad aplicada"
echo "Recuerda configurar las contraseñas en Kibana y Logstash"
```

### Configuración de Alerts con Watcher

```bash
# Crear alerta para errores críticos
curl -X PUT "localhost:9200/_watcher/watch/error_alert" -H 'Content-Type: application/json' -d'
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "search": {
      "request": {
        "indices": ["logs-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "term": {
                    "level.keyword": "ERROR"
                  }
                },
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-1m"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "gt": 10
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": ["admin@company.com"],
        "subject": "Critical Error Alert",
        "body": "More than 10 errors detected in the last minute"
      }
    },
    "log_alert": {
      "logging": {
        "text": "Critical error threshold exceeded: {{ctx.payload.hits.total}} errors"
      }
    }
  }
}
'
```

## 8. Monitoreo y Mantenimiento {#monitoreo-mantenimiento}

### Script de Monitoreo del Cluster

```bash
#!/bin/bash
# elasticsearch_health_check.sh

ES_URL="http://localhost:9200"
LOG_FILE="/var/log/elasticsearch-health.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Verificar salud del cluster
check_cluster_health() {
    local health=$(curl -s "$ES_URL/_cluster/health" | jq -r '.status')
    
    case $health in
        "green")
            log_message "✓ Cluster health: GREEN"
            return 0
            ;;
        "yellow")
            log_message "⚠ Cluster health: YELLOW"
            return 1
            ;;
        "red")
            log_message "✗ Cluster health: RED"
            return 2
            ;;
        *)
            log_message "✗ Cannot determine cluster health"
            return 3
            ;;
    esac
}

# Verificar uso de disco
check_disk_usage() {
    local disk_usage=$(curl -s "$ES_URL/_cat/allocation?h=disk.percent&format=json" | jq -r '.[].["disk.percent"]' | sort -nr | head -1)
    
    if [ $(echo "$disk_usage > 85" | bc -l) -eq 1 ]; then
        log_message "⚠ High disk usage: ${disk_usage}%"
        return 1
    else
        log_message "✓ Disk usage OK: ${disk_usage}%"
        return 0
    fi
}

# Verificar índices problemáticos
check_problematic_indices() {
    local red_indices=$(curl -s "$ES_URL/_cat/indices?h=health,index&format=json" | jq -r '.[] | select(.health=="red") | .index')
    
    if [ -n "$red_indices" ]; then
        log_message "✗ Red indices detected: $red_indices"
        return 1
    else
        log_message "✓ No problematic indices"
        return 0
    fi
}

# Verificar memoria JVM
check_jvm_memory() {
    local heap_used=$(curl -s "$ES_URL/_nodes/stats/jvm" | jq -r '.nodes | to_entries[0].value.jvm.mem.heap_used_percent')
    
    if [ $(echo "$heap_used > 85" | bc -l) -eq 1 ]; then
        log_message "⚠ High JVM heap usage: ${heap_used}%"
        return 1
    else
        log_message "✓ JVM memory OK: ${heap_used}%"
        return 0
    fi
}

# Ejecutar todas las verificaciones
main() {
    log_message "=== Elasticsearch Health Check ==="
    
    check_cluster_health
    check_disk_usage
    check_problematic_indices
    check_jvm_memory
    
    log_message "=== Health Check Complete ==="
}

main
```

### Mantenimiento de Índices

```bash
#!/bin/bash
# elasticsearch_maintenance.sh

ES_URL="http://localhost:9200"

# Función para optimizar índices antiguos
optimize_old_indices() {
    echo "Optimizando índices antiguos..."
    
    # Obtener índices más antiguos de 7 días
    local old_indices=$(curl -s "$ES_URL/_cat/indices?h=index,creation.date.string&format=json" | \
        jq -r --arg date "$(date -d '7 days ago' '+%Y-%m-%d')" \
        '.[] | select(.["creation.date.string"] < $date) | .index')
    
    for index in $old_indices; do
        echo "Optimizando índice: $index"
        curl -X POST "$ES_URL/$index/_forcemerge?max_num_segments=1"
    done
}

# Función para limpiar índices muy antiguos
cleanup_old_indices() {
    echo "Limpiando índices antiguos..."
    
    # Eliminar índices más antiguos de 30 días
    local very_old_indices=$(curl -s "$ES_URL/_cat/indices?h=index,creation.date.string&format=json" | \
        jq -r --arg date "$(date -d '30 days ago' '+%Y-%m-%d')" \
        '.[] | select(.["creation.date.string"] < $date) | .index')
    
    for index in $very_old_indices; do
        echo "Eliminando índice: $index"
        curl -X DELETE "$ES_URL/$index"
    done
}

# Función para rebalancear shards
rebalance_shards() {
    echo "Rebalanceando shards..."
    
    curl -X PUT "$ES_URL/_cluster/settings" -H 'Content-Type: application/json' -d'
    {
      "transient": {
        "cluster.routing.rebalance.enable": "all",
        "cluster.routing.allocation.allow_rebalance": "always"
      }
    }
    '
}

# Función para crear snapshot
create_snapshot() {
    local snapshot_name="snapshot-$(date +%Y%m%d-%H%M%S)"
    
    echo "Creando snapshot: $snapshot_name"
    
    curl -X PUT "$ES_URL/_snapshot/backup/$snapshot_name" -H 'Content-Type: application/json' -d'
    {
      "indices": "*",
      "ignore_unavailable": true,
      "include_global_state": false,
      "metadata": {
        "taken_by": "elasticsearch_maintenance_script",
        "taken_because": "scheduled_backup"
      }
    }
    '
}

# Menú principal
case "$1" in
    optimize)
        optimize_old_indices
        ;;
    cleanup)
        cleanup_old_indices
        ;;
    rebalance)
        rebalance_shards
        ;;
    snapshot)
        create_snapshot
        ;;
    all)
        optimize_old_indices
        cleanup_old_indices
        rebalance_shards
        create_snapshot
        ;;
    *)
        echo "Uso: $0 {optimize|cleanup|rebalance|snapshot|all}"
        exit 1
        ;;
esac
```

## 9. Casos de Uso Prácticos {#casos-uso-practicos}

### Caso 1: Monitoreo de Aplicación Web

```bash
#!/bin/bash
# setup_web_monitoring.sh

# Configurar Filebeat para logs de aplicación
cat << EOF > /etc/filebeat/conf.d/webapp.yml
- type: log
  enabled: true
  paths:
    - /var/log/webapp/*.log
  fields:
    service: webapp
    environment: production
  fields_under_root: true
  multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
  multiline.negate: true
  multiline.match: after
  processors:
    - add_host_metadata: ~
    - add_fields:
        target: ''
        fields:
          datacenter: us-east-1
EOF

# Pipeline de Logstash para aplicación web
cat << 'EOF' > /etc/logstash/conf.d/webapp.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [service] == "webapp" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] \[%{DATA:thread}\] %{DATA:logger} - %{GREEDYDATA:log_message}" 
      }
    }
    
    date {
      match => [ "timestamp", "yyyy-MM-dd HH:mm:ss,SSS" ]
    }
    
    if [level] == "ERROR" {
      mutate {
        add_tag => [ "error" ]
      }
    }
    
    if "Exception" in [log_message] {
      mutate {
        add_tag => [ "exception" ]
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "webapp-logs-%{+YYYY.MM.dd}"
  }
}
EOF

echo "Configuración de monitoreo de aplicación web completada"
```

### Caso 2: Análisis de Logs de Seguridad

```bash
#!/bin/bash
# setup_security_monitoring.sh

# Pipeline para logs de seguridad
cat << 'EOF' > /etc/logstash/conf.d/security.conf
input {
  file {
    path => "/var/log/auth.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "syslog"
  }
  
  file {
    path => "/var/log/fail2ban.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "fail2ban"
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { 
        "message" => "%{SYSLOGTIMESTAMP:timestamp} %{HOSTNAME:hostname} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}" 
      }
      overwrite => [ "message" ]
    }
    
    if "Failed password" in [message] {
      grok {
        match => { 
          "message" => "Failed password for %{DATA:username} from %{IP:src_ip} port %{INT:src_port}" 
        }
      }
      mutate {
        add_tag => [ "failed_login" ]
        add_field => { "event_type" => "authentication_failure" }
      }
    }
    
    if "Accepted password" in [message] {
      grok {
        match => { 
          "message" => "Accepted password for %{DATA:username} from %{IP:src_ip} port %{INT:src_port}" 
        }
      }
      mutate {
        add_tag => [ "successful_login" ]
        add_field => { "event_type" => "authentication_success" }
      }
    }
  }
  
  if [type] == "fail2ban" {
    if "Ban" in [message] {
      grok {
        match => { 
          "message" => "Ban %{IP:banned_ip}" 
        }
      }
      mutate {
        add_tag => [ "ip_banned" ]
        add_field => { "event_type" => "security_action" }
      }
    }
  }
  
  # Enriquecimiento GeoIP
  if [src_ip] {
    geoip {
      source => "src_ip"
      target => "geoip"
    }
  }
  
  if [banned_ip] {
    geoip {
      source => "banned_ip"
      target => "geoip"
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "security-logs-%{+YYYY.MM.dd}"
  }
  
  # Alertas para eventos críticos
  if "failed_login" in [tags] {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "security-alerts-%{+YYYY.MM.dd}"
    }
  }
}
EOF

echo "Configuración de monitoreo de seguridad completada"
```

### Caso 3: Análisis de Performance de API

```bash
#!/bin/bash
# setup_api_performance_monitoring.sh

# Pipeline para logs de API
cat << 'EOF' > /etc/logstash/conf.d/api-performance.conf
input {
  beats {
    port => 5046
    type => "api-logs"
  }
}

filter {
  if [type] == "api-logs" {
    grok {
      match => { 
        "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} \[%{DATA:request_id}\] %{WORD:method} %{URIPATH:endpoint} - %{NUMBER:response_time:float}ms - %{INT:status_code} - %{DATA:user_agent}" 
      }
    }
    
    date {
      match => [ "timestamp", "yyyy-MM-dd'T'HH:mm:ss.SSSZ" ]
    }
    
    # Categorizar por tiempo de respuesta
    if [response_time] {
      if [response_time] < 100 {
        mutate { add_field => { "performance_category" => "fast" } }
      } else if [response_time] < 500 {
        mutate { add_field => { "performance_category" => "normal" } }
      } else if [response_time] < 1000 {
        mutate { add_field => { "performance_category" => "slow" } }
      } else {
        mutate { add_field => { "performance_category" => "very_slow" } }
      }
    }
    
    # Categorizar por status code
    if [status_code] {
      if [status_code] >= 500 {
        mutate { add_field => { "error_category" => "server_error" } }
      } else if [status_code] >= 400 {
        mutate { add_field => { "error_category" => "client_error" } }
      } else {
        mutate { add_field => { "error_category" => "success" } }
      }
    }
    
    # Extraer información del endpoint
    if [endpoint] {
      grok {
        match => { "endpoint" => "/api/v(?<api_version>\d+)/(?<resource>\w+)" }
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "api-performance-%{+YYYY.MM.dd}"
  }
  
  # Alertas para APIs lentas
  if [performance_category] == "very_slow" {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "api-alerts-%{+YYYY.MM.dd}"
    }
  }
}
EOF

echo "Configuración de monitoreo de performance de API completada"
```

## 📚 Recursos y Herramientas Adicionales

### Herramientas de Gestión

```bash
# Curator para gestión automática de índices
pip install elasticsearch-curator

# Configuración de Curator
cat << EOF > /etc/curator/curator.yml
client:
  hosts:
    - 127.0.0.1
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth:
  timeout: 30
  master_only: False

logging:
  loglevel: INFO
  logfile: /var/log/curator.log
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
EOF

# Acción para eliminar índices antiguos
cat << EOF > /etc/curator/delete_old_indices.yml
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 30 days
    options:
      ignore_empty_list: True
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: logs-
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 30
EOF
```

### Scripts de Monitoreo

```bash
#!/bin/bash
# elk_stack_monitor.sh

# Verificar todos los servicios del stack ELK
check_service() {
    local service=$1
    local port=$2
    
    if systemctl is-active --quiet $service; then
        if nc -z localhost $port; then
            echo "✓ $service está corriendo y respondiendo en puerto $port"
        else
            echo "⚠ $service está corriendo pero no responde en puerto $port"
        fi
    else
        echo "✗ $service no está corriendo"
    fi
}

echo "=== ELK Stack Status Check ==="
check_service elasticsearch 9200
check_service kibana 5601
check_service logstash 5044
check_service filebeat 0  # Filebeat no tiene puerto de escucha
check_service metricbeat 0

# Verificar uso de recursos
echo ""
echo "=== Resource Usage ==="
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1"%"}'

echo "Memory Usage:"
free -h | awk 'NR==2{printf "Memory Usage: %s/%s (%.2f%%)\n", $3,$2,$3*100/$2 }'

echo "Disk Usage:"
df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }'
```

## 🎯 Conclusiones

El stack ELK es una solución poderosa para:

1. **Centralización de logs** de múltiples fuentes
2. **Análisis en tiempo real** de grandes volúmenes de datos
3. **Visualización** intuitiva con dashboards
4. **Alertas** proactivas para problemas críticos
5. **Escalabilidad** horizontal para grandes infraestructuras

### Mejores Prácticas

- **Planificar la arquitectura** según el volumen de datos
- **Configurar ILM** para gestión automática de índices
- **Implementar monitoreo** del propio stack ELK
- **Asegurar** con autenticación y cifrado
- **Optimizar consultas** para mejor rendimiento

El dominio del stack ELK es esencial para cualquier administrador de sistemas moderno, especialmente en entornos DevOps y de microservicios.

---

*Este post forma parte de mi serie sobre tecnologías de Big Data y análisis. Si tienes preguntas o sugerencias, no dudes en contactarme.*

---
**Andrés Nuñez - t4ifi**
