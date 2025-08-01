Este script deve ser executado no **PowerShell do Windows como Administrador**. Ele vai verificar se as distribuições Arch Linux e Fedora estão instaladas no WSL. Se não estiverem, ele as baixará e registrará.

- Windows powershell

```powershell
# SCRIPT POWERSHELL (EXECUTAR NO WINDOWS COMO ADMINISTRADOR)
# Nome do arquivo: Install-ArchLinuxWSL.ps1

Write-Host "Iniciando a instalação de Arch Linux para WSL2..." -ForegroundColor Cyan

# Define o nome da distro no WSL
$DistroName = "ArchLinuxWSL"
# URL para download do ArchWSL (fonte da comunidade, a mais próxima de "oficial" para Arch no WSL)
$InstallerUrl = "https://github.com/yuk7/ArchWSL/releases/latest/download/Arch.zip"
# Diretório temporário para download
$DownloadDir = "$env:TEMP\WSL_Distro_Downloads"
$ZipFile = Join-Path $DownloadDir "Arch.zip"
# Diretório onde o Arch.exe será extraído e executado
$InstallDir = "$env:LOCALAPPDATA\ArchLinuxWSL"

# --- Funções Auxiliares ---
Function Test-WslDistroInstalled {
    Param (
        [string]$Distro
    )
    $wslList = wsl.exe --list --quiet
    return ($wslList | Select-String -Pattern "^$Distro$") -ne $null
}

Function Ensure-DownloadDir {
    Param (
        [string]$Path
    )
    If (-not (Test-Path $Path)) {
        try {
            New-Item -ItemType Directory -Path $Path -ErrorAction Stop | Out-Null
            Write-Host "Diretório de download '$Path' criado." -ForegroundColor Green
        } catch {
            Write-Host "ERRO: Não foi possível criar o diretório de download '$Path'. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
            exit 1
        }
    } Else {
        Write-Host "Diretório de download '$Path' já existe." -ForegroundColor DarkYellow
    }
}

# --- Início do Script Principal ---

# 1. Verifica se a distro já está instalada
If (Test-WslDistroInstalled $DistroName) {
    Write-Host "Arch Linux ($DistroName) já está instalada. Nenhuma ação necessária." -ForegroundColor Green
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 0
}

Write-Host "Arch Linux ($DistroName) não encontrada. Iniciando processo de instalação..." -ForegroundColor Yellow

# 2. Garante o diretório de download
Ensure-DownloadDir $DownloadDir

# 3. Baixa o instalador do ArchWSL
Write-Host "Baixando ArchWSL de $InstallerUrl para $ZipFile..." -ForegroundColor Green
try {
    Invoke-WebRequest -Uri $InstallerUrl -OutFile $ZipFile -UseBasicParsing -ErrorAction Stop
    Write-Host "Download de ArchWSL concluído com sucesso!" -ForegroundColor Green
} catch {
    Write-Host "ERRO: Falha ao baixar ArchWSL. Verifique a URL, sua conexão ou as permissões. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 1
}

# 4. Cria o diretório de instalação e extrai o arquivo
Write-Host "Criando diretório de instalação '$InstallDir' e extraindo '$ZipFile'..." -ForegroundColor Green
If (-not (Test-Path $InstallDir)) {
    New-Item -ItemType Directory -Path $InstallDir | Out-Null
}
try {
    Expand-Archive -Path $ZipFile -DestinationPath $InstallDir -Force -ErrorAction Stop
    Write-Host "Extração de ArchWSL concluída com sucesso!" -ForegroundColor Green
} catch {
    Write-Host "ERRO: Falha ao extrair Arch.zip. O arquivo pode estar corrompido ou o diretório inacessível. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 1
}

# 5. Registra a distro no WSL
Write-Host "Registrando '$DistroName' no WSL..." -ForegroundColor Green
$ArchExePath = Join-Path $InstallDir "Arch.exe"

If (-not (Test-Path $ArchExePath)) {
    Write-Host "ERRO: Executável 'Arch.exe' não encontrado em '$InstallDir'. Algo deu errado na extração." -ForegroundColor Red
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 1
}

Write-Host "A instalação do Arch.exe pode levar alguns instantes e pode pedir para você criar um usuário e senha." -ForegroundColor Yellow
Write-Host "A instância será configurada para o usuário padrão '$env:USERNAME' no Windows." -ForegroundColor Cyan
try {
    Start-Process -FilePath "$ArchExePath" -ArgumentList "install", "--root", "--auto-create-user", "--default-user", "$env:USERNAME" -Wait -NoNewWindow -ErrorAction Stop
    Write-Host "Comando de instalação de Arch.exe executado. Verificando o resultado..." -ForegroundColor Green
} catch {
    Write-Host "ERRO: Falha ao executar 'Arch.exe install'. Verifique as permissões ou se o WSL está configurado corretamente. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 1
}

# 6. Verifica a instalação pós-comando
If (Test-WslDistroInstalled $DistroName) {
    Write-Host "Arch Linux ($DistroName) instalado e registrado com sucesso!" -ForegroundColor Green
    Write-Host "Definindo '$DistroName' como a distro padrão do WSL..." -ForegroundColor Green
    wsl.exe --set-default $DistroName
    Write-Host "Arch Linux definida como a distro padrão." -ForegroundColor Green
} Else {
    Write-Host "ERRO: Arch Linux ($DistroName) não foi detectada como instalada após a execução do comando. Algo pode ter falhado silenciosamente." -ForegroundColor Red
    Write-Host "Por favor, verifique manualmente o status com 'wsl --list --verbose'." -ForegroundColor Red
    exit 1
}

Write-Host "Limpeza de arquivos temporários de download..." -ForegroundColor Cyan
Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue

Write-Host "Instalação de Arch Linux no WSL concluída!" -ForegroundColor Cyan
Write-Host "Próximo passo: Acesse sua nova distro com 'wsl -d ArchLinuxWSL' e execute o script de pós-instalação para ambiente gráfico e Atuin." -ForegroundColor Yellow
Write-Host "Lembre-se de instalar um X Server (VcXsrv) e um servidor PulseAudio no Windows antes de tentar usar a GUI." -ForegroundColor DarkYellow
```

- Bash - Arch - Plasma
```bash
#!/bin/bash

# --- Variáveis de Configuração ---
USERNAME=$(whoami) # Deve ser o seu usuário normal no Arch, não o root
LOCALE="en_US.UTF-8" # ou pt_BR.UTF-8
KEYMAP="br-abnt2"
HOSTNAME="arch-kde-wsl"

# --- Cores para Saída do Script ---
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# --- Funções Auxiliares ---
print_header() { echo -e "\n${BLUE}========================================${NC}\n${BLUE}>>> $1 <<<${NC}\n${BLUE}========================================${NC}\n"; }
print_info() { echo -e "${GREEN}INFO: $1${NC}"; }
print_warning() { echo -e "${YELLOW}WARNING: $1${NC}"; }
print_error() { echo -e "${RED}ERROR: $1${NC}"; }
confirm_action() { read -p "$(echo -e "${YELLOW}Deseja continuar com a instalação de '$1'? (s/N) ${NC}")" choice; [[ "$choice" = [sS] ]]; }

# --- Início do Script ---
print_header "Iniciando Pós-Instalação do Arch Linux com KDE Plasma no WSL2"

# Verifica se é Arch Linux
if ! grep -q "ID=arch" /etc/os-release; then
    print_error "Este script é para Arch Linux. A distro detectada não é Arch."
    exit 1
fi

# Garante que o usuário tem sudo
if ! groups $USERNAME | grep -q "wheel"; then
    print_warning "Usuário '$USERNAME' não está no grupo 'wheel'. Adicionando..."
    sudo usermod -aG wheel $USERNAME
    print_warning "Usuário '$USERNAME' adicionado ao grupo 'wheel'. Você pode precisar reiniciar o WSL para que as permissões tenham efeito."
    print_warning "Execute 'exit' e reabra a distro Arch no WSL para continuar, ou execute como root a partir daqui se for um novo usuário."
    # Se for a primeira vez e o usuário não for root, podemos pedir para ele reiniciar.
    if [[ "$EUID" -ne 0 ]]; then
        print_error "Por favor, reinicie a distro WSL e execute o script novamente para garantir permissões de sudo."
        exit 1
    fi
fi

# --- 1. Sincronizar e Atualizar o Sistema ---
print_header "1. Sincronizando e Atualizando o Sistema"
print_info "Sincronizando espelhos e atualizando todos os pacotes..."
sudo pacman -Syu --noconfirm || { print_error "Falha ao sincronizar e atualizar o sistema. Verifique sua conexão e tente novamente."; exit 1; }
print_info "Sistema atualizado com sucesso!"

# --- 2. Configurações Essenciais ---
print_header "2. Configurações Essenciais"
print_info "Configurando Hostname para: $HOSTNAME"
echo "$HOSTNAME" | sudo tee /etc/hostname > /dev/null
echo "127.0.0.1 localhost" | sudo tee /etc/hosts > /dev/null
echo "::1       localhost" | sudo tee -a /etc/hosts > /dev/null
echo "127.0.1.1 $HOSTNAME.localdomain $HOSTNAME" | sudo tee -a /etc/hosts > /dev/null
print_info "Locale: $LOCALE, Keymap: $KEYMAP"
sudo sed -i "/^#$LOCALE UTF-8/s/^#//" /etc/locale.gen
sudo locale-gen
echo "LANG=$LOCALE" | sudo tee /etc/locale.conf > /dev/null
echo "KEYMAP=$KEYMAP" | sudo tee /etc/vconsole.conf > /dev/null
print_info "Fuso Horário: Usando o do Windows/WSL por padrão (timedatectl já sincroniza)."
sudo timedatectl set-timezone America/Sao_Paulo # Ou use 'timedatectl set-local-rtc 0' para desabilitar RTC local
print_info "Instalando NTP (systemd-timesyncd)..."
sudo pacman -S --noconfirm systemd-timesyncd || print_warning "Falha ao instalar systemd-timesyncd."
sudo systemctl enable systemd-timesyncd --now || print_warning "Falha ao habilitar systemd-timesyncd."

# --- 3. Instalação do KDE Plasma e Ferramentas Gráficas ---
print_header "3. Instalação do KDE Plasma"
if ! confirm_action "KDE Plasma e componentes essenciais"; then
    print_info "Instalação do KDE Plasma ignorada."
    exit 0 # Sai do script se o KDE não for desejado
fi

print_info "Instalando pacotes essenciais para KDE Plasma..."
# Pacotes essenciais para KDE Plasma
sudo pacman -S --noconfirm xorg-server xorg-xinit sddm plasma kde-applications mesa || print_error "Falha na instalação básica do KDE Plasma. Verifique as dependências."

# Para áudio no WSL (via PulseAudio)
print_info "Instalando PulseAudio e pavucontrol para áudio..."
sudo pacman -S --noconfirm pulseaudio pavucontrol || print_warning "Falha ao instalar PulseAudio."

# Ferramentas e utilitários úteis para ambiente gráfico
print_info "Instalando ferramentas adicionais..."
sudo pacman -S --noconfirm firefox dolphin konsole kate okular ark spectacle discover packagekit || print_warning "Falha ao instalar pacotes adicionais do KDE."

# Habilitar o Display Manager (SDDM)
print_info "Habilitando SDDM (Display Manager) para iniciar o ambiente gráfico..."
sudo systemctl enable sddm --now || print_warning "Falha ao habilitar e iniciar SDDM."
print_info "SDDM habilitado. Você precisará iniciar um servidor X no Windows para ver o KDE."

# --- 4. Configuração do X Server para WSL ---
print_header "4. Configuração do X Server para WSL"
print_info "Adicionando configuração de DISPLAY ao ~/.bashrc..."

# Detecta o IP do Windows Host dinamicamente
WINDOWS_HOST_IP=$(grep nameserver /etc/resolv.conf | awk '{print $2}')
# Exporta variáveis de ambiente para X Server e PulseAudio
echo "export DISPLAY=$WINDOWS_HOST_IP:0.0" >> "$HOME/.bashrc"
echo "export LIBGL_ALWAYS_INDIRECT=1" >> "$HOME/.bashrc" # Essencial para GL no WSL
echo "export PULSE_SERVER=tcp:$WINDOWS_HOST_IP" >> "$HOME/.bashrc" # Para áudio com PulseAudio
print_info "As variáveis DISPLAY, LIBGL_ALWAYS_INDIRECT e PULSE_SERVER foram adicionadas ao ~/.bashrc."
print_warning "Certifique-se de ter um servidor X (como VcXsrv) e um servidor PulseAudio no Windows."
print_warning "Para aplicar, feche e reabra este terminal WSL, ou execute 'source ~/.bashrc'."

# --- 5. Otimizações e Ferramentas Adicionais ---
print_header "5. Otimizações e Ferramentas Adicionais"
if confirm_action "Instalar Yay (AUR Helper)"; then
    print_info "Instalando Yay (AUR helper)..."
    sudo pacman -S --noconfirm git base-devel # Garante as dependências
    git clone https://aur.archlinux.org/yay.git /tmp/yay || print_warning "Falha ao clonar repositório yay."
    (cd /tmp/yay && makepkg -si --noconfirm) || print_warning "Falha ao compilar e instalar yay."
    rm -rf /tmp/yay
    print_info "Yay instalado com sucesso!"
fi

if confirm_action "Habilitar Repositório Multilib"; then
    print_info "Habilitando repositório Multilib..."
    sudo sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf
    sudo pacman -Syyu --noconfirm # Sincroniza e atualiza novamente após habilitar multilib
    print_info "Repositório Multilib habilitado e sistema atualizado."
fi

# --- 6. Limpeza Final ---
print_header "6. Limpeza Final"
print_info "Limpando cache do Pacman e removendo pacotes órfãos..."
sudo pacman -Sc --noconfirm || print_warning "Falha ao limpar cache do Pacman."
sudo pacman -Qtdq | sudo pacman -Rns --noconfirm || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
print_info "Cache e pacotes órfãos limpos."

print_header "Pós-Instalação do Arch Linux com KDE Plasma Concluída!"
print_warning "Para iniciar o KDE, certifique-se de que seu servidor X (e de áudio) está rodando no Windows."
print_warning "Então, você pode reiniciar a instância WSL ('wsl --shutdown' no PowerShell) e reabri-la."
print_warning "Ou, se SDDM não iniciar automaticamente, tente 'sudo systemctl start sddm'."
print_info "Você verá a tela de login do SDDM na janela do seu X Server no Windows."
```

