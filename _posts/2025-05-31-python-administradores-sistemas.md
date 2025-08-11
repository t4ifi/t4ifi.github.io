---
title: "Python para Administradores de Sistemas: Automatización y Scripting Avanzado"
date: 2025-05-31 20:15:00 +0000
categories: [Python, Administración de Sistemas]
tags: [python, automation, scripting, sysadmin, devops, monitoring, administration]
---

Python se ha convertido en el lenguaje de facto para administradores de sistemas modernos. Su sintaxis clara, extensas librerías y capacidades de automatización lo hacen perfecto para gestionar infraestructura, monitorear sistemas y automatizar tareas repetitivas.

> 🐍 **Objetivo:** Dominar Python aplicado a administración de sistemas, desde scripts básicos hasta soluciones complejas de automatización, monitoreo y gestión de infraestructura.

## 🚀 Por qué Python para SysAdmins

### Ventajas sobre bash scripting

**Bash:**
```bash
# Código difícil de mantener y escalar
for server in $(cat servers.txt); do
  ssh $server "df -h | grep -E '^/dev' | awk '{print $5}' | sed 's/%//'" | while read usage; do
    if [ $usage -gt 80 ]; then
      echo "ALERT: $server disk usage: $usage%"
    fi
  done
done
```

**Python equivalente:**
```python
import subprocess
import paramiko
from concurrent.futures import ThreadPoolExecutor

def check_disk_usage(server):
    """Verifica uso de disco en servidor remoto"""
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(server)
        
        stdin, stdout, stderr = ssh.exec_command("df -h / | tail -1 | awk '{print $5}' | sed 's/%//'")
        usage = int(stdout.read().decode().strip())
        
        if usage > 80:
            return f"ALERT: {server} disk usage: {usage}%"
        return f"OK: {server} disk usage: {usage}%"
    
    except Exception as e:
        return f"ERROR: {server} - {str(e)}"
    finally:
        ssh.close()

# Ejecución paralela
with open('servers.txt') as f:
    servers = [line.strip() for line in f]

with ThreadPoolExecutor(max_workers=10) as executor:
    results = list(executor.map(check_disk_usage, servers))

for result in results:
    print(result)
```

### Librerías esenciales para SysAdmins

```python
# Gestión del sistema operativo
import os, sys, subprocess, shutil, glob
from pathlib import Path

# Trabajo con archivos y text
import json, yaml, csv, configparser
import re  # expresiones regulares

# Red y protocolos
import socket, requests, urllib
import paramiko  # SSH
import smtplib  # Email

# Concurrencia y paralelismo
import threading, multiprocessing
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Fechas y tiempo
import datetime, time
from dateutil import parser

# Monitoreo y métricas
import psutil  # información del sistema
import logging  # logging estructurado

# Bases de datos
import sqlite3, pymongo
from sqlalchemy import create_engine
```

## 📁 Gestión de Archivos y Directorios

### Operaciones básicas con pathlib

```python
from pathlib import Path
import shutil
import stat

class FileManager:
    """Gestión avanzada de archivos del sistema"""
    
    def __init__(self, base_path):
        self.base_path = Path(base_path)
    
    def find_files(self, pattern, recursive=True):
        """Buscar archivos con patrón específico"""
        if recursive:
            return list(self.base_path.rglob(pattern))
        else:
            return list(self.base_path.glob(pattern))
    
    def clean_old_files(self, days_old=7, pattern="*", dry_run=True):
        """Eliminar archivos antiguos"""
        cutoff_date = datetime.now() - timedelta(days=days_old)
        removed_files = []
        
        for file_path in self.find_files(pattern):
            if file_path.is_file():
                file_time = datetime.fromtimestamp(file_path.stat().st_mtime)
                if file_time < cutoff_date:
                    removed_files.append(str(file_path))
                    if not dry_run:
                        file_path.unlink()
                        print(f"Eliminado: {file_path}")
        
        return removed_files
    
    def backup_directory(self, dest_path, exclude_patterns=None):
        """Crear backup de directorio con exclusiones"""
        exclude_patterns = exclude_patterns or []
        dest_path = Path(dest_path)
        
        def should_exclude(path):
            for pattern in exclude_patterns:
                if path.match(pattern):
                    return True
            return False
        
        for item in self.base_path.rglob('*'):
            if should_exclude(item):
                continue
                
            relative_path = item.relative_to(self.base_path)
            dest_item = dest_path / relative_path
            
            if item.is_dir():
                dest_item.mkdir(parents=True, exist_ok=True)
            else:
                dest_item.parent.mkdir(parents=True, exist_ok=True)
                shutil.copy2(item, dest_item)
    
    def set_permissions_recursive(self, file_perms=0o644, dir_perms=0o755):
        """Establecer permisos recursivamente"""
        for item in self.base_path.rglob('*'):
            if item.is_file():
                item.chmod(file_perms)
            elif item.is_dir():
                item.chmod(dir_perms)

# Uso práctico
fm = FileManager('/var/log')
old_logs = fm.clean_old_files(days_old=30, pattern="*.log", dry_run=False)
print(f"Eliminados {len(old_logs)} archivos de log antiguos")
```

### Monitoreo de cambios en archivos

```python
import time
import hashlib
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class ConfigFileMonitor(FileSystemEventHandler):
    """Monitor para cambios en archivos de configuración"""
    
    def __init__(self, callback=None):
        self.callback = callback
        self.file_hashes = {}
    
    def on_modified(self, event):
        if event.is_directory:
            return
            
        file_path = event.src_path
        current_hash = self.get_file_hash(file_path)
        
        if file_path in self.file_hashes:
            if self.file_hashes[file_path] != current_hash:
                print(f"Archivo modificado: {file_path}")
                if self.callback:
                    self.callback(file_path)
        
        self.file_hashes[file_path] = current_hash
    
    def get_file_hash(self, file_path):
        """Calcular hash MD5 de archivo"""
        try:
            with open(file_path, 'rb') as f:
                return hashlib.md5(f.read()).hexdigest()
        except IOError:
            return None

def reload_nginx_config(file_path):
    """Callback para recargar configuración de nginx"""
    if 'nginx' in file_path:
        subprocess.run(['nginx', '-t'])  # Test config
        subprocess.run(['systemctl', 'reload', 'nginx'])
        print("Nginx recargado exitosamente")

# Monitoreo en tiempo real
monitor = ConfigFileMonitor(callback=reload_nginx_config)
observer = Observer()
observer.schedule(monitor, '/etc/nginx/', recursive=True)
observer.start()

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    observer.stop()
observer.join()
```

## 🌐 Automatización de Red y SSH

### Cliente SSH robusto

```python
import paramiko
import socket
from contextlib import contextmanager

class SSHManager:
    """Gestión robusta de conexiones SSH"""
    
    def __init__(self, hostname, username, password=None, key_filename=None, port=22):
        self.hostname = hostname
        self.username = username
        self.password = password
        self.key_filename = key_filename
        self.port = port
        self.client = None
    
    @contextmanager
    def connect(self):
        """Context manager para conexiones SSH"""
        try:
            self.client = paramiko.SSHClient()
            self.client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            
            # Intentar conexión con clave primero, luego contraseña
            try:
                self.client.connect(
                    hostname=self.hostname,
                    username=self.username,
                    key_filename=self.key_filename,
                    port=self.port,
                    timeout=10
                )
            except paramiko.AuthenticationException:
                if self.password:
                    self.client.connect(
                        hostname=self.hostname,
                        username=self.username,
                        password=self.password,
                        port=self.port,
                        timeout=10
                    )
                else:
                    raise
            
            yield self
            
        except Exception as e:
            print(f"Error conectando a {self.hostname}: {e}")
            raise
        finally:
            if self.client:
                self.client.close()
    
    def execute_command(self, command, timeout=30):
        """Ejecutar comando remoto"""
        stdin, stdout, stderr = self.client.exec_command(command, timeout=timeout)
        
        exit_code = stdout.channel.recv_exit_status()
        output = stdout.read().decode('utf-8')
        error = stderr.read().decode('utf-8')
        
        return {
            'exit_code': exit_code,
            'stdout': output,
            'stderr': error
        }
    
    def upload_file(self, local_path, remote_path):
        """Subir archivo vía SFTP"""
        sftp = self.client.open_sftp()
        try:
            sftp.put(local_path, remote_path)
            return True
        except Exception as e:
            print(f"Error subiendo archivo: {e}")
            return False
        finally:
            sftp.close()
    
    def download_file(self, remote_path, local_path):
        """Descargar archivo vía SFTP"""
        sftp = self.client.open_sftp()
        try:
            sftp.get(remote_path, local_path)
            return True
        except Exception as e:
            print(f"Error descargando archivo: {e}")
            return False
        finally:
            sftp.close()

# Gestión de múltiples servidores
class ServerFleet:
    """Gestión de flota de servidores"""
    
    def __init__(self, config_file):
        self.servers = self.load_config(config_file)
    
    def load_config(self, config_file):
        """Cargar configuración de servidores"""
        with open(config_file) as f:
            return yaml.safe_load(f)
    
    def execute_on_all(self, command, parallel=True):
        """Ejecutar comando en todos los servidores"""
        if parallel:
            return self._execute_parallel(command)
        else:
            return self._execute_sequential(command)
    
    def _execute_parallel(self, command):
        """Ejecución paralela con ThreadPoolExecutor"""
        def execute_on_server(server_config):
            ssh = SSHManager(**server_config)
            with ssh.connect():
                result = ssh.execute_command(command)
                return {
                    'server': server_config['hostname'],
                    'result': result
                }
        
        with ThreadPoolExecutor(max_workers=10) as executor:
            futures = [executor.submit(execute_on_server, server) 
                      for server in self.servers]
            return [future.result() for future in futures]
    
    def _execute_sequential(self, command):
        """Ejecución secuencial"""
        results = []
        for server_config in self.servers:
            ssh = SSHManager(**server_config)
            with ssh.connect():
                result = ssh.execute_command(command)
                results.append({
                    'server': server_config['hostname'],
                    'result': result
                })
        return results

# Ejemplo de uso
# servers.yaml
"""
- hostname: web1.example.com
  username: admin
  key_filename: /home/user/.ssh/id_rsa
- hostname: web2.example.com
  username: admin
  key_filename: /home/user/.ssh/id_rsa
- hostname: db1.example.com
  username: admin
  password: secret_password
"""

fleet = ServerFleet('servers.yaml')
results = fleet.execute_on_all('uptime')

for result in results:
    print(f"{result['server']}: {result['result']['stdout'].strip()}")
```

### Monitoreo de red y puertos

```python
import socket
import requests
from concurrent.futures import ThreadPoolExecutor
import time

class NetworkMonitor:
    """Monitor de servicios de red"""
    
    def __init__(self):
        self.results = []
    
    def check_port(self, host, port, timeout=5):
        """Verificar si un puerto está abierto"""
        try:
            with socket.create_connection((host, port), timeout=timeout):
                return True
        except (socket.timeout, socket.error):
            return False
    
    def check_http_service(self, url, expected_status=200, timeout=10):
        """Verificar servicio HTTP/HTTPS"""
        try:
            response = requests.get(url, timeout=timeout)
            return {
                'url': url,
                'status_code': response.status_code,
                'response_time': response.elapsed.total_seconds(),
                'success': response.status_code == expected_status
            }
        except requests.RequestException as e:
            return {
                'url': url,
                'error': str(e),
                'success': False
            }
    
    def ping_host(self, host):
        """Ping a host usando subprocess"""
        try:
            result = subprocess.run(
                ['ping', '-c', '1', '-W', '2', host],
                capture_output=True,
                text=True
            )
            return result.returncode == 0
        except Exception:
            return False
    
    def comprehensive_check(self, target):
        """Verificación completa de un objetivo"""
        results = {
            'target': target,
            'timestamp': datetime.now().isoformat(),
            'checks': {}
        }
        
        # Parse target (puede ser URL o host:port)
        if target.startswith('http'):
            results['checks']['http'] = self.check_http_service(target)
            host = target.split('//')[1].split('/')[0].split(':')[0]
        else:
            host = target.split(':')[0]
            port = int(target.split(':')[1]) if ':' in target else 80
            results['checks']['port'] = self.check_port(host, port)
        
        results['checks']['ping'] = self.ping_host(host)
        
        return results
    
    def monitor_services(self, targets, interval=60):
        """Monitoreo continuo de servicios"""
        while True:
            print(f"\n--- Verificación de servicios: {datetime.now()} ---")
            
            with ThreadPoolExecutor(max_workers=len(targets)) as executor:
                futures = {executor.submit(self.comprehensive_check, target): target 
                          for target in targets}
                
                for future in futures:
                    try:
                        result = future.result()
                        self.print_status(result)
                        self.results.append(result)
                    except Exception as e:
                        print(f"Error verificando {futures[future]}: {e}")
            
            time.sleep(interval)
    
    def print_status(self, result):
        """Imprimir estado de verificación"""
        target = result['target']
        checks = result['checks']
        
        status_symbols = {True: '✅', False: '❌'}
        
        print(f"{target}:")
        for check_type, check_result in checks.items():
            if isinstance(check_result, bool):
                symbol = status_symbols[check_result]
                print(f"  {check_type}: {symbol}")
            elif isinstance(check_result, dict):
                success = check_result.get('success', False)
                symbol = status_symbols[success]
                print(f"  {check_type}: {symbol}")
                if 'response_time' in check_result:
                    print(f"    Response time: {check_result['response_time']:.2f}s")

# Uso del monitor
targets = [
    'https://google.com',
    'https://github.com',
    'web1.example.com:80',
    'db1.example.com:3306'
]

monitor = NetworkMonitor()
# monitor.monitor_services(targets, interval=30)  # Descomenta para monitoreo continuo
```

## 📊 Monitoreo del Sistema con psutil

### Monitor completo del sistema

```python
import psutil
import json
import time
from datetime import datetime
import smtplib
from email.mime.text import MIMEText

class SystemMonitor:
    """Monitor completo del sistema"""
    
    def __init__(self, config=None):
        self.config = config or self.default_config()
        self.alerts_sent = set()
    
    def default_config(self):
        return {
            'cpu_threshold': 80,
            'memory_threshold': 85,
            'disk_threshold': 90,
            'smtp_server': 'localhost',
            'smtp_port': 587,
            'alert_email': 'admin@example.com',
            'check_interval': 60
        }
    
    def get_cpu_info(self):
        """Información detallada de CPU"""
        return {
            'usage_percent': psutil.cpu_percent(interval=1),
            'usage_per_cpu': psutil.cpu_percent(interval=1, percpu=True),
            'load_average': psutil.getloadavg(),
            'cpu_count': psutil.cpu_count(),
            'cpu_freq': psutil.cpu_freq()._asdict() if psutil.cpu_freq() else None
        }
    
    def get_memory_info(self):
        """Información de memoria"""
        virtual = psutil.virtual_memory()
        swap = psutil.swap_memory()
        
        return {
            'virtual': {
                'total': virtual.total,
                'available': virtual.available,
                'used': virtual.used,
                'percent': virtual.percent
            },
            'swap': {
                'total': swap.total,
                'used': swap.used,
                'percent': swap.percent
            }
        }
    
    def get_disk_info(self):
        """Información de discos"""
        disk_info = []
        
        for partition in psutil.disk_partitions():
            try:
                usage = psutil.disk_usage(partition.mountpoint)
                disk_info.append({
                    'device': partition.device,
                    'mountpoint': partition.mountpoint,
                    'fstype': partition.fstype,
                    'total': usage.total,
                    'used': usage.used,
                    'free': usage.free,
                    'percent': (usage.used / usage.total) * 100
                })
            except PermissionError:
                continue
        
        return disk_info
    
    def get_network_info(self):
        """Información de red"""
        net_io = psutil.net_io_counters()
        connections = len(psutil.net_connections())
        
        return {
            'io_counters': {
                'bytes_sent': net_io.bytes_sent,
                'bytes_recv': net_io.bytes_recv,
                'packets_sent': net_io.packets_sent,
                'packets_recv': net_io.packets_recv
            },
            'connections_count': connections
        }
    
    def get_process_info(self, limit=10):
        """Top procesos por CPU y memoria"""
        processes = []
        
        for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 'memory_percent', 'username']):
            try:
                processes.append(proc.info)
            except (psutil.NoSuchProcess, psutil.AccessDenied):
                pass
        
        # Top por CPU
        top_cpu = sorted(processes, key=lambda x: x['cpu_percent'] or 0, reverse=True)[:limit]
        
        # Top por memoria
        top_memory = sorted(processes, key=lambda x: x['memory_percent'] or 0, reverse=True)[:limit]
        
        return {
            'top_cpu': top_cpu,
            'top_memory': top_memory,
            'total_processes': len(processes)
        }
    
    def collect_metrics(self):
        """Recopilar todas las métricas"""
        return {
            'timestamp': datetime.now().isoformat(),
            'cpu': self.get_cpu_info(),
            'memory': self.get_memory_info(),
            'disk': self.get_disk_info(),
            'network': self.get_network_info(),
            'processes': self.get_process_info()
        }
    
    def check_alerts(self, metrics):
        """Verificar condiciones de alerta"""
        alerts = []
        
        # CPU alert
        if metrics['cpu']['usage_percent'] > self.config['cpu_threshold']:
            alerts.append(f"HIGH CPU: {metrics['cpu']['usage_percent']:.1f}%")
        
        # Memory alert
        if metrics['memory']['virtual']['percent'] > self.config['memory_threshold']:
            alerts.append(f"HIGH MEMORY: {metrics['memory']['virtual']['percent']:.1f}%")
        
        # Disk alerts
        for disk in metrics['disk']:
            if disk['percent'] > self.config['disk_threshold']:
                alerts.append(f"HIGH DISK: {disk['mountpoint']} {disk['percent']:.1f}%")
        
        return alerts
    
    def send_alert(self, alerts):
        """Enviar alerta por email"""
        if not alerts:
            return
        
        alert_key = "|".join(sorted(alerts))
        if alert_key in self.alerts_sent:
            return  # No enviar alertas duplicadas
        
        subject = f"System Alert - {socket.gethostname()}"
        body = f"Se detectaron las siguientes alertas:\n\n" + "\n".join(alerts)
        
        try:
            msg = MIMEText(body)
            msg['Subject'] = subject
            msg['From'] = 'system@example.com'
            msg['To'] = self.config['alert_email']
            
            with smtplib.SMTP(self.config['smtp_server'], self.config['smtp_port']) as server:
                server.send_message(msg)
            
            self.alerts_sent.add(alert_key)
            print(f"Alerta enviada: {subject}")
            
        except Exception as e:
            print(f"Error enviando alerta: {e}")
    
    def save_metrics(self, metrics, filename=None):
        """Guardar métricas en archivo"""
        if filename is None:
            filename = f"/var/log/system_metrics_{datetime.now().strftime('%Y%m%d')}.json"
        
        with open(filename, 'a') as f:
            f.write(json.dumps(metrics) + '\n')
    
    def run_monitoring(self):
        """Ejecutar monitoreo continuo"""
        print("Iniciando monitoreo del sistema...")
        
        while True:
            try:
                metrics = self.collect_metrics()
                alerts = self.check_alerts(metrics)
                
                # Mostrar estado actual
                print(f"\n[{metrics['timestamp']}] Sistema:")
                print(f"  CPU: {metrics['cpu']['usage_percent']:.1f}%")
                print(f"  Memoria: {metrics['memory']['virtual']['percent']:.1f}%")
                print(f"  Procesos: {metrics['processes']['total_processes']}")
                
                if alerts:
                    print(f"  🚨 ALERTAS: {', '.join(alerts)}")
                    self.send_alert(alerts)
                
                # Guardar métricas
                self.save_metrics(metrics)
                
                time.sleep(self.config['check_interval'])
                
            except KeyboardInterrupt:
                print("\nMonitoreo detenido por usuario")
                break
            except Exception as e:
                print(f"Error en monitoreo: {e}")
                time.sleep(self.config['check_interval'])

# Uso del monitor
if __name__ == "__main__":
    monitor = SystemMonitor({
        'cpu_threshold': 75,
        'memory_threshold': 80,
        'disk_threshold': 85,
        'check_interval': 30
    })
    
    # monitor.run_monitoring()  # Descomenta para ejecutar
```

## 🗄️ Gestión de Bases de Datos

### Cliente universal de base de datos

```python
import sqlite3
import psycopg2
from sqlalchemy import create_engine
import pymongo
from contextlib import contextmanager

class DatabaseManager:
    """Gestor universal de bases de datos"""
    
    def __init__(self, db_type, **kwargs):
        self.db_type = db_type.lower()
        self.config = kwargs
        self.connection = None
    
    @contextmanager
    def connect(self):
        """Context manager para conexiones"""
        try:
            if self.db_type == 'sqlite':
                self.connection = sqlite3.connect(self.config['database'])
                self.connection.row_factory = sqlite3.Row
            
            elif self.db_type == 'postgresql':
                self.connection = psycopg2.connect(**self.config)
            
            elif self.db_type == 'mysql':
                import mysql.connector
                self.connection = mysql.connector.connect(**self.config)
            
            elif self.db_type == 'mongodb':
                client = pymongo.MongoClient(**self.config)
                self.connection = client[self.config.get('database', 'admin')]
            
            yield self
            
        except Exception as e:
            print(f"Error conectando a base de datos: {e}")
            raise
        finally:
            if self.connection and self.db_type != 'mongodb':
                self.connection.close()
    
    def execute_query(self, query, params=None):
        """Ejecutar query SQL"""
        if self.db_type == 'mongodb':
            raise ValueError("Use métodos específicos de MongoDB")
        
        cursor = self.connection.cursor()
        try:
            if params:
                cursor.execute(query, params)
            else:
                cursor.execute(query)
            
            if query.strip().upper().startswith('SELECT'):
                return cursor.fetchall()
            else:
                self.connection.commit()
                return cursor.rowcount
                
        except Exception as e:
            self.connection.rollback()
            raise e
        finally:
            cursor.close()
    
    def backup_database(self, output_file):
        """Crear backup de base de datos"""
        if self.db_type == 'postgresql':
            cmd = [
                'pg_dump',
                '-h', self.config.get('host', 'localhost'),
                '-U', self.config['user'],
                '-d', self.config['database'],
                '-f', output_file
            ]
            subprocess.run(cmd, env={'PGPASSWORD': self.config['password']})
        
        elif self.db_type == 'mysql':
            cmd = [
                'mysqldump',
                '-h', self.config.get('host', 'localhost'),
                '-u', self.config['user'],
                f'-p{self.config["password"]}',
                self.config['database']
            ]
            with open(output_file, 'w') as f:
                subprocess.run(cmd, stdout=f)
        
        elif self.db_type == 'sqlite':
            shutil.copy2(self.config['database'], output_file)
        
        elif self.db_type == 'mongodb':
            cmd = [
                'mongodump',
                '--host', f"{self.config.get('host', 'localhost')}:{self.config.get('port', 27017)}",
                '--db', self.config['database'],
                '--out', output_file
            ]
            subprocess.run(cmd)

# Ejemplo de monitoreo de bases de datos
class DatabaseMonitor:
    """Monitor de salud de bases de datos"""
    
    def __init__(self):
        self.databases = {}
    
    def add_database(self, name, db_manager):
        """Añadir base de datos al monitoreo"""
        self.databases[name] = db_manager
    
    def check_connection(self, name):
        """Verificar conectividad"""
        try:
            with self.databases[name].connect():
                return True
        except Exception as e:
            print(f"Error conectando a {name}: {e}")
            return False
    
    def get_database_size(self, name):
        """Obtener tamaño de base de datos"""
        db_manager = self.databases[name]
        
        with db_manager.connect():
            if db_manager.db_type == 'postgresql':
                query = """
                SELECT pg_size_pretty(pg_database_size(current_database())) as size
                """
                result = db_manager.execute_query(query)
                return result[0][0]
            
            elif db_manager.db_type == 'mysql':
                query = """
                SELECT ROUND(SUM(data_length + index_length) / 1024 / 1024, 1) AS 'DB Size in MB'
                FROM information_schema.tables
                WHERE table_schema = DATABASE()
                """
                result = db_manager.execute_query(query)
                return f"{result[0][0]} MB"
            
            elif db_manager.db_type == 'sqlite':
                file_size = os.path.getsize(db_manager.config['database'])
                return f"{file_size / 1024 / 1024:.1f} MB"
    
    def monitor_all(self):
        """Monitorear todas las bases de datos"""
        print("Estado de bases de datos:")
        for name, db_manager in self.databases.items():
            connected = self.check_connection(name)
            status = "✅ Online" if connected else "❌ Offline"
            
            if connected:
                try:
                    size = self.get_database_size(name)
                    print(f"  {name}: {status} - Size: {size}")
                except Exception as e:
                    print(f"  {name}: {status} - Error getting size: {e}")
            else:
                print(f"  {name}: {status}")

# Configuración de ejemplo
pg_config = {
    'host': 'localhost',
    'database': 'myapp',
    'user': 'admin',
    'password': 'password'
}

mysql_config = {
    'host': 'localhost',
    'database': 'myapp',
    'user': 'admin',
    'password': 'password'
}

# Uso del monitor
monitor = DatabaseMonitor()
monitor.add_database('postgres', DatabaseManager('postgresql', **pg_config))
monitor.add_database('mysql', DatabaseManager('mysql', **mysql_config))
monitor.monitor_all()
```

## 📈 Logging y Métricas

### Sistema de logging estructurado

```python
import logging
import json
import sys
from datetime import datetime
from logging.handlers import RotatingFileHandler, SysLogHandler

class StructuredLogger:
    """Logger estructurado para aplicaciones de sistema"""
    
    def __init__(self, name, level=logging.INFO):
        self.logger = logging.getLogger(name)
        self.logger.setLevel(level)
        
        # Evitar handlers duplicados
        if not self.logger.handlers:
            self.setup_handlers()
    
    def setup_handlers(self):
        """Configurar handlers de logging"""
        # Console handler
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(logging.INFO)
        console_formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        console_handler.setFormatter(console_formatter)
        
        # File handler con rotación
        file_handler = RotatingFileHandler(
            '/var/log/sysadmin.log',
            maxBytes=10*1024*1024,  # 10MB
            backupCount=5
        )
        file_handler.setLevel(logging.DEBUG)
        file_formatter = logging.Formatter(
            '%(asctime)s - %(name)s - %(levelname)s - %(funcName)s:%(lineno)d - %(message)s'
        )
        file_handler.setFormatter(file_formatter)
        
        # Syslog handler
        try:
            syslog_handler = SysLogHandler(address='/dev/log')
            syslog_handler.setLevel(logging.WARNING)
            syslog_formatter = logging.Formatter(
                '%(name)s: %(levelname)s %(message)s'
            )
            syslog_handler.setFormatter(syslog_formatter)
            self.logger.addHandler(syslog_handler)
        except Exception:
            pass  # Syslog no disponible
        
        self.logger.addHandler(console_handler)
        self.logger.addHandler(file_handler)
    
    def log_structured(self, level, message, **kwargs):
        """Log con datos estructurados"""
        log_data = {
            'timestamp': datetime.now().isoformat(),
            'message': message,
            'hostname': socket.gethostname(),
            **kwargs
        }
        
        # Log como JSON para facilitar parsing
        json_message = json.dumps(log_data)
        self.logger.log(level, json_message)
    
    def info(self, message, **kwargs):
        self.log_structured(logging.INFO, message, **kwargs)
    
    def warning(self, message, **kwargs):
        self.log_structured(logging.WARNING, message, **kwargs)
    
    def error(self, message, **kwargs):
        self.log_structured(logging.ERROR, message, **kwargs)
    
    def critical(self, message, **kwargs):
        self.log_structured(logging.CRITICAL, message, **kwargs)

# Decorador para logging automático
def log_execution(logger):
    """Decorador para loggear ejecución de funciones"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            start_time = time.time()
            logger.info(f"Iniciando ejecución de {func.__name__}", 
                       function=func.__name__, args=str(args)[:100])
            
            try:
                result = func(*args, **kwargs)
                execution_time = time.time() - start_time
                logger.info(f"Completado {func.__name__}", 
                           function=func.__name__, 
                           execution_time=execution_time,
                           success=True)
                return result
            
            except Exception as e:
                execution_time = time.time() - start_time
                logger.error(f"Error en {func.__name__}: {str(e)}", 
                            function=func.__name__, 
                            execution_time=execution_time,
                            error=str(e),
                            success=False)
                raise
        
        return wrapper
    return decorator

# Uso del sistema de logging
logger = StructuredLogger('sysadmin_scripts')

@log_execution(logger)
def deploy_application(app_name, version):
    """Ejemplo de función con logging automático"""
    logger.info("Iniciando deployment", app=app_name, version=version)
    
    # Simular deployment
    time.sleep(2)
    
    if app_name == "fail_app":
        raise Exception("Deployment failed")
    
    logger.info("Deployment completado exitosamente", 
               app=app_name, version=version)
    return True

# Ejemplo de uso
try:
    deploy_application("web_app", "1.2.3")
    deploy_application("fail_app", "1.0.0")
except Exception:
    pass
```

---

## 🎓 Conclusión

Python ofrece herramientas poderosas para automatizar prácticamente cualquier tarea de administración de sistemas:

### Puntos clave cubiertos:

1. **Gestión de archivos** con pathlib y monitoreo de cambios
2. **Automatización SSH** para gestión de flotas de servidores
3. **Monitoreo del sistema** con psutil y alertas automatizadas
4. **Gestión de bases de datos** multiplataforma
5. **Logging estructurado** para debugging y auditoría
6. **Monitoreo de red** y servicios

## 🧠 Teoría Avanzada: Fundamentos de la Automatización

### Paradigmas de Programación en Administración de Sistemas

**Programación Imperativa vs Declarativa:**

**Imperativa (Cómo hacer):**
```python
# Bash tradicional - imperativo
servers = ['web1', 'web2', 'web3']
for server in servers:
    ssh.connect(server)
    ssh.execute('systemctl restart nginx')
    ssh.execute('systemctl enable nginx')
    ssh.disconnect()
```

**Declarativa (Qué queremos):**
```python
# Enfoque declarativo con estado deseado
desired_state = {
    'service': 'nginx',
    'state': 'running',
    'enabled': True
}

# El framework determina los pasos necesarios
for server in servers:
    ensure_service_state(server, desired_state)
```

**Ventajas del enfoque declarativo:**
- **Idempotencia**: Ejecutar múltiples veces produce el mismo resultado
- **Predictibilidad**: El estado final es conocido y verificable
- **Reversibilidad**: Fácil rollback a estados anteriores

### Teoría de Concurrencia y Paralelismo

**Diferencias fundamentales:**

**Concurrencia (Concurrency):**
- **Definición**: Múltiples tareas **parecen** ejecutarse simultáneamente
- **Implementación**: Threading, asyncio
- **Uso**: I/O bound operations (SSH, HTTP requests, file operations)

**Paralelismo (Parallelism):**
- **Definición**: Múltiples tareas **realmente** se ejecutan simultáneamente
- **Implementación**: Multiprocessing
- **Uso**: CPU bound operations (cálculos intensivos, compresión)

**Global Interpreter Lock (GIL) en Python:**
```python
import threading
import multiprocessing
import time

# Threading - NO paralelismo real para CPU-bound
def cpu_bound_task():
    return sum(i*i for i in range(1000000))

# Multiprocessing - paralelismo real
def parallel_processing():
    with multiprocessing.Pool() as pool:
        results = pool.map(cpu_bound_task, range(4))
    return results

# AsyncIO - concurrencia para I/O-bound
import asyncio
async def io_bound_task():
    await asyncio.sleep(1)  # Simula operación I/O
    return "completed"
```

### Patrones de Diseño en Automatización

**1. Factory Pattern - Creación de objetos:**
```python
class ConnectionFactory:
    @staticmethod
    def create_connection(conn_type, **kwargs):
        if conn_type == 'ssh':
            return SSHConnection(**kwargs)
        elif conn_type == 'database':
            return DatabaseConnection(**kwargs)
        elif conn_type == 'http':
            return HTTPConnection(**kwargs)
        else:
            raise ValueError(f"Unknown connection type: {conn_type}")
```

**2. Observer Pattern - Monitoreo de eventos:**
```python
class SystemEventObserver:
    def __init__(self):
        self.observers = []
    
    def attach(self, observer):
        self.observers.append(observer)
    
    def notify(self, event):
        for observer in self.observers:
            observer.update(event)

class AlertManager:
    def update(self, event):
        if event.severity == 'critical':
            self.send_alert(event)
```

**3. Strategy Pattern - Múltiples algoritmos:**
```python
class BackupStrategy:
    def backup(self, data): pass

class IncrementalBackup(BackupStrategy):
    def backup(self, data):
        # Lógica de backup incremental
        pass

class FullBackup(BackupStrategy):
    def backup(self, data):
        # Lógica de backup completo
        pass
```

### Teoría de Sistemas Distribuidos

**Propiedades fundamentales (CAP Theorem):**
- **Consistency**: Todos los nodos ven los mismos datos simultáneamente
- **Availability**: El sistema sigue operativo ante fallos
- **Partition Tolerance**: El sistema continúa funcionando ante fallos de red

**Implicaciones para SysAdmins:**
```python
# Eventual Consistency en configuración distribuida
class DistributedConfig:
    def __init__(self):
        self.nodes = []
        self.consistency_level = 'eventual'
    
    def update_config(self, key, value):
        # Envía actualización a mayoría de nodos
        success_count = 0
        for node in self.nodes:
            if node.update(key, value):
                success_count += 1
        
        # Quorum-based consistency
        return success_count > len(self.nodes) // 2
```

### Error Handling y Resilience Patterns

**Circuit Breaker Pattern:**
```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = 1    # Normal operation
    OPEN = 2      # Failing fast
    HALF_OPEN = 3 # Testing recovery

class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
    
    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.timeout:
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise e
    
    def on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

**Retry Pattern con Exponential Backoff:**
```python
import time
import random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1, max_delay=60):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries:
                        raise e
                    
                    # Exponential backoff with jitter
                    delay = min(base_delay * (2 ** attempt), max_delay)
                    jitter = random.uniform(0, delay * 0.1)
                    time.sleep(delay + jitter)
                    
                    print(f"Retry {attempt + 1}/{max_retries} after {delay:.2f}s")
            
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3, base_delay=2)
def unreliable_api_call():
    # Función que puede fallar
    pass
```

### Teoría de Observabilidad

**Los Tres Pilares de la Observabilidad:**

**1. Logging (Qué pasó):**
```python
# Structured logging para máquinas
logger.info("User login", 
           user_id=12345, 
           ip_address="192.168.1.100",
           session_id="abc123",
           timestamp=datetime.now().isoformat())
```

**2. Metrics (Cuánto pasó):**
```python
# Time series data
metrics = {
    'cpu_usage': 75.5,
    'memory_usage': 82.1,
    'disk_io_read': 1024000,
    'network_bytes_in': 5500000
}
```

**3. Tracing (Por dónde pasó):**
```python
# Distributed tracing
@trace_span("database_query")
def get_user(user_id):
    with trace_span("validate_input"):
        validate_user_id(user_id)
    
    with trace_span("database_lookup"):
        return database.query(f"SELECT * FROM users WHERE id = {user_id}")
```

### Testing en Automatización

**Pirámide de Testing:**
```
    E2E Tests (Pocos, lentos, costosos)
         ↗ ↖
Integration Tests (Algunos, medianos)
         ↗ ↖
Unit Tests (Muchos, rápidos, baratos)
```

**Implementación práctica:**
```python
import unittest
from unittest.mock import patch, MagicMock

class TestSystemMonitor(unittest.TestCase):
    
    def setUp(self):
        self.monitor = SystemMonitor()
    
    @patch('psutil.cpu_percent')
    def test_cpu_alert_threshold(self, mock_cpu):
        # Unit test - componente aislado
        mock_cpu.return_value = 85
        metrics = self.monitor.collect_metrics()
        alerts = self.monitor.check_alerts(metrics)
        
        self.assertIn('HIGH CPU', alerts[0])
    
    @patch('subprocess.run')
    def test_ssh_connection_integration(self, mock_subprocess):
        # Integration test - múltiples componentes
        mock_subprocess.return_value.returncode = 0
        mock_subprocess.return_value.stdout = b"uptime output"
        
        ssh = SSHManager('test-server', 'test-user')
        with ssh.connect():
            result = ssh.execute_command('uptime')
        
        self.assertEqual(result['exit_code'], 0)

# Property-based testing para casos edge
from hypothesis import given, strategies as st

@given(st.integers(min_value=0, max_value=100))
def test_cpu_percentage_always_valid(cpu_percent):
    monitor = SystemMonitor()
    # Test que cualquier porcentaje válido no cause errores
    assert 0 <= cpu_percent <= 100
```

### Infrastructure as Code - Principios Teóricos

**Immutable Infrastructure:**
```python
# En lugar de modificar servidores existentes
def patch_server(server):
    server.update_package('nginx')
    server.restart_service('nginx')

# Crear nueva instancia con configuración deseada
def deploy_new_version(config):
    new_server = create_server(config)
    test_server(new_server)
    swap_traffic(old_server, new_server)
    terminate_server(old_server)
```

**GitOps Workflow:**
```
Git Repository (Source of Truth)
    ↓
CI/CD Pipeline
    ↓
Automated Deployment
    ↓
Production Environment
    ↓
Monitoring & Feedback
    ↓
Git Repository (Updates)
```

### Mejores prácticas:

- Usar **context managers** para recursos (archivos, conexiones)
- Implementar **logging estructurado** desde el inicio
- **Manejo de errores** robusto con try/except
- **Paralelización** para operaciones en múltiples servidores
- **Configuración externa** (YAML, JSON) para flexibilidad
- **Testing automático** de scripts críticos
- **Versionado** de scripts y configuraciones
- **Documentación** inline y external

### Recursos recomendados:

- **Fabric**: Deployment y ejecución remota simplificada
- **Ansible**: Automatización de infraestructura
- **Docker SDK**: Gestión de containers desde Python
- **Kubernetes Python Client**: Orquestación de containers
- **Prometheus Python Client**: Métricas y monitoreo
- **pytest**: Framework de testing avanzado
- **Black**: Formateo automático de código
- **mypy**: Type checking para Python

---

## 📝 Reflexiones Finales

Python ha revolucionado la administración de sistemas moderna porque combina:

1. **Simplicidad sintáctica** que permite enfocarse en la lógica del problema
2. **Ecosistema rico** de librerías especializadas
3. **Capacidades de integración** con prácticamente cualquier sistema
4. **Escalabilidad** desde scripts simples hasta sistemas distribuidos complejos

Como administradores de sistemas, Python nos permite **elevar nuestro trabajo** desde tareas manuales repetitivas hacia **ingeniería de sistemas** sofisticada. La clave está en entender no solo *cómo* usar las herramientas, sino *por qué* funcionan de cierta manera y *cuándo* aplicar cada patrón.

El futuro de la administración de sistemas está en la **automatización inteligente**, donde los sistemas se auto-gestionan, auto-reparan y auto-optimizan. Python es nuestra herramienta principal para construir ese futuro.

¿Tienes una tarea específica de administración que quieres automatizar? ¡Comparte tu caso y te ayudo a implementarlo!

**Escrito por Andrés Núñez**  