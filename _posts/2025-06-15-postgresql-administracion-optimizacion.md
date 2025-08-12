---
title: "PostgreSQL: Administración y Optimización Completa para Producción"
date: 2025-06-15 14:20:00 +0100
categories: [Bases de Datos, PostgreSQL]
tags: [postgresql, database, sql, administration, performance, optimization, backup]
---

# PostgreSQL: Administración y Optimización Completa para Producción

**PostgreSQL** es uno de los sistemas de gestión de bases de datos relacionales más avanzados y robustos del mundo. Su reputación como "la base de datos más avanzada del mundo de código abierto" no es casualidad: ofrece características empresariales, extensibilidad única y un rendimiento excepcional cuando se configura correctamente.

## ¿Por Qué PostgreSQL?

### Ventajas Clave

1. **ACID Compliant**: Garantiza transacciones confiables
2. **Extensibilidad**: Tipos de datos personalizados, funciones, operadores
3. **Concurrencia Avanzada**: MVCC (Multi-Version Concurrency Control)
4. **Estándares SQL**: Cumplimiento robusto con SQL estándar
5. **Replicación**: Múltiples opciones de alta disponibilidad
6. **Performance**: Optimizaciones avanzadas y paralelización

### Casos de Uso Ideales

- **Aplicaciones OLTP** de alta concurrencia
- **Data warehousing** y análisis complejos
- **Aplicaciones geoespaciales** (PostGIS)
- **APIs REST** con JSON nativo
- **Sistemas críticos** que requieren consistencia

## Instalación y Configuración Inicial

### Instalación en Ubuntu/Debian

```bash
# Actualizar repositorios
sudo apt update

# Instalar PostgreSQL y herramientas adicionales
sudo apt install postgresql postgresql-contrib postgresql-client

# Verificar instalación
sudo systemctl status postgresql

# Ver versión instalada
psql --version
```

### Instalación en CentOS/RHEL

```bash
# Instalar repositorio oficial
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Instalar PostgreSQL 15
sudo dnf install -y postgresql15-server postgresql15

# Inicializar base de datos
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb

# Habilitar y iniciar servicio
sudo systemctl enable postgresql-15
sudo systemctl start postgresql-15
```

### Configuración Inicial

```bash
# Cambiar a usuario postgres
sudo -u postgres psql

# Dentro de psql:
# Cambiar contraseña del usuario postgres
ALTER USER postgres PASSWORD 'tu_contraseña_segura';

# Crear usuario para aplicación
CREATE USER app_user WITH PASSWORD 'password_segura';

# Crear base de datos
CREATE DATABASE mi_aplicacion OWNER app_user;

# Otorgar privilegios
GRANT ALL PRIVILEGES ON DATABASE mi_aplicacion TO app_user;

# Salir
\q
```

## Configuración del Servidor

### Archivos de Configuración Principales

```bash
# Ubicación típica de archivos de configuración
/etc/postgresql/15/main/postgresql.conf    # Configuración principal
/etc/postgresql/15/main/pg_hba.conf        # Autenticación
/etc/postgresql/15/main/pg_ident.conf      # Mapeo de usuarios

# Localizar archivos en tu sistema
sudo -u postgres psql -c "SHOW config_file;"
sudo -u postgres psql -c "SHOW hba_file;"
```

### postgresql.conf - Configuración Principal

```bash
# postgresql.conf
# Configuración de memoria
shared_buffers = 256MB              # 25% de RAM total (para sistemas dedicados)
effective_cache_size = 1GB          # 75% de RAM total
work_mem = 4MB                      # Memoria por operación de ordenamiento
maintenance_work_mem = 64MB         # Memoria para operaciones de mantenimiento

# Configuración de WAL (Write Ahead Log)
wal_buffers = 16MB
checkpoint_completion_target = 0.7
wal_keep_size = 1GB
max_wal_size = 1GB
min_wal_size = 80MB

# Configuración de conexiones
max_connections = 200               # Ajustar según necesidades
listen_addresses = '*'              # Escuchar en todas las interfaces
port = 5432

# Logging
log_destination = 'stderr'
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 10MB
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 0
log_autovacuum_min_duration = 0
log_error_verbosity = default

# Performance
random_page_cost = 1.1              # Para SSDs
effective_io_concurrency = 200      # Para SSDs
max_worker_processes = 8
max_parallel_workers_per_gather = 2
max_parallel_workers = 8
max_parallel_maintenance_workers = 2
```

### pg_hba.conf - Configuración de Autenticación

```bash
# pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections
local   all             postgres                                peer
local   all             all                                     md5

# IPv4 local connections
host    all             all             127.0.0.1/32            md5

# IPv6 local connections
host    all             all             ::1/128                 md5

# Remote connections (ajustar según necesidades)
host    all             all             10.0.0.0/8              md5
host    all             all             192.168.0.0/16          md5

# SSL connections (recomendado para producción)
hostssl all             all             0.0.0.0/0               md5

# Replication connections
host    replication     replicator      192.168.1.0/24          md5
```

### Aplicar Configuración

```bash
# Recargar configuración sin reiniciar
sudo systemctl reload postgresql

# O desde psql
SELECT pg_reload_conf();

# Verificar configuración actual
SHOW shared_buffers;
SHOW effective_cache_size;
SHOW max_connections;
```

## Administración de Bases de Datos

### Gestión de Usuarios y Roles

```sql
-- Crear roles
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin;

-- Otorgar privilegios básicos
GRANT CONNECT ON DATABASE mi_aplicacion TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

GRANT CONNECT ON DATABASE mi_aplicacion TO readwrite;
GRANT USAGE, CREATE ON SCHEMA public TO readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO readwrite;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO readwrite;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO readwrite;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO readwrite;

-- Crear usuarios y asignar roles
CREATE USER app_readonly WITH PASSWORD 'password_readonly';
GRANT readonly TO app_readonly;

CREATE USER app_readwrite WITH PASSWORD 'password_readwrite';
GRANT readwrite TO app_readwrite;

CREATE USER app_admin WITH PASSWORD 'password_admin';
GRANT admin TO app_admin;
ALTER USER app_admin CREATEDB;
```

### Gestión de Esquemas

```sql
-- Crear esquemas para organizar tablas
CREATE SCHEMA ventas;
CREATE SCHEMA inventario;
CREATE SCHEMA usuarios;

-- Cambiar ownership
ALTER SCHEMA ventas OWNER TO app_user;

-- Configurar search_path
ALTER DATABASE mi_aplicacion SET search_path TO ventas, inventario, public;

-- Para un usuario específico
ALTER USER app_user SET search_path TO ventas, inventario, public;
```

### Backup y Restauración

#### pg_dump - Backup Lógico

```bash
# Backup completo de base de datos
pg_dump -h localhost -U postgres -d mi_aplicacion > backup_completo.sql

# Backup con compresión
pg_dump -h localhost -U postgres -d mi_aplicacion | gzip > backup_comprimido.sql.gz

# Backup en formato custom (recomendado)
pg_dump -h localhost -U postgres -d mi_aplicacion -Fc > backup_custom.dump

# Backup solo de esquema
pg_dump -h localhost -U postgres -d mi_aplicacion -s > esquema_only.sql

# Backup solo de datos
pg_dump -h localhost -U postgres -d mi_aplicacion -a > datos_only.sql

# Backup de tablas específicas
pg_dump -h localhost -U postgres -d mi_aplicacion -t usuarios -t productos > tablas_especificas.sql

# Backup con jobs paralelos (más rápido)
pg_dump -h localhost -U postgres -d mi_aplicacion -Fd -j 4 -f backup_paralelo/
```

#### Restauración

```bash
# Restaurar desde SQL
psql -h localhost -U postgres -d nueva_bd < backup_completo.sql

# Restaurar desde formato custom
pg_restore -h localhost -U postgres -d nueva_bd backup_custom.dump

# Restaurar con paralelismo
pg_restore -h localhost -U postgres -d nueva_bd -j 4 backup_custom.dump

# Restaurar solo esquema
pg_restore -h localhost -U postgres -d nueva_bd -s backup_custom.dump

# Restaurar solo datos
pg_restore -h localhost -U postgres -d nueva_bd -a backup_custom.dump

# Restaurar tabla específica
pg_restore -h localhost -U postgres -d nueva_bd -t usuarios backup_custom.dump
```

#### Backup Físico con pg_basebackup

```bash
# Configurar replicación en postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64MB

# Crear usuario de replicación
sudo -u postgres psql
CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'repl_password';

# Realizar backup físico
pg_basebackup -h localhost -D /backup/base/ -U replicator -v -P -W -Ft -z -Z 9

# Backup continuo de WAL
archive_mode = on
archive_command = 'test ! -f /backup/wal/%f && cp %p /backup/wal/%f'
```

### Scripts de Automatización

#### Script de Backup Automatizado

```bash
#!/bin/bash
# backup_postgresql.sh

set -e

# Configuración
DB_HOST="localhost"
DB_USER="postgres"
DB_NAME="mi_aplicacion"
BACKUP_DIR="/backups/postgresql"
RETENTION_DAYS=7

# Crear directorio si no existe
mkdir -p "$BACKUP_DIR"

# Timestamp para el archivo
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.dump"

echo "Iniciando backup de $DB_NAME..."

# Realizar backup
pg_dump -h "$DB_HOST" -U "$DB_USER" -d "$DB_NAME" -Fc > "$BACKUP_FILE"

# Verificar que el backup se creó correctamente
if [ $? -eq 0 ]; then
    echo "Backup completado: $BACKUP_FILE"
    
    # Comprimir backup
    gzip "$BACKUP_FILE"
    echo "Backup comprimido: ${BACKUP_FILE}.gz"
    
    # Limpiar backups antiguos
    find "$BACKUP_DIR" -name "*.dump.gz" -mtime +$RETENTION_DAYS -delete
    echo "Backups antiguos eliminados (más de $RETENTION_DAYS días)"
    
else
    echo "Error: Falló el backup de $DB_NAME" >&2
    exit 1
fi

echo "Script de backup completado"
```

#### Cron Job para Backup Automático

```bash
# Editar crontab
crontab -e

# Backup diario a las 2:00 AM
0 2 * * * /scripts/backup_postgresql.sh >> /var/log/postgresql_backup.log 2>&1

# Backup cada 6 horas
0 */6 * * * /scripts/backup_postgresql.sh >> /var/log/postgresql_backup.log 2>&1
```

## Optimización de Performance

### Análisis de Consultas

#### EXPLAIN y EXPLAIN ANALYZE

```sql
-- Ver plan de ejecución
EXPLAIN SELECT * FROM usuarios WHERE email = 'usuario@ejemplo.com';

-- Ver plan con estadísticas reales
EXPLAIN ANALYZE SELECT * FROM usuarios WHERE email = 'usuario@ejemplo.com';

-- Con más detalles
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT u.nombre, p.titulo 
FROM usuarios u 
JOIN posts p ON u.id = p.usuario_id 
WHERE u.activo = true;
```

#### Ejemplo de Interpretación

```sql
-- Consulta lenta
EXPLAIN ANALYZE
SELECT * FROM pedidos p
JOIN clientes c ON p.cliente_id = c.id
WHERE p.fecha_pedido >= '2025-01-01'
AND c.pais = 'España';

-- Resultado (ejemplo):
-- Nested Loop  (cost=0.29..8.32 rows=1 width=68) (actual time=0.123..145.234 rows=1500 loops=1)
--   ->  Seq Scan on clientes c  (cost=0.00..4.00 rows=1 width=36) (actual time=0.045..89.123 rows=750 loops=1)
--         Filter: ((pais)::text = 'España'::text)
--         Rows Removed by Filter: 9250
--   ->  Index Scan using idx_pedidos_cliente on pedidos p  (cost=0.29..4.31 rows=1 width=32) (actual time=0.045..0.067 rows=2 loops=750)
--         Index Cond: (cliente_id = c.id)
--         Filter: (fecha_pedido >= '2025-01-01'::date)
--         Rows Removed by Filter: 1
```

### Índices Estratégicos

#### Tipos de Índices

```sql
-- Índice B-tree (por defecto)
CREATE INDEX idx_usuarios_email ON usuarios (email);

-- Índice único
CREATE UNIQUE INDEX idx_usuarios_email_unique ON usuarios (email);

-- Índice compuesto
CREATE INDEX idx_pedidos_fecha_estado ON pedidos (fecha_pedido, estado);

-- Índice parcial
CREATE INDEX idx_usuarios_activos ON usuarios (email) WHERE activo = true;

-- Índice de expresión
CREATE INDEX idx_usuarios_email_lower ON usuarios (lower(email));

-- Índice GIN para búsqueda de texto completo
CREATE INDEX idx_posts_contenido_gin ON posts USING gin(to_tsvector('spanish', contenido));

-- Índice GiST para datos geoespaciales
CREATE INDEX idx_ubicaciones_gist ON ubicaciones USING gist(coordenadas);

-- Índice BRIN para datos muy grandes ordenados por tiempo
CREATE INDEX idx_logs_fecha_brin ON logs USING brin(fecha_creacion);
```

#### Estrategias de Indexación

```sql
-- Analizar uso de índices
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes 
ORDER BY idx_scan DESC;

-- Encontrar índices no utilizados
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan
FROM pg_stat_user_indexes 
WHERE idx_scan = 0;

-- Tamaño de índices
SELECT 
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) as size
FROM pg_indexes 
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexname::regclass) DESC;
```

### Optimización de Consultas

#### Reescritura de Consultas Lentas

```sql
-- MALO: Función en WHERE que impide uso de índices
SELECT * FROM usuarios WHERE upper(email) = 'USUARIO@EJEMPLO.COM';

-- BUENO: Usar índice funcional o comparación directa
CREATE INDEX idx_usuarios_email_upper ON usuarios (upper(email));
-- O mejor aún:
SELECT * FROM usuarios WHERE email = 'usuario@ejemplo.com';

-- MALO: OR que impide uso eficiente de índices
SELECT * FROM productos WHERE categoria = 'electrónicos' OR precio < 100;

-- BUENO: Usar UNION ALL
SELECT * FROM productos WHERE categoria = 'electrónicos'
UNION ALL
SELECT * FROM productos WHERE precio < 100 AND categoria != 'electrónicos';

-- MALO: Subconsulta correlacionada
SELECT u.nombre, 
       (SELECT COUNT(*) FROM pedidos p WHERE p.usuario_id = u.id) as total_pedidos
FROM usuarios u;

-- BUENO: JOIN con agregación
SELECT u.nombre, COALESCE(p.total_pedidos, 0) as total_pedidos
FROM usuarios u
LEFT JOIN (
    SELECT usuario_id, COUNT(*) as total_pedidos
    FROM pedidos
    GROUP BY usuario_id
) p ON u.id = p.usuario_id;
```

#### Window Functions para Performance

```sql
-- Ranking sin subconsultas
SELECT 
    nombre,
    salario,
    ROW_NUMBER() OVER (ORDER BY salario DESC) as ranking,
    DENSE_RANK() OVER (PARTITION BY departamento ORDER BY salario DESC) as ranking_dept
FROM empleados;

-- Agregaciones móviles
SELECT 
    fecha,
    ventas,
    AVG(ventas) OVER (ORDER BY fecha ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as media_7_dias
FROM ventas_diarias;
```

### Configuración Avanzada

#### Tuning de Memoria

```sql
-- Calcular configuración óptima basada en RAM disponible
-- Para un servidor con 8GB RAM dedicado a PostgreSQL:

shared_buffers = 2GB                    -- 25% de RAM
effective_cache_size = 6GB              -- 75% de RAM
work_mem = 16MB                         -- Para 100 conexiones: (RAM - shared_buffers) / (max_connections * 3)
maintenance_work_mem = 512MB            -- Para operaciones de mantenimiento
```

#### Configuración de Autovacuum

```sql
-- Configuración agresiva para alta concurrencia
autovacuum = on
autovacuum_max_workers = 6
autovacuum_naptime = 15s                -- Revisar tablas cada 15 segundos
autovacuum_vacuum_threshold = 25        -- Vacuum después de 25 tuplas modificadas
autovacuum_vacuum_scale_factor = 0.1    -- O 10% de la tabla
autovacuum_analyze_threshold = 25
autovacuum_analyze_scale_factor = 0.05  -- O 5% de la tabla

-- Para tablas específicas muy activas
ALTER TABLE tabla_muy_activa SET (
    autovacuum_vacuum_scale_factor = 0.01,
    autovacuum_analyze_scale_factor = 0.005
);
```

## Monitoreo y Diagnóstico

### Consultas de Monitoreo Esenciales

```sql
-- Estado general del servidor
SELECT 
    version(),
    current_setting('max_connections'),
    current_setting('shared_buffers'),
    current_setting('effective_cache_size');

-- Conexiones activas
SELECT 
    count(*) as total_connections,
    count(*) FILTER (WHERE state = 'active') as active_connections,
    count(*) FILTER (WHERE state = 'idle') as idle_connections,
    count(*) FILTER (WHERE state = 'idle in transaction') as idle_in_transaction
FROM pg_stat_activity;

-- Consultas lentas actuales
SELECT 
    pid,
    now() - pg_stat_activity.query_start AS duration,
    query,
    state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes'
  AND state != 'idle'
ORDER BY duration DESC;

-- Bloqueos activos
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS current_statement_in_blocking_process
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- Uso de espacio por tabla
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Estadísticas de actividad de tablas
SELECT 
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del
FROM pg_stat_user_tables
ORDER BY seq_scan DESC;
```

### Script de Monitoreo Automatizado

```bash
#!/bin/bash
# monitor_postgresql.sh

DB_HOST="localhost"
DB_USER="postgres"
DB_NAME="mi_aplicacion"
LOG_FILE="/var/log/postgresql_monitor.log"

echo "$(date): Iniciando monitoreo PostgreSQL" >> "$LOG_FILE"

# Verificar conexiones
CONNECTIONS=$(psql -h "$DB_HOST" -U "$DB_USER" -d "$DB_NAME" -t -c "SELECT count(*) FROM pg_stat_activity;")
MAX_CONNECTIONS=$(psql -h "$DB_HOST" -U "$DB_USER" -d "$DB_NAME" -t -c "SELECT setting FROM pg_settings WHERE name='max_connections';")

CONNECTION_USAGE=$(echo "scale=2; $CONNECTIONS * 100 / $MAX_CONNECTIONS" | bc)

echo "$(date): Conexiones: $CONNECTIONS/$MAX_CONNECTIONS (${CONNECTION_USAGE}%)" >> "$LOG_FILE"

# Alerta si uso de conexiones > 80%
if (( $(echo "$CONNECTION_USAGE > 80" | bc -l) )); then
    echo "$(date): ALERTA: Uso alto de conexiones: ${CONNECTION_USAGE}%" >> "$LOG_FILE"
    # Enviar alerta por email o Slack
fi

# Verificar consultas lentas
SLOW_QUERIES=$(psql -h "$DB_HOST" -U "$DB_USER" -d "$DB_NAME" -t -c "
SELECT count(*) 
FROM pg_stat_activity 
WHERE state = 'active' 
  AND now() - query_start > interval '30 seconds';")

if [ "$SLOW_QUERIES" -gt 0 ]; then
    echo "$(date): ALERTA: $SLOW_QUERIES consultas lentas detectadas" >> "$LOG_FILE"
fi

# Verificar espacio en disco
DISK_USAGE=$(df -h /var/lib/postgresql | awk 'NR==2 {print $5}' | sed 's/%//')
if [ "$DISK_USAGE" -gt 85 ]; then
    echo "$(date): ALERTA: Uso de disco: ${DISK_USAGE}%" >> "$LOG_FILE"
fi

echo "$(date): Monitoreo completado" >> "$LOG_FILE"
```

## Alta Disponibilidad y Replicación

### Streaming Replication

#### Configuración del Master

```bash
# postgresql.conf en el master
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64MB
synchronous_commit = on
synchronous_standby_names = 'standby1'

# pg_hba.conf en el master
host    replication     replicator      192.168.1.100/32        md5
```

#### Configuración del Standby

```bash
# Crear backup base
pg_basebackup -h master_ip -D /var/lib/postgresql/15/main -U replicator -v -P -W

# Crear standby.signal
touch /var/lib/postgresql/15/main/standby.signal

# postgresql.conf en el standby
primary_conninfo = 'host=master_ip port=5432 user=replicator password=repl_password'
restore_command = 'cp /archive/%f %p'
archive_cleanup_command = 'pg_archivecleanup /archive %r'
```

### Configuración de pgBouncer (Connection Pooling)

```ini
# /etc/pgbouncer/pgbouncer.ini
[databases]
mi_aplicacion = host=localhost port=5432 dbname=mi_aplicacion

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
logfile = /var/log/pgbouncer/pgbouncer.log
pidfile = /var/run/pgbouncer/pgbouncer.pid
admin_users = postgres
stats_users = postgres

# Pool settings
pool_mode = transaction
max_client_conn = 200
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
reserve_pool_timeout = 5
```

## Mejores Prácticas

### Seguridad

```sql
-- Crear roles con privilegios mínimos
CREATE ROLE app_reader;
GRANT CONNECT ON DATABASE mi_aplicacion TO app_reader;
GRANT USAGE ON SCHEMA public TO app_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_reader;

-- Auditoría con log_statement
log_statement = 'mod'                # Loguear todas las modificaciones
log_min_duration_statement = 1000    # Loguear consultas > 1 segundo

-- Encriptación en tránsito
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_ca_file = 'ca.crt'
```

### Mantenimiento Regular

```sql
-- Script de mantenimiento semanal
DO $$
DECLARE
    table_name text;
BEGIN
    -- Vacuum y analyze para todas las tablas
    FOR table_name IN 
        SELECT tablename FROM pg_tables WHERE schemaname = 'public'
    LOOP
        EXECUTE 'VACUUM ANALYZE ' || table_name;
        RAISE NOTICE 'Processed table: %', table_name;
    END LOOP;
END
$$;

-- Reindex mensual
REINDEX DATABASE mi_aplicacion;

-- Limpiar logs antiguos
SELECT pg_rotate_logfile();
```

### Optimización Continua

```sql
-- Análisis de estadísticas de consultas
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Ver top 10 consultas más lentas
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- Reset de estadísticas
SELECT pg_stat_statements_reset();
```

## Herramientas Útiles

### pgAdmin - Interfaz Gráfica

```bash
# Instalar pgAdmin 4
curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add
sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list'
sudo apt update
sudo apt install pgadmin4
```

### psql Tips y Tricks

```sql
-- Comandos útiles en psql
\l          -- Listar bases de datos
\c db_name  -- Conectar a base de datos
\dt         -- Listar tablas
\d+ tabla   -- Describir tabla con detalles
\di         -- Listar índices
\du         -- Listar usuarios/roles
\timing     -- Activar medición de tiempo
\x          -- Formato expandido para resultados
\copy       -- Importar/exportar datos

-- Configurar .psqlrc para mejores defaults
\set QUIET 1
\set ON_ERROR_ROLLBACK interactive
\set VERBOSITY verbose
\set PROMPT1 '%[%033[1m%]%M %n@%/%R%[%033[0m%]%# '
\set PROMPT2 '[more] %R > '
\timing
\set QUIET 0
```

## Conclusión

PostgreSQL es una base de datos tremendamente poderosa que, cuando se configura y administra correctamente, puede manejar cargas de trabajo empresariales exigentes. Las claves del éxito incluyen:

1. **Configuración apropiada** de memoria y WAL
2. **Índices estratégicos** basados en patrones de consulta
3. **Monitoreo proactivo** de performance y recursos
4. **Backups regulares** y estrategias de recuperación
5. **Mantenimiento preventivo** con vacuum y análisis
6. **Seguridad robusta** con roles y permisos granulares

La inversión en aprender PostgreSQL a fondo se paga rápidamente en términos de performance, confiabilidad y capacidades avanzadas que otros sistemas de bases de datos simplemente no pueden ofrecer.


---
**Andrés Nuñez - t4ifi**
