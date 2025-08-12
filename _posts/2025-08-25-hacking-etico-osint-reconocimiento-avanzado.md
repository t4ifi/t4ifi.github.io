---
title: "Hacking Ético #3: OSINT y Reconocimiento Avanzado - Information Gathering Profesional"
date: 2025-08-25 10:00:00 +0000
categories: [Ciberseguridad, Hacking Ético]
tags: [osint, reconocimiento, google-dorking, information-gathering, recon, footprinting]
---

## 🎯 OSINT: El Arte del Reconocimiento Silencioso

El **OSINT** (Open Source Intelligence) es la disciplina de recopilar información de fuentes públicamente disponibles. En pentesting, representa el **80% del éxito** - cuanta más información tengas, más preciso será tu ataque.

> 💡 **Regla de oro:** El reconocimiento nunca termina. Siempre hay más información que descubrir.

---

## 🧠 Metodología OSINT Profesional

### Fases del reconocimiento:

1. **Footprinting pasivo** - Sin tocar el objetivo
2. **Footprinting activo** - Interacción mínima
3. **Scanning** - Interacción directa
4. **Enumeration** - Extracción detallada
5. **Vulnerability Assessment** - Identificación de debilidades

### Pirámide de información:

```
           🎯 OBJETIVO
          /           \
    🌐 DOMINIOS     📧 EMAILS
    /       \       /       \
🖥️ IPS    📱 APPS  👥 PERSONAS  🏢 ORG
```

---

## 🌐 Reconocimiento de Dominios y Subdominios

### 1. Información básica del dominio:

```bash
#!/bin/bash
# domain-recon.sh

TARGET_DOMAIN=$1

if [ -z "$TARGET_DOMAIN" ]; then
    echo "Uso: $0 <dominio>"
    exit 1
fi

echo "🎯 Reconocimiento de: $TARGET_DOMAIN"
echo "====================================="

# WHOIS lookup
echo "📋 WHOIS Information:"
whois $TARGET_DOMAIN | tee whois_$TARGET_DOMAIN.txt

# DNS básico
echo -e "\n🔍 DNS Records:"
dig $TARGET_DOMAIN ANY +noall +answer | tee dns_$TARGET_DOMAIN.txt

# Subdominios con múltiples herramientas
echo -e "\n🕸️ Subdomain Enumeration:"

# Subfinder
subfinder -d $TARGET_DOMAIN -o subfinder_$TARGET_DOMAIN.txt

# Assetfinder
assetfinder $TARGET_DOMAIN | tee assetfinder_$TARGET_DOMAIN.txt

# Amass (más completo pero más lento)
amass enum -d $TARGET_DOMAIN -o amass_$TARGET_DOMAIN.txt

# Consolidar resultados
cat subfinder_$TARGET_DOMAIN.txt assetfinder_$TARGET_DOMAIN.txt amass_$TARGET_DOMAIN.txt | sort -u > all_subdomains_$TARGET_DOMAIN.txt

echo "✅ Subdominios encontrados: $(wc -l < all_subdomains_$TARGET_DOMAIN.txt)"
```

### 2. DNS enumeration avanzada:

```bash
#!/bin/bash
# advanced-dns.sh

DOMAIN=$1

# DNS Zone Transfer attempt
echo "🔄 Intentando Zone Transfer..."
dig axfr $DOMAIN @$(dig ns $DOMAIN +short | head -1)

# DNS Brute force
echo "💥 DNS Brute Force..."
dnsrecon -d $DOMAIN -D /usr/share/wordlists/dnsmap.txt -t brt

# Reverse DNS lookup en rangos de IP
echo "🔄 Reverse DNS Lookup..."
for ip in $(dig $DOMAIN +short); do
    dig -x $ip +short
done

# DNS Cache Snooping
echo "👁️ DNS Cache Snooping..."
dnsrecon -d $DOMAIN -t snoop

# Buscar registros específicos
echo "📝 Registros específicos:"
for record in A AAAA CNAME MX TXT NS SOA; do
    echo "=== $record Records ==="
    dig $DOMAIN $record +short
done
```

### 3. Herramientas especializadas:

```bash
# Nuclei para subdomain discovery
nuclei -t ~/nuclei-templates/dns/ -target $DOMAIN

# HTTPx para verificar servicios web activos
cat all_subdomains_$DOMAIN.txt | httpx -ports 80,443,8080,8443 -o live_subdomains.txt

# Screenshot de todos los servicios web
cat live_subdomains.txt | aquatone

# Tecnologías web
cat live_subdomains.txt | httpx -tech-detect
```

---

## 🕵️ Google Dorking Avanzado

### Operadores básicos potentes:

```bash
# Búsquedas en sitio específico
site:example.com

# Tipos de archivo específicos
site:example.com filetype:pdf
site:example.com filetype:xlsx
site:example.com filetype:docx
site:example.com filetype:sql

# Información sensible
site:example.com "confidential"
site:example.com "password"
site:example.com "login"
site:example.com "admin"

# Directorios expuestos
site:example.com intitle:"index of"
site:example.com "parent directory"

# Configuraciones expuestas
site:example.com filetype:env
site:example.com filetype:config
site:example.com filetype:ini

# Errores que revelan información
site:example.com "Fatal error"
site:example.com "Warning: mysql_"
site:example.com "sql syntax error"
```

### Dorking automatizado:

```bash
#!/bin/bash
# google-dorking.sh

DOMAIN=$1
OUTPUT_DIR="google_dorking_$DOMAIN"
mkdir -p $OUTPUT_DIR

# Lista de dorks específicos
DORKS=(
    "site:$DOMAIN filetype:pdf"
    "site:$DOMAIN filetype:xlsx"
    "site:$DOMAIN intitle:login"
    "site:$DOMAIN intitle:admin"
    "site:$DOMAIN \"password\""
    "site:$DOMAIN \"confidential\""
    "site:$DOMAIN filetype:env"
    "site:$DOMAIN \"index of\""
    "site:$DOMAIN \"sql error\""
    "site:$DOMAIN inurl:wp-admin"
    "site:$DOMAIN inurl:/admin/"
    "site:$DOMAIN inurl:login.php"
)

echo "🔍 Ejecutando Google Dorking para $DOMAIN..."

for dork in "${DORKS[@]}"; do
    echo "Buscando: $dork"
    # Usar googler o similar para automatizar
    googler --count 20 --json "$dork" > "$OUTPUT_DIR/$(echo $dork | tr ' :/' '_').json"
    sleep 2  # Rate limiting
done

echo "✅ Google Dorking completado. Revisa: $OUTPUT_DIR"
```

### Dorking con múltiples motores:

```python
#!/usr/bin/env python3
# multi-search.py

import requests
import time
import json
from bs4 import BeautifulSoup

class MultiSearchDorking:
    def __init__(self, domain):
        self.domain = domain
        self.results = {}
        
    def bing_search(self, query):
        """Búsqueda en Bing"""
        url = f"https://www.bing.com/search?q={query}"
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'}
        
        try:
            response = requests.get(url, headers=headers)
            soup = BeautifulSoup(response.content, 'html.parser')
            links = [a.get('href') for a in soup.find_all('a', href=True)]
            return links
        except Exception as e:
            print(f"Error en Bing: {e}")
            return []
    
    def duckduckgo_search(self, query):
        """Búsqueda en DuckDuckGo"""
        url = f"https://duckduckgo.com/html/?q={query}"
        headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'}
        
        try:
            response = requests.get(url, headers=headers)
            soup = BeautifulSoup(response.content, 'html.parser')
            links = [a.get('href') for a in soup.find_all('a', class_='result__a')]
            return links
        except Exception as e:
            print(f"Error en DuckDuckGo: {e}")
            return []
    
    def comprehensive_dork(self):
        """Dorking comprehensivo"""
        dorks = [
            f"site:{self.domain} filetype:pdf",
            f"site:{self.domain} intitle:login",
            f"site:{self.domain} inurl:admin",
            f'site:{self.domain} "password"',
            f'site:{self.domain} "config"'
        ]
        
        for dork in dorks:
            print(f"🔍 Dorking: {dork}")
            
            # Buscar en múltiples motores
            bing_results = self.bing_search(dork)
            duck_results = self.duckduckgo_search(dork)
            
            self.results[dork] = {
                'bing': bing_results,
                'duckduckgo': duck_results
            }
            
            time.sleep(3)  # Rate limiting
        
        return self.results

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) != 2:
        print("Uso: python3 multi-search.py <dominio>")
        sys.exit(1)
    
    domain = sys.argv[1]
    dorker = MultiSearchDorking(domain)
    results = dorker.comprehensive_dork()
    
    # Guardar resultados
    with open(f'dorking_results_{domain}.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    print(f"✅ Resultados guardados en dorking_results_{domain}.json")
```

---

## 📧 Email Harvesting y Social Engineering

### 1. Recolección de emails:

```bash
#!/bin/bash
# email-harvesting.sh

DOMAIN=$1

echo "📧 Email Harvesting para: $DOMAIN"

# TheHarvester
theharvester -d $DOMAIN -b all -l 500 -f theharvester_$DOMAIN.html

# Hunter.io (requiere API key)
if [ ! -z "$HUNTER_API_KEY" ]; then
    curl "https://api.hunter.io/v2/domain-search?domain=$DOMAIN&api_key=$HUNTER_API_KEY" | jq '.data.emails[].value' > hunter_emails.txt
fi

# Google dorking para emails
googler --count 50 "site:$DOMAIN \"@$DOMAIN\"" | grep -oE "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" > google_emails.txt

# LinkedIn enumeration con linkedin2username
if command -v linkedin2username &> /dev/null; then
    linkedin2username -c "company name" -n $DOMAIN
fi

# Consolidar resultados
cat theharvester_*.txt hunter_emails.txt google_emails.txt | grep -E "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" | sort -u > all_emails_$DOMAIN.txt

echo "✅ Emails encontrados: $(wc -l < all_emails_$DOMAIN.txt)"
```

### 2. OSINT de personas:

```python
#!/usr/bin/env python3
# people-osint.py

import requests
import json
import re

class PeopleOSINT:
    def __init__(self):
        self.results = {}
    
    def search_haveibeenpwned(self, email):
        """Verificar si el email ha sido comprometido"""
        url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}"
        headers = {'User-Agent': 'OSINT-Tool'}
        
        try:
            response = requests.get(url, headers=headers)
            if response.status_code == 200:
                return response.json()
            elif response.status_code == 404:
                return {"status": "No breaches found"}
            else:
                return {"error": f"HTTP {response.status_code}"}
        except Exception as e:
            return {"error": str(e)}
    
    def social_media_search(self, username):
        """Buscar en redes sociales"""
        platforms = {
            'twitter': f'https://twitter.com/{username}',
            'linkedin': f'https://linkedin.com/in/{username}',
            'github': f'https://github.com/{username}',
            'instagram': f'https://instagram.com/{username}'
        }
        
        results = {}
        for platform, url in platforms.items():
            try:
                response = requests.head(url, timeout=5)
                results[platform] = {
                    'url': url,
                    'exists': response.status_code == 200
                }
            except:
                results[platform] = {
                    'url': url,
                    'exists': False
                }
        
        return results
    
    def comprehensive_search(self, target):
        """Búsqueda comprehensiva"""
        # Si es email
        if '@' in target:
            username = target.split('@')[0]
            self.results['email'] = target
            self.results['breaches'] = self.search_haveibeenpwned(target)
        else:
            username = target
        
        # Búsqueda en redes sociales
        self.results['social_media'] = self.social_media_search(username)
        
        return self.results

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) != 2:
        print("Uso: python3 people-osint.py <email|username>")
        sys.exit(1)
    
    target = sys.argv[1]
    osint = PeopleOSINT()
    results = osint.comprehensive_search(target)
    
    print(json.dumps(results, indent=2))
```

---

## 🌐 Reconocimiento de Infraestructura

### 1. Descubrimiento de IPs y rangos:

```bash
#!/bin/bash
# ip-discovery.sh

DOMAIN=$1

echo "🌐 Descubrimiento de infraestructura para: $DOMAIN"

# Obtener IPs del dominio principal
echo "📍 IPs del dominio principal:"
dig $DOMAIN +short | tee domain_ips.txt

# ASN (Autonomous System Number) lookup
echo "🏢 Información de ASN:"
for ip in $(cat domain_ips.txt); do
    whois $ip | grep -E "(OrgName|NetRange|CIDR)" | tee -a asn_info.txt
done

# Rangos de IP de la organización
echo "📊 Rangos de IP de la organización:"
amass intel -org "Organization Name" -ip

# Certificados SSL para encontrar más subdominios
echo "🔐 Búsqueda por certificados SSL:"
curl -s "https://crt.sh/?q=%25.$DOMAIN&output=json" | jq -r '.[].name_value' | sort -u | tee cert_subdomains.txt

# Reverse IP lookup
echo "🔄 Reverse IP Lookup:"
for ip in $(cat domain_ips.txt); do
    echo "=== $ip ==="
    dig -x $ip +short
    # Buscar otros dominios en la misma IP
    curl -s "https://api.hackertarget.com/reverseiplookup/?q=$ip" | tee -a reverse_ip_$ip.txt
done
```

### 2. CDN y tecnologías:

```bash
#!/bin/bash
# tech-discovery.sh

DOMAIN=$1

echo "🔧 Análisis de tecnologías para: $DOMAIN"

# WAF detection
wafw00f http://$DOMAIN
wafw00f https://$DOMAIN

# Technology stack detection
whatweb $DOMAIN | tee whatweb_$DOMAIN.txt

# HTTP headers analysis
curl -I http://$DOMAIN | tee -a headers_$DOMAIN.txt
curl -I https://$DOMAIN | tee -a headers_$DOMAIN.txt

# SSL/TLS analysis
echo "🔐 Análisis SSL/TLS:"
sslscan $DOMAIN | tee sslscan_$DOMAIN.txt
testssl.sh $DOMAIN | tee testssl_$DOMAIN.txt

# CDN detection
echo "☁️ Detección de CDN:"
dig $DOMAIN | grep -E "(cloudflare|fastly|akamai|amazonaws|googleusercontent)"
```

---

## 🔍 Automatización Completa del Reconocimiento

### Script maestro de reconocimiento:

```bash
#!/bin/bash
# master-recon.sh

TARGET=$1
OUTPUT_DIR="recon_$(date +%Y%m%d_%H%M%S)_$TARGET"

if [ -z "$TARGET" ]; then
    echo "Uso: $0 <dominio>"
    exit 1
fi

echo "🎯 RECONOCIMIENTO COMPLETO DE: $TARGET"
echo "======================================="

# Crear estructura de directorios
mkdir -p $OUTPUT_DIR/{dns,subdomains,emails,ips,tech,social,dorking}
cd $OUTPUT_DIR

# Función de logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a recon.log
}

log "Iniciando reconocimiento de $TARGET"

# Fase 1: DNS y subdominios
log "Fase 1: DNS y subdominios"
./domain-recon.sh $TARGET
mv *_$TARGET.txt dns/

# Fase 2: Google Dorking
log "Fase 2: Google Dorking"
./google-dorking.sh $TARGET
mv google_dorking_$TARGET dorking/

# Fase 3: Email harvesting
log "Fase 3: Email harvesting"
./email-harvesting.sh $TARGET
mv *emails*.txt emails/

# Fase 4: IP y infraestructura
log "Fase 4: Infraestructura"
./ip-discovery.sh $TARGET
mv *ips*.txt *asn*.txt ips/

# Fase 5: Tecnologías
log "Fase 5: Tecnologías"
./tech-discovery.sh $TARGET
mv *tech*.txt *ssl*.txt tech/

# Fase 6: Screenshots y análisis visual
log "Fase 6: Screenshots"
if [ -f dns/live_subdomains.txt ]; then
    cat dns/live_subdomains.txt | aquatone -out screenshots/
fi

# Fase 7: Generación de reporte
log "Generando reporte final"
cat > recon_report.md << EOF
# Reporte de Reconocimiento - $TARGET
**Fecha:** $(date)
**Pentester:** $(whoami)

## Resumen Ejecutivo
- **Subdominios encontrados:** $(wc -l < dns/all_subdomains_$TARGET.txt 2>/dev/null || echo "N/A")
- **Emails encontrados:** $(wc -l < emails/all_emails_$TARGET.txt 2>/dev/null || echo "N/A")
- **IPs identificadas:** $(wc -l < ips/domain_ips.txt 2>/dev/null || echo "N/A")

## Subdominios
\`\`\`
$(cat dns/all_subdomains_$TARGET.txt 2>/dev/null || echo "No encontrados")
\`\`\`

## Emails
\`\`\`
$(head -20 emails/all_emails_$TARGET.txt 2>/dev/null || echo "No encontrados")
\`\`\`

## Tecnologías Detectadas
\`\`\`
$(cat tech/whatweb_$TARGET.txt 2>/dev/null || echo "No detectadas")
\`\`\`

## Próximos Pasos
1. Escanear puertos en IPs identificadas
2. Enumerar servicios web encontrados
3. Verificar credenciales por defecto
4. Buscar vulnerabilidades conocidas
EOF

log "Reconocimiento completado. Reporte disponible en: $PWD/recon_report.md"

# Mostrar estadísticas finales
echo ""
echo "📊 ESTADÍSTICAS FINALES:"
echo "========================"
echo "Subdominios: $(wc -l < dns/all_subdomains_$TARGET.txt 2>/dev/null || echo "0")"
echo "Emails: $(wc -l < emails/all_emails_$TARGET.txt 2>/dev/null || echo "0")"
echo "IPs: $(wc -l < ips/domain_ips.txt 2>/dev/null || echo "0")"
echo "Archivos generados: $(find . -type f | wc -l)"
echo ""
echo "🎯 Siguiente fase: Port scanning y service enumeration"
```

---

## 🛡️ OSINT Defensivo

### Monitoreo de tu propia exposición:

```bash
#!/bin/bash
# defensive-osint.sh

MY_DOMAIN=$1

echo "🛡️ OSINT Defensivo para: $MY_DOMAIN"

# Verificar qué información está expuesta
echo "🔍 Información expuesta públicamente:"

# DNS records que pueden revelar infraestructura
dig $MY_DOMAIN ANY +short | tee exposed_dns.txt

# Subdominios conocidos públicamente
subfinder -d $MY_DOMAIN -silent | tee exposed_subdomains.txt

# Información en motores de búsqueda
googler --count 100 "site:$MY_DOMAIN" | tee google_exposure.txt

# Emails expuestos
theharvester -d $MY_DOMAIN -b all -l 100 | grep "@" | tee exposed_emails.txt

# Certificados SSL públicos
curl -s "https://crt.sh/?q=%25.$MY_DOMAIN&output=json" | jq -r '.[].name_value' | sort -u | tee cert_exposure.txt

# GitHub/GitLab exposure
echo "🔍 Buscando exposición en repositorios:"
for keyword in $MY_DOMAIN api_key password token; do
    echo "Buscando: $keyword"
    # Usar github-search o similar
done

# Generar reporte de exposición
cat > exposure_report.md << EOF
# Reporte de Exposición - $MY_DOMAIN

## Subdominios Expuestos
$(cat exposed_subdomains.txt)

## Emails Expuestos
$(cat exposed_emails.txt)

## Recomendaciones
1. Revisar subdominios innecesarios
2. Implementar políticas de email
3. Monitorear certificados SSL
4. Revisar repositorios de código
EOF

echo "✅ Reporte de exposición generado: exposure_report.md"
```

---

## 🎯 Laboratorio Práctico

### Ejercicio 1: Reconocimiento de Metasploitable

```bash
# 1. Iniciar Metasploitable
lab start

# 2. Descubrir la IP
nmap -sn 10.0.2.0/24

# 3. Reconocimiento básico (simular dominio)
echo "10.0.2.4 metasploitable.local" | sudo tee -a /etc/hosts

# 4. Ejecutar reconocimiento completo
./master-recon.sh metasploitable.local

# 5. Analizar resultados
cat recon_*/recon_report.md
```

### Ejercicio 2: Tu primer OSINT real

```bash
# Usar un dominio de prueba público como example.com
./master-recon.sh example.com

# Comparar resultados con:
# - Shodan.io
# - Censys.io
# - SecurityTrails
```

---

## 📚 Herramientas OSINT Avanzadas

### Suite completa de herramientas:

```bash
#!/bin/bash
# install-osint-tools.sh

echo "🔧 Instalando herramientas OSINT avanzadas..."

# Go tools
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install github.com/tomnomnom/assetfinder@latest
go install github.com/hakluke/hakrawler@latest

# Python tools
pip3 install theHarvester
pip3 install spiderfoot
pip3 install recon-ng
pip3 install photon
pip3 install osrframework

# Specialized tools
sudo apt install -y \
    maltego \
    fierce \
    dnsrecon \
    dmitry \
    whois \
    dnsutils \
    sublist3r

# OSINT framework
git clone https://github.com/j3ssie/osmedeus
git clone https://github.com/lanmaster53/recon-ng
git clone https://github.com/yogeshojha/rengine

echo "✅ Herramientas OSINT instaladas"
```

---

## 🎯 Próximo Tutorial

En la siguiente entrega veremos:
- **Network scanning avanzado con Nmap**
- **Service enumeration detallada**
- **Vulnerability assessment automatizado**
- **Primer laboratorio de explotación básica**

💡 **Recuerda:** El reconocimiento es iterativo. Cada pieza de información puede llevar a descubrir más datos valiosos.

---
**Andrés Nuñez - t4ifi**
