---
title: "Redis: Base de Datos en Memoria para Alto Rendimiento"
date: 2025-07-31 10:00:00 +0000
categories: [Bases de Datos, NoSQL]
tags: [redis, cache, nosql, memoria, performance]
---

# Redis: Base de Datos en Memoria para Alto Rendimiento

Redis (Remote Dictionary Server) es una estructura de datos en memoria open-source que se utiliza como base de datos, cache y message broker, conocida por su velocidad excepcional y versatilidad.

## Introducción a Redis

### ¿Qué es Redis?

Redis es una base de datos NoSQL en memoria que almacena datos como pares clave-valor, soportando múltiples estructuras de datos complejas y operaciones atómicas con latencia submilisegundo.

### Características Principales

#### Ventajas de Redis
- **Velocidad**: Todas las operaciones en memoria RAM
- **Versatilidad**: Múltiples estructuras de datos
- **Persistencia**: Opciones flexibles de almacenamiento en disco
- **Escalabilidad**: Clustering y replicación nativa
- **Atomicidad**: Operaciones transaccionales
- **Pub/Sub**: Sistema de mensajería integrado

#### Casos de Uso Típicos
```bash
# 1. Cache de aplicaciones web
# 2. Almacén de sesiones
# 3. Leaderboards y contadores
# 4. Cola de mensajes
# 5. Rate limiting
# 6. Real-time analytics
```

## Instalación y Configuración

### Instalación de Redis

#### Ubuntu/Debian
```bash
# Actualizar repositorios
apt update

# Instalar Redis
apt install redis-server

# Verificar instalación
redis-server --version
redis-cli --version

# Iniciar servicio
systemctl start redis-server
systemctl enable redis-server

# Verificar estado
systemctl status redis-server
```

#### CentOS/RHEL
```bash
# Instalar EPEL repository
yum install epel-release

# Instalar Redis
yum install redis

# O compilar desde fuente
wget http://download.redis.io/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
cd redis-stable
make
make install
```

#### Docker
```bash
# Ejecutar Redis en Docker
docker run -d --name redis-server -p 6379:6379 redis:latest

# Con configuración personalizada
docker run -d --name redis-server \
  -p 6379:6379 \
  -v /ruta/redis.conf:/usr/local/etc/redis/redis.conf \
  redis:latest redis-server /usr/local/etc/redis/redis.conf
```

### Configuración Básica

#### Archivo de Configuración Principal
```bash
# Ubicación típica: /etc/redis/redis.conf
vim /etc/redis/redis.conf

# Configuraciones básicas importantes:
bind 127.0.0.1 ::1          # IPs permitidas
port 6379                   # Puerto por defecto
daemonize yes               # Ejecutar como daemon
pidfile /var/run/redis.pid  # Archivo PID
logfile /var/log/redis.log  # Archivo de log
dir /var/lib/redis/         # Directorio de trabajo
```

#### Configuración de Memoria
```bash
# redis.conf
maxmemory 2gb                    # Límite de memoria
maxmemory-policy allkeys-lru     # Política de eliminación

# Políticas disponibles:
# noeviction: Sin eliminación (por defecto)
# allkeys-lru: LRU en todas las claves
# volatile-lru: LRU solo en claves con TTL
# allkeys-random: Aleatorio en todas las claves
# volatile-random: Aleatorio en claves con TTL
# volatile-ttl: Eliminar las que expiran primero
```

#### Configuración de Red
```bash
# Permitir conexiones remotas
bind 0.0.0.0

# Configurar password
requirepass tu_password_seguro

# Timeout de conexión inactiva
timeout 300

# Límites de conexión
maxclients 10000
```

## Tipos de Datos en Redis

### Strings: Tipo Básico

#### Operaciones con Strings
```bash
# Conectar a Redis CLI
redis-cli

# Operaciones básicas
SET usuario:1:nombre "Juan Pérez"
GET usuario:1:nombre

# Con expiración
SET sesion:abc123 "datos_sesion" EX 3600  # Expira en 1 hora
SETEX token:xyz789 300 "bearer_token"     # Expira en 5 minutos

# Operaciones atómicas de incremento
SET contador 0
INCR contador          # Incrementa en 1
INCRBY contador 5      # Incrementa en 5
DECR contador          # Decrementa en 1

# Operaciones de bit
SETBIT usuario:activo 1001 1  # Marcar usuario 1001 como activo
GETBIT usuario:activo 1001    # Verificar si está activo
BITCOUNT usuario:activo       # Contar usuarios activos
```

#### Casos de Uso para Strings
```bash
# Cache de contenido web
SET "pagina:/home" "<html>contenido</html>" EX 300

# Contadores de visitas
INCR "visitas:pagina:/home"

# Rate limiting
SET "rate_limit:usuario:123" "1" EX 60
INCR "rate_limit:usuario:123"
```

### Hashes: Objetos Estructurados

#### Operaciones con Hashes
```bash
# Crear hash de usuario
HSET usuario:1 nombre "Juan" apellido "Pérez" edad 30 email "juan@ejemplo.com"

# Obtener campo específico
HGET usuario:1 nombre

# Obtener múltiples campos
HMGET usuario:1 nombre edad

# Obtener todos los campos
HGETALL usuario:1

# Verificar existencia de campo
HEXISTS usuario:1 telefono

# Incrementar campo numérico
HINCRBY usuario:1 puntos 100

# Obtener todas las claves
HKEYS usuario:1

# Obtener todos los valores
HVALS usuario:1
```

#### Casos de Uso para Hashes
```bash
# Perfiles de usuario
HSET perfil:usuario123 nombre "Ana" puntos 1500 nivel "premium"

# Configuraciones de aplicación
HSET config:app debug "true" timeout "30" max_connections "100"

# Métricas por categoría
HSET metricas:diarias ventas 150 usuarios_nuevos 45 ingresos 75000
```

### Lists: Colas y Pilas

#### Operaciones con Lists
```bash
# Agregar elementos (izquierda/derecha)
LPUSH cola:tareas "tarea1" "tarea2"
RPUSH cola:tareas "tarea3"

# Obtener elementos
LRANGE cola:tareas 0 -1    # Todos los elementos
LRANGE cola:tareas 0 5     # Primeros 5 elementos

# Extraer elementos (pop)
LPOP cola:tareas           # Extraer del inicio
RPOP cola:tareas           # Extraer del final

# Operaciones bloqueantes
BLPOP cola:tareas 10       # Esperar hasta 10 segundos

# Longitud de la lista
LLEN cola:tareas

# Obtener elemento por índice
LINDEX cola:tareas 0

# Establecer elemento por índice
LSET cola:tareas 0 "nueva_tarea"
```

#### Casos de Uso para Lists
```bash
# Cola de trabajos
LPUSH cola:emails "enviar_email_123"
RPOP cola:emails

# Timeline de actividades
LPUSH timeline:usuario123 "accion_reciente"
LRANGE timeline:usuario123 0 9  # Últimas 10 actividades

# Log de eventos
LPUSH logs:errores "$(date): Error en función X"
```

### Sets: Conjuntos Únicos

#### Operaciones con Sets
```bash
# Agregar miembros
SADD tags:articulo1 "python" "programming" "tutorial"

# Verificar membresía
SISMEMBER tags:articulo1 "python"

# Obtener todos los miembros
SMEMBERS tags:articulo1

# Obtener miembro aleatorio
SRANDMEMBER tags:articulo1

# Remover miembro
SREM tags:articulo1 "tutorial"

# Operaciones de conjuntos
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"

SINTER set1 set2           # Intersección
SUNION set1 set2           # Unión
SDIFF set1 set2            # Diferencia

# Cardinalidad (tamaño)
SCARD tags:articulo1
```

#### Casos de Uso para Sets
```bash
# Etiquetas de artículos
SADD tags:post123 "redis" "database" "nosql"

# Usuarios únicos por día
SADD usuarios:2025-07-31 "user1" "user2" "user3"
SCARD usuarios:2025-07-31

# Seguidores únicos
SADD seguidores:usuario123 "follower1" "follower2"
SINTER seguidores:usuario123 seguidores:usuario456  # Seguidores comunes
```

### Sorted Sets: Conjuntos Ordenados

#### Operaciones con Sorted Sets
```bash
# Agregar miembros con score
ZADD leaderboard 1500 "jugador1" 2000 "jugador2" 1200 "jugador3"

# Obtener rango por posición
ZRANGE leaderboard 0 -1           # Todos (orden ascendente)
ZREVRANGE leaderboard 0 -1        # Todos (orden descendente)
ZRANGE leaderboard 0 2 WITHSCORES # Top 3 con scores

# Obtener rango por score
ZRANGEBYSCORE leaderboard 1000 2000

# Obtener score de miembro
ZSCORE leaderboard "jugador1"

# Incrementar score
ZINCRBY leaderboard 100 "jugador1"

# Obtener ranking de miembro
ZRANK leaderboard "jugador1"      # Posición ascendente
ZREVRANK leaderboard "jugador1"   # Posición descendente

# Eliminar miembros
ZREM leaderboard "jugador3"
```

#### Casos de Uso para Sorted Sets
```bash
# Tabla de clasificación
ZADD puntuacion:juego $(date +%s) "usuario123:1500"

# Trending topics por tiempo
ZADD trending $(date +%s) "topic1" $(date +%s) "topic2"

# Prioridad de tareas
ZADD cola:prioridad 1 "tarea_urgente" 5 "tarea_normal" 10 "tarea_baja"
```

## Persistencia y Backup

### Tipos de Persistencia

#### RDB (Redis Database Backup)
```bash
# Configuración en redis.conf
save 900 1      # Guardar si al menos 1 clave cambió en 900 segundos
save 300 10     # Guardar si al menos 10 claves cambiaron en 300 segundos
save 60 10000   # Guardar si al menos 10000 claves cambiaron en 60 segundos

# Configuraciones adicionales
dbfilename dump.rdb
dir /var/lib/redis/
rdbcompression yes
rdbchecksum yes

# Comandos para backup manual
SAVE        # Bloquea Redis durante el backup
BGSAVE      # Backup en background
LASTSAVE    # Timestamp del último backup exitoso
```

#### AOF (Append Only File)
```bash
# Habilitar AOF en redis.conf
appendonly yes
appendfilename "appendonly.aof"

# Políticas de sincronización
appendfsync everysec    # Sincronizar cada segundo (recomendado)
appendfsync always      # Sincronizar cada escritura (más lento)
appendfsync no          # Dejar al SO decidir

# Reescritura automática del AOF
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Comando manual de reescritura
BGREWRITEAOF
```

### Estrategias de Backup

#### Script de Backup Automatizado
```bash
#!/bin/bash
# redis_backup.sh

REDIS_CLI="/usr/bin/redis-cli"
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Crear directorio de backup
mkdir -p "$BACKUP_DIR"

# Ejecutar BGSAVE
echo "Iniciando backup de Redis..."
$REDIS_CLI BGSAVE

# Esperar a que termine el backup
while [ $($REDIS_CLI LASTSAVE) -eq $($REDIS_CLI LASTSAVE) ]; do
    sleep 1
done

# Copiar dump.rdb
cp /var/lib/redis/dump.rdb "$BACKUP_DIR/dump_$DATE.rdb"

# Comprimir backup
gzip "$BACKUP_DIR/dump_$DATE.rdb"

# Limpiar backups antiguos
find "$BACKUP_DIR" -name "dump_*.rdb.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completado: dump_$DATE.rdb.gz"
```

#### Backup de AOF
```bash
#!/bin/bash
# aof_backup.sh

AOF_FILE="/var/lib/redis/appendonly.aof"
BACKUP_DIR="/backup/redis/aof"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Reescribir AOF para optimizar
redis-cli BGREWRITEAOF

# Esperar a que termine la reescritura
while [ $(redis-cli INFO persistence | grep aof_rewrite_in_progress:1 | wc -l) -eq 1 ]; do
    sleep 1
done

# Copiar AOF
cp "$AOF_FILE" "$BACKUP_DIR/appendonly_$DATE.aof"

# Comprimir
gzip "$BACKUP_DIR/appendonly_$DATE.aof"

echo "Backup AOF completado: appendonly_$DATE.aof.gz"
```

## Replicación y Alta Disponibilidad

### Configuración de Replicación

#### Master-Slave Setup

##### Configuración del Master
```bash
# redis-master.conf
bind 0.0.0.0
port 6379
requirepass master_password
masterauth slave_password

# Configuraciones para replicación
repl-diskless-sync yes
repl-diskless-sync-delay 5
```

##### Configuración del Slave
```bash
# redis-slave.conf
bind 0.0.0.0
port 6379
replicaof 192.168.1.100 6379  # IP del master
masterauth master_password
requirepass slave_password

# Configuraciones de slave
replica-read-only yes
replica-serve-stale-data yes
```

#### Comandos de Replicación
```bash
# Ver información de replicación
INFO replication

# Promover slave a master
REPLICAOF NO ONE

# Cambiar master
REPLICAOF 192.168.1.101 6379
```

### Redis Sentinel: Failover Automático

#### Configuración de Sentinel
```bash
# sentinel.conf
port 26379
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel auth-pass mymaster master_password
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

# Configurar múltiples instancias de Sentinel
# Mínimo 3 instancias para quorum
```

#### Iniciar Sentinel
```bash
# Iniciar Redis Sentinel
redis-sentinel /etc/redis/sentinel.conf

# O usando redis-server
redis-server /etc/redis/sentinel.conf --sentinel
```

#### Monitoreo con Sentinel
```bash
# Conectar a Sentinel
redis-cli -p 26379

# Comandos de Sentinel
SENTINEL masters
SENTINEL slaves mymaster
SENTINEL sentinels mymaster
SENTINEL get-master-addr-by-name mymaster
```

## Redis Cluster: Escalabilidad Horizontal

### Configuración de Cluster

#### Configuración de Nodos
```bash
# redis-cluster.conf (para cada nodo)
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 15000
appendonly yes
```

#### Crear Cluster
```bash
# Crear cluster con 6 nodos (3 masters, 3 slaves)
redis-cli --cluster create \
  192.168.1.100:7000 192.168.1.101:7000 192.168.1.102:7000 \
  192.168.1.100:7001 192.168.1.101:7001 192.168.1.102:7001 \
  --cluster-replicas 1

# Verificar cluster
redis-cli --cluster check 192.168.1.100:7000
```

#### Gestión de Cluster
```bash
# Información del cluster
redis-cli -c -p 7000
CLUSTER INFO
CLUSTER NODES

# Añadir nodo al cluster
redis-cli --cluster add-node 192.168.1.103:7000 192.168.1.100:7000

# Rebalancear slots
redis-cli --cluster rebalance 192.168.1.100:7000

# Eliminar nodo
redis-cli --cluster del-node 192.168.1.100:7000 <node-id>
```

## Monitoreo y Performance

### Comandos de Monitoreo

#### Información del Sistema
```bash
# Información general
INFO

# Secciones específicas
INFO memory
INFO replication
INFO persistence
INFO clients
INFO stats

# Estadísticas en tiempo real
MONITOR

# Comandos lentos
SLOWLOG GET 10
SLOWLOG LEN
SLOWLOG RESET
```

#### Métricas de Memoria
```bash
# Uso de memoria
INFO memory

# Analizar uso por tipo de datos
MEMORY USAGE key_name

# Estadísticas de memoria
MEMORY STATS

# Información de clientes conectados
CLIENT LIST
CLIENT INFO
```

### Redis-cli Tools Avanzadas

#### Análisis de Performance
```bash
# Latencia de comandos
redis-cli --latency -h 192.168.1.100 -p 6379

# Latencia de red
redis-cli --latency-history -h 192.168.1.100 -p 6379

# Throughput de operaciones
redis-cli --stat

# Big keys (claves que usan mucha memoria)
redis-cli --bigkeys

# Patrón de acceso
redis-cli --hotkeys
```

#### Debugging y Profiling
```bash
# Analizar comandos ejecutándose
redis-cli MONITOR

# Configurar slow log
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 1000

# Ver memoria por categorías
MEMORY DOCTOR
```

### Scripting con Lua

#### Ejecutar Scripts Lua
```bash
# Script simple
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 mykey myvalue

# Script de contador atómico
EVAL "
local current = redis.call('get', KEYS[1])
if current == false then
    current = 0
else
    current = tonumber(current)
end
local new_value = current + tonumber(ARGV[1])
redis.call('set', KEYS[1], new_value)
return new_value
" 1 contador 5
```

#### Cargar y Ejecutar Scripts
```bash
# Cargar script
SCRIPT LOAD "return redis.call('get', KEYS[1])"
# Retorna SHA1 del script

# Ejecutar por SHA1
EVALSHA <sha1> 1 mykey

# Verificar si script existe
SCRIPT EXISTS <sha1>

# Limpiar cache de scripts
SCRIPT FLUSH
```

## Casos de Uso Avanzados

### Cache Distribuido

#### Implementación de Cache
```python
import redis
import json
import hashlib

class RedisCache:
    def __init__(self, host='localhost', port=6379, db=0):
        self.redis = redis.Redis(host=host, port=port, db=db)
        self.default_timeout = 3600  # 1 hora
    
    def get(self, key):
        """Obtener valor del cache"""
        value = self.redis.get(key)
        if value:
            return json.loads(value)
        return None
    
    def set(self, key, value, timeout=None):
        """Guardar valor en cache"""
        timeout = timeout or self.default_timeout
        return self.redis.setex(key, timeout, json.dumps(value))
    
    def delete(self, key):
        """Eliminar del cache"""
        return self.redis.delete(key)
    
    def cache_function(self, timeout=None):
        """Decorador para cachear funciones"""
        def decorator(func):
            def wrapper(*args, **kwargs):
                # Generar clave única
                key_data = f"{func.__name__}:{args}:{sorted(kwargs.items())}"
                cache_key = hashlib.md5(key_data.encode()).hexdigest()
                
                # Verificar cache
                result = self.get(cache_key)
                if result is not None:
                    return result
                
                # Ejecutar función y cachear resultado
                result = func(*args, **kwargs)
                self.set(cache_key, result, timeout)
                return result
            return wrapper
        return decorator

# Uso del cache
cache = RedisCache()

@cache.cache_function(timeout=1800)
def consulta_pesada(user_id):
    # Simulación de consulta costosa
    return {"user_id": user_id, "data": "información compleja"}
```

### Session Store

#### Gestión de Sesiones Web
```python
import redis
import json
import uuid
from datetime import datetime, timedelta

class RedisSessionStore:
    def __init__(self, redis_host='localhost', redis_port=6379):
        self.redis = redis.Redis(host=redis_host, port=redis_port)
        self.session_timeout = 3600  # 1 hora
    
    def create_session(self, user_id, user_data=None):
        """Crear nueva sesión"""
        session_id = str(uuid.uuid4())
        session_data = {
            'user_id': user_id,
            'created_at': datetime.now().isoformat(),
            'last_accessed': datetime.now().isoformat(),
            'user_data': user_data or {}
        }
        
        session_key = f"session:{session_id}"
        self.redis.setex(session_key, self.session_timeout, json.dumps(session_data))
        return session_id
    
    def get_session(self, session_id):
        """Obtener datos de sesión"""
        session_key = f"session:{session_id}"
        session_data = self.redis.get(session_key)
        
        if session_data:
            data = json.loads(session_data)
            # Actualizar last_accessed
            data['last_accessed'] = datetime.now().isoformat()
            self.redis.setex(session_key, self.session_timeout, json.dumps(data))
            return data
        return None
    
    def update_session(self, session_id, user_data):
        """Actualizar datos de sesión"""
        session_data = self.get_session(session_id)
        if session_data:
            session_data['user_data'].update(user_data)
            session_key = f"session:{session_id}"
            self.redis.setex(session_key, self.session_timeout, json.dumps(session_data))
            return True
        return False
    
    def destroy_session(self, session_id):
        """Eliminar sesión"""
        session_key = f"session:{session_id}"
        return self.redis.delete(session_key)
```

### Rate Limiting

#### Implementación de Rate Limiting
```bash
# Script Lua para rate limiting
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current_time = tonumber(ARGV[3])

-- Limpiar entradas expiradas
redis.call('ZREMRANGEBYSCORE', key, 0, current_time - window)

-- Contar requests actuales
local current_requests = redis.call('ZCARD', key)

if current_requests < limit then
    -- Agregar nueva request
    redis.call('ZADD', key, current_time, current_time)
    redis.call('EXPIRE', key, window)
    return {1, limit - current_requests - 1}
else
    return {0, 0}
end
```

#### Rate Limiter en Python
```python
import redis
import time

class RateLimiter:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.script = """
        local key = KEYS[1]
        local limit = tonumber(ARGV[1])
        local window = tonumber(ARGV[2])
        local current_time = tonumber(ARGV[3])
        
        redis.call('ZREMRANGEBYSCORE', key, 0, current_time - window)
        local current_requests = redis.call('ZCARD', key)
        
        if current_requests < limit then
            redis.call('ZADD', key, current_time, current_time)
            redis.call('EXPIRE', key, window)
            return {1, limit - current_requests - 1}
        else
            return {0, 0}
        end
        """
        self.script_sha = self.redis.script_load(self.script)
    
    def is_allowed(self, identifier, limit, window_seconds):
        """Verificar si la request está permitida"""
        key = f"rate_limit:{identifier}"
        current_time = int(time.time())
        
        result = self.redis.evalsha(
            self.script_sha, 1, key, limit, window_seconds, current_time
        )
        
        return {
            'allowed': bool(result[0]),
            'remaining': result[1]
        }

# Uso
redis_client = redis.Redis()
limiter = RateLimiter(redis_client)

# Permitir 100 requests por minuto
result = limiter.is_allowed('user:123', 100, 60)
if result['allowed']:
    print(f"Request permitida. Quedan: {result['remaining']}")
else:
    print("Rate limit excedido")
```

## Seguridad en Redis

### Configuración de Seguridad

#### Autenticación y Autorización
```bash
# redis.conf
requirepass tu_password_muy_seguro

# Cambiar password en runtime
CONFIG SET requirepass nuevo_password

# ACL (Access Control Lists) - Redis 6+
ACL SETUSER usuario1 on >password1 ~cached:* +get +set
ACL SETUSER readonly on >readonly_pass ~* +@read -@write

# Ver usuarios ACL
ACL LIST
ACL WHOAMI
```

#### Configuraciones de Red Seguras
```bash
# Binding seguro
bind 127.0.0.1 192.168.1.100

# Deshabilitar comandos peligrosos
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""
rename-command CONFIG "CONFIG_b840fc02d524045429941cc15f59e41cb7be6c52"

# Protected mode
protected-mode yes
```

#### TLS/SSL
```bash
# Configurar TLS
port 0
tls-port 6380
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt

# Conectar con TLS
redis-cli --tls --cert /path/to/client.crt --key /path/to/client.key
```

## Optimización y Best Practices

### Optimización de Memoria

#### Configuraciones de Memoria
```bash
# redis.conf
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# Compresión LZF
rdbcompression yes

# Política de memoria
maxmemory-policy allkeys-lru
```

#### Naming Conventions
```bash
# Usar prefijos consistentes
user:1001:profile
user:1001:sessions
cache:homepage:html
metrics:daily:2025-07-31
queue:emails:high_priority

# Evitar nombres muy largos
# Mal: "very_long_key_name_that_uses_too_much_memory"
# Bien: "vlkn:tmum" o usar hash si es necesario
```

### Performance Best Practices

#### Optimización de Operaciones
```bash
# Usar pipelines para múltiples comandos
redis-cli --pipe < comandos.txt

# Usar MGET/MSET para múltiples claves
MSET key1 value1 key2 value2 key3 value3
MGET key1 key2 key3

# Evitar KEYS en producción, usar SCAN
SCAN 0 MATCH user:* COUNT 100
```

#### Monitoreo Continuo
```bash
#!/bin/bash
# redis_health_check.sh

REDIS_CLI="redis-cli"
THRESHOLDS_MEM=80
THRESHOLD_CLIENTS=1000

# Verificar conectividad
if ! $REDIS_CLI ping > /dev/null 2>&1; then
    echo "CRITICAL: Redis no responde"
    exit 2
fi

# Verificar memoria
MEMORY_USAGE=$($REDIS_CLI INFO memory | grep used_memory_rss_human | cut -d: -f2 | tr -d '\r')
MAX_MEMORY=$($REDIS_CLI CONFIG GET maxmemory | tail -1)

# Verificar clientes conectados
CONNECTED_CLIENTS=$($REDIS_CLI INFO clients | grep connected_clients | cut -d: -f2 | tr -d '\r')

echo "Redis Health Check:"
echo "Memory Usage: $MEMORY_USAGE"
echo "Max Memory: $MAX_MEMORY"
echo "Connected Clients: $CONNECTED_CLIENTS"

# Alertas si excede umbrales
if [ "$CONNECTED_CLIENTS" -gt "$THRESHOLD_CLIENTS" ]; then
    echo "WARNING: Demasiados clientes conectados"
fi
```

## Conclusiones

Redis es una herramienta fundamental para:

1. **Caching**: Reducir latencia y carga en bases de datos principales
2. **Session Storage**: Gestión escalable de sesiones web
3. **Real-time Analytics**: Contadores y métricas en tiempo real
4. **Message Queues**: Comunicación asíncrona entre servicios
5. **Rate Limiting**: Control de acceso y protección de APIs

Dominar Redis es esencial para construir aplicaciones web modernas de alto rendimiento.

---

*Andrés Núñez - Especialista en Bases de Datos NoSQL y Arquitectura de Alto Rendimiento*
