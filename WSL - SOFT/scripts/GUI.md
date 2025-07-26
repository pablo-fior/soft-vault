- Este script assume que você já tem uma **instalação básica do Arch Linux funcionando**, com acesso à internet e um usuário não-root configurado com `sudo`.
    
- **Execute este script como seu usuário normal**, e ele solicitará `sudo` quando necessário.
    
- **Leia cada seção cuidadosamente** antes de executá-la para entender o que está acontecendo.
    
- **Faça backups** de quaisquer arquivos de configuração importantes antes de executar o script, especialmente se você estiver adaptando-o para um sistema já em uso.
    
- O script está desenhado para ser **interativo em alguns pontos**, permitindo que você tome decisões durante a execução (como a escolha do ambiente de desktop).

```bash
#!/bin/bash

# --- Variáveis de Configuração ---
# Você pode alterar estas variáveis conforme suas preferências.

# Nome do seu usuário (garanta que este usuário já exista e tenha sudo)
USERNAME=$(whoami)

# Idioma e Localização (LC_ALL pode ser removido se quiser configurar apenas LOCALE)
LOCALE="en_US.UTF-8" # Exemplo: en_US.UTF-8 ou pt_BR.UTF-8
KEYMAP="br-abnt2"    # Layout do teclado no console
TIMEZONE="America/Sao_Paulo" # Exemplo: America/Sao_Paulo

# Hostname do seu sistema
HOSTNAME="archlinux-desktop"

# Diretório para baixar arquivos temporários (se necessário)
DOWNLOAD_DIR="/tmp/arch_post_install"

# --- Cores para Saída do Script ---
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# --- Funções Auxiliares ---

print_header() {
    echo -e "\n${BLUE}========================================${NC}"
    echo -e "${BLUE}>>> $1 <<<${NC}"
    echo -e "${BLUE}========================================${NC}\n"
}

print_info() {
    echo -e "${GREEN}INFO: $1${NC}"
}

print_warning() {
    echo -e "${YELLOW}WARNING: $1${NC}"
}

print_error() {
    echo -e "${RED}ERROR: $1${NC}"
}

confirm_action() {
    read -p "$(echo -e "${YELLOW}Deseja continuar com a instalação de '$1'? (s/N) ${NC}")" choice
    [[ "$choice" = [sS] ]]
}

# --- Início do Script ---

print_header "Iniciando Configuração Pós-Instalação do Arch Linux"
print_info "Executando como usuário: $USERNAME"
sleep 2

# --- 1. Sincronizar e Atualizar o Sistema ---
print_header "1. Sincronizando e Atualizando o Sistema"
print_info "Sincronizando espelhos e atualizando todos os pacotes..."
sudo pacman -Syu --noconfirm || { print_error "Falha ao sincronizar e atualizar o sistema. Verifique sua conexão e tente novamente."; exit 1; }
print_info "Sistema atualizado com sucesso!"

---
## 2. Configurações Essenciais do Sistema

print_header "2. Configurações Essenciais do Sistema"

# --- 2.1. Configuração de Hostname ---
print_info "Configurando Hostname para: $HOSTNAME"
echo "$HOSTNAME" | sudo tee /etc/hostname > /dev/null
echo "127.0.0.1 localhost" | sudo tee /etc/hosts > /dev/null
echo "::1       localhost" | sudo tee -a /etc/hosts > /dev/null
echo "127.0.1.1 $HOSTNAME.localdomain $HOSTNAME" | sudo tee -a /etc/hosts > /dev/null
print_info "Hostname e /etc/hosts configurados."

# --- 2.2. Configuração de Localização (Locale) ---
print_info "Configurando Localização (Locale) para: $LOCALE"
sudo sed -i "/^#$LOCALE UTF-8/s/^#//" /etc/locale.gen
sudo locale-gen
echo "LANG=$LOCALE" | sudo tee /etc/locale.conf > /dev/null
echo "KEYMAP=$KEYMAP" | sudo tee /etc/vconsole.conf > /dev/null
print_info "Localização e keymap do console configurados."

# --- 2.3. Configuração de Fuso Horário ---
print_info "Configurando Fuso Horário para: $TIMEZONE"
sudo ln -sf "/usr/share/zoneinfo/$TIMEZONE" /etc/localtime
sudo hwclock --systohc --utc # Sincroniza o relógio de hardware com o sistema (UTC é recomendado)
print_info "Fuso horário configurado e relógio de hardware sincronizado."

# --- 2.4. Instalação e Configuração do NTP (Network Time Protocol) ---
print_info "Instalando e configurando NTP para sincronização de tempo..."
sudo pacman -S --noconfirm ntp || print_warning "Falha ao instalar ntp. O tempo pode não ser sincronizado automaticamente."
sudo systemctl enable ntpd --now || print_warning "Falha ao habilitar e iniciar ntpd."
print_info "NTP configurado e ativado."

---
## 3. Instalação de Ferramentas Essenciais

print_header "3. Instalação de Ferramentas Essenciais"

# --- 3.1. Ferramentas de Desenvolvimento e Build ---
print_info "Instalando base-devel (essencial para compilar pacotes do AUR)..."
sudo pacman -S --noconfirm base-devel || print_warning "Falha ao instalar base-devel."
print_info "Instalando git para clonar repositórios..."
sudo pacman -S --noconfirm git || print_warning "Falha ao instalar git."
print_info "Instalando wget e curl para downloads..."
sudo pacman -S --noconfirm wget curl || print_warning "Falha ao instalar wget/curl."
print_info "Instalando unzip, unrar, p7zip para manipulação de arquivos..."
sudo pacman -S --noconfirm unzip unrar p7zip || print_warning "Falha ao instalar unzip/unrar/p7zip."

# --- 3.2. Gerenciador de Rede (NetworkManager é o mais comum) ---
print_info "Instalando e ativando NetworkManager..."
sudo pacman -S --noconfirm networkmanager || { print_error "Falha ao instalar networkmanager."; exit 1; }
sudo systemctl enable NetworkManager --now || { print_error "Falha ao habilitar e iniciar NetworkManager."; exit 1; }
print_info "NetworkManager instalado e ativado."

# --- 3.3. Configuração de Firewall (UFW é simples e eficaz) ---
if confirm_action "Firewall (UFW)"; then
    print_info "Instalando e configurando UFW (Uncomplicated Firewall)..."
    sudo pacman -S --noconfirm ufw || print_warning "Falha ao instalar ufw."
    sudo ufw enable || print_warning "Falha ao habilitar UFW. Verifique o status manualmente."
    sudo ufw default deny incoming || print_warning "Falha ao configurar UFW para negar conexões de entrada."
    sudo ufw default allow outgoing || print_warning "Falha ao configurar UFW para permitir conexões de saída."
    print_info "UFW configurado e ativado. Regras básicas aplicadas."
    print_info "Você pode adicionar regras específicas com 'sudo ufw allow/deny <porta>/<protocolo>'."
else
    print_info "Ignorando instalação de Firewall (UFW)."
fi

# --- 3.4. Drivers Gráficos ---
print_info "Detectando e instalando drivers gráficos..."
if lspci | grep -E "VGA|3D|Display" | grep -i nvidia; then
    print_info "Detectado placa de vídeo NVIDIA."
    if confirm_action "Drivers NVIDIA (recomendado para desempenho em jogos/aplicações gráficas)"; then
        sudo pacman -S --noconfirm nvidia nvidia-utils nvidia-settings || print_warning "Falha ao instalar drivers NVIDIA."
        print_info "Drivers NVIDIA instalados. Reinicie para aplicar as mudanças."
    else
        print_warning "Drivers NVIDIA ignorados. Usando Nouveau (open source) por padrão."
    fi
elif lspci | grep -E "VGA|3D|Display" | grep -i amd; then
    print_info "Detectado placa de vídeo AMD/ATI."
    print_info "Instalando drivers AMD (amdvlk, vulkan-radeon, mesa)..."
    sudo pacman -S --noconfirm amdvlk vulkan-radeon mesa || print_warning "Falha ao instalar drivers AMD/Mesa."
    print_info "Drivers AMD/Mesa instalados."
elif lspci | grep -E "VGA|3D|Display" | grep -i intel; then
    print_info "Detectado placa de vídeo Intel."
    print_info "Instalando drivers Intel (mesa, vulkan-intel)..."
    sudo pacman -S --noconfirm mesa vulkan-intel || print_warning "Falha ao instalar drivers Intel/Mesa."
    print_info "Drivers Intel/Mesa instalados."
else
    print_warning "Nenhum driver gráfico proprietário detectado/instalado. Usando drivers Mesa (open source)."
    sudo pacman -S --noconfirm mesa || print_warning "Falha ao instalar pacote Mesa genérico."
fi

---
## 4. Ambiente de Desktop ou Gerenciador de Janelas

print_header "4. Instalação de Ambiente de Desktop (DE) ou Gerenciador de Janelas (WM)"

echo -e "${YELLOW}Escolha o ambiente de desktop/gerenciador de janelas que deseja instalar:${NC}"
echo "1) GNOME"
echo "2) KDE Plasma"
echo "3) XFCE"
echo "4) Cinnamon"
echo "5) MATE"
echo "6) LXQt"
echo "7) i3 (Gerenciador de Janelas - mais leve, exige mais configuração manual)"
echo "8) Nenhuma (Instalação mínima, sem DE/WM)"
read -p "Digite o número da sua escolha (1-8): " DESKTOP_CHOICE

# --- 4.1. Instalação do Xorg (Essencial para qualquer ambiente gráfico) ---
if [[ "$DESKTOP_CHOICE" -ge 1 && "$DESKTOP_CHOICE" -le 7 ]]; then
    print_info "Instalando Xorg (servidor gráfico)..."
    sudo pacman -S --noconfirm xorg-server xorg-xinit xterm || { print_error "Falha ao instalar Xorg. Não será possível iniciar um ambiente gráfico."; exit 1; }
    print_info "Xorg instalado."
fi

# --- 4.2. Instalação do Ambiente Escolhido ---
case "$DESKTOP_CHOICE" in
    1) # GNOME
        if confirm_action "GNOME"; then
            print_info "Instalando GNOME..."
            sudo pacman -S --noconfirm gnome gnome-extra gdm gnome-tweaks gnome-shell-extensions || print_warning "Falha ao instalar GNOME."
            sudo systemctl enable gdm --now || print_warning "Falha ao habilitar e iniciar GDM."
            print_info "GNOME instalado e GDM ativado."
        fi
        ;;
    2) # KDE Plasma
        if confirm_action "KDE Plasma"; then
            print_info "Instalando KDE Plasma..."
            sudo pacman -S --noconfirm plasma sddm kde-applications || print_warning "Falha ao instalar KDE Plasma."
            sudo systemctl enable sddm --now || print_warning "Falha ao habilitar e iniciar SDDM."
            print_info "KDE Plasma instalado e SDDM ativado."
        fi
        ;;
    3) # XFCE
        if confirm_action "XFCE"; then
            print_info "Instalando XFCE..."
            sudo pacman -S --noconfirm xfce4 xfce4-goodies lightdm lightdm-gtk-greeter || print_warning "Falha ao instalar XFCE."
            sudo systemctl enable lightdm --now || print_warning "Falha ao habilitar e iniciar LightDM."
            print_info "XFCE instalado e LightDM ativado."
        fi
        ;;
    4) # Cinnamon
        if confirm_action "Cinnamon"; then
            print_info "Instalando Cinnamon..."
            sudo pacman -S --noconfirm cinnamon lightdm lightdm-gtk-greeter || print_warning "Falha ao instalar Cinnamon."
            sudo systemctl enable lightdm --now || print_warning "Falha ao habilitar e iniciar LightDM."
            print_info "Cinnamon instalado e LightDM ativado."
        fi
        ;;
    5) # MATE
        if confirm_action "MATE"; then
            print_info "Instalando MATE..."
            sudo pacman -S --noconfirm mate mate-extra lightdm lightdm-gtk-greeter || print_warning "Falha ao instalar MATE."
            sudo systemctl enable lightdm --now || print_warning "Falha ao habilitar e iniciar LightDM."
            print_info "MATE instalado e LightDM ativado."
        fi
        ;;
    6) # LXQt
        if confirm_action "LXQt"; then
            print_info "Instalando LXQt..."
            sudo pacman -S --noconfirm lxqt sddm || print_warning "Falha ao instalar LXQt."
            sudo systemctl enable sddm --now || print_warning "Falha ao habilitar e iniciar SDDM."
            print_info "LXQt instalado e SDDM ativado."
        fi
        ;;
    7) # i3
        if confirm_action "i3 (Gerenciador de Janelas)"; then
            print_info "Instalando i3 (i3-wm, i3status, i3lock, dmenu)..."
            sudo pacman -S --noconfirm i3-wm i3status i3lock dmenu || print_warning "Falha ao instalar i3."
            print_warning "i3 é um WM, exige configuração manual. Você precisará de um Display Manager (ex: LightDM ou SDDM) ou iniciar via startx."
            read -p "$(echo -e "${YELLOW}Deseja instalar e habilitar LightDM para i3? (s/N) ${NC}")" install_lightdm_i3
            if [[ "$install_lightdm_i3" = [sS] ]]; then
                sudo pacman -S --noconfirm lightdm lightdm-gtk-greeter || print_warning "Falha ao instalar LightDM para i3."
                sudo systemctl enable lightdm --now || print_warning "Falha ao habilitar e iniciar LightDM para i3."
                print_info "LightDM instalado e ativado para i3."
            else
                print_info "LightDM não será instalado. Você precisará iniciar i3 manualmente via 'startx' ou configurar outro Display Manager."
            fi
        fi
        ;;
    8)
        print_info "Nenhum ambiente de desktop/gerenciador de janelas será instalado."
        ;;
    *)
        print_warning "Opção inválida. Nenhum ambiente de desktop/gerenciador de janelas será instalado."
        ;;
esac

---
## 5. Aplicativos Essenciais

print_header "5. Instalação de Aplicativos Essenciais"

# --- 5.1. Navegador Web ---
if confirm_action "Navegador Web (Firefox)"; then
    print_info "Instalando Firefox..."
    sudo pacman -S --noconfirm firefox || print_warning "Falha ao instalar Firefox."
else
    print_info "Ignorando instalação do Firefox."
fi

# --- 5.2. Terminal Alternativo (Ex: Alacritty, Kitty) ---
if confirm_action "Terminal Alternativo (Alacritty)"; then
    print_info "Instalando Alacritty (terminal acelerado por GPU)..."
    sudo pacman -S --noconfirm alacritty || print_warning "Falha ao instalar Alacritty."
else
    print_info "Ignorando instalação de terminal alternativo."
fi

# --- 5.3. Editor de Texto (Ex: Neovim, VS Code, Sublime Text) ---
if confirm_action "Editor de Texto (Neovim e VS Code)"; then
    print_info "Instalando Neovim..."
    sudo pacman -S --noconfirm neovim || print_warning "Falha ao instalar Neovim."
    print_info "Instalando Visual Studio Code (via Flathub, se Flatpak for configurado ou AUR se você o configurar mais tarde)..."
    # VS Code geralmente é instalado via Flatpak ou AUR.
    # Por enquanto, vamos apenas notificar, a instalação do Flatpak virá mais abaixo.
    print_warning "VS Code será instalado via Flatpak (se habilitado) ou AUR. Adicione-o manualmente se preferir."
else
    print_info "Ignorando instalação de editores de texto."
fi

# --- 5.4. Leitor de PDF ---
if confirm_action "Leitor de PDF (Evince)"; then
    print_info "Instalando Evince (leitor de PDF)..."
    sudo pacman -S --noconfirm evince || print_warning "Falha ao instalar Evince."
else
    print_info "Ignorando instalação de leitor de PDF."
fi

# --- 5.5. Cliente de Email (Ex: Thunderbird) ---
if confirm_action "Cliente de Email (Thunderbird)"; then
    print_info "Instalando Thunderbird..."
    sudo pacman -S --noconfirm thunderbird || print_warning "Falha ao instalar Thunderbird."
else
    print_info "Ignorando instalação de cliente de email."
fi

# --- 5.6. Reprodutor de Mídia (Ex: VLC) ---
if confirm_action "Reprodutor de Mídia (VLC)"; then
    print_info "Instalando VLC..."
    sudo pacman -S --noconfirm vlc || print_warning "Falha ao instalar VLC."
else
    print_info "Ignorando instalação de reprodutor de mídia."
fi

# --- 5.7. Utilitários de Compactação/Descompactação (se não instalados acima) ---
# Já cobrimos unzip, unrar, p7zip na seção 3.1. Mas podemos adicionar outros gráficos aqui.
# Ex: file-roller (para GNOME), ark (para KDE)
if [[ "$DESKTOP_CHOICE" -eq 1 ]]; then # GNOME
    if confirm_action "File Roller (GNOME archive manager)"; then
        sudo pacman -S --noconfirm file-roller || print_warning "Falha ao instalar file-roller."
    fi
elif [[ "$DESKTOP_CHOICE" -eq 2 ]]; then # KDE Plasma
    if confirm_action "Ark (KDE archive manager)"; then
        sudo pacman -S --noconfirm ark || print_warning "Falha ao instalar ark."
    fi
fi

# --- 5.8. Suites de Escritório (Ex: LibreOffice) ---
if confirm_action "Suíte de Escritório (LibreOffice-fresh)"; then
    print_info "Instalando LibreOffice (versão mais recente)..."
    sudo pacman -S --noconfirm libreoffice-fresh || print_warning "Falha ao instalar LibreOffice."
else
    print_info "Ignorando instalação de suíte de escritório."
fi

---
## 6. Configurações de Áudio

print_header "6. Configurações de Áudio"

# PipeWire é o substituto moderno para PulseAudio e JACK.
if confirm_action "Configurar Áudio com PipeWire (recomendado)"; then
    print_info "Instalando PipeWire e componentes essenciais..."
    sudo pacman -S --noconfirm pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber pavucontrol || print_warning "Falha ao instalar PipeWire e componentes."
    sudo systemctl --user enable pipewire pipewire-pulse wireplumber || print_warning "Falha ao habilitar serviços PipeWire para o usuário."
    print_info "PipeWire configurado. Use 'pavucontrol' para gerenciar áudio."
else
    print_warning "PipeWire não será configurado. Você pode precisar configurar PulseAudio/ALSA manualmente."
    print_info "Instalando PulseAudio (opção alternativa)..."
    sudo pacman -S --noconfirm pulseaudio pulseaudio-alsa pavucontrol || print_warning "Falha ao instalar PulseAudio."
fi

---
## 7. Otimizações e Aprimoramentos

print_header "7. Otimizações e Aprimoramentos"

# --- 7.1. Habilitar Multilib (para Wine, Steam, etc.) ---
if confirm_action "Habilitar Repositório Multilib (para programas de 32-bit como Wine/Steam)"; then
    print_info "Habilitando repositório Multilib..."
    sudo sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf
    sudo pacman -Syyu --noconfirm # Sincroniza e atualiza novamente após habilitar multilib
    print_info "Repositório Multilib habilitado e sistema atualizado."
else
    print_info "Repositório Multilib não será habilitado."
fi

# --- 7.2. Habilitar Serviço de FSTRIM (para SSDs) ---
if confirm_action "Habilitar fstrim.timer (otimização para SSDs)"; then
    print_info "Habilitando fstrim.timer para otimização de SSDs..."
    sudo systemctl enable fstrim.timer --now || print_warning "Falha ao habilitar fstrim.timer."
    print_info "fstrim.timer habilitado. Ele executará o TRIM semanalmente."
else
    print_info "fstrim.timer não será habilitado."
fi

# --- 7.3. Instalar Microcode (para patches de segurança e desempenho da CPU) ---
if confirm_action "Instalar Microcode da CPU (Intel/AMD)"; then
    print_info "Instalando microcode da CPU..."
    if grep -q "AuthenticAMD" /proc/cpuinfo; then
        sudo pacman -S --noconfirm amd-ucode || print_warning "Falha ao instalar microcode AMD."
        print_info "Microcode AMD instalado."
    elif grep -q "GenuineIntel" /proc/cpuinfo; then
        sudo pacman -S --noconfirm intel-ucode || print_warning "Falha ao instalar microcode Intel."
        print_info "Microcode Intel instalado."
    else
        print_warning "CPU não detectada como Intel ou AMD. Microcode não instalado."
    fi
    print_info "Lembre-se de regenerar o GRUB (ou seu bootloader) se o microcode não for automaticamente detectado, ou para garantir que ele seja carregado corretamente."
    print_info "Execute 'sudo grub-mkconfig -o /boot/grub/grub.cfg' após a instalação do microcode."
else
    print_info "Microcode da CPU não será instalado."
fi

# --- 7.4. Configurar TLP (para economia de energia em notebooks) ---
if confirm_action "Instalar e configurar TLP (para otimização de bateria em notebooks)"; then
    print_info "Instalando TLP para gerenciamento de energia..."
    sudo pacman -S --noconfirm tlp tlp-rdw || print_warning "Falha ao instalar TLP."
    sudo systemctl enable tlp --now || print_warning "Falha ao habilitar e iniciar TLP."
    print_info "TLP instalado e ativado. Reinicie para aplicar as configurações de energia."
    print_info "Para mais configurações, edite /etc/tlp.conf."
else
    print_info "TLP não será instalado."
fi

# --- 7.5. Configurar o AUR Helper (Yay é o mais popular) ---
if confirm_action "Instalar Yay (AUR Helper)"; then
    print_info "Instalando Yay (AUR helper)..."
    print_info "Clonando repositório yay..."
    mkdir -p "$DOWNLOAD_DIR"
    cd "$DOWNLOAD_DIR"
    git clone https://aur.archlinux.org/yay.git || { print_error "Falha ao clonar repositório yay."; cd -; exit 1; }
    cd yay
    makepkg -si --noconfirm || { print_error "Falha ao compilar e instalar yay."; cd -; exit 1; }
    cd - > /dev/null # Retorna ao diretório anterior
    rm -rf "$DOWNLOAD_DIR" # Limpa arquivos temporários
    print_info "Yay instalado com sucesso!"
    print_info "Agora você pode instalar pacotes do AUR com 'yay -S <nome_do_pacote>'."
else
    print_info "Yay (AUR Helper) não será instalado. Você precisará instalar pacotes do AUR manualmente."
fi

# --- 7.6. Configurar Flatpak (para aplicativos adicionais) ---
if confirm_action "Instalar e configurar Flatpak"; then
    print_info "Instalando Flatpak..."
    sudo pacman -S --noconfirm flatpak || print_warning "Falha ao instalar flatpak."
    print_info "Adicionando repositório Flathub..."
    flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo || print_warning "Falha ao adicionar repositório Flathub."
    print_info "Flatpak configurado. Você pode instalar aplicativos com 'flatpak install flathub <nome_do_aplicativo>'."
else
    print_info "Flatpak não será configurado."
fi

---
## 8. Limpeza Final

print_header "8. Limpeza Final"

print_info "Limpando cache do Pacman..."
sudo pacman -Sc --noconfirm || print_warning "Falha ao limpar cache do Pacman."
print_info "Removendo pacotes órfãos (não usados por outros pacotes)..."
sudo pacman -Qtdq | sudo pacman -Rns --noconfirm || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
print_info "Cache e pacotes órfãos limpos."

---
## 9. Finalização

print_header "Processo de Pós-Instalação Concluído!"

print_info "Reinicie o sistema para que todas as mudanças entrem em vigor."
print_info "Após o reboot, você poderá fazer login no seu novo ambiente de desktop."

echo -e "\n${BLUE}========================================${NC}"
echo -e "${BLUE}Script Concluído!${NC}"
echo -e "${BLUE}========================================${NC}\n"

read -p "$(echo -e "${YELLOW}Deseja reiniciar agora? (s/N) ${NC}")" reboot_choice
if [[ "$reboot_choice" = [sS] ]]; then
    sudo reboot
else
    print_info "Reinicie manualmente quando estiver pronto com 'sudo reboot'."
fi
```