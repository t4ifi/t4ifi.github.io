---
title: "Redes y Troubleshooting: Guía Completa para Diagnóstico y Resolución de Problemas de Red"
date: 2025-06-17 12:00:00 +0200
categories: [Redes, Troubleshooting]
tags: [redes, troubleshooting, diagnóstico, tcp-ip, wireshark, netstat, tcpdump]
pin: false
---

# Redes y Troubleshooting: Guía Completa para Diagnóstico y Resolución de Problemas de Red

El troubleshooting de redes es una habilidad fundamental para cualquier administrador de sistemas. Los problemas de red pueden manifestarse de múltiples formas y requieren un enfoque sistemático para su diagnóstico y resolución. Esta guía completa te proporcionará las herramientas, técnicas y metodologías necesarias para convertirte en un experto en resolución de problemas de red.

## Fundamentos de Redes TCP/IP

### El Modelo OSI y TCP/IP

Antes de diagnosticar problemas, es crucial entender cómo funcionan las redes. El modelo TCP/IP organiza las comunicaciones de red en capas:

```
┌─────────────────┬─────────────────┐
│   Aplicación    │    HTTP/HTTPS   │
├─────────────────┼─────────────────┤
│   Transporte    │    TCP/UDP      │
├─────────────────┼─────────────────┤
│   Red           │    IP/ICMP      │
├─────────────────┼─────────────────┤
│   Enlace        │  Ethernet/WiFi  │
└─────────────────┴─────────────────┘
```

#### Capa de Aplicación (HTTP/HTTPS, DNS, SMTP)
- **Problemas comunes**: Timeouts de aplicación, errores 404/500, problemas de autenticación
- **Herramientas**: curl, wget, telnet, navegadores

#### Capa de Transporte (TCP/UDP)
- **Problemas comunes**: Puertos cerrados, conexiones rechazadas, pérdida de paquetes
- **Herramientas**: netstat, ss, nmap, tcpdump

#### Capa de Red (IP)
- **Problemas comunes**: Routing incorrecto, IP duplicadas, problemas de subred
- **Herramientas**: ping, traceroute, ip route, arp

#### Capa de Enlace (Ethernet/WiFi)
- **Problemas comunes**: Cables defectuosos, problemas de switch, interferencias WiFi
- **Herramientas**: ethtool, iwconfig, dmesg

## Metodología de Troubleshooting

### Enfoque Sistemático: Bottom-Up vs Top-Down

#### Método Bottom-Up (De abajo hacia arriba)
Comenzar desde la capa física y subir:

```bash
#!/bin/bash

# troubleshoot_bottom_up.sh - Diagnóstico bottom-up
# Autor: Andrés Núñez

set -euo pipefail

# Colores para output
RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
    local level=$1
    shift
    local message="$*"
    
    case $level in
        "ERROR")   echo -e "${RED}[ERROR]${NC} $message" ;;
        "WARN")    echo -e "${YELLOW}[WARN]${NC} $message" ;;
        "INFO")    echo -e "${BLUE}[INFO]${NC} $message" ;;
        "SUCCESS") echo -e "${GREEN}[SUCCESS]${NC} $message" ;;
    esac
}

# Capa 1: Física
check_physical_layer() {
    log "INFO" "=== VERIFICANDO CAPA FÍSICA ==="
    
    # Verificar interfaces de red
    log "INFO" "Interfaces de red disponibles:"
    ip link show | grep -E "^[0-9]+:" | while read line; do
        interface=$(echo "$line" | awk '{print $2}' | sed 's/://')
        state=$(echo "$line" | grep -o "state [A-Z]*" | awk '{print $2}')
        
        if [ "$state" = "UP" ]; then
            log "SUCCESS" "  $interface está UP"
            
            # Verificar estadísticas de la interfaz
            if command -v ethtool > /dev/null; then
                speed=$(ethtool "$interface" 2>/dev/null | grep "Speed:" | awk '{print $2}' || echo "Unknown")
                duplex=$(ethtool "$interface" 2>/dev/null | grep "Duplex:" | awk '{print $2}' || echo "Unknown")
                log "INFO" "    Speed: $speed, Duplex: $duplex"
            fi
        else
            log "ERROR" "  $interface está $state"
        fi
    done
    
    # Verificar errores en interfaces
    log "INFO" "Verificando errores en interfaces:"
    cat /proc/net/dev | grep -v "lo:" | grep ":" | while read line; do
        interface=$(echo "$line" | awk '{print $1}' | sed 's/://')
        rx_errors=$(echo "$line" | awk '{print $4}')
        tx_errors=$(echo "$line" | awk '{print $12}')
        
        if [ "$rx_errors" -gt 0 ] || [ "$tx_errors" -gt 0 ]; then
            log "WARN" "  $interface - RX errors: $rx_errors, TX errors: $tx_errors"
        fi
    done
}

# Capa 2: Enlace de datos
check_data_link_layer() {
    log "INFO" "=== VERIFICANDO CAPA DE ENLACE ==="
    
    # Verificar tabla ARP
    log "INFO" "Tabla ARP:"
    arp -a | head -10
    
    # Verificar conectividad en la misma subred
    local gateway=$(ip route | grep default | awk '{print $3}' | head -1)
    if [ -n "$gateway" ]; then
        log "INFO" "Verificando conectividad con gateway: $gateway"
        if ping -c 3 -W 2 "$gateway" > /dev/null 2>&1; then
            log "SUCCESS" "Gateway $gateway es alcanzable"
        else
            log "ERROR" "Gateway $gateway NO es alcanzable"
        fi
    fi
}

# Capa 3: Red
check_network_layer() {
    log "INFO" "=== VERIFICANDO CAPA DE RED ==="
    
    # Verificar configuración IP
    log "INFO" "Configuración IP:"
    ip addr show | grep -E "(inet |inet6)" | while read line; do
        log "INFO" "  $line"
    done
    
    # Verificar tabla de routing
    log "INFO" "Tabla de routing:"
    ip route show
    
    # Verificar DNS
    log "INFO" "Configuración DNS:"
    if [ -f /etc/resolv.conf ]; then
        grep nameserver /etc/resolv.conf | while read line; do
            dns_server=$(echo "$line" | awk '{print $2}')
            log "INFO" "  DNS: $dns_server"
            
            if ping -c 2 -W 2 "$dns_server" > /dev/null 2>&1; then
                log "SUCCESS" "    DNS $dns_server es alcanzable"
            else
                log "ERROR" "    DNS $dns_server NO es alcanzable"
            fi
        done
    fi
    
    # Verificar conectividad externa
    log "INFO" "Verificando conectividad externa:"
    if ping -c 3 -W 3 8.8.8.8 > /dev/null 2>&1; then
        log "SUCCESS" "Conectividad a 8.8.8.8 OK"
    else
        log "ERROR" "Sin conectividad a 8.8.8.8"
    fi
    
    if nslookup google.com > /dev/null 2>&1; then
        log "SUCCESS" "Resolución DNS OK"
    else
        log "ERROR" "Problemas con resolución DNS"
    fi
}

# Capa 4: Transporte
check_transport_layer() {
    log "INFO" "=== VERIFICANDO CAPA DE TRANSPORTE ==="
    
    # Verificar puertos en escucha
    log "INFO" "Puertos en escucha:"
    ss -tuln | head -20
    
    # Verificar conexiones activas
    log "INFO" "Conexiones activas (top 10):"
    ss -tun | grep ESTAB | head -10
    
    # Verificar estadísticas TCP
    log "INFO" "Estadísticas TCP:"
    ss -s
}

# Ejecutar verificaciones
main() {
    log "INFO" "Iniciando diagnóstico de red bottom-up..."
    echo
    
    check_physical_layer
    echo
    check_data_link_layer
    echo
    check_network_layer
    echo
    check_transport_layer
    
    echo
    log "INFO" "Diagnóstico completado"
}

main "$@"
```

#### Método Top-Down (De arriba hacia abajo)
Comenzar desde la aplicación y bajar:

```bash
#!/bin/bash

# troubleshoot_top_down.sh - Diagnóstico top-down
# Autor: Andrés Núñez

test_application_connectivity() {
    local target_host=$1
    local target_port=$2
    local service_name=$3
    
    log "INFO" "=== VERIFICANDO APLICACIÓN: $service_name ==="
    
    # Test HTTP/HTTPS
    if [ "$target_port" = "80" ] || [ "$target_port" = "443" ]; then
        local protocol="http"
        [ "$target_port" = "443" ] && protocol="https"
        
        log "INFO" "Probando $protocol://$target_host:$target_port"
        
        response=$(curl -s -o /dev/null -w "%{http_code},%{time_total},%{time_connect}" \
                       --connect-timeout 10 --max-time 30 \
                       "$protocol://$target_host:$target_port" 2>/dev/null || echo "000,999,999")
        
        IFS=',' read -r http_code total_time connect_time <<< "$response"
        
        if [ "$http_code" = "200" ]; then
            log "SUCCESS" "HTTP respuesta OK (${total_time}s)"
        else
            log "ERROR" "HTTP falló - Código: $http_code"
            return 1
        fi
    else
        # Test genérico TCP
        log "INFO" "Probando conexión TCP a $target_host:$target_port"
        
        if timeout 10 bash -c "</dev/tcp/$target_host/$target_port" 2>/dev/null; then
            log "SUCCESS" "Puerto $target_port accesible"
        else
            log "ERROR" "Puerto $target_port NO accesible"
            return 1
        fi
    fi
    
    return 0
}

test_dns_resolution() {
    local hostname=$1
    
    log "INFO" "=== VERIFICANDO RESOLUCIÓN DNS para $hostname ==="
    
    # Test con nslookup
    if command -v nslookup > /dev/null; then
        local ip=$(nslookup "$hostname" 2>/dev/null | grep "Address:" | grep -v "#53" | awk '{print $2}' | head -1)
        if [ -n "$ip" ]; then
            log "SUCCESS" "DNS resuelve $hostname -> $ip"
        else
            log "ERROR" "DNS no puede resolver $hostname"
            return 1
        fi
    fi
    
    # Test con dig
    if command -v dig > /dev/null; then
        local query_time=$(dig "$hostname" | grep "Query time:" | awk '{print $4}')
        if [ -n "$query_time" ]; then
            log "INFO" "Tiempo de consulta DNS: ${query_time}ms"
        fi
    fi
    
    return 0
}

test_network_connectivity() {
    local target_host=$1
    
    log "INFO" "=== VERIFICANDO CONECTIVIDAD DE RED a $target_host ==="
    
    # Ping test
    if ping -c 4 -W 3 "$target_host" > /dev/null 2>&1; then
        local avg_time=$(ping -c 4 "$target_host" 2>/dev/null | tail -1 | awk -F'/' '{print $5}')
        log "SUCCESS" "Ping OK - Tiempo promedio: ${avg_time}ms"
    else
        log "ERROR" "Ping falló a $target_host"
        return 1
    fi
    
    # Traceroute para ver la ruta
    log "INFO" "Trazando ruta a $target_host:"
    if command -v traceroute > /dev/null; then
        traceroute -n -m 10 "$target_host" 2>/dev/null | head -10 | while read line; do
            log "INFO" "  $line"
        done
    fi
    
    return 0
}

# Función principal para test top-down
test_connectivity() {
    local target=$1
    local port=${2:-80}
    local service=${3:-"Web"}
    
    # Extraer hostname de URL si es necesario
    local hostname
    if [[ "$target" =~ ^https?:// ]]; then
        hostname=$(echo "$target" | sed -E 's|^https?://([^/]+).*|\1|')
    else
        hostname="$target"
    fi
    
    log "INFO" "Iniciando diagnóstico top-down para $hostname:$port"
    echo
    
    # Paso 1: Test de aplicación
    if ! test_application_connectivity "$hostname" "$port" "$service"; then
        echo
        # Paso 2: Test DNS
        if ! test_dns_resolution "$hostname"; then
            log "ERROR" "Problema en resolución DNS"
            return 1
        fi
        
        echo
        # Paso 3: Test de conectividad
        if ! test_network_connectivity "$hostname"; then
            log "ERROR" "Problema de conectividad de red"
            return 1
        fi
        
        log "ERROR" "Red OK pero aplicación no responde"
        return 1
    fi
    
    log "SUCCESS" "Todos los tests pasaron"
    return 0
}

# Ejemplo de uso
if [ $# -gt 0 ]; then
    test_connectivity "$@"
else
    echo "Uso: $0 <hostname/url> [puerto] [nombre_servicio]"
    echo "Ejemplos:"
    echo "  $0 google.com 80 Google"
    echo "  $0 https://www.github.com 443 GitHub"
    echo "  $0 192.168.1.1 22 SSH"
fi
```

## Herramientas Esenciales de Diagnóstico

### 1. Análisis de Tráfico con tcpdump y Wireshark

```bash
#!/bin/bash

# network_capture.sh - Script para captura y análisis de tráfico
# Autor: Andrés Núñez

set -euo pipefail

# Configuración
CAPTURE_DIR="/tmp/network_captures"
MAX_CAPTURE_SIZE="100M"
CAPTURE_DURATION=60

# Crear directorio para capturas
mkdir -p "$CAPTURE_DIR"

# Función para captura básica
basic_capture() {
    local interface=$1
    local filter=${2:-""}
    local output_file="$CAPTURE_DIR/capture_$(date +%Y%m%d_%H%M%S).pcap"
    
    log "INFO" "Iniciando captura en $interface por ${CAPTURE_DURATION}s"
    log "INFO" "Filtro: ${filter:-'ninguno'}"
    log "INFO" "Archivo: $output_file"
    
    if [ -n "$filter" ]; then
        timeout "$CAPTURE_DURATION" tcpdump -i "$interface" -w "$output_file" \
                -s 65535 -C "$MAX_CAPTURE_SIZE" "$filter"
    else
        timeout "$CAPTURE_DURATION" tcpdump -i "$interface" -w "$output_file" \
                -s 65535 -C "$MAX_CAPTURE_SIZE"
    fi
    
    log "SUCCESS" "Captura completada: $output_file"
    
    # Análisis básico
    analyze_capture "$output_file"
}

# Análisis automático de captura
analyze_capture() {
    local pcap_file=$1
    
    log "INFO" "=== ANÁLISIS DE CAPTURA ==="
    
    # Estadísticas generales
    log "INFO" "Estadísticas generales:"
    capinfos "$pcap_file" 2>/dev/null || log "WARN" "capinfos no disponible"
    
    # Top protocolos
    log "INFO" "Top 10 protocolos:"
    tshark -r "$pcap_file" -q -z io,phs 2>/dev/null | head -15 || \
        log "WARN" "tshark no disponible para análisis"
    
    # Top conversaciones
    log "INFO" "Top 10 conversaciones:"
    tshark -r "$pcap_file" -q -z conv,ip 2>/dev/null | head -15 || \
        tcpdump -r "$pcap_file" -nn | awk '{print $3 " -> " $5}' | sort | uniq -c | sort -nr | head -10
    
    # Errores y retransmisiones TCP
    log "INFO" "Buscando problemas TCP:"
    local tcp_errors=$(tshark -r "$pcap_file" -Y "tcp.analysis.flags" 2>/dev/null | wc -l || echo "0")
    if [ "$tcp_errors" -gt 0 ]; then
        log "WARN" "Se encontraron $tcp_errors posibles problemas TCP"
        tshark -r "$pcap_file" -Y "tcp.analysis.flags" -T fields -e frame.number -e ip.src -e ip.dst -e tcp.analysis.flags 2>/dev/null | head -10
    else
        log "SUCCESS" "No se detectaron problemas TCP obvios"
    fi
}

# Capturas especializadas
capture_http_traffic() {
    local interface=$1
    basic_capture "$interface" "tcp port 80 or tcp port 443"
}

capture_dns_traffic() {
    local interface=$1
    basic_capture "$interface" "udp port 53 or tcp port 53"
}

capture_ssh_traffic() {
    local interface=$1
    basic_capture "$interface" "tcp port 22"
}

capture_slow_connections() {
    local interface=$1
    # Capturar solo SYN, RST, FIN para análisis de conexiones
    basic_capture "$interface" "tcp[tcpflags] & (tcp-syn|tcp-rst|tcp-fin) != 0"
}

# Análisis en tiempo real
real_time_analysis() {
    local interface=$1
    
    log "INFO" "Iniciando análisis en tiempo real en $interface"
    log "INFO" "Presiona Ctrl+C para detener"
    
    # Monitor de conexiones
    (
        while true; do
            echo "=== $(date) ==="
            echo "Conexiones activas:"
            ss -tuln | grep LISTEN | wc -l | xargs echo "Puertos en escucha:"
            ss -tun | grep ESTAB | wc -l | xargs echo "Conexiones establecidas:"
            echo "Top 5 conexiones por IP:"
            ss -tun | grep ESTAB | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr | head -5
            echo
            sleep 10
        done
    ) &
    
    # Monitor de tráfico
    tcpdump -i "$interface" -nn -l | while read line; do
        # Detectar patrones sospechosos
        if echo "$line" | grep -q "RST"; then
            echo "[$(date '+%H:%M:%S')] RESET: $line"
        elif echo "$line" | grep -q "ICMP.*unreachable"; then
            echo "[$(date '+%H:%M:%S')] UNREACHABLE: $line"
        fi
    done
}

# Menú principal
show_menu() {
    cat << EOF
=== HERRAMIENTA DE CAPTURA Y ANÁLISIS DE RED ===

Opciones:
1) Captura básica
2) Captura HTTP/HTTPS
3) Captura DNS
4) Captura SSH
5) Análisis de conexiones lentas
6) Análisis en tiempo real
7) Analizar archivo existente
8) Listar interfaces
9) Mostrar capturas guardadas

EOF
}

main() {
    if [ "$EUID" -ne 0 ]; then
        echo "Este script requiere permisos de root para captura de paquetes"
        exit 1
    fi
    
    case "${1:-menu}" in
        "menu")
            while true; do
                show_menu
                read -p "Selecciona una opción (1-9, q para salir): " choice
                case $choice in
                    1) 
                        read -p "Interfaz: " iface
                        read -p "Filtro (opcional): " filter
                        basic_capture "$iface" "$filter"
                        ;;
                    2) 
                        read -p "Interfaz: " iface
                        capture_http_traffic "$iface"
                        ;;
                    3)
                        read -p "Interfaz: " iface
                        capture_dns_traffic "$iface"
                        ;;
                    4)
                        read -p "Interfaz: " iface
                        capture_ssh_traffic "$iface"
                        ;;
                    5)
                        read -p "Interfaz: " iface
                        capture_slow_connections "$iface"
                        ;;
                    6)
                        read -p "Interfaz: " iface
                        real_time_analysis "$iface"
                        ;;
                    7)
                        read -p "Ruta del archivo .pcap: " pcap_file
                        if [ -f "$pcap_file" ]; then
                            analyze_capture "$pcap_file"
                        else
                            log "ERROR" "Archivo no encontrado"
                        fi
                        ;;
                    8)
                        ip link show
                        ;;
                    9)
                        ls -la "$CAPTURE_DIR"
                        ;;
                    q)
                        break
                        ;;
                    *)
                        echo "Opción inválida"
                        ;;
                esac
                echo
                read -p "Presiona Enter para continuar..."
            done
            ;;
        "capture")
            basic_capture "$2" "${3:-}"
            ;;
        "analyze")
            analyze_capture "$2"
            ;;
        "http")
            capture_http_traffic "$2"
            ;;
        "dns")
            capture_dns_traffic "$2"
            ;;
        *)
            echo "Uso: $0 [menu|capture|analyze|http|dns] [args...]"
            ;;
    esac
}

main "$@"
```

### 2. Análisis de Configuración de Red

```bash
#!/bin/bash

# network_config_analyzer.sh - Analizador de configuración de red
# Autor: Andrés Núñez

analyze_network_configuration() {
    log "INFO" "=== ANÁLISIS DE CONFIGURACIÓN DE RED ==="
    
    # Verificar configuración IP
    log "INFO" "Configuración de interfaces:"
    ip addr show | grep -E "^[0-9]+:|inet " | while read line; do
        if [[ "$line" =~ ^[0-9]+: ]]; then
            echo "  $line"
        else
            echo "    $line"
        fi
    done
    
    # Verificar tabla de routing
    log "INFO" "Tabla de routing:"
    ip route show | while read line; do
        echo "  $line"
    done
    
    # Verificar reglas de firewall
    log "INFO" "Reglas de firewall (iptables):"
    if iptables -L -n | grep -q "Chain"; then
        iptables -L -n --line-numbers | head -20
    else
        log "WARN" "No se pudieron obtener reglas de iptables"
    fi
    
    # Verificar configuración de red permanente
    log "INFO" "Archivos de configuración de red:"
    
    # Debian/Ubuntu
    if [ -d "/etc/netplan" ]; then
        log "INFO" "Configuración Netplan:"
        find /etc/netplan -name "*.yaml" -exec cat {} \; 2>/dev/null
    fi
    
    if [ -f "/etc/network/interfaces" ]; then
        log "INFO" "Configuración /etc/network/interfaces:"
        grep -v "^#" /etc/network/interfaces | grep -v "^$"
    fi
    
    # Red Hat/CentOS
    if [ -d "/etc/sysconfig/network-scripts" ]; then
        log "INFO" "Scripts de red Red Hat:"
        ls /etc/sysconfig/network-scripts/ifcfg-* 2>/dev/null | head -5 | while read file; do
            echo "  $file:"
            cat "$file" | grep -v "^#" | grep -v "^$" | sed 's/^/    /'
        done
    fi
}

# Verificar problemas comunes
check_common_issues() {
    log "INFO" "=== VERIFICANDO PROBLEMAS COMUNES ==="
    
    # IPs duplicadas
    log "INFO" "Verificando IPs duplicadas en la red:"
    local my_ip=$(ip route get 8.8.8.8 | grep -oP 'src \K\S+')
    if [ -n "$my_ip" ]; then
        if arping -c 3 -I "$(ip route get 8.8.8.8 | grep -oP 'dev \K\S+')" "$my_ip" 2>/dev/null | grep -q "reply"; then
            log "WARN" "Posible IP duplicada detectada para $my_ip"
        else
            log "SUCCESS" "No hay IPs duplicadas detectadas"
        fi
    fi
    
    # Verificar MTU
    log "INFO" "Verificando MTU de interfaces:"
    ip link show | grep mtu | while read line; do
        interface=$(echo "$line" | awk '{print $2}' | sed 's/://')
        mtu=$(echo "$line" | grep -o "mtu [0-9]*" | awk '{print $2}')
        
        if [ "$mtu" -lt 1500 ] && [ "$interface" != "lo" ]; then
            log "WARN" "  $interface tiene MTU bajo: $mtu"
        else
            log "INFO" "  $interface MTU: $mtu"
        fi
    done
    
    # Verificar límites de conexión
    log "INFO" "Verificando límites del sistema:"
    local max_files=$(ulimit -n)
    local current_connections=$(ss -tun | grep -c ESTAB)
    
    log "INFO" "  Máximo archivos abiertos: $max_files"
    log "INFO" "  Conexiones actuales: $current_connections"
    
    if [ "$current_connections" -gt $((max_files / 2)) ]; then
        log "WARN" "  Alto número de conexiones vs límite de archivos"
    fi
    
    # Verificar puertos en TIME_WAIT
    local time_wait_count=$(ss -tun | grep -c TIME-WAIT)
    log "INFO" "  Conexiones en TIME_WAIT: $time_wait_count"
    
    if [ "$time_wait_count" -gt 1000 ]; then
        log "WARN" "  Alto número de conexiones TIME_WAIT"
        log "INFO" "  Considerar ajustar net.ipv4.tcp_tw_reuse"
    fi
}

# Optimizaciones sugeridas
suggest_optimizations() {
    log "INFO" "=== SUGERENCIAS DE OPTIMIZACIÓN ==="
    
    # Parámetros de kernel
    log "INFO" "Parámetros de red del kernel actuales:"
    
    local params=(
        "net.core.rmem_max"
        "net.core.wmem_max"
        "net.ipv4.tcp_rmem"
        "net.ipv4.tcp_wmem"
        "net.ipv4.tcp_congestion_control"
        "net.core.netdev_max_backlog"
        "net.ipv4.tcp_tw_reuse"
    )
    
    for param in "${params[@]}"; do
        local value=$(sysctl -n "$param" 2>/dev/null || echo "N/A")
        log "INFO" "  $param = $value"
    done
    
    # Sugerencias específicas
    echo
    log "INFO" "Sugerencias de optimización:"
    
    # Buffer sizes
    local rmem_max=$(sysctl -n net.core.rmem_max)
    if [ "$rmem_max" -lt 16777216 ]; then
        log "WARN" "  Considerar aumentar net.core.rmem_max a 16777216"
    fi
    
    # TCP congestion control
    local cc_algo=$(sysctl -n net.ipv4.tcp_congestion_control)
    if [ "$cc_algo" != "bbr" ] && [ "$cc_algo" != "cubic" ]; then
        log "INFO" "  Considerar usar BBR o CUBIC para control de congestión"
    fi
    
    # TIME_WAIT reuse
    local tw_reuse=$(sysctl -n net.ipv4.tcp_tw_reuse)
    if [ "$tw_reuse" = "0" ]; then
        log "INFO" "  Considerar habilitar net.ipv4.tcp_tw_reuse=1"
    fi
}
```

### 3. Monitoreo de Rendimiento de Red

```bash
#!/bin/bash

# network_performance_monitor.sh - Monitor de rendimiento de red
# Autor: Andrés Núñez

monitor_network_performance() {
    local interface=${1:-$(ip route | grep default | awk '{print $5}' | head -1)}
    local duration=${2:-60}
    
    log "INFO" "Monitoreando rendimiento de $interface por ${duration}s"
    
    # Obtener estadísticas iniciales
    local initial_stats="/tmp/net_stats_initial"
    local final_stats="/tmp/net_stats_final"
    
    get_interface_stats "$interface" > "$initial_stats"
    sleep "$duration"
    get_interface_stats "$interface" > "$final_stats"
    
    # Calcular diferencias
    calculate_network_metrics "$initial_stats" "$final_stats" "$duration"
}

get_interface_stats() {
    local interface=$1
    
    cat "/sys/class/net/$interface/statistics/rx_bytes"
    cat "/sys/class/net/$interface/statistics/tx_bytes"
    cat "/sys/class/net/$interface/statistics/rx_packets"
    cat "/sys/class/net/$interface/statistics/tx_packets"
    cat "/sys/class/net/$interface/statistics/rx_errors"
    cat "/sys/class/net/$interface/statistics/tx_errors"
    cat "/sys/class/net/$interface/statistics/rx_dropped"
    cat "/sys/class/net/$interface/statistics/tx_dropped"
}

calculate_network_metrics() {
    local initial_file=$1
    local final_file=$2
    local duration=$3
    
    # Leer estadísticas
    local initial=($(cat "$initial_file"))
    local final=($(cat "$final_file"))
    
    # Calcular diferencias
    local rx_bytes_diff=$((final[0] - initial[0]))
    local tx_bytes_diff=$((final[1] - initial[1]))
    local rx_packets_diff=$((final[2] - initial[2]))
    local tx_packets_diff=$((final[3] - initial[3]))
    local rx_errors_diff=$((final[4] - initial[4]))
    local tx_errors_diff=$((final[5] - initial[5]))
    
    # Calcular tasas
    local rx_rate_mbps=$(echo "scale=2; $rx_bytes_diff * 8 / $duration / 1000000" | bc)
    local tx_rate_mbps=$(echo "scale=2; $tx_bytes_diff * 8 / $duration / 1000000" | bc)
    local rx_pps=$(echo "scale=0; $rx_packets_diff / $duration" | bc)
    local tx_pps=$(echo "scale=0; $tx_packets_diff / $duration" | bc)
    
    # Mostrar resultados
    log "INFO" "=== MÉTRICAS DE RENDIMIENTO (${duration}s) ==="
    log "INFO" "Recepción:"
    log "INFO" "  Velocidad: ${rx_rate_mbps} Mbps"
    log "INFO" "  Paquetes: ${rx_pps} pps"
    log "INFO" "  Bytes: $(numfmt --to=iec $rx_bytes_diff)"
    log "INFO" "  Errores: $rx_errors_diff"
    
    log "INFO" "Transmisión:"
    log "INFO" "  Velocidad: ${tx_rate_mbps} Mbps"
    log "INFO" "  Paquetes: ${tx_pps} pps"
    log "INFO" "  Bytes: $(numfmt --to=iec $tx_bytes_diff)"
    log "INFO" "  Errores: $tx_errors_diff"
    
    # Alertas
    if [ "$rx_errors_diff" -gt 0 ] || [ "$tx_errors_diff" -gt 0 ]; then
        log "WARN" "Se detectaron errores de red durante el monitoreo"
    fi
    
    # Cleanup
    rm -f "$initial_file" "$final_stats"
}

# Test de latencia y pérdida de paquetes
latency_loss_test() {
    local target=${1:-8.8.8.8}
    local count=${2:-100}
    
    log "INFO" "=== TEST DE LATENCIA Y PÉRDIDA ==="
    log "INFO" "Objetivo: $target, Paquetes: $count"
    
    local ping_result=$(ping -c "$count" -i 0.2 "$target" 2>/dev/null)
    
    if [ $? -eq 0 ]; then
        # Extraer estadísticas
        local packet_loss=$(echo "$ping_result" | grep "packet loss" | awk '{print $6}' | sed 's/%//')
        local latency_stats=$(echo "$ping_result" | tail -1 | awk -F'/' '{print $4 "/" $5 "/" $6}')
        
        IFS='/' read -r min_lat avg_lat max_lat <<< "$latency_stats"
        
        log "INFO" "Resultados:"
        log "INFO" "  Pérdida de paquetes: ${packet_loss}%"
        log "INFO" "  Latencia mínima: ${min_lat}ms"
        log "INFO" "  Latencia promedio: ${avg_lat}ms"
        log "INFO" "  Latencia máxima: ${max_lat}ms"
        
        # Evaluación
        if (( $(echo "$packet_loss > 1" | bc -l) )); then
            log "WARN" "Alta pérdida de paquetes"
        fi
        
        if (( $(echo "$avg_lat > 100" | bc -l) )); then
            log "WARN" "Latencia alta"
        fi
        
        # Calcular jitter (estimación)
        local jitter=$(echo "scale=2; $max_lat - $min_lat" | bc)
        log "INFO" "  Jitter estimado: ${jitter}ms"
        
        if (( $(echo "$jitter > 50" | bc -l) )); then
            log "WARN" "Alto jitter detectado"
        fi
    else
        log "ERROR" "No se pudo realizar el test de ping a $target"
    fi
}

# Test de ancho de banda
bandwidth_test() {
    local target_host=$1
    local target_port=${2:-22}
    
    log "INFO" "=== TEST DE ANCHO DE BANDA ==="
    
    if command -v iperf3 > /dev/null; then
        log "INFO" "Usando iperf3 para test de ancho de banda"
        
        # Test TCP
        log "INFO" "Test TCP:"
        iperf3 -c "$target_host" -p "$target_port" -t 10 2>/dev/null | tail -4
        
        # Test UDP
        log "INFO" "Test UDP:"
        iperf3 -c "$target_host" -p "$target_port" -u -b 100M -t 10 2>/dev/null | tail -4
        
    else
        log "WARN" "iperf3 no está disponible"
        log "INFO" "Realizando test básico con dd y nc"
        
        # Test simple con dd y netcat
        if command -v nc > /dev/null; then
            log "INFO" "Enviando 10MB de datos a $target_host:$target_port"
            time (dd if=/dev/zero bs=1M count=10 2>/dev/null | nc "$target_host" "$target_port")
        else
            log "WARN" "nc no está disponible para test de ancho de banda"
        fi
    fi
}
```

## Resolución de Problemas Específicos

### Problemas Comunes y Soluciones

#### 1. Conectividad intermitente

```bash
#!/bin/bash

# diagnose_intermittent.sh - Diagnóstico de problemas intermitentes
# Autor: Andrés Núñez

monitor_intermittent_connectivity() {
    local target=${1:-8.8.8.8}
    local interval=${2:-1}
    local log_file="/tmp/connectivity_monitor.log"
    
    log "INFO" "Monitoreando conectividad intermitente a $target"
    log "INFO" "Intervalo: ${interval}s, Log: $log_file"
    
    local consecutive_failures=0
    local total_tests=0
    local total_failures=0
    
    while true; do
        local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        total_tests=$((total_tests + 1))
        
        if ping -c 1 -W 3 "$target" > /dev/null 2>&1; then
            if [ $consecutive_failures -gt 0 ]; then
                log "SUCCESS" "[$timestamp] Conectividad restaurada después de $consecutive_failures fallos"
                echo "[$timestamp] RECOVERED after $consecutive_failures failures" >> "$log_file"
            fi
            consecutive_failures=0
        else
            consecutive_failures=$((consecutive_failures + 1))
            total_failures=$((total_failures + 1))
            
            log "ERROR" "[$timestamp] Fallo $consecutive_failures - Total: $total_failures/$total_tests"
            echo "[$timestamp] FAILURE $consecutive_failures" >> "$log_file"
            
            # Diagnóstico adicional en fallos consecutivos
            if [ $consecutive_failures -eq 3 ]; then
                log "WARN" "Realizando diagnóstico adicional..."
                
                # Verificar tabla ARP
                arp -a | grep -E "(incomplete|FAILED)" && log "WARN" "Problemas ARP detectados"
                
                # Verificar interfaz
                local main_interface=$(ip route | grep default | awk '{print $5}' | head -1)
                if ip link show "$main_interface" | grep -q "DOWN"; then
                    log "ERROR" "Interfaz principal $main_interface está DOWN"
                fi
                
                # Verificar gateway
                local gateway=$(ip route | grep default | awk '{print $3}' | head -1)
                if [ -n "$gateway" ]; then
                    if ! ping -c 1 -W 2 "$gateway" > /dev/null 2>&1; then
                        log "ERROR" "Gateway $gateway no responde"
                    fi
                fi
            fi
        fi
        
        sleep "$interval"
    done
}

# Análisis de patrones en logs
analyze_connectivity_patterns() {
    local log_file="/tmp/connectivity_monitor.log"
    
    if [ ! -f "$log_file" ]; then
        log "ERROR" "Archivo de log no encontrado: $log_file"
        return 1
    fi
    
    log "INFO" "=== ANÁLISIS DE PATRONES DE CONECTIVIDAD ==="
    
    # Estadísticas generales
    local total_entries=$(wc -l < "$log_file")
    local failures=$(grep -c "FAILURE" "$log_file")
    local recoveries=$(grep -c "RECOVERED" "$log_file")
    
    log "INFO" "Total de entradas: $total_entries"
    log "INFO" "Fallos registrados: $failures"
    log "INFO" "Recuperaciones: $recoveries"
    
    # Patrones temporales
    log "INFO" "Distribución por hora:"
    awk '{print $2}' "$log_file" | cut -d: -f1 | sort | uniq -c | sort -nr
    
    # Duración de interrupciones
    log "INFO" "Análisis de interrupciones:"
    grep "RECOVERED" "$log_file" | awk '{print $6}' | sort -nr | head -10 | while read duration; do
        log "INFO" "  Interrupción de $duration fallos consecutivos"
    done
}
```

#### 2. Problemas de DNS

```bash
#!/bin/bash

# dns_troubleshoot.sh - Diagnóstico de problemas DNS
# Autor: Andrés Núñez

comprehensive_dns_test() {
    local domain=${1:-google.com}
    
    log "INFO" "=== DIAGNÓSTICO COMPLETO DNS para $domain ==="
    
    # Test básico de resolución
    log "INFO" "1. Test básico de resolución:"
    if nslookup "$domain" > /dev/null 2>&1; then
        local ip=$(nslookup "$domain" | grep "Address:" | grep -v "#53" | awk '{print $2}' | head -1)
        log "SUCCESS" "  Resuelve a: $ip"
    else
        log "ERROR" "  Fallo en resolución básica"
        return 1
    fi
    
    # Test con diferentes servidores DNS
    log "INFO" "2. Test con diferentes servidores DNS:"
    local dns_servers=("8.8.8.8" "1.1.1.1" "9.9.9.9")
    
    for dns in "${dns_servers[@]}"; do
        local result=$(dig @"$dns" "$domain" +short +time=3 2>/dev/null | head -1)
        if [ -n "$result" ]; then
            log "SUCCESS" "  $dns: $result"
        else
            log "ERROR" "  $dns: Sin respuesta"
        fi
    done
    
    # Test de tipos de registro
    log "INFO" "3. Tipos de registro DNS:"
    local record_types=("A" "AAAA" "MX" "NS" "TXT")
    
    for type in "${record_types[@]}"; do
        local result=$(dig "$domain" "$type" +short 2>/dev/null | head -1)
        if [ -n "$result" ]; then
            log "INFO" "  $type: $result"
        fi
    done
    
    # Test de DNS reverso
    log "INFO" "4. DNS reverso:"
    if [ -n "$ip" ]; then
        local reverse=$(dig -x "$ip" +short 2>/dev/null)
        if [ -n "$reverse" ]; then
            log "INFO" "  $ip -> $reverse"
        else
            log "WARN" "  No hay registro PTR para $ip"
        fi
    fi
    
    # Verificar configuración local
    log "INFO" "5. Configuración DNS local:"
    if [ -f /etc/resolv.conf ]; then
        log "INFO" "  Servidores configurados:"
        grep nameserver /etc/resolv.conf | while read line; do
            log "INFO" "    $line"
        done
        
        local search_domains=$(grep search /etc/resolv.conf | cut -d' ' -f2-)
        if [ -n "$search_domains" ]; then
            log "INFO" "  Dominios de búsqueda: $search_domains"
        fi
    fi
    
    # Test de tiempo de respuesta
    log "INFO" "6. Tiempo de respuesta DNS:"
    local query_time=$(dig "$domain" | grep "Query time:" | awk '{print $4}')
    if [ -n "$query_time" ]; then
        log "INFO" "  Tiempo de consulta: ${query_time}ms"
        
        if [ "$query_time" -gt 100 ]; then
            log "WARN" "  Tiempo de respuesta DNS alto"
        fi
    fi
}

# Verificar problemas comunes de DNS
check_dns_issues() {
    log "INFO" "=== VERIFICANDO PROBLEMAS COMUNES DNS ==="
    
    # Verificar si el servicio DNS está corriendo
    log "INFO" "1. Estado del servicio DNS local:"
    if systemctl is-active --quiet systemd-resolved; then
        log "SUCCESS" "  systemd-resolved está activo"
    elif systemctl is-active --quiet dnsmasq; then
        log "SUCCESS" "  dnsmasq está activo"
    elif systemctl is-active --quiet bind9; then
        log "SUCCESS" "  bind9 está activo"
    else
        log "WARN" "  No se detectó servicio DNS local activo"
    fi
    
    # Verificar cache DNS
    log "INFO" "2. Estado del cache DNS:"
    if command -v systemd-resolve > /dev/null; then
        systemd-resolve --statistics | grep -E "(Current|Cache)"
    fi
    
    # Verificar conectividad a servidores DNS
    log "INFO" "3. Conectividad a servidores DNS:"
    grep nameserver /etc/resolv.conf | awk '{print $2}' | while read dns_server; do
        if ping -c 2 -W 2 "$dns_server" > /dev/null 2>&1; then
            log "SUCCESS" "  $dns_server es alcanzable"
            
            # Test del puerto 53
            if timeout 3 bash -c "</dev/tcp/$dns_server/53" 2>/dev/null; then
                log "SUCCESS" "    Puerto 53 TCP accesible"
            else
                log "WARN" "    Puerto 53 TCP no accesible"
            fi
        else
            log "ERROR" "  $dns_server NO es alcanzable"
        fi
    done
    
    # Verificar archivos de configuración
    log "INFO" "4. Archivos de configuración:"
    
    # /etc/hosts
    if [ -f /etc/hosts ]; then
        local hosts_entries=$(grep -v "^#" /etc/hosts | grep -v "^$" | wc -l)
        log "INFO" "  /etc/hosts: $hosts_entries entradas"
        
        # Buscar duplicados
        local duplicates=$(awk '{print $2}' /etc/hosts | grep -v "^$" | sort | uniq -d)
        if [ -n "$duplicates" ]; then
            log "WARN" "  Duplicados en /etc/hosts: $duplicates"
        fi
    fi
    
    # nsswitch.conf
    if [ -f /etc/nsswitch.conf ]; then
        local hosts_line=$(grep "^hosts:" /etc/nsswitch.conf)
        log "INFO" "  Orden de resolución: $hosts_line"
    fi
}

# Limpiar cache DNS
flush_dns_cache() {
    log "INFO" "=== LIMPIANDO CACHE DNS ==="
    
    # systemd-resolved
    if command -v systemd-resolve > /dev/null; then
        systemd-resolve --flush-caches
        log "SUCCESS" "Cache systemd-resolved limpiado"
    fi
    
    # dnsmasq
    if systemctl is-active --quiet dnsmasq; then
        systemctl restart dnsmasq
        log "SUCCESS" "dnsmasq reiniciado"
    fi
    
    # nscd
    if command -v nscd > /dev/null; then
        nscd -i hosts
        log "SUCCESS" "Cache nscd limpiado"
    fi
    
    log "INFO" "Cache DNS limpiado. Probando resolución..."
    sleep 2
    
    # Test post-limpieza
    if nslookup google.com > /dev/null 2>&1; then
        log "SUCCESS" "Resolución DNS funcionando después de limpiar cache"
    else
        log "ERROR" "Problemas de DNS persisten después de limpiar cache"
    fi
}
```

## Conclusión

El troubleshooting de redes requiere un enfoque sistemático y el dominio de múltiples herramientas. Los elementos clave para un diagnóstico efectivo incluyen:

### Principios fundamentales:

1. **Metodología estructurada**: Usar enfoques bottom-up o top-down según el problema
2. **Documentación**: Registrar todos los pasos y resultados
3. **Aislamiento**: Dividir el problema en componentes más pequeños
4. **Verificación**: Confirmar que las soluciones resuelven el problema real
5. **Prevención**: Implementar monitoreo para detectar problemas futuros

### Herramientas esenciales:

- **Diagnóstico básico**: ping, traceroute, nslookup, dig
- **Análisis de conectividad**: telnet, nc, curl, wget
- **Monitoreo de interfaces**: ip, ss, netstat, iftop
- **Captura de paquetes**: tcpdump, wireshark, tshark
- **Análisis de rendimiento**: iperf3, mtr, speedtest-cli

### Problemas más comunes:

1. **Configuración IP incorrecta**: IPs duplicadas, máscara de subred errónea
2. **Problemas de routing**: Gateway incorrecto, rutas faltantes
3. **DNS mal configurado**: Servidores DNS incorrectos, cache corrupto
4. **Firewall**: Reglas que bloquean tráfico legítimo
5. **Problemas físicos**: Cables defectuosos, puertos dañados

La experiencia en troubleshooting de redes se desarrolla con la práctica constante y la exposición a diferentes tipos de problemas. Mantén siempre un enfoque metódico, documenta tus hallazgos y nunca subestimes la importancia de verificar los fundamentos básicos antes de buscar soluciones complejas.

---

*Escrito por Andrés Núñez, especialista en redes, diagnóstico de infraestructuras y administración de sistemas distribuidos.*
