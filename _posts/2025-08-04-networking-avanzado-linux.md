---
title: "Networking Avanzado en Linux: Configuración, Troubleshooting y Optimización"
date: 2025-08-31 08:00:00 +0100
categories: [Linux, Networking]
tags: [linux, networking, tcp, ip, routing, vlan, bonding, troubleshooting, performance]
---

# Networking Avanzado en Linux: Configuración, Troubleshooting y Optimización

El networking en Linux es uno de los aspectos más críticos para cualquier administrador de sistemas. En este post profundizaremos en técnicas avanzadas de configuración de red, troubleshooting y optimización de rendimiento.

## 📋 Tabla de Contenidos

1. [Fundamentos Avanzados de Networking](#fundamentos-avanzados)
2. [Configuración de Interfaces de Red](#configuracion-interfaces)
3. [Routing Avanzado y Tablas de Enrutamiento](#routing-avanzado)
4. [VLANs y Bridge](#vlans-bridge)
5. [Bonding y Agregación de Enlaces](#bonding-agregacion)
6. [Troubleshooting de Red](#troubleshooting)
7. [Monitoreo y Análisis de Tráfico](#monitoreo-trafico)
8. [Optimización de Rendimiento](#optimizacion-rendimiento)
9. [Casos Prácticos](#casos-practicos)

## 1. Fundamentos Avanzados de Networking {#fundamentos-avanzados}

### Arquitectura del Stack de Red en Linux

```bash
# Ver todas las interfaces de red
ip link show

# Información detallada de interfaces
ip addr show

# Estadísticas de red
cat /proc/net/dev

# Ver conexiones activas
ss -tuln
```

### Namespaces de Red

```bash
# Crear un namespace de red
sudo ip netns add test-ns

# Listar namespaces
ip netns list

# Ejecutar comando en namespace
sudo ip netns exec test-ns ip link show

# Configurar interface en namespace
sudo ip netns exec test-ns ip link set lo up
sudo ip netns exec test-ns ip addr add 127.0.0.1/8 dev lo
```

## 2. Configuración de Interfaces de Red {#configuracion-interfaces}

### Configuración Estática con NetworkManager

```bash
# Configuración con nmcli
sudo nmcli connection add type ethernet con-name eth0-static \
    ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1

# Configurar DNS
sudo nmcli connection modify eth0-static ipv4.dns "8.8.8.8,8.8.4.4"

# Activar conexión
sudo nmcli connection up eth0-static
```

### Configuración con Netplan (Ubuntu)

```yaml
# /etc/netplan/01-network-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    eth1:
      dhcp4: true
```

```bash
# Aplicar configuración
sudo netplan apply

# Verificar configuración
sudo netplan --debug apply
```

### Configuración Manual con iproute2

```bash
# Configurar IP estática
sudo ip addr add 192.168.1.100/24 dev eth0

# Configurar gateway
sudo ip route add default via 192.168.1.1

# Configurar rutas específicas
sudo ip route add 10.0.0.0/8 via 192.168.1.254

# Hacer persistente
echo "192.168.1.100/24" | sudo tee /etc/systemd/network/eth0.network
```

## 3. Routing Avanzado y Tablas de Enrutamiento {#routing-avanzado}

### Múltiples Tablas de Routing

```bash
# Crear tabla de routing personalizada
echo "200 custom" | sudo tee -a /etc/iproute2/rt_tables

# Agregar rutas a tabla personalizada
sudo ip route add 10.0.0.0/8 via 192.168.1.254 table custom

# Crear regla de routing
sudo ip rule add from 192.168.1.0/24 table custom

# Ver todas las tablas
ip route show table all
```

### Policy-Based Routing

```bash
# Routing basado en origen
sudo ip rule add from 192.168.1.0/24 table 100

# Routing basado en destino
sudo ip rule add to 10.0.0.0/8 table 100

# Routing basado en puerto
sudo ip rule add sport 80 table 100

# Ver reglas
ip rule show
```

### Load Balancing con ECMP

```bash
# Configurar múltiples gateways con balanceo
sudo ip route add default \
    nexthop via 192.168.1.1 weight 1 \
    nexthop via 192.168.1.2 weight 1

# Verificar balanceo
ip route show
```

## 4. VLANs y Bridge {#vlans-bridge}

### Configuración de VLANs

```bash
# Crear VLAN interface
sudo ip link add link eth0 name eth0.100 type vlan id 100

# Configurar IP en VLAN
sudo ip addr add 192.168.100.1/24 dev eth0.100

# Activar interface VLAN
sudo ip link set eth0.100 up

# Configuración persistente con systemd
cat << EOF | sudo tee /etc/systemd/network/eth0.100.netdev
[NetDev]
Name=eth0.100
Kind=vlan

[VLAN]
Id=100
EOF
```

### Configuración de Bridge

```bash
# Crear bridge
sudo ip link add br0 type bridge

# Agregar interfaces al bridge
sudo ip link set eth0 master br0
sudo ip link set eth1 master br0

# Configurar bridge
sudo ip addr add 192.168.1.1/24 dev br0
sudo ip link set br0 up

# Configuración avanzada de bridge
sudo echo 1 > /sys/class/net/br0/bridge/stp_state
sudo echo 10 > /sys/class/net/br0/bridge/forward_delay
```

## 5. Bonding y Agregación de Enlaces {#bonding-agregacion}

### Configuración de Bonding

```bash
# Cargar módulo bonding
sudo modprobe bonding

# Crear interface bond
sudo ip link add bond0 type bond mode 802.3ad

# Agregar interfaces slave
sudo ip link set eth0 down
sudo ip link set eth1 down
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0

# Configurar bond
sudo ip addr add 192.168.1.100/24 dev bond0
sudo ip link set bond0 up
```

### Configuración Persistente de Bonding

```bash
# /etc/systemd/network/bond0.netdev
cat << EOF | sudo tee /etc/systemd/network/bond0.netdev
[NetDev]
Name=bond0
Kind=bond

[Bond]
Mode=802.3ad
MIIMonitorSec=100
EOF

# /etc/systemd/network/bond0.network
cat << EOF | sudo tee /etc/systemd/network/bond0.network
[Match]
Name=bond0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
EOF
```

### Monitoreo de Bonding

```bash
# Ver estado del bond
cat /proc/net/bonding/bond0

# Estadísticas detalladas
sudo ethtool bond0

# Cambiar modo de bonding dinámicamente
echo "balance-rr" | sudo tee /sys/class/net/bond0/bonding/mode
```

## 6. Troubleshooting de Red {#troubleshooting}

### Herramientas de Diagnóstico

```bash
# Diagnóstico completo de conectividad
#!/bin/bash
echo "=== NETWORK TROUBLESHOOTING SCRIPT ==="

# Interfaces
echo "--- Interfaces ---"
ip link show

# Direcciones IP
echo "--- IP Addresses ---"
ip addr show

# Rutas
echo "--- Routing Table ---"
ip route show

# DNS
echo "--- DNS Resolution ---"
nslookup google.com

# Conectividad
echo "--- Connectivity Tests ---"
ping -c 3 8.8.8.8
ping -c 3 google.com

# Puertos
echo "--- Listening Ports ---"
ss -tuln
```

### Análisis de Paquetes

```bash
# Captura con tcpdump
sudo tcpdump -i eth0 -w capture.pcap

# Filtros específicos
sudo tcpdump -i eth0 host 192.168.1.100
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 'tcp and port 22'

# Análisis en tiempo real
sudo tcpdump -i eth0 -n -A
```

### Troubleshooting de Latencia

```bash
# Medir latencia con ping
ping -c 100 8.8.8.8 | tail -1

# Traceroute avanzado
mtr --report --report-cycles 10 google.com

# Análisis de jitter
#!/bin/bash
for i in {1..20}; do
    ping -c 1 8.8.8.8 | grep time= | cut -d= -f4
done | awk '{sum+=$1; sumsq+=$1*$1} END {print "Avg:",sum/NR,"ms Jitter:",sqrt(sumsq/NR-(sum/NR)^2),"ms"}'
```

## 7. Monitoreo y Análisis de Tráfico {#monitoreo-trafico}

### Monitoreo con iftop

```bash
# Instalar iftop
sudo apt install iftop

# Monitoreo básico
sudo iftop

# Monitoreo por puerto
sudo iftop -P

# Monitoreo específico
sudo iftop -i eth0 -f "host 192.168.1.100"
```

### Análisis con netstat y ss

```bash
# Conexiones por estado
ss -s

# Top conexiones por IP
netstat -an | grep :80 | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr

# Monitoreo de ancho de banda por proceso
sudo nethogs eth0
```

### Script de Monitoreo de Red

```bash
#!/bin/bash
# network_monitor.sh

INTERFACE="eth0"
INTERVAL=5

echo "Monitoring network on $INTERFACE every $INTERVAL seconds"
echo "Press Ctrl+C to stop"

while true; do
    RX_BYTES=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
    TX_BYTES=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
    RX_PACKETS=$(cat /sys/class/net/$INTERFACE/statistics/rx_packets)
    TX_PACKETS=$(cat /sys/class/net/$INTERFACE/statistics/tx_packets)
    
    sleep $INTERVAL
    
    RX_BYTES_NEW=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
    TX_BYTES_NEW=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
    RX_PACKETS_NEW=$(cat /sys/class/net/$INTERFACE/statistics/rx_packets)
    TX_PACKETS_NEW=$(cat /sys/class/net/$INTERFACE/statistics/tx_packets)
    
    RX_RATE=$(((RX_BYTES_NEW - RX_BYTES) / INTERVAL))
    TX_RATE=$(((TX_BYTES_NEW - TX_BYTES) / INTERVAL))
    RX_PPS=$(((RX_PACKETS_NEW - RX_PACKETS) / INTERVAL))
    TX_PPS=$(((TX_PACKETS_NEW - TX_PACKETS) / INTERVAL))
    
    echo "$(date): RX: ${RX_RATE} B/s ($RX_PPS pps) TX: ${TX_RATE} B/s ($TX_PPS pps)"
done
```

## 8. Optimización de Rendimiento {#optimizacion-rendimiento}

### Tuning del Kernel

```bash
# Optimizaciones de red en sysctl
cat << EOF | sudo tee /etc/sysctl.d/99-network-tuning.conf
# TCP Buffer Sizes
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_default = 262144
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# TCP Congestion Control
net.ipv4.tcp_congestion_control = bbr

# Network Device Settings
net.core.netdev_max_backlog = 5000
net.core.netdev_budget = 600

# TCP Optimizations
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_sack = 1
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_moderate_rcvbuf = 1

# Security
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
EOF

# Aplicar configuración
sudo sysctl -p /etc/sysctl.d/99-network-tuning.conf
```

### Optimización de Interfaces

```bash
# Configurar ring buffers
sudo ethtool -G eth0 rx 4096 tx 4096

# Configurar coalescing
sudo ethtool -C eth0 adaptive-rx on adaptive-tx on rx-usecs 50

# Habilitar offloading
sudo ethtool -K eth0 gso on tso on gro on lro on

# CPU affinity para interrupciones
echo 2 | sudo tee /proc/irq/24/smp_affinity
```

### Benchmark de Red

```bash
# iperf3 servidor
iperf3 -s

# iperf3 cliente
iperf3 -c 192.168.1.100 -t 60 -P 4

# Benchmark UDP
iperf3 -c 192.168.1.100 -u -b 1G

# Script de benchmark completo
#!/bin/bash
echo "=== NETWORK BENCHMARK ==="
TARGET_IP="192.168.1.100"

echo "--- TCP Bandwidth ---"
iperf3 -c $TARGET_IP -t 30

echo "--- TCP Multiple Streams ---"
iperf3 -c $TARGET_IP -t 30 -P 4

echo "--- UDP Bandwidth ---"
iperf3 -c $TARGET_IP -u -b 1G -t 30

echo "--- Latency Test ---"
ping -c 100 $TARGET_IP | tail -1
```

## 9. Casos Prácticos {#casos-practicos}

### Caso 1: Red Corporativa con Múltiples VLANs

```bash
#!/bin/bash
# setup_corporate_network.sh

# Crear VLANs
sudo ip link add link eth0 name eth0.10 type vlan id 10  # Administración
sudo ip link add link eth0 name eth0.20 type vlan id 20  # Usuarios
sudo ip link add link eth0 name eth0.30 type vlan id 30  # Servidores

# Configurar IPs
sudo ip addr add 192.168.10.1/24 dev eth0.10
sudo ip addr add 192.168.20.1/24 dev eth0.20
sudo ip addr add 192.168.30.1/24 dev eth0.30

# Activar interfaces
sudo ip link set eth0.10 up
sudo ip link set eth0.20 up
sudo ip link set eth0.30 up

# Configurar forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Reglas de firewall entre VLANs
sudo iptables -A FORWARD -i eth0.10 -o eth0.20 -j ACCEPT
sudo iptables -A FORWARD -i eth0.20 -o eth0.30 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

### Caso 2: Alta Disponibilidad con Bonding

```bash
#!/bin/bash
# setup_ha_network.sh

# Configurar bonding activo-pasivo
sudo modprobe bonding mode=active-backup miimon=100

# Crear bond
sudo ip link add bond0 type bond

# Configurar modo
echo "active-backup" | sudo tee /sys/class/net/bond0/bonding/mode
echo "100" | sudo tee /sys/class/net/bond0/bonding/miimon

# Agregar interfaces
sudo ip link set eth0 down
sudo ip link set eth1 down
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0

# Configurar IP
sudo ip addr add 192.168.1.100/24 dev bond0
sudo ip link set bond0 up

# Configurar gateway
sudo ip route add default via 192.168.1.1
```

### Caso 3: Troubleshooting de Conectividad

```bash
#!/bin/bash
# network_troubleshoot.sh

TARGET_HOST="$1"
if [ -z "$TARGET_HOST" ]; then
    echo "Uso: $0 <host>"
    exit 1
fi

echo "=== TROUBLESHOOTING CONNECTIVITY TO $TARGET_HOST ==="

# Test básico de conectividad
echo "--- Ping Test ---"
if ping -c 3 "$TARGET_HOST" > /dev/null 2>&1; then
    echo "✓ Host is reachable"
    ping -c 3 "$TARGET_HOST"
else
    echo "✗ Host is NOT reachable"
fi

# Test de resolución DNS
echo "--- DNS Resolution ---"
if nslookup "$TARGET_HOST" > /dev/null 2>&1; then
    echo "✓ DNS resolution works"
    nslookup "$TARGET_HOST"
else
    echo "✗ DNS resolution failed"
fi

# Traceroute
echo "--- Route Analysis ---"
traceroute "$TARGET_HOST"

# Test de puertos comunes
echo "--- Port Tests ---"
for port in 22 80 443; do
    if timeout 3 bash -c "</dev/tcp/$TARGET_HOST/$port" 2>/dev/null; then
        echo "✓ Port $port is open"
    else
        echo "✗ Port $port is closed or filtered"
    fi
done
```

## 🔧 Scripts de Automatización

### Script de Configuración Completa

```bash
#!/bin/bash
# complete_network_setup.sh

set -e

CONFIG_FILE="/etc/network/advanced.conf"

# Función de logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a /var/log/network-setup.log
}

# Validar privilegios
if [ "$EUID" -ne 0 ]; then
    echo "Este script debe ejecutarse como root"
    exit 1
fi

log "Iniciando configuración avanzada de red"

# Backup de configuración actual
log "Creando backup de configuración actual"
mkdir -p /backup/network/$(date +%Y%m%d)
cp -r /etc/systemd/network/ /backup/network/$(date +%Y%m%d)/ 2>/dev/null || true

# Aplicar optimizaciones del kernel
log "Aplicando optimizaciones del kernel"
sysctl -p /etc/sysctl.d/99-network-tuning.conf

# Configurar interfaces según archivo de configuración
if [ -f "$CONFIG_FILE" ]; then
    log "Aplicando configuración desde $CONFIG_FILE"
    source "$CONFIG_FILE"
else
    log "Archivo de configuración no encontrado, usando valores por defecto"
fi

log "Configuración de red completada exitosamente"

# Verificar conectividad
log "Verificando conectividad"
if ping -c 3 8.8.8.8 > /dev/null 2>&1; then
    log "✓ Conectividad verificada"
else
    log "✗ Problema de conectividad detectado"
fi
```

## 📚 Recursos y Referencias

### Comandos de Diagnóstico Útiles

```bash
# Resumen de comandos esenciales
alias net-info='ip addr show && echo "--- Routes ---" && ip route show'
alias net-ports='ss -tuln'
alias net-connections='ss -tuln | grep LISTEN'
alias net-stats='cat /proc/net/dev'
```

### Archivos de Configuración Importantes

- `/etc/systemd/network/` - Configuración NetworkD
- `/etc/netplan/` - Configuración Netplan (Ubuntu)
- `/etc/sysctl.conf` - Parámetros del kernel
- `/proc/net/` - Estadísticas de red del kernel
- `/sys/class/net/` - Información de interfaces

## 🎯 Conclusiones

El networking avanzado en Linux requiere dominar múltiples herramientas y conceptos. Los puntos clave incluyen:

1. **Comprensión profunda** del stack TCP/IP
2. **Herramientas modernas** como iproute2 y systemd-networkd
3. **Monitoreo continuo** y troubleshooting proactivo
4. **Optimización basada** en métricas reales
5. **Automatización** de tareas repetitivas

La práctica constante con estos conceptos y herramientas es fundamental para convertirse en un experto en networking en Linux.

---

*Este post forma parte de mi serie sobre administración avanzada de sistemas Linux. Si tienes preguntas o sugerencias, no dudes en contactarme.*

---
**Andrés Nuñez - t4ifi**
