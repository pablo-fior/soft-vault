---
state: "[[Volte aqui]]"
---
```bash
#!/bin/bash

# --- Variáveis de Configuração ---
USERNAME=$(whoami)
LOCALE="pt_BR.UTF-8"
KEYMAP="br-abnt2"
HOSTNAME="arch-kde-dev-wsl"

# Arquivo de estado para controlar o que já foi executado
STATE_FILE="$HOME/.arch_wsl_setup_state"

# Cores para Saída do Script
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
RED='\033[0;31m'
CYAN='\033[0;36m'
NC='\033[0m'

# --- Funções Auxiliares ---
print_header() { echo -e "\n${BLUE}========================================${NC}\n${BLUE}>>> $1 <<<${NC}\n${BLUE}========================================${NC}\n"; }
print_info() { echo -e "${GREEN}INFO: $1${NC}"; }
print_warning() { echo -e "${YELLOW}AVISO: $1${NC}"; }
print_error() { echo -e "${RED}ERRO: $1${NC}"; }
print_skip() { echo -e "${CYAN}SKIP: $1 (já executado)${NC}"; }
confirm_action() { read -p "$(echo -e "${YELLOW}Deseja continuar com a ação '$1'? (s/N) ${NC}")" choice; [[ "$choice" = [sS] ]]; }

# --- Funções de Estado ---
init_state_file() {
    if [ ! -f "$STATE_FILE" ]; then
        touch "$STATE_FILE"
        print_info "Arquivo de estado criado: $STATE_FILE"
    fi
}

mark_completed() {
    echo "$1=completed" >> "$STATE_FILE"
}

is_completed() {
    grep -q "^$1=completed$" "$STATE_FILE" 2>/dev/null
}

show_status() {
    echo -e "\n${CYAN}=== STATUS ATUAL ===${NC}"
    is_completed "system_update" && echo -e "${GREEN}✓${NC} Sistema atualizado" || echo -e "${RED}✗${NC} Sistema não atualizado"
    is_completed "locale_config" && echo -e "${GREEN}✓${NC} Locale configurado" || echo -e "${RED}✗${NC} Locale não configurado"
    is_completed "kde_install" && echo -e "${GREEN}✓${NC} KDE Plasma instalado" || echo -e "${RED}✗${NC} KDE Plasma não instalado"
    is_completed "nvm_install" && echo -e "${GREEN}✓${NC} Node.js (NVM) instalado" || echo -e "${RED}✗${NC} Node.js (NVM) não instalado"
    is_completed "fish_install" && echo -e "${GREEN}✓${NC} Fish Shell instalado" || echo -e "${RED}✗${NC} Fish Shell não instalado"
    is_completed "fisher_install" && echo -e "${GREEN}✓${NC} Fisher instalado" || echo -e "${RED}✗${NC} Fisher não instalado"
    is_completed "chaotic_aur" && echo -e "${GREEN}✓${NC} Chaotic AUR configurado" || echo -e "${RED}✗${NC} Chaotic AUR não configurado"
    is_completed "yay_install" && echo -e "${GREEN}✓${NC} Yay instalado" || echo -e "${RED}✗${NC} Yay não instalado"
    is_completed "vscode_install" && echo -e "${GREEN}✓${NC} VS Code instalado" || echo -e "${RED}✗${NC} VS Code não instalado"
    is_completed "cursor_install" && echo -e "${GREEN}✓${NC} Cursor instalado" || echo -e "${RED}✗${NC} Cursor não instalado"
    is_completed "obsidian_install" && echo -e "${GREEN}✓${NC} Obsidian instalado" || echo -e "${RED}✗${NC} Obsidian não instalado"
    is_completed "x11_config" && echo -e "${GREEN}✓${NC} X11 configurado" || echo -e "${RED}✗${NC} X11 não configurado"
    is_completed "utils_install" && echo -e "${GREEN}✓${NC} Utilitários instalados" || echo -e "${RED}✗${NC} Utilitários não instalados"
    echo -e "${CYAN}===================${NC}\n"
}

reset_state() {
    if [ -f "$STATE_FILE" ]; then
        rm "$STATE_FILE"
        print_info "Estado resetado. Todas as verificações serão executadas novamente."
    fi
}

# --- Verificações Específicas ---
check_kde_installed() {
    pacman -Qq plasma &>/dev/null && pacman -Qq sddm &>/dev/null
}

check_nvm_installed() {
    [ -s "$HOME/.nvm/nvm.sh" ] && [ -d "$HOME/.nvm" ]
}

check_fish_installed() {
    command -v fish &>/dev/null && [ "$(getent passwd "$USER" | cut -d: -f7)" = "$(which fish)" ]
}

check_fisher_installed() {
    fish -c "functions -q fisher" &>/dev/null
}

check_chaotic_aur_configured() {
    grep -q "\[chaotic-aur\]" /etc/pacman.conf
}

check_yay_installed() {
    command -v yay &>/dev/null
}

check_vscode_installed() {
    command -v code &>/dev/null || pacman -Qq visual-studio-code-bin &>/dev/null
}

check_cursor_installed() {
    command -v cursor &>/dev/null || pacman -Qq cursor-bin &>/dev/null
}

check_obsidian_installed() {
    command -v obsidian &>/dev/null || [ -f "$HOME/.local/bin/Obsidian.AppImage" ] || pacman -Qq obsidian &>/dev/null
}

check_x11_configured() {
    grep -q "DISPLAY.*:" "$HOME/.bashrc" &>/dev/null || grep -q "DISPLAY.*:" "$HOME/.config/fish/config.fish" &>/dev/null
}

check_locale_configured() {
    [ -f /etc/locale.conf ] && grep -q "$LOCALE" /etc/locale.conf
}

check_utils_installed() {
    command -v htop &>/dev/null && command -v neofetch &>/dev/null && command -v tree &>/dev/null
}

# --- Início do Script ---
print_header "Configurador Inteligente do Arch Linux WSL"
print_info "Este script verifica o que já foi instalado antes de executar qualquer ação."

# Inicializar arquivo de estado
init_state_file

# Verificar argumentos da linha de comando
case "$1" in
    --status)
        show_status
        exit 0
        ;;
    --reset)
        reset_state
        exit 0
        ;;
    --help)
        echo "Uso: $0 [opções]"
        echo "Opções:"
        echo "  --status    Mostra o status atual das instalações"
        echo "  --reset     Remove o arquivo de estado (força reexecução)"
        echo "  --help      Mostra esta ajuda"
        exit 0
        ;;
esac

# Mostrar status atual
show_status

# 1. Verificação da Distro
if ! grep -q "ID=arch" /etc/os-release; then
    print_error "Este script é para Arch Linux. Sua distro não é Arch."
    exit 1
fi

# 2. Verificação se está rodando no WSL
if ! grep -qE "(microsoft|WSL)" /proc/version; then
    print_warning "Este script foi projetado para WSL. Continuando mesmo assim..."
fi

# --- 3. Sincronizar e Atualizar o Sistema ---
print_header "3. Sincronizando e Atualizando o Sistema"
if is_completed "system_update"; then
    print_skip "Sistema já foi atualizado"
else
    print_info "Sincronizando espelhos e atualizando todos os pacotes..."
    if sudo pacman -Syu --noconfirm; then
        mark_completed "system_update"
        print_info "Sistema Arch Linux WSL atualizado com sucesso!"
    else
        print_error "Falha ao sincronizar e atualizar o sistema."
        exit 1
    fi
fi

# --- 4. Configurar Locale ---
print_header "4. Configurando Locale"
if is_completed "locale_config" || check_locale_configured; then
    print_skip "Locale já está configurado"
    is_completed "locale_config" || mark_completed "locale_config"
else
    if confirm_action "Configurar locale $LOCALE"; then
        print_info "Descomentando $LOCALE no /etc/locale.gen..."
        sudo sed -i "s/#$LOCALE/$LOCALE/" /etc/locale.gen
        if sudo locale-gen && echo "LANG=$LOCALE" | sudo tee /etc/locale.conf > /dev/null; then
            mark_completed "locale_config"
            print_info "Locale $LOCALE configurado."
        else
            print_warning "Falha ao configurar locale."
        fi
    else
        print_info "Configuração de locale ignorada."
    fi
fi

# --- 5. Instalar Interface Gráfica KDE Plasma ---
print_header "5. Instalação da Interface Gráfica KDE Plasma"
if is_completed "kde_install" || check_kde_installed; then
    print_skip "KDE Plasma já está instalado"
    is_completed "kde_install" || mark_completed "kde_install"
else
    if confirm_action "KDE Plasma Desktop Environment e componentes essenciais"; then
        print_info "Instalando xorg-server, xorg-xinit, sddm, plasma, kde-applications..."
        
        if sudo pacman -S --noconfirm xorg-server xorg-xinit && \
           sudo pacman -S --noconfirm sddm && \
           sudo pacman -S --noconfirm plasma; then
            
            sudo pacman -S --noconfirm kde-applications || print_warning "Falha ao instalar kde-applications. Continuando..."
            sudo systemctl enable sddm || print_warning "Falha ao habilitar SDDM. No WSL, isso é esperado."
            
            mark_completed "kde_install"
            print_info "KDE Plasma Desktop instalado."
        else
            print_error "Falha crítica na instalação do KDE Plasma."
            exit 1
        fi
    else
        print_info "Instalação do KDE Plasma ignorada."
    fi
fi

# --- 6. Instalar Ambiente de Desenvolvimento Node.js (via NVM) ---
print_header "6. Instalação de Ambiente de Desenvolvimento Node.js (via NVM)"
if is_completed "nvm_install" || check_nvm_installed; then
    print_skip "NVM/Node.js já está instalado"
    is_completed "nvm_install" || mark_completed "nvm_install"
    
    # Mostrar versão atual se disponível
    if check_nvm_installed; then
        export NVM_DIR="$HOME/.nvm"
        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
        if command -v node &>/dev/null; then
            print_info "Node.js versão atual: $(node --version)"
        fi
    fi
else
    if confirm_action "Node.js via Node Version Manager (NVM)"; then
        print_info "Instalando dependências para NVM (curl)..."
        sudo pacman -S --noconfirm curl || print_warning "Falha ao instalar curl."

        print_info "Baixando e instalando NVM..."
        if curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash; then
            # Carregar NVM no shell atual
            export NVM_DIR="$HOME/.nvm"
            [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
            [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
            
            if command -v nvm &>/dev/null; then
                print_info "Instalando Node.js LTS..."
                if nvm install --lts && nvm use --lts && nvm alias default 'lts/*'; then
                    print_info "Node.js versão: $(node --version)"
                    print_info "NPM versão: $(npm --version)"
                    
                    # Configurar shells
                    if ! grep -q "NVM_DIR" "$HOME/.bashrc" 2>/dev/null; then
                        {
                            echo 'export NVM_DIR="$HOME/.nvm"'
                            echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"'
                            echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"'
                        } >> "$HOME/.bashrc"
                    fi
                    
                    if [ -f "$HOME/.zshrc" ] && ! grep -q "NVM_DIR" "$HOME/.zshrc"; then
                        {
                            echo 'export NVM_DIR="$HOME/.nvm"'
                            echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"'
                            echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"'
                        } >> "$HOME/.zshrc"
                    fi
                    
                    mark_completed "nvm_install"
                    print_info "NVM e Node.js instalados com sucesso!"
                else
                    print_error "Falha ao instalar Node.js via NVM."
                fi
            else
                print_error "NVM não foi instalado corretamente."
            fi
        else
            print_error "Falha ao baixar/instalar NVM."
        fi
    else
        print_info "Instalação de Node.js via NVM ignorada."
    fi
fi

# --- 7. Instalar Shell Fish ---
print_header "7. Instalação e Configuração do Shell Fish"
if is_completed "fish_install" || check_fish_installed; then
    print_skip "Fish Shell já está instalado e configurado como padrão"
    is_completed "fish_install" || mark_completed "fish_install"
else
    if confirm_action "Shell Fish"; then
        print_info "Instalando Fish..."
        if sudo pacman -S --noconfirm fish; then
            print_info "Definindo Fish como shell padrão..."
            if sudo chsh -s "$(which fish)" "$USERNAME"; then
                print_info "Criando configuração do Fish..."
                mkdir -p "$HOME/.config/fish"
                
                cat > "$HOME/.config/fish/config.fish" << 'EOF'
# Configuração básica do Fish Shell
set -g fish_greeting 'Bem-vindo ao Fish Shell no Arch Linux WSL!'

# NVM para Fish (requer bass plugin)
if test -d ~/.nvm
    function nvm
        bass source ~/.nvm/nvm.sh --no-use ';' nvm $argv
    end
    
    # Carregar node automaticamente
    if test -f ~/.nvm/alias/default
        nvm use default --silent
    end
end
EOF
                mark_completed "fish_install"
                print_info "Fish Shell instalado e configurado."
            else
                print_warning "Falha ao definir Fish como shell padrão."
            fi
        else
            print_error "Falha ao instalar Fish Shell."
        fi
    else
        print_info "Instalação do Fish Shell ignorada."
    fi
fi

# --- 7.1. Instalação do Fisher ---
print_header "7.1. Instalação do Fisher (Plugin Manager para Fish)"
if is_completed "fisher_install" || check_fisher_installed; then
    print_skip "Fisher já está instalado"
    is_completed "fisher_install" || mark_completed "fisher_install"
else
    if command -v fish &>/dev/null; then
        if confirm_action "Fisher (Plugin Manager para Fish)"; then
            print_info "Instalando Fisher..."
            if fish -c "curl -sL https://raw.githubusercontent.com/jorgebucaran/fisher/main/functions/fisher.fish | source && fisher install jorgebucaran/fisher"; then
                # Instalar plugin útil para NVM
                fish -c "fisher install edc/bass" || print_warning "Falha ao instalar plugin bass."
                mark_completed "fisher_install"
                print_info "Fisher instalado com sucesso!"
            else
                print_error "Falha ao instalar Fisher."
            fi
        else
            print_info "Instalação do Fisher ignorada."
        fi
    else
        print_warning "Fish não está instalado. Fisher será ignorado."
    fi
fi

# --- 8. Configurar Chaotic AUR ---
print_header "8. Configurando o Repositório Chaotic AUR"
if is_completed "chaotic_aur" || check_chaotic_aur_configured; then
    print_skip "Chaotic AUR já está configurado"
    is_completed "chaotic_aur" || mark_completed "chaotic_aur"
else
    if confirm_action "Configurar Chaotic AUR"; then
        print_info "Configurando Chaotic AUR..."
        if sudo pacman -S --noconfirm archlinux-keyring && \
           sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com && \
           sudo pacman-key --lsign-key 3056513887B78AEB && \
           sudo pacman -U --noconfirm 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst' && \
           sudo pacman -U --noconfirm 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'; then
            
            {
                echo ""
                echo "[chaotic-aur]"
                echo "Include = /etc/pacman.d/chaotic-mirrorlist"
            } | sudo tee -a /etc/pacman.conf > /dev/null
            
            if sudo pacman -Sy; then
                mark_completed "chaotic_aur"
                print_info "Chaotic AUR configurado com sucesso!"
            else
                print_warning "Falha ao sincronizar após adicionar Chaotic AUR."
            fi
        else
            print_error "Falha ao configurar Chaotic AUR."
        fi
    else
        print_info "Configuração do Chaotic AUR ignorada."
    fi
fi

# --- 9. Instalar Yay (AUR Helper) ---
print_header "9. Instalação do Yay (AUR Helper)"
if is_completed "yay_install" || check_yay_installed; then
    print_skip "Yay já está instalado"
    is_completed "yay_install" || mark_completed "yay_install"
    command -v yay &>/dev/null && print_info "Yay versão: $(yay --version | head -n1)"
else
    if confirm_action "Yay (AUR Helper)"; then
        print_info "Instalando dependências e compilando Yay..."
        if sudo pacman -S --noconfirm git base-devel; then
            cd /tmp || exit 1
            if git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si --noconfirm; then
                cd "$HOME" || exit 1
                rm -rf /tmp/yay
                mark_completed "yay_install"
                print_info "Yay instalado com sucesso!"
            else
                print_error "Falha ao compilar Yay."
            fi
        else
            print_error "Falha ao instalar dependências do Yay."
        fi
    else
        print_info "Instalação do Yay ignorada."
    fi
fi

# --- 10. Instalar VS Code ---
print_header "10. Instalação do Visual Studio Code"
if is_completed "vscode_install" || check_vscode_installed; then
    print_skip "Visual Studio Code já está instalado"
    is_completed "vscode_install" || mark_completed "vscode_install"
else
    if command -v yay &>/dev/null; then
        if confirm_action "Visual Studio Code (via AUR)"; then
            print_info "Instalando Visual Studio Code..."
            if yay -S --noconfirm visual-studio-code-bin; then
                mark_completed "vscode_install"
                print_info "Visual Studio Code instalado."
            else
                print_error "Falha ao instalar VS Code."
            fi
        else
            print_info "Instalação do VS Code ignorada."
        fi
    else
        print_warning "Yay não disponível. VS Code não pode ser instalado via AUR."
    fi
fi

# --- 11. Instalar Cursor ---
print_header "11. Instalação do Cursor"
if is_completed "cursor_install" || check_cursor_installed; then
    print_skip "Cursor já está instalado"
    is_completed "cursor_install" || mark_completed "cursor_install"
else
    if command -v yay &>/dev/null; then
        if confirm_action "Cursor (Editor com IA)"; then
            print_info "Instalando Cursor..."
            if yay -S --noconfirm cursor-bin; then
                mark_completed "cursor_install"
                print_info "Cursor instalado."
            else
                print_error "Falha ao instalar Cursor."
            fi
        else
            print_info "Instalação do Cursor ignorada."
        fi
    else
        print_warning "Yay não disponível. Cursor não pode ser instalado via AUR."
    fi
fi

# --- 12. Instalar Obsidian ---
print_header "12. Instalação do Obsidian"
if is_completed "obsidian_install" || check_obsidian_installed; then
    print_skip "Obsidian já está instalado"
    is_completed "obsidian_install" || mark_completed "obsidian_install"
else
    if confirm_action "Obsidian"; then
        if command -v yay &>/dev/null; then
            print_info "Tentando instalar via AUR..."
            if yay -S --noconfirm obsidian; then
                mark_completed "obsidian_install"
                print_info "Obsidian instalado via AUR."
            else
                print_warning "Falha via AUR. Tentando AppImage..."
                mkdir -p "$HOME/.local/bin"
                
                OBSIDIAN_URL=$(curl -s https://api.github.com/repos/obsidianmd/obsidian-releases/releases/latest | grep "browser_download_url.*AppImage" | head -n1 | cut -d '"' -f 4)
                
                if [ -n "$OBSIDIAN_URL" ] && wget -O "$HOME/.local/bin/Obsidian.AppImage" "$OBSIDIAN_URL"; then
                    chmod +x "$HOME/.local/bin/Obsidian.AppImage"
                    
                    mkdir -p "$HOME/.local/share/applications"
                    cat > "$HOME/.local/share/applications/obsidian.desktop" << EOF
[Desktop Entry]
Name=Obsidian
Exec=$HOME/.local/bin/Obsidian.AppImage
Icon=obsidian
Type=Application
Categories=Office;Utility;
EOF
                    mark_completed "obsidian_install"
                    print_info "Obsidian AppImage instalado."
                else
                    print_error "Falha ao baixar Obsidian AppImage."
                fi
            fi
        else
            print_warning "Yay não disponível. Obsidian não pode ser instalado."
        fi
    else
        print_info "Instalação do Obsidian ignorada."
    fi
fi

# --- 13. Configuração do X Server para WSL ---
print_header "13. Configuração do X Server para WSL"
if is_completed "x11_config" || check_x11_configured; then
    print_skip "X11 já está configurado"
    is_completed "x11_config" || mark_completed "x11_config"
else
    print_info "Configurando X11 para WSL..."
    WINDOWS_HOST_IP=$(awk '/^nameserver / { print $2; exit }' /etc/resolv.conf 2>/dev/null)
    
    if [ -n "$WINDOWS_HOST_IP" ]; then
        print_info "IP do host detectado: $WINDOWS_HOST_IP"
        
        # Bash configuration
        if ! grep -q "DISPLAY.*$WINDOWS_HOST_IP" "$HOME/.bashrc" 2>/dev/null; then
            {
                echo "# WSL X11 Configuration"
                echo "export DISPLAY=\"$WINDOWS_HOST_IP:0.0\""
                echo "export LIBGL_ALWAYS_INDIRECT=1"
                echo "export PULSE_SERVER=\"tcp:$WINDOWS_HOST_IP\""
            } >> "$HOME/.bashrc"
        fi
        
        # Fish configuration
        if [ -f "$HOME/.config/fish/config.fish" ] && ! grep -q "DISPLAY.*$WINDOWS_HOST_IP" "$HOME/.config/fish/config.fish"; then
            {
                echo ""
                echo "# WSL X11 Configuration"
                echo "set -gx DISPLAY \"$WINDOWS_HOST_IP:0.0\""
                echo "set -gx LIBGL_ALWAYS_INDIRECT 1"
                echo "set -gx PULSE_SERVER \"tcp:$WINDOWS_HOST_IP\""
            } >> "$HOME/.config/fish/config.fish"
        fi
        
        mark_completed "x11_config"
        print_info "X11 configurado para WSL."
    else
        print_error "Não foi possível detectar IP do host."
    fi
fi

# --- 14. Instalar Utilitários ---
print_header "14. Instalação de Utilitários Adicionais"
if is_completed "utils_install" || check_utils_installed; then
    print_skip "Utilitários já estão instalados"
    is_completed "utils_install" || mark_completed "utils_install"
else
    if confirm_action "Instalar utilitários úteis"; then
        print_info "Instalando utilitários..."
        if sudo pacman -S --noconfirm wget unzip htop neofetch tree vim nano; then
            mark_completed "utils_install"
            print_info "Utilitários instalados."
        else
            print_warning "Falha ao instalar alguns utilitários."
        fi
    else
        print_info "Instalação de utilitários ignorada."
    fi
fi

# --- 15. Limpeza Final ---
print_header "15. Limpeza Final"
print_info "Limpando cache e arquivos temporários..."
sudo pacman -Sc --noconfirm || print_warning "Falha ao limpar cache do Pacman."

ORPHANS=$(pacman -Qtdq 2>/dev/null)
if [ -n "$ORPHANS" ]; then
    echo "$ORPHANS" | sudo pacman -Rns --noconfirm - || print_warning "Falha ao remover pacotes órfãos."
else
    print_info "Nenhum pacote órfão encontrado."
fi

command -v yay &>/dev/null && yay -Sc --noconfirm

# --- Status Final ---
print_header "Configuração Finalizada!"
show_status

print_warning "PRÓXIMOS PASSOS:"
print_warning "1. Reinicie o WSL: 'wsl --shutdown' no PowerShell"
print_warning "2. Instale VcXsrv no Windows para aplicações gráficas"
print_warning "3. Configure VcXsrv com 'Disable access control'"
print_warning "4. Para KDE: execute 'startplasma-x11'"

echo -e "\n${GREEN}Para ver o status: $0 --status${NC}"
echo -e "${YELLOW}Para resetar: $0 --reset${NC}"
echo -e "\n${BLUE}Script finalizado!${NC}\n"
```