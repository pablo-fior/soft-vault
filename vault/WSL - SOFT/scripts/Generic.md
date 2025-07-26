Este script é projetado para ser executado **dentro da sua distribuição Linux no WSL2** após a instalação básica. Ele abordará otimizações de sistema, instalação de ferramentas essenciais e configurações de integração com o Windows.

**Observações Importantes:**

- **Execute este script como seu usuário normal** (não `root`), e ele solicitará `sudo` quando necessário.
    
- **Este script é genérico para qualquer distribuição Linux (Debian/Ubuntu, Arch, Fedora, etc.) no WSL2**, mas as seções de `pacman` (Arch) ou `apt` (Debian/Ubuntu) serão condicionais. Você deve descomentar a seção que corresponde à sua distro.
    
- **Leia cada seção cuidadosamente** antes de executá-la para entender o que está acontecendo.
    
- **Faça backups** de quaisquer arquivos de configuração importantes se estiver adaptando para um sistema já em uso.
    
- O script tenta ser **interativo em alguns pontos**, permitindo que você tome decisões durante a execução.

### Importante
**Gerenciamento de Recursos (.wslconfig):** É fundamental ajustar o arquivo `.wslconfig` no Windows (geralmente em `C:\Users\<SeuUsuario>\.wslconfig`) para limitar a RAM e a CPU que o WSL pode usar, evitando que ele consuma todos os recursos do seu sistema Windows.

```bash
#!/bin/bash

# --- Variáveis de Configuração ---
# Você pode alterar estas variáveis conforme suas preferências.

# Nome do seu usuário no WSL (garanta que este usuário já exista e tenha sudo)
USERNAME=$(whoami)

# Idioma e Localização
LOCALE="en_US.UTF-8" # Exemplo: en_US.UTF-8 ou pt_BR.UTF-8
KEYMAP="br-abnt2"    # Layout do teclado (pode não ser totalmente relevante para o WSL console)

# Hostname do seu ambiente WSL
HOSTNAME="wsl-linux-distro"

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

detect_distro() {
    if grep -q "ID=debian" /etc/os-release || grep -q "ID=ubuntu" /etc/os-release; then
        echo "debian_ubuntu"
    elif grep -q "ID=arch" /etc/os-release; then
        echo "arch"
    elif grep -q "ID=fedora" /etc/os-release; then
        echo "fedora"
    else
        echo "unknown"
    fi
}

DISTRO_TYPE=$(detect_distro)

install_package() {
    local package_name="$1"
    case "$DISTRO_TYPE" in
        debian_ubuntu)
            sudo apt install -y "$package_name" || print_warning "Falha ao instalar $package_name (apt)."
            ;;
        arch)
            sudo pacman -S --noconfirm "$package_name" || print_warning "Falha ao instalar $package_name (pacman)."
            ;;
        fedora)
            sudo dnf install -y "$package_name" || print_warning "Falha ao instalar $package_name (dnf)."
            ;;
        *)
            print_warning "Gerenciador de pacotes desconhecido. Instale $package_name manualmente."
            ;;
    esac
}

update_system() {
    case "$DISTRO_TYPE" in
        debian_ubuntu)
            sudo apt update && sudo apt upgrade -y || { print_error "Falha ao atualizar o sistema (apt)."; exit 1; }
            ;;
        arch)
            sudo pacman -Syu --noconfirm || { print_error "Falha ao atualizar o sistema (pacman)."; exit 1; }
            ;;
        fedora)
            sudo dnf upgrade -y || { print_error "Falha ao atualizar o sistema (dnf)."; exit 1; }
            ;;
        *)
            print_error "Gerenciador de pacotes desconhecido. Não foi possível atualizar o sistema."
            exit 1
            ;;
    esac
}

# --- Início do Script ---

print_header "Iniciando Configuração Pós-Instalação para WSL2"
print_info "Distribuição detectada: $DISTRO_TYPE"
print_info "Executando como usuário: $USERNAME"
sleep 2

# --- 1. Sincronizar e Atualizar o Sistema ---
print_header "1. Sincronizando e Atualizando o Sistema"
print_info "Sincronizando espelhos e atualizando todos os pacotes..."
update_system
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
if [ "$DISTRO_TYPE" = "debian_ubuntu" ]; then
    sudo sed -i "/^# $LOCALE UTF-8/s/^# //" /etc/locale.gen
    sudo locale-gen
    echo "LANG=$LOCALE" | sudo tee /etc/default/locale > /dev/null
    print_info "Localização configurada para Debian/Ubuntu."
elif [ "$DISTRO_TYPE" = "arch" ]; then
    sudo sed -i "/^#$LOCALE UTF-8/s/^#//" /etc/locale.gen
    sudo locale-gen
    echo "LANG=$LOCALE" | sudo tee /etc/locale.conf > /dev/null
    echo "KEYMAP=$KEYMAP" | sudo tee /etc/vconsole.conf > /dev/null
    print_info "Localização e keymap do console configurados para Arch."
elif [ "$DISTRO_TYPE" = "fedora" ]; then
    sudo sed -i "/^#$LOCALE UTF-8/s/^#//" /etc/locale.gen
    sudo localectl set-locale LANG="$LOCALE"
    sudo localectl set-keymap "$KEYMAP"
    print_info "Localização e keymap do console configurados para Fedora."
else
    print_warning "Não foi possível configurar o locale para a distribuição desconhecida. Faça manualmente."
fi
print_info "Localização configurada."

# --- 2.3. Configuração de Fuso Horário ---
print_info "Configurando Fuso Horário (herdado do Windows ou padrão se não configurado)..."
# No WSL2, o relógio do sistema Linux geralmente é sincronizado com o Windows.
# Podemos garantir que o fuso horário reflita o do sistema host ou um fuso desejado.
# Exemplo: TZ=$(curl -s ipinfo.io/json | grep -oP '"timezone": "\K[^"]+') # Precisa de 'curl'
# Ou apenas defina um padrão.
DEFAULT_TIMEZONE="America/Sao_Paulo"
read -p "$(echo -e "${YELLOW}Deseja definir o fuso horário para $DEFAULT_TIMEZONE? (s/N) ${NC}")" set_timezone
if [[ "$set_timezone" = [sS] ]]; then
    sudo timedatectl set-timezone "$DEFAULT_TIMEZONE" || print_warning "Falha ao definir fuso horário com timedatectl. Verifique o serviço systemd-timesyncd."
    print_info "Fuso horário definido para: $DEFAULT_TIMEZONE."
else
    print_info "Fuso horário não alterado. Manterá o padrão ou o herdado."
fi

# --- 2.4. Instalação e Configuração do NTP (Network Time Protocol) ---
# O WSL já lida com sincronização de tempo via Windows, mas ter um cliente é bom.
if confirm_action "Cliente NTP (para sincronização de tempo)"; then
    print_info "Instalando cliente NTP..."
    if [ "$DISTRO_TYPE" = "debian_ubuntu" ] || [ "$DISTRO_TYPE" = "fedora" ]; then
        install_package "systemd-timesyncd" # Usado por padrão em muitas distros modernas
        sudo systemctl enable systemd-timesyncd --now || print_warning "Falha ao habilitar e iniciar systemd-timesyncd."
    elif [ "$DISTRO_TYPE" = "arch" ]; then
        install_package "ntp"
        sudo systemctl enable ntpd --now || print_warning "Falha ao habilitar e iniciar ntpd."
    fi
    print_info "Cliente NTP configurado e ativado."
else
    print_info "Ignorando instalação de cliente NTP."
fi

---
## 3. Instalação de Ferramentas Essenciais

print_header "3. Instalação de Ferramentas Essenciais"

# --- 3.1. Ferramentas de Desenvolvimento e Build ---
print_info "Instalando ferramentas de desenvolvimento e build..."
install_package "build-essential" # Debian/Ubuntu
install_package "base-devel"     # Arch
install_package "gcc"            # Fedora (e outras, se build-essential não existe)
install_package "make"
install_package "git"
install_package "wget"
install_package "curl"
install_package "unzip"
install_package "unrar"
install_package "p7zip"
print_info "Ferramentas essenciais instaladas."

# --- 3.2. Gerenciamento de Rede (WSL) ---
print_info "WSL2 gerencia a rede, então não precisamos de NetworkManager aqui."
print_info "Instalando iputils (para 'ip' e outras ferramentas de rede)."
install_package "iputils-ip" # Ou 'iproute2' se preferir

# --- 3.3. Firewall ---
print_info "Configuração de firewall (UFW) dentro do WSL é limitada."
print_info "O tráfego é roteado pelo Windows. Gerencie o firewall do Windows para controle externo."
if confirm_action "Instalar UFW (para controle interno de processos, se desejado)"; then
    print_info "Instalando UFW (Uncomplicated Firewall)..."
    install_package "ufw"
    # Não vamos habilitar por padrão, pois pode causar problemas com a rede do WSL
    print_warning "UFW instalado, mas não habilitado por padrão. Use com cautela no WSL."
    print_warning "Execute 'sudo ufw enable' por sua conta e risco."
else
    print_info "Ignorando instalação de Firewall (UFW)."
fi

# --- 3.4. Docker (WSL integra-se com Docker Desktop) ---
if confirm_action "Instalar Docker (dentro do WSL - requer Docker Desktop no Windows)"; then
    print_info "Configurando Docker para integração com Docker Desktop..."
    # A instalação do Docker Engine pode ser feita, mas o Docker Desktop para Windows é o ideal
    # para gerenciar contêineres e imagens que rodam no backend WSL.
    # O comando abaixo adiciona o usuário ao grupo docker.
    sudo usermod -aG docker "$USERNAME" || print_warning "Falha ao adicionar usuário ao grupo docker. Reinicie o WSL para aplicar."
    print_info "Certifique-se de ter o Docker Desktop instalado no Windows e configurado para usar o WSL2."
    print_info "Você precisará reiniciar a instância WSL para que a mudança de grupo tenha efeito."
    print_info "Para instalar o Docker Engine *dentro* do WSL, você precisaria de mais passos."
else
    print_info "Ignorando configuração de Docker."
fi

---
## 4. Ferramentas de Produtividade e Desenvolvimento

print_header "4. Ferramentas de Produtividade e Desenvolvimento"

# --- 4.1. Editores de Texto de Terminal ---
if confirm_action "Editores de Texto (Vim, Nano, Neovim)"; then
    print_info "Instalando Vim e Nano..."
    install_package "vim"
    install_package "nano"
    print_info "Instalando Neovim..."
    install_package "neovim"
else
    print_info "Ignorando instalação de editores de texto de terminal."
fi

# --- 4.2. Utilitários de Produtividade em Terminal ---
if confirm_action "Utilitários de Produtividade (htop, neofetch, glances, tree)"; then
    print_info "Instalando htop (gerenciador de processos)..."
    install_package "htop"
    print_info "Instalando neofetch (informações do sistema)..."
    install_package "neofetch"
    print_info "Instalando glances (monitor de sistema avançado)..."
    install_package "glances"
    print_info "Instalando tree (exibição de diretórios)..."
    install_package "tree"
else
    print_info "Ignorando instalação de utilitários de produtividade."
fi

# --- 4.3. Cliente SSH e Servidor (para acessar outros hosts e ser acessado pelo Windows) ---
if confirm_action "OpenSSH (Cliente e Servidor)"; then
    print_info "Instalando OpenSSH..."
    install_package "openssh-client" # Debian/Ubuntu
    install_package "openssh-server" # Debian/Ubuntu
    install_package "openssh"        # Arch/Fedora (geralmente inclui ambos)

    # Configuração básica do servidor SSH no WSL
    print_info "Configurando o servidor OpenSSH..."
    sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config # Mudar porta para evitar conflitos
    sudo sed -i 's/#ListenAddress 0.0.0.0/ListenAddress 0.0.0.0/' /etc/ssh/sshd_config
    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config # Habilitar senha para facilidade inicial
    sudo systemctl enable ssh --now || print_warning "Falha ao habilitar e iniciar sshd. Verifique se o systemd está ativo no WSL."
    print_info "Servidor SSH configurado e ativado na porta 2222."
    print_warning "Lembre-se de configurar regras de firewall no Windows para permitir conexões à porta 2222 se você quiser acessar de outras máquinas na rede."
    print_warning "Para acessar do Windows, use 'ssh $USERNAME@localhost -p 2222'."
else
    print_info "Ignorando instalação e configuração de OpenSSH."
fi

# --- 4.4. Zsh e Oh My Zsh (opcional, para um shell mais poderoso) ---
if confirm_action "Zsh e Oh My Zsh (shell aprimorado)"; then
    print_info "Instalando Zsh..."
    install_package "zsh"
    print_info "Configurando Oh My Zsh para o usuário $USERNAME..."
    if [ ! -d "$HOME/.oh-my-zsh" ]; then
        # --unattended para instalação não interativa
        sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended || print_warning "Falha ao instalar Oh My Zsh. Tente novamente manualmente."
        print_info "Oh My Zsh instalado. Para definir Zsh como shell padrão: 'chsh -s $(which zsh)'."
    else
        print_info "Oh My Zsh já parece estar instalado."
    fi
else
    print_info "Ignorando instalação de Zsh e Oh My Zsh."
fi

---
## 5. Integração e Otimizações WSL Específicas

print_header "5. Integração e Otimizações WSL Específicas"

# --- 5.1. Montagem Automática de Drives do Windows ---
print_info "Verificando montagem automática de drives do Windows..."
print_info "Os drives do Windows (C:, D:, etc.) são montados automaticamente em /mnt/c, /mnt/d, etc."
print_info "Não é necessário configuração adicional, apenas um lembrete."

# --- 5.2. Otimizações .wslconfig (para o lado do Windows) ---
print_info "Otimizações de recursos de CPU/Memória são feitas no arquivo .wslconfig no Windows."
print_info "Este script não pode modificar o .wslconfig diretamente. Crie/edite C:\\Users\\<SeuUsuario>\\.wslconfig"
print_info "Exemplo de .wslconfig:"
echo -e "${YELLOW}[wsl2]
memory=4GB       # Limita a memória que o WSL pode usar
processors=2     # Limita o número de processadores virtuais
swap=0           # Desabilita swap (se você tem muita RAM)
localhostForwarding=true # Permite acesso a serviços do WSL via localhost no Windows
#autoMemoryReclaim=true # Libera memória não utilizada (Windows 11)
${NC}"
print_warning "Após editar .wslconfig, execute 'wsl --shutdown' no PowerShell para aplicar as mudanças."

# --- 5.3. Configuração de X Server para GUI (Opcional) ---
if confirm_action "Configurar X Server para GUI (requer XLaunch/VcXsrv no Windows)"; then
    print_info "Instalando pacotes básicos para X Server (ex: xterm, xauth, mesa-utils)..."
    install_package "xterm"
    install_package "xauth"
    install_package "mesa-utils" # Para glxgears, etc.

    echo -e "${YELLOW}Configurando DISPLAY para o X Server do Windows...${NC}"
    # Detecta o IP do Windows Host dinamicamente
    # No WSL2, o IP do host Windows varia.
    # Uma forma comum é acessar via o arquivo resolv.conf
    WINDOWS_HOST_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
    echo "export DISPLAY=$WINDOWS_HOST_IP:0.0" >> "$HOME/.bashrc"
    echo "export LIBGL_ALWAYS_INDIRECT=1" >> "$HOME/.bashrc" # Pode ser necessário para apps OpenGL
    echo "export PULSE_SERVER=tcp:$WINDOWS_HOST_IP" >> "$HOME/.bashrc" # Para áudio com PulseAudio
    print_info "Configuração DISPLAY e PULSE_SERVER adicionada ao ~/.bashrc"
    print_warning "Você precisará instalar um X Server como VcXsrv ou XLaunch no Windows e iniciá-lo."
    print_warning "Recarregue seu shell ('source ~/.bashrc' ou abra um novo terminal) após esta etapa."
    print_warning "Para áudio, instale PulseAudio e configure o servidor no Windows."
else
    print_info "Ignorando configuração de X Server para GUI."
fi

---
## 6. Limpeza Final

print_header "6. Limpeza Final"

print_info "Limpando cache do gerenciador de pacotes..."
case "$DISTRO_TYPE" in
    debian_ubuntu)
        sudo apt clean && sudo apt autoremove -y || print_warning "Falha ao limpar cache/remover pacotes órfãos (apt)."
        ;;
    arch)
        sudo pacman -Sc --noconfirm || print_warning "Falha ao limpar cache do Pacman."
        sudo pacman -Qtdq | sudo pacman -Rns --noconfirm || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
        ;;
    fedora)
        sudo dnf clean all || print_warning "Falha ao limpar cache (dnf)."
        sudo dnf autoremove -y || print_warning "Falha ao remover pacotes órfãos (dnf)."
        ;;
    *)
        print_warning "Gerenciador de pacotes desconhecido. Limpeza manual pode ser necessária."
        ;;
esac
print_info "Cache e pacotes órfãos limpos."

---
## 7. Finalização

print_header "Processo de Pós-Instalação para WSL2 Concluído!"

print_info "Algumas mudanças (como grupo 'docker' ou shell padrão) podem exigir que você reinicie a instância do WSL."
print_info "Para reiniciar a instância, feche todos os terminais WSL e, no PowerShell do Windows, execute 'wsl --shutdown'."
print_info "Em seguida, abra novamente sua distribuição WSL."

echo -e "\n${BLUE}========================================${NC}"
echo -e "${BLUE}Script Concluído!${NC}"
echo -e "${BLUE}========================================${NC}\n"
```