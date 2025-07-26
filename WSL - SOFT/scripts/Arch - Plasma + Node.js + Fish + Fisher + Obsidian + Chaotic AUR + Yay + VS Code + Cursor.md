```bash
#!/bin/bash

# --- Variáveis de Configuração ---
USERNAME=$(whoami) # Detecta o usuário atual
LOCALE="pt_BR.UTF-8"
KEYMAP="br-abnt2"
HOSTNAME="arch-kde-dev-wsl"

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
print_header "Iniciando Configuração Completa do Arch Linux WSL: KDE Plasma, Node.js, Fish, Fisher, Obsidian, Chaotic AUR, Yay, VS Code e Cursor"

# 1. Verificação da Distro
if ! grep -q "ID=arch" /etc/os-release; then
    print_error "Este script é para Arch Linux. Sua distro não é Arch."
    exit 1
fi

# 2. Verificação e Adição de Permissões Sudo (grupo wheel)
if ! groups "$USERNAME" | grep -q "wheel"; then
    print_warning "Usuário '$USERNAME' não está no grupo 'wheel'. Adicionando..."
    sudo usermod -aG wheel "$USERNAME"
    print_warning "Usuário '$USERNAME' adicionado ao grupo 'wheel'. Pode ser necessário reiniciar o WSL para que as permissões tenham efeito."
    print_error "Por favor, reinicie a distro WSL e execute o script novamente para garantir permissões de sudo."
    exit 1
fi
print_info "Executando como usuário: $USERNAME com permissões sudo."
sleep 2

# --- 3. Sincronizar e Atualizar o Sistema ---
print_header "3. Sincronizando e Atualizando o Sistema"
print_info "Sincronizando espelhos e atualizando todos os pacotes..."
sudo pacman -Syu --noconfirm || { print_error "Falha ao sincronizar e atualizar o sistema."; exit 1; }
print_info "Sistema Arch Linux WSL atualizado com sucesso!"

# --- 4. Configurações Essenciais do Sistema ---
print_header "4. Configurações Essenciais do Sistema"
print_info "Configurando Hostname para: $HOSTNAME"
echo "$HOSTNAME" | sudo tee /etc/hostname > /dev/null
echo "127.0.0.1 localhost" | sudo tee /etc/hosts > /dev/null
echo "::1       localhost" | sudo tee -a /etc/hosts > /dev/null
echo "127.0.1.1 $HOSTNAME.localdomain $HOSTNAME" | sudo tee -a /etc/hosts > /dev/null
print_info "Hostname e /etc/hosts configurados."

if confirm_action "Configurar Localização para $LOCALE e Keymap para $KEYMAP"; then
    print_info "Configurando Localização (Locale) para: $LOCALE e Keymap para $KEYMAP"
    sudo sed -i "/^#$LOCALE UTF-8/s/^#//" /etc/locale.gen
    sudo locale-gen
    echo "LANG=$LOCALE" | sudo tee /etc/locale.conf > /dev/null
    echo "KEYMAP=$KEYMAP" | sudo tee /etc/vconsole.conf > /dev/null
    print_info "Localização e keymap do console configurados."
fi

DEFAULT_TIMEZONE="America/Sao_Paulo"
if confirm_action "Definir Fuso Horário para $DEFAULT_TIMEZONE (Ponta Grossa, PR)"; then
    print_info "Configurando Fuso Horário para: $DEFAULT_TIMEZONE"
    sudo timedatectl set-timezone "$DEFAULT_TIMEZONE" || print_warning "Falha ao definir fuso horário."
    print_info "Fuso horário definido."
fi

print_info "Instalando cliente NTP (systemd-timesyncd)..."
sudo pacman -S --noconfirm systemd-timesyncd || print_warning "Falha ao instalar systemd-timesyncd."
sudo systemctl enable systemd-timesyncd --now || print_warning "Falha ao habilitar/iniciar systemd-timesyncd."
print_info "Cliente NTP configurado e ativado."

# --- 5. Instalar Interface Gráfica KDE Plasma ---
print_header "5. Instalação da Interface Gráfica KDE Plasma"
if confirm_action "KDE Plasma Desktop Environment e componentes essenciais"; then
    print_info "Instalando xorg-server, xorg-xinit, sddm, plasma, kde-applications, mesa..."
    # Adicionando um pouco de redundância: Se a primeira tentativa falhar, tentamos novamente
    sudo pacman -S --noconfirm xorg-server xorg-xinit sddm plasma kde-applications mesa || \
    (print_warning "Primeira tentativa de instalação do KDE falhou. Tentando novamente..." && sudo pacman -S --noconfirm xorg-server xorg-xinit sddm plasma kde-applications mesa) || \
    { print_error "Falha crítica na instalação do KDE Plasma. Verifique a conexão e os repositórios."; exit 1; }
    print_info "KDE Plasma Desktop instalado."

    print_info "Habilitando SDDM (Display Manager) para iniciar o ambiente gráfico..."
    sudo systemctl enable sddm --now || print_warning "Falha ao habilitar/iniciar SDDM. Verifique se o systemd está ativo."
    print_info "SDDM habilitado."
else
    print_info "Instalação do KDE Plasma ignorada."
fi

# Instalação do PulseAudio para áudio
print_info "Instalando PulseAudio e pavucontrol para áudio..."
sudo pacman -S --noconfirm pulseaudio pavucontrol || print_warning "Falha ao instalar PulseAudio. O áudio pode não funcionar."
print_info "PulseAudio instalado."

# --- 6. Instalar Ambiente de Desenvolvimento Node.js (via NVM) ---
print_header "6. Instalação de Ambiente de Desenvolvimento Node.js (via NVM)"
if confirm_action "Node.js via Node Version Manager (NVM)"; then
    print_info "Instalando dependências para NVM (curl)..."
    sudo pacman -S --noconfirm curl || print_warning "Falha ao instalar curl."

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

# --- 7. Instalar Shell Fish ---
print_header "7. Instalação e Configuração do Shell Fish"
if confirm_action "Shell Fish"; then
    print_info "Instalando Fish..."
    sudo pacman -S --noconfirm fish || print_error "Falha ao instalar Fish."

    print_info "Definindo Fish como shell padrão para o usuário '$USERNAME'..."
    chsh -s "$(which fish)" || print_warning "Falha ao definir Fish como shell padrão. Pode ser necessário executar 'chsh -s $(which fish)' manualmente após reiniciar o terminal."
    
    print_info "Criando diretório de configuração do Fish (~/.config/fish)..."
    mkdir -p "$HOME/.config/fish" || print_warning "Falha ao criar diretório de configuração do Fish."
    echo "set -g fish_greeting 'Bem-vindo ao Fish Shell no Arch Linux WSL!'" > "$HOME/.config/fish/config.fish"
    print_info "Fish configurado. Reinicie seu terminal para usar o Fish como shell padrão."

    # --- Nova seção: Instalação do Fisher ---
    print_header "7.1. Instalação do Fisher (Plugin Manager para Fish)"
    if confirm_action "Fisher (Plugin Manager para Fish)"; then
        print_info "Baixando e instalando Fisher..."
        # Usar o Fish para instalar Fisher, garantindo que o comando seja executado no contexto do Fish.
        # Assegura que o script Fisher seja executado com a opção --no-confirm para automação
        fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher --no-confirm" || print_error "Falha ao instalar Fisher."
        print_info "Fisher instalado com sucesso! Você pode adicionar plugins com 'fisher install <plugin>'."
    else
        print_info "Instalação do Fisher ignorada."
    fi
    # --- Fim da nova seção: Instalação do Fisher ---

else
    print_info "Instalação do Shell Fish ignorada."
fi

---
# --- 8. Configurar Chaotic AUR ---
print_header "8. Configurando o Repositório Chaotic AUR"
if confirm_action "Configurar Chaotic AUR"; then
    print_info "Instalando a chave GPG do Chaotic AUR..."
    # Instala o pacote archlinux-keyring para garantir que as chaves mais recentes estejam disponíveis
    sudo pacman -S --noconfirm archlinux-keyring || print_warning "Falha ao atualizar archlinux-keyring. Chaves podem estar desatualizadas."
    # Adiciona a chave GPG do Chaotic AUR diretamente
    sudo pacman-key --recv-key 3056513887B78AEB --keyserver hkps://keyserver.ubuntu.com || print_error "Falha ao receber a chave GPG do Chaotic AUR."
    sudo pacman-key --lsign-key 3056513887B78AEB || print_error "Falha ao assinar a chave GPG do Chaotic AUR."

    print_info "Adicionando o repositório Chaotic AUR ao /etc/pacman.conf..."
    # Verifica se o repositório já existe para evitar duplicação
    if ! grep -q "\[chaotic-aur\]" /etc/pacman.conf; then
        echo -e "\n[chaotic-aur]" | sudo tee -a /etc/pacman.conf > /dev/null
        echo "Include = /etc/pacman.d/chaotic-aur" | sudo tee -a /etc/pacman.conf > /dev/null
        # Cria o arquivo de inclusão para o Chaotic AUR com os servidores de espelho
        sudo bash -c "cat <<EOF > /etc/pacman.d/chaotic-aur
[chaotic-aur]
SigLevel = PackageRequired
Include = /etc/pacman.d/chaotic-mirrorlist
EOF"
        sudo bash -c "cat <<EOF > /etc/pacman.d/chaotic-mirrorlist
# Chaotic-AUR mirrors
Server = https://eu.mirrors.unitedwe.stand.icu/\$repo/\$arch
Server = https://repo.kitsunemimi.pw/\$repo/\$arch
Server = https://mirror.moson.org/\$repo/\$arch
Server = https://id.mirror.chaotic.cx/\$repo/\$arch
EOF"
        print_info "Repositório Chaotic AUR adicionado. Sincronizando novos repositórios..."
        sudo pacman -Syyu --noconfirm || print_warning "Falha ao sincronizar e atualizar o sistema após adicionar Chaotic AUR."
    else
        print_info "Repositório Chaotic AUR já existe no /etc/pacman.conf."
        print_info "Sincronizando repositórios para garantir que o Chaotic AUR esteja atualizado..."
        sudo pacman -Syyu --noconfirm || print_warning "Falha ao sincronizar e atualizar o sistema (Chaotic AUR já presente)."
    fi
    print_info "Chaotic AUR configurado com sucesso!"
else
    print_info "Configuração do Chaotic AUR ignorada."
fi

---

# --- 9. Instalar Yay (AUR Helper) ---
print_header "9. Instalação do Yay (AUR Helper)"
if confirm_action "Yay (AUR Helper)"; then
    print_info "Instalando dependências para Yay (git e base-devel)..."
    # base-devel inclui make, gcc, fakeroot, etc., essenciais para compilar pacotes do AUR
    sudo pacman -S --noconfirm git base-devel || { print_error "Falha ao instalar dependências para Yay."; }

    if ! command -v yay &> /dev/null; then
        print_info "Clonando o repositório do Yay..."
        git clone https://aur.archlinux.org/yay.git "$HOME/yay-build" || { print_error "Falha ao clonar o repositório do Yay."; }
        
        if [ -d "$HOME/yay-build" ]; then
            print_info "Compilando e instalando Yay..."
            (cd "$HOME/yay-build" && makepkg -si --noconfirm) || { print_error "Falha ao compilar e instalar Yay."; }
            print_info "Limpando diretório de build do Yay..."
            rm -rf "$HOME/yay-build"
        else
            print_error "Diretório de build do Yay não encontrado. Instalação ignorada."
        fi
    else
        print_info "Yay já está instalado."
    fi

    if command -v yay &> /dev/null; then
        print_info "Yay instalado com sucesso!"
    else
        print_error "Yay não foi instalado corretamente. VS Code e Cursor não serão instalados via AUR."
    fi
else
    print_info "Instalação do Yay ignorada. VS Code e Cursor não serão instalados via AUR."
fi

# --- 10. Instalar VS Code (visual-studio-code-bin do AUR) ---
print_header "10. Instalação do Visual Studio Code (via Yay)"
if command -v yay &> /dev/null; then
    if confirm_action "Visual Studio Code (visual-studio-code-bin do AUR)"; then
        print_info "Instalando visual-studio-code-bin via Yay..."
        # yay --noconfirm tenta automatizar as confirmações
        yay -S --noconfirm visual-studio-code-bin || print_error "Falha ao instalar Visual Studio Code via Yay."
        print_info "Visual Studio Code instalado. Pode ser necessário reiniciar o ambiente gráfico para que o atalho apareça."
    else
        print_info "Instalação do Visual Studio Code ignorada."
    fi
else
    print_warning "Yay não está instalado. Não é possível instalar Visual Studio Code via AUR."
fi

# --- 11. Instalar Cursor (cursor-bin do AUR) ---
print_header "11. Instalação do Cursor (via Yay)"
if command -v yay &> /dev/null; then
    if confirm_action "Cursor (cursor-bin do AUR)"; then
        print_info "Instalando cursor-bin via Yay..."
        yay -S --noconfirm cursor-bin || print_error "Falha ao instalar Cursor via Yay."
        print_info "Cursor instalado. Pode ser necessário reiniciar o ambiente gráfico para que o atalho apareça."
    else
        print_info "Instalação do Cursor ignorada."
    fi
else
    print_warning "Yay não está instalado. Não é possível instalar Cursor via AUR."
fi

# --- 12. Instalar Obsidian ---
print_header "12. Instalação do Obsidian"
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
        # Isso geralmente requer o comando `xdg-utils` e um display manager funcionando
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

# --- 13. Configuração do X Server para WSL ---
print_header "13. Configuração do X Server para WSL"
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

# --- 14. Limpeza Final ---
print_header "14. Limpeza Final"
print_info "Limpando cache do Pacman e removendo pacotes órfãos..."
sudo pacman -Sc --noconfirm || print_warning "Falha ao limpar cache do Pacman."
sudo pacman -Qtdq | sudo pacman -Rns --noconfirm || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
print_info "Limpeza concluída."

print_header "Processo de Configuração de Arch Linux WSL Concluído!"
print_warning "Para aplicar todas as mudanças (GUI, Fish como shell padrão, NVM, novos editores), é **fortemente recomendado reiniciar sua instância Arch Linux WSL**."
print_warning "Feche todos os terminais Arch WSL e, no PowerShell do Windows, execute 'wsl --shutdown'."
print_warning "Em seguida, reabra sua instância Arch Linux ('wsl -d ArchLinuxWSL')."
print_warning "Após reiniciar, você deverá ver a tela de login do KDE Plasma na janela do seu X Server (VcXsrv)."
print_warning "Para usar o Atuin, siga as instruções pós-instalação dele para configurar a sincronização."
echo -e "\n${BLUE}========================================${NC}\n${BLUE}Script Finalizado!${NC}\n${BLUE}========================================${NC}\n"
```