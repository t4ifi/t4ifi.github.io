---
title: "Backup y Recuperación en Linux: Estrategias Profesionales para Proteger Datos"
date: 2025-07-10 10:00:00 +0000
categories: [Administración, Backup]
tags: [backup, recovery, rsync, tar, dd, bacula, disaster recovery]
---

# Backup y Recuperación en Linux: Estrategias Profesionales para Proteger Datos

Los backups son la última línea de defensa contra la pérdida de datos. Una estrategia de backup bien diseñada puede ser la diferencia entre un pequeño inconveniente y un desastre catastrófico.

## Fundamentos de Backup y Recuperación

### Tipos de Backup

#### 1. Backup Completo (Full Backup)
```bash
# Backup completo con tar
tar -czf backup_completo_$(date +%Y%m%d).tar.gz /home /etc /var/www

# Backup completo con rsync
rsync -avH --delete /home/ /backup/home/
```

#### 2. Backup Incremental
```bash
# Solo archivos modificados desde el último backup
find /home -newer /var/log/last_backup.timestamp -type f | \
    tar -czf incremental_$(date +%Y%m%d).tar.gz -T -

# Actualizar timestamp
touch /var/log/last_backup.timestamp
```

#### 3. Backup Diferencial
```bash
# Archivos modificados desde el último backup completo
find /home -newer /var/log/full_backup.timestamp -type f | \
    tar -czf diferencial_$(date +%Y%m%d).tar.gz -T -
```

### Principios Fundamentales

#### Regla 3-2-1
- **3** copias de los datos importantes
- **2** medios de almacenamiento diferentes
- **1** copia offsite (fuera del sitio)

#### RTO y RPO
```bash
# RTO (Recovery Time Objective): Tiempo máximo aceptable de downtime
# RPO (Recovery Point Objective): Pérdida máxima de datos aceptable

# Configurar backups según RPO
# RPO = 1 hora -> Backup cada hora
0 * * * * /script/backup_incremental.sh

# RPO = 1 día -> Backup diario
0 2 * * * /script/backup_completo.sh
```

## Herramientas de Backup Nativas

### rsync: Sincronización Eficiente

#### Configuración Básica
```bash
# Backup local
rsync -avH --progress /home/ /backup/home/

# Backup remoto
rsync -avH --progress -e ssh /home/ usuario@servidor:/backup/home/

# Backup con exclusiones
rsync -avH --progress \
    --exclude='*.tmp' \
    --exclude='*.log' \
    --exclude='.cache' \
    /home/ /backup/home/
```

#### Script de Backup con rsync
```bash
#!/bin/bash
# backup_rsync.sh

ORIGEN="/home"
DESTINO="/backup/home"
LOG="/var/log/backup.log"
DATE=$(date +"%Y-%m-%d %H:%M:%S")

echo "[$DATE] Iniciando backup..." >> $LOG

# Crear directorio de backup con fecha
BACKUP_DIR="$DESTINO/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Backup con hard links para ahorrar espacio
rsync -avH --link-dest="$DESTINO/latest" "$ORIGEN/" "$BACKUP_DIR/"

# Actualizar enlace a último backup
rm -f "$DESTINO/latest"
ln -s "$BACKUP_DIR" "$DESTINO/latest"

if [ $? -eq 0 ]; then
    echo "[$DATE] Backup completado exitosamente" >> $LOG
else
    echo "[$DATE] Error en backup" >> $LOG
    exit 1
fi
```

### tar: Archivos Comprimidos

#### Backup Completo con tar
```bash
# Backup con compresión gzip
tar -czf backup_$(date +%Y%m%d).tar.gz \
    --exclude='/proc' \
    --exclude='/sys' \
    --exclude='/dev' \
    --exclude='/tmp' \
    --exclude='/var/cache' \
    /

# Backup con compresión xz (mejor compresión)
tar -cJf backup_$(date +%Y%m%d).tar.xz /home

# Backup con verificación
tar -czf backup.tar.gz /home && tar -tzf backup.tar.gz > /dev/null
```

#### Restauración con tar
```bash
# Listar contenido del archivo
tar -tzf backup.tar.gz | head -20

# Extraer archivos específicos
tar -xzf backup.tar.gz home/usuario/documento.txt

# Restaurar todo
tar -xzf backup.tar.gz -C /restore/
```

### dd: Backup a Nivel de Bloque

#### Clonado de Discos
```bash
# Clonar disco completo
dd if=/dev/sda of=/dev/sdb bs=64K status=progress

# Crear imagen de disco
dd if=/dev/sda of=/backup/disk_image.img bs=64K status=progress

# Backup con compresión on-the-fly
dd if=/dev/sda bs=64K | gzip > /backup/disk_image.img.gz
```

#### Backup de Particiones
```bash
# Backup de partición boot
dd if=/dev/sda1 of=/backup/boot_partition.img bs=64K

# Backup de MBR
dd if=/dev/sda of=/backup/mbr.img bs=512 count=1
```

## Herramientas Profesionales

### Bacula: Enterprise Backup Solution

#### Instalación de Bacula
```bash
# Ubuntu/Debian
apt-get install bacula-server bacula-client

# Configurar base de datos
sudo -u postgres createuser bacula
sudo -u postgres createdb bacula
/usr/share/bacula/make_postgresql_tables
```

#### Configuración Director (bacula-dir.conf)
```conf
Director {
  Name = "servidor-dir"
  DIRport = 9101
  QueryFile = "/etc/bacula/scripts/query.sql"
  WorkingDirectory = "/var/lib/bacula"
  PidDirectory = "/var/run/bacula"
  Maximum Concurrent Jobs = 4
  Password = "password_director"
  Messages = Daemon
}

JobDefs {
  Name = "DefaultJob"
  Type = Backup
  Level = Incremental
  Client = servidor-fd
  FileSet = "Full Set"
  Schedule = "WeeklyCycle"
  Storage = File
  Messages = Standard
  Pool = File
  Priority = 10
  Write Bootstrap = "/var/lib/bacula/%c.bsr"
}

Job {
  Name = "BackupCliente1"
  JobDefs = "DefaultJob"
  Client = cliente1-fd
}

FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
      compression = GZIP
    }
    File = /home
    File = /etc
    File = /var/www
  }
  
  Exclude {
    File = /var/lib/bacula
    File = /tmp
    File = /proc
    File = /sys
  }
}
```

#### Configuración Storage Daemon
```conf
Storage {
  Name = servidor-sd
  SDPort = 9103
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/var/run/bacula"
  Maximum Concurrent Jobs = 20
}

Device {
  Name = FileStorage
  Media Type = File
  Archive Device = /backup/bacula
  LabelMedia = yes;
  Random Access = Yes;
  AutomaticMount = yes;
  RemovableMedia = no;
  AlwaysOpen = no;
}
```

### Amanda: Advanced Maryland Automatic Network Disk Archiver

#### Configuración Amanda
```bash
# Instalación
apt-get install amanda-server amanda-client

# Configuración básica
# /etc/amanda/DailySet1/amanda.conf
org "DailySet1"
mailto "admin@ejemplo.com"

dumpcycle 4 weeks
runspercycle 20
tapecycle 25 tapes

define dumptype global {
    comment "Global definitions"
    auth "bsdtcp"
}

define dumptype root-tar {
    global
    program "GNUTAR"
    comment "root partitions dumped with tar"
    compress none
    index yes
}
```

### Duplicity: Encrypted Backup

#### Backup Encriptado
```bash
# Backup encriptado a S3
export AWS_ACCESS_KEY_ID="tu_access_key"
export AWS_SECRET_ACCESS_KEY="tu_secret_key"
export PASSPHRASE="tu_passphrase"

duplicity /home s3://tu-bucket/backup/home

# Backup incremental
duplicity --full-if-older-than 1M /home s3://tu-bucket/backup/home

# Restaurar archivo específico
duplicity restore --file-to-restore home/usuario/archivo.txt \
    s3://tu-bucket/backup/home /tmp/restaurado/
```

## Strategies de Backup Automatizadas

### Scripts de Backup Inteligentes

#### Backup Rotativo con Retención
```bash
#!/bin/bash
# backup_inteligente.sh

BACKUP_DIR="/backup"
RETENTION_DAYS=30
SOURCE_DIRS="/home /etc /var/www"
EXCLUDE_FILE="/etc/backup/excludes.txt"

# Crear directorio con timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/$TIMESTAMP"
mkdir -p "$BACKUP_PATH"

# Función de logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a /var/log/backup.log
}

# Función de backup
perform_backup() {
    log "Iniciando backup en $BACKUP_PATH"
    
    tar -czf "$BACKUP_PATH/system_backup.tar.gz" \
        --exclude-from="$EXCLUDE_FILE" \
        $SOURCE_DIRS
    
    if [ $? -eq 0 ]; then
        log "Backup completado exitosamente"
        echo "$TIMESTAMP" > "$BACKUP_DIR/latest.txt"
    else
        log "ERROR: Backup falló"
        return 1
    fi
}

# Función de limpieza
cleanup_old_backups() {
    log "Iniciando limpieza de backups antiguos"
    
    find "$BACKUP_DIR" -type d -name "20*" -mtime +$RETENTION_DAYS -exec rm -rf {} \;
    
    log "Limpieza completada"
}

# Verificar espacio en disco
check_disk_space() {
    AVAILABLE=$(df "$BACKUP_DIR" | awk 'NR==2 {print $4}')
    REQUIRED=1000000  # 1GB en KB
    
    if [ "$AVAILABLE" -lt "$REQUIRED" ]; then
        log "ERROR: Espacio insuficiente en disco"
        exit 1
    fi
}

# Ejecutar backup
check_disk_space
perform_backup
cleanup_old_backups

# Enviar notificación
if [ $? -eq 0 ]; then
    echo "Backup completado en $(hostname)" | mail -s "Backup OK" admin@ejemplo.com
else
    echo "Backup falló en $(hostname)" | mail -s "Backup ERROR" admin@ejemplo.com
fi
```

### Backup con MySQL/PostgreSQL

#### Backup de Base de Datos MySQL
```bash
#!/bin/bash
# mysql_backup.sh

DB_USER="backup_user"
DB_PASS="backup_password"
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

# Crear directorio
mkdir -p "$BACKUP_DIR"

# Backup de todas las bases de datos
mysqldump --user="$DB_USER" --password="$DB_PASS" \
    --all-databases \
    --routines \
    --triggers \
    --single-transaction \
    --master-data=2 | gzip > "$BACKUP_DIR/all_databases_$DATE.sql.gz"

# Backup individual por base de datos
mysql --user="$DB_USER" --password="$DB_PASS" -e "SHOW DATABASES;" | \
grep -Ev '^(Database|information_schema|performance_schema|mysql|sys)$' | \
while read db; do
    mysqldump --user="$DB_USER" --password="$DB_PASS" \
        --routines \
        --triggers \
        --single-transaction \
        "$db" | gzip > "$BACKUP_DIR/${db}_$DATE.sql.gz"
done
```

#### Backup de PostgreSQL
```bash
#!/bin/bash
# postgresql_backup.sh

export PGPASSWORD="password"
BACKUP_DIR="/backup/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Backup global
pg_dumpall -h localhost -U postgres | gzip > "$BACKUP_DIR/global_$DATE.sql.gz"

# Backup por base de datos
psql -h localhost -U postgres -l -t | cut -d'|' -f1 | \
sed -e 's/ //g' -e '/^$/d' | \
grep -v template | \
while read db; do
    pg_dump -h localhost -U postgres -Fc "$db" > "$BACKUP_DIR/${db}_$DATE.dump"
done
```

## Backup Remoto y Cloud

### Backup a Servidores Remotos

#### rsync sobre SSH
```bash
#!/bin/bash
# backup_remoto.sh

REMOTE_USER="backup"
REMOTE_HOST="backup.ejemplo.com"
REMOTE_PATH="/backup/$(hostname)"
LOCAL_PATH="/home"

# Configurar SSH sin contraseña
# ssh-keygen -t rsa
# ssh-copy-id $REMOTE_USER@$REMOTE_HOST

# Backup con rsync
rsync -avz --delete \
    -e "ssh -o StrictHostKeyChecking=no" \
    "$LOCAL_PATH/" \
    "$REMOTE_USER@$REMOTE_HOST:$REMOTE_PATH/"
```

### Backup a Cloud Providers

#### AWS S3 con aws-cli
```bash
#!/bin/bash
# backup_s3.sh

AWS_PROFILE="backup"
S3_BUCKET="mi-bucket-backup"
LOCAL_DIR="/backup/daily"

# Configurar perfil AWS
# aws configure --profile backup

# Sincronizar con S3
aws s3 sync "$LOCAL_DIR" "s3://$S3_BUCKET/$(hostname)/" \
    --profile "$AWS_PROFILE" \
    --delete \
    --storage-class STANDARD_IA

# Backup con encriptación
aws s3 cp backup.tar.gz "s3://$S3_BUCKET/encrypted/" \
    --sse AES256 \
    --profile "$AWS_PROFILE"
```

#### Google Cloud Storage
```bash
#!/bin/bash
# backup_gcs.sh

GCS_BUCKET="gs://mi-bucket-backup"
LOCAL_DIR="/backup/daily"

# Instalar y configurar gcloud
# gcloud auth login
# gcloud config set project mi-proyecto

# Sincronizar con GCS
gsutil -m rsync -r -d "$LOCAL_DIR" "$GCS_BUCKET/$(hostname)/"

# Backup con compresión
tar -czf - /home | gsutil cp - "$GCS_BUCKET/home_$(date +%Y%m%d).tar.gz"
```

## Recuperación de Datos

### Estrategias de Restauración

#### Restauración Granular
```bash
# Restaurar archivo específico desde tar
tar -xzf backup.tar.gz home/usuario/archivo.txt --strip-components=2

# Restaurar desde rsync backup
rsync -avH /backup/home/latest/usuario/archivo.txt /home/usuario/

# Restaurar base de datos MySQL
gunzip < database_backup.sql.gz | mysql -u root -p database_name
```

#### Restauración Completa del Sistema
```bash
#!/bin/bash
# restauracion_completa.sh

BACKUP_FILE="/backup/system_backup.tar.gz"
RESTORE_ROOT="/mnt/restore"

# Montar sistema de archivos destino
mount /dev/sdb1 "$RESTORE_ROOT"

# Restaurar sistema base
cd "$RESTORE_ROOT"
tar -xzf "$BACKUP_FILE"

# Restaurar bootloader
mount --bind /dev "$RESTORE_ROOT/dev"
mount --bind /proc "$RESTORE_ROOT/proc"
mount --bind /sys "$RESTORE_ROOT/sys"

chroot "$RESTORE_ROOT" grub-install /dev/sdb
chroot "$RESTORE_ROOT" update-grub

# Ajustar fstab si es necesario
vim "$RESTORE_ROOT/etc/fstab"
```

### Disaster Recovery Planning

#### Documento de DR
```bash
# Crear inventario del sistema
#!/bin/bash
# inventario_sistema.sh

echo "=== INVENTARIO DEL SISTEMA $(date) ===" > inventario.txt
echo "" >> inventario.txt

echo "HARDWARE:" >> inventario.txt
lscpu | grep -E "Model name|CPU\(s\)|Architecture" >> inventario.txt
free -h >> inventario.txt
lsblk >> inventario.txt
echo "" >> inventario.txt

echo "SISTEMA OPERATIVO:" >> inventario.txt
cat /etc/os-release >> inventario.txt
uname -a >> inventario.txt
echo "" >> inventario.txt

echo "SERVICIOS ACTIVOS:" >> inventario.txt
systemctl list-units --type=service --state=active >> inventario.txt
echo "" >> inventario.txt

echo "PUERTOS ABIERTOS:" >> inventario.txt
netstat -tlnp >> inventario.txt
echo "" >> inventario.txt

echo "APLICACIONES INSTALADAS:" >> inventario.txt
dpkg -l >> inventario.txt
```

## Monitoreo y Alertas

### Verificación de Backups

#### Script de Verificación
```bash
#!/bin/bash
# verificar_backups.sh

BACKUP_DIR="/backup"
LOG_FILE="/var/log/backup_verification.log"
MAX_AGE_HOURS=25  # Backup debe ser menor a 25 horas

check_backup_age() {
    local backup_file="$1"
    local file_age_seconds=$(( $(date +%s) - $(stat -c %Y "$backup_file") ))
    local file_age_hours=$(( file_age_seconds / 3600 ))
    
    if [ $file_age_hours -gt $MAX_AGE_HOURS ]; then
        echo "ALERTA: Backup antiguo detectado: $backup_file ($file_age_hours horas)" | \
            tee -a "$LOG_FILE"
        return 1
    else
        echo "OK: Backup reciente: $backup_file ($file_age_hours horas)" | \
            tee -a "$LOG_FILE"
        return 0
    fi
}

verify_backup_integrity() {
    local backup_file="$1"
    
    case "$backup_file" in
        *.tar.gz)
            if tar -tzf "$backup_file" > /dev/null 2>&1; then
                echo "OK: Integridad verificada: $backup_file"
                return 0
            else
                echo "ERROR: Backup corrupto: $backup_file"
                return 1
            fi
            ;;
        *.sql.gz)
            if gunzip -t "$backup_file" 2>/dev/null; then
                echo "OK: Integridad verificada: $backup_file"
                return 0
            else
                echo "ERROR: Backup corrupto: $backup_file"
                return 1
            fi
            ;;
    esac
}

# Verificar todos los backups
find "$BACKUP_DIR" -name "*.tar.gz" -o -name "*.sql.gz" | \
while read backup_file; do
    check_backup_age "$backup_file"
    verify_backup_integrity "$backup_file"
done
```

### Alertas y Notificaciones

#### Sistema de Alertas con Nagios
```bash
# check_backup_age.sh para Nagios

#!/bin/bash
BACKUP_FILE="$1"
WARNING_HOURS="$2"
CRITICAL_HOURS="$3"

if [ ! -f "$BACKUP_FILE" ]; then
    echo "CRITICAL: Archivo de backup no encontrado: $BACKUP_FILE"
    exit 2
fi

file_age_seconds=$(( $(date +%s) - $(stat -c %Y "$BACKUP_FILE") ))
file_age_hours=$(( file_age_seconds / 3600 ))

if [ $file_age_hours -gt $CRITICAL_HOURS ]; then
    echo "CRITICAL: Backup demasiado antiguo: $file_age_hours horas"
    exit 2
elif [ $file_age_hours -gt $WARNING_HOURS ]; then
    echo "WARNING: Backup antiguo: $file_age_hours horas"
    exit 1
else
    echo "OK: Backup reciente: $file_age_hours horas"
    exit 0
fi
```

## Best Practices y Seguridad

### Seguridad en Backups

#### Encriptación de Backups
```bash
# Backup encriptado con GPG
tar -czf - /home | gpg --cipher-algo AES256 --compress-algo 1 \
    --symmetric --output backup_encrypted.tar.gz.gpg

# Restaurar backup encriptado
gpg --decrypt backup_encrypted.tar.gz.gpg | tar -xzf -

# Backup con password
tar -czf - /home | openssl enc -aes-256-cbc -salt > backup_encrypted.tar.gz
```

#### Control de Acceso
```bash
# Configurar permisos restrictivos
chmod 700 /backup
chown backup:backup /backup

# Usuario dedicado para backups
useradd -r -s /bin/bash -d /var/lib/backup backup
```

### Testing de Recuperación

#### Plan de Testing Regular
```bash
#!/bin/bash
# test_restauracion.sh

TEST_DIR="/tmp/test_restore_$(date +%s)"
BACKUP_FILE="/backup/latest.tar.gz"

mkdir -p "$TEST_DIR"

# Restaurar backup de prueba
tar -xzf "$BACKUP_FILE" -C "$TEST_DIR"

# Verificar archivos críticos
critical_files=(
    "etc/passwd"
    "etc/fstab"
    "etc/ssh/sshd_config"
    "home/usuario/.bashrc"
)

for file in "${critical_files[@]}"; do
    if [ -f "$TEST_DIR/$file" ]; then
        echo "OK: $file restaurado correctamente"
    else
        echo "ERROR: $file no encontrado en backup"
    fi
done

# Limpiar
rm -rf "$TEST_DIR"
```

## Conclusiones

Una estrategia de backup efectiva combina:

1. **Automatización**: Scripts programados y herramientas profesionales
2. **Redundancia**: Múltiples copias en diferentes ubicaciones
3. **Verificación**: Testing regular de integridad y recuperación
4. **Seguridad**: Encriptación y control de acceso
5. **Documentación**: Procedimientos claros de recuperación
6. **Monitoreo**: Alertas proactivas sobre fallos

Los backups no son un lujo, son una necesidad fundamental en cualquier infraestructura seria.

---
**Andrés Nuñez - t4ifi**
