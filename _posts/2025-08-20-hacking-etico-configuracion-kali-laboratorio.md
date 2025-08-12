---
title: "Hacking Ético #2: Configuración Avanzada de Kali Linux y Laboratorio de Práctica"
date: 2025-08-20 10:00:00 +0000
categories: [Ciberseguridad, Hacking Ético]
tags: [kali-linux, laboratorio, pentesting, configuracion, herramientas, metasploitable]
---

## 🎯 Configuración Profesional de Kali Linux

En este tutorial vamos a configurar Kali Linux como un verdadero pentest profesional, optimizado para máximo rendimiento y productividad.

> 💡 **Tip:** Esta configuración está basada en años de experiencia en pentesting real y te ahorrará horas de configuración manual.

---

## 🚀 Instalación y Configuración Inicial

### Descarga e instalación optimizada:

```bash
# 1. Descargar Kali Linux más reciente
wget -c https://cdimage.kali.org/kali-2023.3/kali-linux-2023.3-installer-amd64.iso

# 2. Verificar integridad
wget https://cdimage.kali.org/kali-2023.3/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing

# 3. Configuración de VM recomendada:
# - CPU: 4 cores (mínimo 2)
# - RAM: 8GB (mínimo 4GB)
# - Disco: 80GB (mínimo 40GB)
# - Red: NAT + Host-only adapter
# - Graphics: 128MB VRAM, 3D acceleration habilitado
```

### Primera configuración del sistema:

```bash
#!/bin/bash
# kali-setup.sh - Script de configuración inicial

echo "🔧 Configurando Kali Linux para pentesting profesional..."

# Actualizar repositorios y sistema
sudo apt update && sudo apt full-upgrade -y

# Instalar herramientas esenciales
sudo apt install -y \
    curl \
    wget \
    git \
    vim \
    tmux \
    htop \
    tree \
    unzip \
    p7zip-full \
    terminator \
    firefox-esr \
    chromium \
    flameshot \
    keepassxc \
    openvpn \
    network-manager-openvpn-gnome

# Herramientas adicionales de pentesting
sudo apt install -y \
    gobuster \
    ffuf \
    subfinder \
    amass \
    nuclei \
    httpx \
    katana \
    gau \
    waybackurls

# Lenguajes de programación y librerías
sudo apt install -y \
    python3-pip \
    golang-go \
    nodejs \
    npm \
    ruby \
    ruby-dev \
    gem

# Instalar herramientas de Python
pip3 install --user \
    requests \
    beautifulsoup4 \
    lxml \
    scapy \
    pwntools \
    impacket \
    bloodhound \
    mitm6 \
    crackmapexec \
    dnsrecon \
    dirsearch

# Configurar Go tools
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Instalar herramientas de Go
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/lc/gau@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/tomnomnom/assetfinder@latest

echo "✅ Configuración inicial completada"
```

---

## 🎨 Personalización del Entorno

### Configuración de terminal avanzada:

```bash
# ~/.zshrc optimizado para pentesting
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="agnoster"

plugins=(
    git
    sudo
    history
    zsh-autosuggestions
    zsh-syntax-highlighting
    docker
    kubectl
    python
)

# Variables de entorno para pentesting
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:$HOME/.local/bin

# Aliases útiles para pentesting
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias nse='ls /usr/share/nmap/scripts/ | grep'
alias http-server='python3 -m http.server'
alias decode64='base64 -d'
alias encode64='base64 -w 0'

# Funciones útiles
function extract-ips() {
    grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" $1 | sort -u
}

function port-scan() {
    if [ $# -eq 0 ]; then
        echo "Uso: port-scan <IP>"
        return 1
    fi
    nmap -sS -sV -sC -O --script=vuln -oA "scan_$1" $1
}

function web-enum() {
    if [ $# -eq 0 ]; then
        echo "Uso: web-enum <URL>"
        return 1
    fi
    
    echo "🔍 Enumerando $1..."
    
    # Directory discovery
    gobuster dir -u $1 -w /usr/share/wordlists/dirb/common.txt -o "gobuster_$1.txt" &
    
    # Technology detection
    whatweb $1 | tee "whatweb_$1.txt"
    
    # Nikto scan
    nikto -h $1 -o "nikto_$1.txt"
    
    wait
    echo "✅ Enumeración web completada"
}

source $ZSH/oh-my-zsh.sh
```

### Configuración de tmux para multitasking:

```bash
# ~/.tmux.conf
# Cambiar prefijo a Ctrl+a
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# División de ventanas más intuitiva
bind | split-window -h
bind - split-window -v
unbind '"'
unbind %

# Navegación entre paneles con Alt+arrows
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# Habilitar mouse
set -g mouse on

# Numeración desde 1
set -g base-index 1
setw -g pane-base-index 1

# Renumeración automática
set -g renumber-windows on

# Colores para pentesting
set -g default-terminal "screen-256color"
set -g status-bg colour235
set -g status-fg colour255
set -g status-left '#[fg=colour200,bold]#S '
set -g status-right '#[fg=colour200,bold]%Y-%m-%d %H:%M'
set -g window-status-current-format '#[fg=colour200,bold]#I:#W'

# Recargar configuración
bind r source-file ~/.tmux.conf \; display "Config reloaded!"
```

---

## 🏗️ Estructura de Directorios para Pentesting

### Organización profesional:

```bash
#!/bin/bash
# setup-directories.sh

# Crear estructura de directorios
mkdir -p ~/pentesting/{
    engagements,
    tools/{custom,external,wordlists},
    resources/{cheatsheets,templates,scripts},
    learning/{courses,ctf,labs},
    reports/{templates,completed}
}

# Templates de engagement
mkdir -p ~/pentesting/engagements/template/{
    recon,
    scanning,
    exploitation,
    post-exploitation,
    reporting,
    evidence
}

# Subdirectorios específicos
mkdir -p ~/pentesting/tools/wordlists/{
    directories,
    files,
    usernames,
    passwords,
    subdomains
}

echo "📁 Estructura de directorios creada:"
tree ~/pentesting
```

### Script de inicio de engagement:

```bash
#!/bin/bash
# new-engagement.sh

if [ $# -eq 0 ]; then
    echo "Uso: $0 <nombre_cliente>"
    exit 1
fi

CLIENT=$1
DATE=$(date +%Y%m%d)
ENGAGEMENT_DIR="$HOME/pentesting/engagements/${CLIENT}_${DATE}"

echo "🎯 Creando nuevo engagement: $CLIENT"

# Crear estructura del engagement
mkdir -p "$ENGAGEMENT_DIR"/{
    01-recon/{osint,dns,subdomains,social},
    02-scanning/{ports,vulnerabilities,web},
    03-exploitation/{initial,persistence,escalation},
    04-post-exploitation/{data,lateral,persistence},
    05-reporting/{screenshots,evidence,final-report},
    notes
}

# Crear archivos iniciales
cat > "$ENGAGEMENT_DIR/notes/engagement-info.md" << EOF
# Engagement: $CLIENT
**Fecha:** $(date)
**Pentester:** $(whoami)

## Scope
- [ ] IPs/Dominios autorizados
- [ ] Restricciones de tiempo
- [ ] Puntos de contacto
- [ ] Reglas de engagement

## Checklist
### Recon
- [ ] OSINT
- [ ] DNS enumeration
- [ ] Subdomain discovery
- [ ] Google dorking

### Scanning
- [ ] Port scanning
- [ ] Service enumeration
- [ ] Vulnerability scanning
- [ ] Web application discovery

### Exploitation
- [ ] Initial access
- [ ] Privilege escalation
- [ ] Lateral movement
- [ ] Persistence

### Reporting
- [ ] Evidence collection
- [ ] Screenshots
- [ ] Final report
- [ ] Client presentation
EOF

echo "✅ Engagement $CLIENT creado en: $ENGAGEMENT_DIR"
echo "📝 Archivo de notas: $ENGAGEMENT_DIR/notes/engagement-info.md"

# Abrir en código para empezar a trabajar
code "$ENGAGEMENT_DIR" 2>/dev/null || echo "Instala VS Code para mejor experiencia"
```

---

## 🔧 Configuración de Herramientas Específicas

### Burp Suite Professional Setup:

```bash
# Configuración de Burp Suite
mkdir -p ~/.java/.userPrefs/burp

# Configurar proxy del sistema para Burp
gsettings set org.gnome.system.proxy mode 'manual'
gsettings set org.gnome.system.proxy.http host '127.0.0.1'
gsettings set org.gnome.system.proxy.http port 8080
gsettings set org.gnome.system.proxy.https host '127.0.0.1'
gsettings set org.gnome.system.proxy.https port 8080

# Script para toggle proxy
cat > ~/pentesting/tools/scripts/toggle-proxy.sh << 'EOF'
#!/bin/bash
current=$(gsettings get org.gnome.system.proxy mode)

if [ "$current" = "'manual'" ]; then
    gsettings set org.gnome.system.proxy mode 'none'
    echo "🔴 Proxy desactivado"
else
    gsettings set org.gnome.system.proxy mode 'manual'
    echo "🟢 Proxy activado (127.0.0.1:8080)"
fi
EOF

chmod +x ~/pentesting/tools/scripts/toggle-proxy.sh
```

### Metasploit Framework configuración:

```bash
# Inicializar base de datos de Metasploit
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo msfdb init

# Configuración personalizada
cat > ~/.msf4/msfconsole.rc << 'EOF'
# Auto-configuración de Metasploit
load auto_add_route
load alias
load sounds

# Aliases útiles
alias handler use exploit/multi/handler
alias p payload
alias s set
alias g get
alias r run
alias e exploit
alias q quit

# Configuración por defecto
setg LHOST 10.0.2.15  # Cambiar por tu IP
setg LPORT 4444
setg ConsoleLogging true
setg LogLevel 3
setg SessionLogging true

# Banner personalizado
banner
puts "[+] Metasploit configurado para pentesting"
puts "[+] LHOST: #{framework.datastore['LHOST']}"
puts "[+] LPORT: #{framework.datastore['LPORT']}"
EOF
```

---

## 🎯 Configuración de Laboratorio Vulnerable

### Descarga e instalación de VMs vulnerables:

```bash
#!/bin/bash
# setup-vulnerable-vms.sh

DOWNLOAD_DIR="$HOME/pentesting/learning/labs"
mkdir -p "$DOWNLOAD_DIR"
cd "$DOWNLOAD_DIR"

echo "📥 Descargando VMs vulnerables..."

# Metasploitable 2
echo "Descargando Metasploitable 2..."
wget -c https://sourceforge.net/projects/metasploitable/files/Metasploitable2/metasploitable-linux-2.0.0.zip

# DVWA setup via Docker
echo "Configurando DVWA..."
docker pull vulnerables/web-dvwa
docker run --rm -it -p 80:80 vulnerables/web-dvwa &

# VulnHub VMs populares
echo "Lista de VMs recomendadas para descargar:"
cat << 'EOF'
📋 VMs Recomendadas por Nivel:

PRINCIPIANTE:
- Metasploitable 2 (ya descargando)
- Basic Pentesting 1
- Kioptrix Level 1
- FristiLeaks 1.3

INTERMEDIO:
- Kioptrix Level 2-5
- HackLAB: Vulnix
- Tr0ll 1 & 2
- Mr. Robot

AVANZADO:
- Vulnhub: Brainpan
- Offensive Security: PWK VMs
- HackTheBox retired machines

APLICACIONES WEB:
- DVWA (Docker - ya configurando)
- bWAPP
- WebGoat
- Mutillidae
EOF

# Configurar red para laboratorio
echo "🌐 Configurando red de laboratorio..."
VBoxManage natnetwork add --netname PentestLab --network 10.0.2.0/24 --enable
VBoxManage natnetwork modify --netname PentestLab --dhcp on

echo "✅ Configuración de laboratorio completada"
echo "📌 Importa las VMs en VirtualBox y configura la red PentestLab"
```

### Script de gestión de laboratorio:

```bash
#!/bin/bash
# lab-manager.sh

VM_PATH="$HOME/pentesting/learning/labs"

case $1 in
    "start")
        echo "🚀 Iniciando laboratorio..."
        VBoxManage startvm "Metasploitable 2" --type headless
        VBoxManage startvm "Kali Linux" --type gui
        docker start dvwa 2>/dev/null || docker run -d --name dvwa -p 80:80 vulnerables/web-dvwa
        echo "✅ Laboratorio iniciado"
        ;;
    
    "stop")
        echo "🛑 Deteniendo laboratorio..."
        VBoxManage controlvm "Metasploitable 2" poweroff
        docker stop dvwa 2>/dev/null
        echo "✅ Laboratorio detenido"
        ;;
    
    "status")
        echo "📊 Estado del laboratorio:"
        VBoxManage list runningvms
        docker ps --filter name=dvwa
        ;;
    
    "reset")
        echo "🔄 Reseteando laboratorio..."
        VBoxManage snapshot "Metasploitable 2" restore "Initial State"
        docker rm -f dvwa 2>/dev/null
        docker run -d --name dvwa -p 80:80 vulnerables/web-dvwa
        echo "✅ Laboratorio reseteado"
        ;;
    
    *)
        echo "Uso: $0 {start|stop|status|reset}"
        exit 1
        ;;
esac
```

---

## 📊 Configuración de Monitoreo y Logging

### Logging centralizado para pentesting:

```bash
#!/bin/bash
# setup-logging.sh

LOG_DIR="$HOME/pentesting/logs"
mkdir -p "$LOG_DIR"/{nmap,web,exploitation,general}

# Configurar logging de terminal
cat >> ~/.zshrc << 'EOF'
# Logging automático de comandos
PROMPT_COMMAND='echo "$(date): $(history 1)" >> ~/.pentesting_history'
EOF

# Script de logging para herramientas
cat > ~/pentesting/tools/scripts/log-command.sh << 'EOF'
#!/bin/bash
# Wrapper para logging automático

TOOL=$1
shift
COMMAND="$@"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
LOG_FILE="$HOME/pentesting/logs/general/${TOOL}_${TIMESTAMP}.log"

echo "[$(date)] Ejecutando: $TOOL $COMMAND" | tee -a "$LOG_FILE"
echo "================================" | tee -a "$LOG_FILE"

# Ejecutar comando y loggear
eval "$TOOL $COMMAND" | tee -a "$LOG_FILE"

echo "✅ Log guardado en: $LOG_FILE"
EOF

chmod +x ~/pentesting/tools/scripts/log-command.sh

# Alias para logging automático
echo "alias nmap-log='~/pentesting/tools/scripts/log-command.sh nmap'" >> ~/.zshrc
echo "alias gobuster-log='~/pentesting/tools/scripts/log-command.sh gobuster'" >> ~/.zshrc
```

---

## 🎨 Personalización Visual y UX

### Configuración de escritorio para pentesting:

```bash
#!/bin/bash
# customize-desktop.sh

# Instalar tema oscuro optimizado
sudo apt install -y arc-theme papirus-icon-theme

# Configurar tema
gsettings set org.gnome.desktop.interface gtk-theme 'Arc-Dark'
gsettings set org.gnome.desktop.interface icon-theme 'Papirus-Dark'
gsettings set org.gnome.desktop.wm.preferences theme 'Arc-Dark'

# Configurar wallpaper de hacking ético
wget -O ~/Pictures/pentesting-wallpaper.jpg https://wallpapercave.com/wp/wp2757819.jpg
gsettings set org.gnome.desktop.background picture-uri "file://$HOME/Pictures/pentesting-wallpaper.jpg"

# Configurar dock con herramientas esenciales
gsettings set org.gnome.shell favorite-apps "['terminator.desktop', 'firefox-esr.desktop', 'burpsuite.desktop', 'org.gnome.Nautilus.desktop', 'code.desktop']"

# Atajos de teclado personalizados
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/', '/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom1/']"

# Ctrl+Alt+T para Terminator
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'Terminator'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command 'terminator'
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding '<Primary><Alt>t'

echo "✅ Personalización de escritorio completada"
```

---

## 🚀 Script de Configuración Todo-en-Uno

### Automatización completa:

```bash
#!/bin/bash
# ultimate-kali-setup.sh

echo "🎯 CONFIGURACIÓN PROFESIONAL DE KALI LINUX"
echo "==========================================="

# Verificar que somos root para algunas operaciones
if [[ $EUID -eq 0 ]]; then
   echo "❌ No ejecutes este script como root"
   exit 1
fi

# Fase 1: Actualización del sistema
echo "📦 Fase 1: Actualizando sistema..."
sudo apt update && sudo apt full-upgrade -y

# Fase 2: Instalación de herramientas
echo "🔧 Fase 2: Instalando herramientas esenciales..."
bash kali-setup.sh

# Fase 3: Configuración del entorno
echo "🎨 Fase 3: Configurando entorno..."
bash setup-directories.sh
bash customize-desktop.sh

# Fase 4: Configuración de herramientas
echo "⚙️ Fase 4: Configurando herramientas específicas..."
bash setup-logging.sh

# Fase 5: Descarga de laboratorio
echo "🏗️ Fase 5: Configurando laboratorio..."
bash setup-vulnerable-vms.sh

# Fase 6: Configuración final
echo "✨ Fase 6: Configuración final..."

# Instalar Oh My Zsh si no existe
if [ ! -d "$HOME/.oh-my-zsh" ]; then
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
fi

# Plugins de Zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# Crear alias útiles
cat >> ~/.zshrc << 'EOF'
# Aliases para pentesting
alias pentesting='cd ~/pentesting'
alias new-engagement='~/pentesting/tools/scripts/new-engagement.sh'
alias lab='~/pentesting/tools/scripts/lab-manager.sh'
alias proxy-toggle='~/pentesting/tools/scripts/toggle-proxy.sh'

# Funciones útiles
function target() {
    export TARGET=$1
    echo "🎯 Target establecido: $TARGET"
}

function note() {
    echo "[$(date)] $*" >> ~/pentesting/notes/quick-notes.txt
    echo "📝 Nota guardada: $*"
}
EOF

# Mensaje final
echo ""
echo "🎉 ¡CONFIGURACIÓN COMPLETADA!"
echo "==============================="
echo "✅ Kali Linux configurado para pentesting profesional"
echo "✅ Estructura de directorios creada"
echo "✅ Herramientas instaladas y configuradas"
echo "✅ Laboratorio preparado"
echo ""
echo "📌 PRÓXIMOS PASOS:"
echo "1. Reinicia la terminal para aplicar cambios"
echo "2. Ejecuta: source ~/.zshrc"
echo "3. Importa VMs vulnerables en VirtualBox"
echo "4. Configura la red PentestLab"
echo "5. Inicia tu primer engagement con: new-engagement cliente-test"
echo ""
echo "🎯 ¡Ya estás listo para hacer pentesting profesional!"
```

---

## 🎯 Verificación de Configuración

### Script de verificación:

```bash
#!/bin/bash
# verify-setup.sh

echo "🔍 VERIFICANDO CONFIGURACIÓN DE KALI LINUX"
echo "==========================================="

# Verificar herramientas esenciales
TOOLS=("nmap" "gobuster" "sqlmap" "burpsuite" "metasploit-framework" "wireshark")

for tool in "${TOOLS[@]}"; do
    if command -v $tool &> /dev/null; then
        echo "✅ $tool instalado"
    else
        echo "❌ $tool NO encontrado"
    fi
done

# Verificar estructura de directorios
if [ -d "$HOME/pentesting" ]; then
    echo "✅ Estructura de directorios OK"
    echo "📁 Directorios encontrados:"
    ls -la ~/pentesting/
else
    echo "❌ Estructura de directorios NO encontrada"
fi

# Verificar configuración de red
if VBoxManage list natnetworks | grep -q "PentestLab"; then
    echo "✅ Red de laboratorio configurada"
else
    echo "❌ Red de laboratorio NO configurada"
fi

# Verificar servicios
echo ""
echo "📊 Estado de servicios:"
systemctl status postgresql --no-pager -l
systemctl status apache2 --no-pager -l

echo ""
echo "🎯 Configuración verificada. ¡Listo para pentesting!"
```

---

## 🎯 Próximo Tutorial

En la siguiente entrega veremos:
- **OSINT y reconocimiento avanzado**
- **Técnicas de Google Dorking profesionales**
- **Automatización de reconocimiento**
- **Primer laboratorio práctico completo**

💡 **Recuerda:** Una configuración sólida es la base del éxito en pentesting. Invierte tiempo en configurar tu entorno correctamente.

---
**Andrés Nuñez - t4ifi**
