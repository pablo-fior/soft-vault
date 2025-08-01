---
state: "[[Volte aqui]]"
---
```bash
#!/bin/bash

# --- Variáveis de Configuração ---
USERNAME=$(whoami) # Detecta o usuário atual
LOCALE="pt_BR.UTF-8"
KEYMAP="br-abnt2"    # Layout do teclado do console (menos relevante para DEs em WSL)
HOSTNAME="ubuntu-wsl-dev" # Nome base do hostname, será complementado pelo DE

# Cores para Saída do Script
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m'

# --- Funções Auxiliares ---
print_header() { echo -e "\n${BLUE}========================================${NC}\n${BLUE}>>> $1 <<<${NC}\n${BLUE}========================================${NC}\n"; }
print_info() { echo -e "${GREEN}INFO: $1${NC}"; }
print_warning() { echo -e "${YELLOW}AVISO: $1${NC}"; }
print_error() { echo -e "${RED}ERRO: $1${NC}"; }
confirm_action() { read -p "$(echo -e "${YELLOW}Deseja continuar com a ação '$1'? (s/N) ${NC}")" choice; [[ "$choice" = [sS] ]]; }

# --- Início do Script ---
print_header "Iniciando Configuração Completa do Ubuntu WSL: DE de Escolha (XFCE4/GNOME/KDE Plasma), Node.js, Fish, Fisher, Obsidian, VS Code e Cursor"

# 1. Verificação da Distro
if ! grep -q "ID=ubuntu" /etc/os-release; then
    print_error "Este script é para Ubuntu Linux. Sua distro não é Ubuntu."
    exit 1
fi

# 2. Verificação de Permissões Sudo
if ! groups "$USERNAME" | grep -q "sudo"; then
    print_warning "Usuário '$USERNAME' não está no grupo 'sudo'. Este script requer permissões sudo."
    print_error "Por favor, adicione seu usuário ao grupo 'sudo' (sudo usermod -aG sudo $USERNAME) e reinicie o WSL antes de executar este script."
    exit 1
fi
print_info "Executando como usuário: $USERNAME com permissões sudo."
sleep 2

# --- 3. Sincronizar e Atualizar o Sistema ---
print_header "3. Atualizando o Sistema"
print_info "Sincronizando repositórios e atualizando todos os pacotes..."
sudo apt update -y && sudo apt upgrade -y || { print_error "Falha ao atualizar o sistema."; exit 1; }
print_info "Sistema Ubuntu WSL atualizado com sucesso!"

# --- 4. Configurações Essenciais do Sistema ---
print_header "4. Configurações Essenciais do Sistema"

# 4.1. Configuração do Hostname
DE_CHOICE=""
while [[ ! "$DE_CHOICE" =~ ^(1|2|3)$ ]]; do
    read -p "$(echo -e "${YELLOW}Escolha o Ambiente de Desktop a ser instalado:\n  1) XFCE4 (Leve e Rápido)\n  2) GNOME (Moderno e Completo)\n  3) KDE Plasma (Altamente Customizável)\nSua escolha (1/2/3): ${NC}")" DE_CHOICE
    case $DE_CHOICE in
        1) HOSTNAME="${HOSTNAME}-xfce"; DE_NAME="XFCE4"; PACKAGE="xfce4 xfce4-goodies lightdm" ;;
        2) HOSTNAME="${HOSTNAME}-gnome"; DE_NAME="GNOME"; PACKAGE="ubuntu-desktop gdm3" ;;
        3) HOSTNAME="${HOSTNAME}-plasma"; DE_NAME="KDE Plasma"; PACKAGE="kde-plasma-desktop sddm" ;;
        *) print_error "Escolha inválida. Por favor, digite 1, 2 ou 3." ;;
    esac
done

print_info "Configurando Hostname para: $HOSTNAME"
echo "$HOSTNAME" | sudo tee /etc/hostname > /dev/null
echo "127.0.0.1 localhost" | sudo tee /etc/hosts > /dev/null
echo "::1       localhost" | sudo tee -a /etc/hosts > /dev/null
echo "127.0.1.1 $HOSTNAME.localdomain $HOSTNAME" | sudo tee -a /etc/hosts > /dev/null
print_info "Hostname e /etc/hosts configurados."

if confirm_action "Configurar Localização para $LOCALE"; then
    print_info "Configurando Localização (Locale) para: $LOCALE"
    sudo apt install -y language-pack-pt-br || print_warning "Falha ao instalar pacote de idioma."
    sudo locale-gen "$LOCALE" || print_warning "Falha ao gerar locale."
    sudo update-locale LANG="$LOCALE" || print_warning "Falha ao definir locale padrão."
    print_info "Localização configurada."
fi

DEFAULT_TIMEZONE="America/Sao_Paulo"
if confirm_action "Definir Fuso Horário para $DEFAULT_TIMEZONE (Ponta Grossa, PR)"; then
    print_info "Configurando Fuso Horário para: $DEFAULT_TIMEZONE"
    sudo timedatectl set-timezone "$DEFAULT_TIMEZONE" || print_warning "Falha ao definir fuso horário."
    print_info "Fuso horário definido."
fi

print_info "Instalando cliente NTP (systemd-timesyncd)..."
sudo apt install -y systemd-timesyncd || print_warning "Falha ao instalar systemd-timesyncd."
sudo systemctl enable systemd-timesyncd --now || print_warning "Falha ao habilitar/iniciar systemd-timesyncd."
print_info "Cliente NTP configurado e ativado."

# --- 5. Instalar Interface Gráfica Escolhida ---
print_header "5. Instalação da Interface Gráfica: $DE_NAME"
if confirm_action "$DE_NAME Desktop Environment e componentes essenciais"; then
    print_info "Instalando xserver-xorg e $DE_NAME com Display Manager..."
    sudo apt install -y xserver-xorg "$PACKAGE" || \
    { print_error "Falha crítica na instalação do $DE_NAME Desktop. Verifique a conexão e os repositórios."; exit 1; }
    print_info "$DE_NAME Desktop instalado."

    # Configura o Display Manager escolhido
    if [ "$DE_CHOICE" == "1" ]; then # XFCE4 com LightDM
        print_info "Configurando LightDM como Display Manager padrão (se não for)."
        sudo dpkg-reconfigure lightdm # Força a reconfiguração para garantir que seja o padrão
        sudo systemctl enable lightdm --now || print_warning "Falha ao habilitar/iniciar LightDM. Verifique se o systemd está ativo."
        print_info "LightDM habilitado."
    elif [ "$DE_CHOICE" == "2" ]; then # GNOME com GDM3
        print_info "Configurando GDM3 como Display Manager padrão (se não for)."
        sudo dpkg-reconfigure gdm3 # Força a reconfiguração para garantir que seja o padrão
        sudo systemctl enable gdm3 --now || print_warning "Falha ao habilitar/iniciar GDM3. Verifique se o systemd está ativo."
        print_info "GDM3 habilitado."
    elif [ "$DE_CHOICE" == "3" ]; then # KDE Plasma com SDDM
        print_info "Configurando SDDM como Display Manager padrão (se não for)."
        sudo dpkg-reconfigure sddm # Força a reconfiguração para garantir que seja o padrão
        sudo systemctl enable sddm --now || print_warning "Falha ao habilitar/iniciar SDDM. Verifique se o systemd está ativo."
        print_info "SDDM habilitado."
    fi
else
    print_info "Instalação da Interface Gráfica $DE_NAME ignorada."
fi

# Instalação do PulseAudio para áudio
print_info "Instalando PulseAudio e pavucontrol para áudio..."
sudo apt install -y pulseaudio pavucontrol || print_warning "Falha ao instalar PulseAudio. O áudio pode não funcionar."
print_info "PulseAudio instalado."

# --- 6. Instalar Ambiente de Desenvolvimento Node.js (via NVM) ---
print_header "6. Instalação de Ambiente de Desenvolvimento Node.js (via NVM)"
if confirm_action "Node.js via Node Version Manager (NVM)"; then
    print_info "Instalando dependências para NVM (curl, build-essential)..."
    # build-essential é importante para compilar módulos nativos do Node.js
    sudo apt install -y curl build-essential || print_warning "Falha ao instalar curl/build-essential."

    print_info "Baixando e instalando NVM..."
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash || print_error "Falha ao instalar NVM."
    
    # Carregar NVM no shell atual para uso imediato
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
    
    if command -v nvm &> /dev/null; then
        print_info "NVM instalado com sucesso. Instalando a versão LTS mais recente do Node.js..."
        nvm install --lts || print_error "Falha ao instalar a versão LTS do Node.js via NVM."
        nvm use --lts || print_warning "Falha ao definir a versão LTS como padrão."
        nvm alias default 'lts/*' || print_warning "Falha ao definir alias default para LTS."
        print_info "Node.js LTS e NPM instalados."
    else
        print_error "NVM não foi instalado corretamente. Node.js não será configurado."
    fi
    
    print_info "Adicionando configuração do NVM ao ~/.bashrc e ~/.zshrc (se existirem)..."
    echo 'export NVM_DIR="$HOME/.nvm"' >> "$HOME/.bashrc"
    echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm' >> "$HOME/.bashrc"
    echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion' >> "$HOME/.bashrc"
    
    if [ -f "$HOME/.zshrc" ]; then
        echo 'export NVM_DIR="$HOME/.nvm"' >> "$HOME/.zshrc"
        echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm' >> "$HOME/.zshrc"
        echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion' >> "$HOME/.zshrc"
    fi
else
    print_info "Instalação de Node.js via NVM ignorada."
fi

# --- 7. Instalar Shell Fish e Fisher ---
print_header "7. Instalação e Configuração do Shell Fish e Fisher"
if confirm_action "Shell Fish e Fisher"; then
    print_info "Instalando Fish..."
    sudo apt install -y fish || print_error "Falha ao instalar Fish."

    print_info "Definindo Fish como shell padrão para o usuário '$USERNAME'..."
    chsh -s "$(which fish)" || print_warning "Falha ao definir Fish como shell padrão. Pode ser necessário executar 'chsh -s $(which fish)' manualmente após reiniciar o terminal."
    
    print_info "Criando diretório de configuração do Fish (~/.config/fish)..."
    mkdir -p "$HOME/.config/fish" || print_warning "Falha ao criar diretório de configuração do Fish."
    echo "set -g fish_greeting 'Bem-vindo ao Fish Shell no Ubuntu Linux WSL!'" > "$HOME/.config/fish/config.fish"
    print_info "Fish configurado. Reinicie seu terminal para usar o Fish como shell padrão."

    # Instalação do Fisher
    print_header "7.1. Instalação do Fisher (Plugin Manager para Fish)"
    print_info "Baixando e instalando Fisher..."
    fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher --no-confirm" || print_error "Falha ao instalar Fisher."
    print_info "Fisher instalado com sucesso! Você pode adicionar plugins com 'fisher install <plugin>'."
else
    print_info "Instalação do Shell Fish e Fisher ignorada."
fi

# --- 8. Instalar VS Code ---
print_header "8. Instalação do Visual Studio Code"
if confirm_action "Visual Studio Code"; then
    print_info "Adicionando a chave GPG e o repositório oficial do VS Code..."
    sudo apt install -y wget apt-transport-https || print_warning "Falha ao instalar wget/apt-transport-https."
    wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null || print_warning "Falha ao importar chave GPG do VS Code."
    echo "deb [arch=amd64,arm64,armhf] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list > /dev/null || print_error "Falha ao adicionar repositório do VS Code."
    
    print_info "Atualizando a lista de pacotes e instalando Visual Studio Code..."
    sudo apt update -y || print_warning "Falha ao atualizar lista de pacotes após adicionar repositório do VS Code."
    sudo apt install -y code || print_error "Falha ao instalar Visual Studio Code."
    print_info "Visual Studio Code instalado."
else
    print_info "Instalação do Visual Studio Code ignorada."
fi

# --- 9. Instalar Cursor ---
print_header "9. Instalação do Cursor"
if confirm_action "Cursor"; then
    print_info "Baixando o pacote DEB do Cursor..."
    # A URL para a última versão do Cursor pode variar.
    # [Inferência] É provável que o Cursor siga um padrão de URL para as releases.
    # [Não Verificado] A URL específica para a última versão estável pode variar.
    # A URL abaixo tenta encontrar o pacote mais recente para amd64 diretamente da página de releases.
    # Adaptei para tentar pegar o link direto para o .deb da página de releases.
    CURSOR_RELEASE_URL="https://releases.cursor.sh/linux/deb/x86_64"
    LATEST_CURSOR_DEB=$(curl -s $CURSOR_RELEASE_URL | grep -oP 'cursor-\d+\.\d+\.\d+[^"]*\.deb' | head -n 1)

    if [ -n "$LATEST_CURSOR_DEB" ]; then
        wget -O "$HOME/cursor.deb" "${CURSOR_RELEASE_URL}/${LATEST_CURSOR_DEB}" || \
        (print_warning "Primeira tentativa de download do Cursor falhou. Tentando novamente..." && wget -O "$HOME/cursor.deb" "${CURSOR_RELEASE_URL}/${LATEST_CURSOR_DEB}") || \
        { print_error "Falha ao baixar Cursor DEB. Verifique a URL ou sua conexão."; }

        if [ -f "$HOME/cursor.deb" ]; then
            print_info "Instalando Cursor DEB..."
            sudo apt install -y "$HOME/cursor.deb" || print_error "Falha ao instalar Cursor via APT."
            print_info "Limpando pacote DEB do Cursor..."
            rm "$HOME/cursor.deb"
            print_info "Cursor instalado."
        else
            print_error "Pacote DEB do Cursor não foi baixado. Instalação ignorada."
        fi
    else
        print_error "Não foi possível encontrar a URL do pacote DEB mais recente do Cursor. Instalação ignorada."
    fi
else
    print_info "Instalação do Cursor ignorada."
fi

# --- 10. Instalar Obsidian ---
print_header "10. Instalação do Obsidian"
if confirm_action "Obsidian.md (AppImage)"; then
    print_info "Baixando Obsidian AppImage..."
    OBSIDIAN_URL=$(curl -s https://api.github.com/repos/obsidianmd/obsidian-releases/releases/latest | grep "browser_download_url.*AppImage" | cut -d : -f 2- | tr -d \" | tr -d " ")
    OBSIDIAN_FILE="Obsidian.AppImage"
    wget -O "$HOME/$OBSIDIAN_FILE" "$OBSIDIAN_URL" || \
    (print_warning "Primeira tentativa de download do Obsidian falhou. Tentando novamente..." && wget -O "$HOME/$OBSIDIAN_FILE" "$OBSIDIAN_URL") || \
    { print_error "Falha ao baixar Obsidian AppImage. Verifique a URL ou sua conexão."; }
    
    if [ -f "$HOME/$OBSIDIAN_FILE" ]; then
        print_info "Tornando Obsidian AppImage executável..."
        chmod +x "$HOME/$OBSIDIAN_FILE" || print_warning "Falha ao tornar AppImage executável."
        print_info "Obsidian baixado para $HOME/$OBSIDIAN_FILE. Você pode executá-lo diretamente."
        print_info "Criando atalho de desktop e entradas de menu para Obsidian..."
        echo "[Desktop Entry]" > "$HOME/.local/share/applications/obsidian.desktop"
        echo "Name=Obsidian" >> "$HOME/.local/share/applications/obsidian.desktop"
        echo "Exec=$HOME/$OBSIDIAN_FILE" >> "$HOME/.local/share/applications/obsidian.desktop"
        echo "Icon=obsidian" >> "$HOME/.local/share/applications/obsidian.desktop" # Assume um ícone padrão ou que o AppImage o forneça
        echo "Type=Application" >> "$HOME/.local/share/applications/obsidian.desktop"
        echo "Categories=Office;Utility;" >> "$HOME/.local/share/applications/obsidian.desktop"
        
        # Tenta atualizar o banco de dados de aplicativos
        sudo update-desktop-database || print_warning "Falha ao atualizar banco de dados de desktop. O atalho pode não aparecer."
        print_info "Atalho de desktop para Obsidian criado em ~/.local/share/applications/obsidian.desktop."
    else
        print_error "Obsidian AppImage não foi baixado. Instalação ignorada."
    fi
else
    print_info "Instalação do Obsidian ignorada."
fi

# --- 11. Configuração do X Server para WSL ---
print_header "11. Configuração do X Server para WSL"
WINDOWS_HOST_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}' 2>/dev/null)

if [ -n "$WINDOWS_HOST_IP" ]; then
    print_info "Detectado IP do host Windows: $WINDOWS_HOST_IP. Adicionando configuração de DISPLAY ao ~/.bashrc e ~/.config/fish/config.fish..."
    echo "export DISPLAY=$WINDOWS_HOST_IP:0.0" >> "$HOME/.bashrc"
    echo "export LIBGL_ALWAYS_INDIRECT=1" >> "$HOME/.bashrc"
    echo "export PULSE_SERVER=tcp:$WINDOWS_HOST_IP" >> "$HOME/.bashrc"
    
    # Para Fish shell
    echo "set -gx DISPLAY \"$WINDOWS_HOST_IP:0.0\"" >> "$HOME/.config/fish/config.fish"
    echo "set -gx LIBGL_ALWAYS_INDIRECT \"1\"" >> "$HOME/.config/fish/config.fish"
    echo "set -gx PULSE_SERVER \"tcp:$WINDOWS_HOST_IP\"" >> "$HOME/.config/fish/config.fish"

    print_info "Variáveis DISPLAY, LIBGL_ALWAYS_INDIRECT e PULSE_SERVER configuradas."
    print_warning "Lembre-se: Você DEVE ter um X Server (VcXsrv) e um servidor PulseAudio rodando no Windows."
else
    print_error "Não foi possível detectar o IP do host Windows. As variáveis DISPLAY e PULSE_SERVER podem precisar ser configuradas manualmente."
fi

# --- 12. Limpeza Final ---
print_header "12. Limpeza Final"
print_info "Limpando cache do APT e removendo pacotes órfãos..."
sudo apt clean -y || print_warning "Falha ao limpar cache do APT."
sudo apt autoremove -y || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
print_info "Limpeza concluída."

print_header "Processo de Configuração de Ubuntu WSL Concluído!"
print_warning "Para aplicar todas as mudanças (GUI, Fish como shell padrão, NVM, novos editores), é **fortemente recomendado reiniciar sua instância Ubuntu WSL**."
print_warning "Feche todos os terminais Ubuntu WSL e, no PowerShell do Windows, execute 'wsl --shutdown'."
print_warning "Em seguida, reabra sua instância Ubuntu ('wsl -d Ubuntu')."
print_warning "Após reiniciar, você deverá ver a tela de login do $DE_NAME na janela do seu X Server (VcXsrv)."
print_warning "Para usar o Atuin, siga as instruções pós-instalação dele para configurar a sincronização."
echo -e "\n${BLUE}========================================${NC}\n${BLUE}Script Finalizado!${NC}\n${BLUE}========================================${NC}\n"
```