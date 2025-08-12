---
title: "Optimización de Servidores Web: Apache vs Nginx Performance Tuning"
date: 2025-05-26 14:30:00 +0000
categories: [Servidores Web, Optimización]
tags: [apache, nginx, performance, optimizacion, web-servers, tuning, caching, ssl]
---

## 🎯 Introducción a la Optimización de Servidores Web

La optimización de servidores web es crucial para garantizar **alta disponibilidad**, **rendimiento óptimo** y **experiencia de usuario** excepcional. En este post veremos técnicas avanzadas de optimización tanto para Apache como para Nginx.

> 💡 **Dato clave:** Una mejora de 1 segundo en tiempo de carga puede aumentar las conversiones hasta en 7% y reducir la tasa de rebote significativamente.

---

## 🔧 Optimización de Apache HTTP Server

### Configuración básica de rendimiento

#### 1. Seleccionar el MPM adecuado
```apache
# /etc/apache2/mods-available/mpm_prefork.conf
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>

# Para sitios con mucho tráfico, usar mpm_worker o mpm_event
# /etc/apache2/mods-available/mpm_event.conf
<IfModule mpm_event_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
    ThreadLimit             64
    ServerLimit             16
</IfModule>
```

#### 2. Configuración de caché y compresión
```apache
# Habilitar módulos necesarios
a2enmod expires
a2enmod headers
a2enmod deflate
a2enmod rewrite

# /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html
    
    # Compresión GZIP
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/plain
        AddOutputFilterByType DEFLATE text/html
        AddOutputFilterByType DEFLATE text/xml
        AddOutputFilterByType DEFLATE text/css
        AddOutputFilterByType DEFLATE application/xml
        AddOutputFilterByType DEFLATE application/xhtml+xml
        AddOutputFilterByType DEFLATE application/rss+xml
        AddOutputFilterByType DEFLATE application/javascript
        AddOutputFilterByType DEFLATE application/x-javascript
        
        # Excluir archivos ya comprimidos
        SetEnvIfNoCase Request_URI \
            \.(?:gif|jpe?g|png|zip|gz|bz2)$ no-gzip dont-vary
    </IfModule>
    
    # Cache headers
    <IfModule mod_expires.c>
        ExpiresActive On
        
        # Cache de imágenes por 1 año
        ExpiresByType image/jpg "access plus 1 year"
        ExpiresByType image/jpeg "access plus 1 year"
        ExpiresByType image/gif "access plus 1 year"
        ExpiresByType image/png "access plus 1 year"
        ExpiresByType image/webp "access plus 1 year"
        
        # Cache de CSS y JS por 1 mes
        ExpiresByType text/css "access plus 1 month"
        ExpiresByType application/javascript "access plus 1 month"
        ExpiresByType application/x-javascript "access plus 1 month"
        
        # Cache de fonts por 1 año
        ExpiresByType application/font-woff "access plus 1 year"
        ExpiresByType application/font-woff2 "access plus 1 year"
        
        # Cache de HTML por 1 hora
        ExpiresByType text/html "access plus 1 hour"
    </IfModule>
    
    # Security headers
    <IfModule mod_headers.c>
        Header always set X-Content-Type-Options nosniff
        Header always set X-Frame-Options DENY
        Header always set X-XSS-Protection "1; mode=block"
        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
        Header always set Referrer-Policy "strict-origin-when-cross-origin"
    </IfModule>
    
    # Optimización de archivos estáticos
    <FilesMatch "\\.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2)$">
        Header set Cache-Control "public, max-age=31536000"
        Header unset ETag
        FileETag None
    </FilesMatch>
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### 3. Optimización de configuración principal
```apache
# /etc/apache2/apache2.conf

# Optimizaciones de performance
ServerTokens Prod
ServerSignature Off
TraceEnable Off
UseCanonicalName Off
HostnameLookups Off

# Timeout optimizado
Timeout 60
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5

# Límites de memoria y conexiones
LimitRequestBody 10485760  # 10MB max upload
LimitRequestFields 100
LimitRequestFieldSize 8190
LimitRequestLine 4094

# Optimizar el logging
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\" %D" combined_with_time
CustomLog ${APACHE_LOG_DIR}/access.log combined_with_time

# Configuración de DirectoryIndex optimizada
DirectoryIndex index.php index.html index.htm

# Prevenir acceso a archivos sensibles
<FilesMatch "^\.">
    Require all denied
</FilesMatch>

<FilesMatch "~$">
    Require all denied
</FilesMatch>
```

---

## ⚡ Optimización de Nginx

### Configuración principal optimizada

#### 1. nginx.conf principal
```nginx
# /etc/nginx/nginx.conf

user www-data;
worker_processes auto;
pid /run/nginx.pid;

# Optimización del worker
worker_rlimit_nofile 65535;

events {
    worker_connections 2048;
    use epoll;
    multi_accept on;
}

http {
    # Configuración básica
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Optimización de performance
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 100;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Buffers optimization
    client_max_body_size 10M;
    client_body_buffer_size 128k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    output_buffers 1 32k;
    postpone_output 1460;
    
    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json
        application/xml
        image/svg+xml;
    gzip_min_length 1000;
    gzip_disable "msie6";
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
    
    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" '
                    'rt=$request_time uct="$upstream_connect_time" '
                    'uht="$upstream_header_time" urt="$upstream_response_time"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;
    
    # Include virtual hosts
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

#### 2. Configuración de sitio optimizada
```nginx
# /etc/nginx/sites-available/example.com
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.php index.html index.htm;
    
    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    # Modern configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
    
    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
    
    # Cache static files
    location ~* \\.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header Vary Accept-Encoding;
        access_log off;
    }
    
    # Cache HTML files
    location ~* \\.(html|htm)$ {
        expires 1h;
        add_header Cache-Control "public";
    }
    
    # Deny access to hidden files
    location ~ /\\. {
        deny all;
        access_log off;
        log_not_found off;
    }
    
    # Rate limiting for login pages
    location /login {
        limit_req zone=login burst=5 nodelay;
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # Rate limiting for API
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        limit_conn conn_limit_per_ip 10;
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    # PHP processing
    location ~ \\.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # FastCGI cache
        fastcgi_cache_valid 200 1h;
        fastcgi_cache_bypass $skip_cache;
        fastcgi_no_cache $skip_cache;
        add_header X-FastCGI-Cache $upstream_cache_status;
    }
    
    # Main location
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

---

## 🚀 Optimización de PHP-FPM

### Configuración de pool optimizada
```ini
; /etc/php/8.1/fpm/pool.d/www.conf

[www]
user = www-data
group = www-data

listen = /var/run/php/php8.1-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

; Pool configuration
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500

; Performance tuning
pm.process_idle_timeout = 60s
pm.max_requests = 1000

; Status page
pm.status_path = /php-fpm-status
ping.path = /php-fpm-ping

; Logging
php_admin_value[error_log] = /var/log/php/fpm-error.log
php_admin_flag[log_errors] = on

; Security
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_value[open_basedir] = /var/www/:/tmp/
```

### Configuración PHP optimizada
```ini
; /etc/php/8.1/fpm/php.ini

; Memory optimization
memory_limit = 256M
max_execution_time = 60
max_input_time = 60
post_max_size = 50M
upload_max_filesize = 50M

; OPcache optimization
opcache.enable = 1
opcache.enable_cli = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 10000
opcache.validate_timestamps = 0
opcache.revalidate_freq = 0
opcache.save_comments = 1
opcache.fast_shutdown = 1

; Realpath cache
realpath_cache_size = 4096K
realpath_cache_ttl = 600

; Session optimization
session.save_handler = redis
session.save_path = "tcp://127.0.0.1:6379"
session.gc_maxlifetime = 3600
```

---

## 📊 Configuración de Caché Avanzado

### Redis para caché de sesiones y objetos
```bash
# Instalación de Redis
sudo apt update
sudo apt install redis-server

# Configuración optimizada
sudo nano /etc/redis/redis.conf
```

```redis
# /etc/redis/redis.conf

# Memory optimization
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence (para caché, podemos deshabilitarlo)
save ""
appendonly no

# Network optimization
tcp-keepalive 60
tcp-backlog 511

# Performance
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

### Configuración de Memcached
```bash
# Instalación
sudo apt install memcached php-memcached

# Configuración
sudo nano /etc/memcached.conf
```

```bash
# /etc/memcached.conf

# Memory allocation
-m 256

# Listen on localhost only
-l 127.0.0.1

# Default port
-p 11211

# User
-u memcache

# Connections
-c 1024

# Verbosity
-v

# Growth factor
-f 1.25

# Minimum space allocated for key+value+flags
-n 48
```

---

## 📈 Monitoreo y Métricas

### Script de monitoreo de rendimiento
```bash
#!/bin/bash
# /usr/local/bin/web-server-monitor.sh

LOG_FILE="/var/log/web-performance.log"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# Función de logging
log_metric() {
    echo "[$TIMESTAMP] $1" >> "$LOG_FILE"
}

# Monitorear conexiones activas
ACTIVE_CONNECTIONS=$(ss -tuln | wc -l)
log_metric "CONNECTIONS: $ACTIVE_CONNECTIONS"

# Monitorear carga del servidor
LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | xargs)
log_metric "LOAD: $LOAD_AVG"

# Monitorear memoria
MEM_USAGE=$(free | grep Mem | awk '{printf("%.1f", $3/$2 * 100.0)}')
log_metric "MEMORY: ${MEM_USAGE}%"

# Monitorear procesos Nginx/Apache
if command -v nginx &> /dev/null; then
    NGINX_WORKERS=$(pgrep nginx | wc -l)
    log_metric "NGINX_WORKERS: $NGINX_WORKERS"
fi

if command -v apache2 &> /dev/null; then
    APACHE_WORKERS=$(pgrep apache2 | wc -l)
    log_metric "APACHE_WORKERS: $APACHE_WORKERS"
fi

# Monitorear PHP-FPM
if command -v php-fpm8.1 &> /dev/null; then
    PHP_WORKERS=$(pgrep php-fpm | wc -l)
    log_metric "PHP_WORKERS: $PHP_WORKERS"
fi

# Verificar respuesta del servidor
RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}\\n' http://localhost)
log_metric "RESPONSE_TIME: ${RESPONSE_TIME}s"

# Verificar SSL si está configurado
if [ -f "/etc/letsencrypt/live/*/cert.pem" ]; then
    SSL_EXPIRY=$(openssl x509 -enddate -noout -in /etc/letsencrypt/live/*/cert.pem | cut -d= -f2)
    log_metric "SSL_EXPIRES: $SSL_EXPIRY"
fi
```

### Configuración de alertas
```bash
#!/bin/bash
# /usr/local/bin/performance-alerts.sh

WEBHOOK_URL="https://hooks.slack.com/your/webhook/url"
EMAIL="admin@example.com"

# Umbrales
CPU_THRESHOLD=80
MEM_THRESHOLD=85
LOAD_THRESHOLD=5.0
RESPONSE_THRESHOLD=2.0

# Obtener métricas
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | grep Mem | awk '{printf("%.1f", $3/$2 * 100.0)}')
LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | xargs)
RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}' http://localhost)

# Función de alerta
send_alert() {
    local message="$1"
    
    # Slack notification
    curl -X POST -H 'Content-type: application/json' \\
        --data "{\"text\":\"🚨 Server Alert: $message\"}" \\
        "$WEBHOOK_URL"
    
    # Email notification
    echo "$message" | mail -s "Server Performance Alert" "$EMAIL"
    
    # Log alert
    echo "[ALERT] $(date): $message" >> /var/log/performance-alerts.log
}

# Verificar umbrales
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
    send_alert "High CPU usage: ${CPU_USAGE}%"
fi

if (( $(echo "$MEM_USAGE > $MEM_THRESHOLD" | bc -l) )); then
    send_alert "High memory usage: ${MEM_USAGE}%"
fi

if (( $(echo "$LOAD_AVG > $LOAD_THRESHOLD" | bc -l) )); then
    send_alert "High load average: $LOAD_AVG"
fi

if (( $(echo "$RESPONSE_TIME > $RESPONSE_THRESHOLD" | bc -l) )); then
    send_alert "Slow response time: ${RESPONSE_TIME}s"
fi
```

---

## 🔧 Herramientas de Testing y Benchmarking

### Scripts de pruebas de carga
```bash
#!/bin/bash
# /usr/local/bin/load-test.sh

DOMAIN="https://example.com"
CONCURRENT_USERS=50
TOTAL_REQUESTS=1000
TEST_DURATION=60

echo "🚀 Iniciando pruebas de carga..."
echo "Domain: $DOMAIN"
echo "Concurrent users: $CONCURRENT_USERS"
echo "Total requests: $TOTAL_REQUESTS"
echo "Duration: ${TEST_DURATION}s"
echo "================================"

# Apache Bench test
echo "📊 Running Apache Bench test..."
ab -n $TOTAL_REQUESTS -c $CONCURRENT_USERS "$DOMAIN/" > ab_results.txt

# cURL test
echo "📊 Running cURL response time test..."
for i in {1..10}; do
    curl -o /dev/null -s -w "Response $i: %{time_total}s\\n" "$DOMAIN"
done

# Siege test (si está instalado)
if command -v siege &> /dev/null; then
    echo "📊 Running Siege test..."
    siege -c $CONCURRENT_USERS -t ${TEST_DURATION}s "$DOMAIN" > siege_results.txt
fi

echo "✅ Pruebas completadas. Revisa los archivos de resultados."
```

### Análisis de logs automatizado
```bash
#!/bin/bash
# /usr/local/bin/analyze-logs.sh

ACCESS_LOG="/var/log/nginx/access.log"
ERROR_LOG="/var/log/nginx/error.log"
OUTPUT_DIR="/var/log/analysis"

mkdir -p "$OUTPUT_DIR"

echo "📊 Analizando logs de acceso..."

# Top 10 IPs con más requests
echo "=== TOP 10 IPs ===" > "$OUTPUT_DIR/top_ips.txt"
awk '{print $1}' "$ACCESS_LOG" | sort | uniq -c | sort -nr | head -10 >> "$OUTPUT_DIR/top_ips.txt"

# Top 10 páginas más visitadas
echo "=== TOP 10 PÁGINAS ===" > "$OUTPUT_DIR/top_pages.txt"
awk '{print $7}' "$ACCESS_LOG" | sort | uniq -c | sort -nr | head -10 >> "$OUTPUT_DIR/top_pages.txt"

# Códigos de respuesta
echo "=== CÓDIGOS DE RESPUESTA ===" > "$OUTPUT_DIR/response_codes.txt"
awk '{print $9}' "$ACCESS_LOG" | sort | uniq -c | sort -nr >> "$OUTPUT_DIR/response_codes.txt"

# User agents más comunes
echo "=== TOP 10 USER AGENTS ===" > "$OUTPUT_DIR/user_agents.txt"
awk -F'"' '{print $6}' "$ACCESS_LOG" | sort | uniq -c | sort -nr | head -10 >> "$OUTPUT_DIR/user_agents.txt"

# Ataques potenciales (404s)
echo "=== 404 ERRORS ===" > "$OUTPUT_DIR/404_errors.txt"
awk '$9 == "404" {print $1, $7}' "$ACCESS_LOG" | sort | uniq -c | sort -nr | head -20 >> "$OUTPUT_DIR/404_errors.txt"

# Tiempos de respuesta lentos (si está configurado)
if grep -q "rt=" "$ACCESS_LOG"; then
    echo "=== SLOW RESPONSES (>2s) ===" > "$OUTPUT_DIR/slow_responses.txt"
    awk '$NF ~ /rt=/ && $NF > "rt=2.000" {print $1, $7, $NF}' "$ACCESS_LOG" >> "$OUTPUT_DIR/slow_responses.txt"
fi

echo "✅ Análisis completado. Resultados en: $OUTPUT_DIR"
```

---

## 🎯 Checklist de Optimización

### Lista de verificación pre-producción:

#### Configuración del servidor:
- [ ] MPM/worker processes optimizados según hardware
- [ ] Buffers y timeouts configurados
- [ ] Compresión gzip/brotli habilitada
- [ ] Headers de caché configurados
- [ ] Headers de seguridad implementados
- [ ] Rate limiting configurado
- [ ] SSL/TLS optimizado (A+ en SSL Labs)

#### PHP (si aplica):
- [ ] OPcache habilitado y optimizado
- [ ] PHP-FPM configurado según carga
- [ ] Memoria y timeouts ajustados
- [ ] Sessions en Redis/Memcached

#### Base de datos:
- [ ] Queries optimizadas con índices
- [ ] Connection pooling configurado
- [ ] Caché de queries habilitado
- [ ] Slow query log monitoreado

#### Monitoreo:
- [ ] Métricas de performance en tiempo real
- [ ] Alertas automáticas configuradas
- [ ] Logs centralizados y rotados
- [ ] Backups automatizados

#### Pruebas:
- [ ] Load testing realizado
- [ ] Performance baseline establecido
- [ ] CDN configurado para estáticos
- [ ] HTTP/2 o HTTP/3 habilitado

---

## 🔚 Conclusión

La optimización de servidores web es un proceso continuo que requiere:

1. **Medición constante** - No optimices sin métricas
2. **Pruebas graduales** - Un cambio a la vez
3. **Monitoreo proactivo** - Detecta problemas antes que los usuarios
4. **Automatización** - Scripts para tareas repetitivas
5. **Documentación** - Registra todos los cambios

### Próximos pasos:
- Implementa CDN (CloudFlare, AWS CloudFront)
- Considera HTTP/3 para mejor performance
- Explora containerización con Docker
- Automatiza deployments con CI/CD
- Implementa observabilidad avanzada

💡 **Recuerda:** La optimización es un balance entre performance, seguridad y mantenibilidad. Siempre prioriza la estabilidad sobre la velocidad.

---
**Andrés Nuñez - t4ifi**
