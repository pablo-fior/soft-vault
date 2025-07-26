Este script instalará o **Fedora** usando o `winget` (Microsoft Store), com um fallback para download e instalação direta do `.appx` se o `winget` não estiver disponível.

```powershell
# SCRIPT POWERSHELL (EXECUTAR NO WINDOWS COMO ADMINISTRADOR)
# Nome do arquivo: Install-FedoraWSL.ps1

Write-Host "Iniciando a instalação de Fedora para WSL2..." -ForegroundColor Cyan

# Define o nome da distro no WSL
$DistroName = "FedoraWSL"
# ID do pacote Fedora Remix for WSL na Microsoft Store / Winget
$WingetId = "9NZL5N6C5FPX"
# URL do fallback para o Appx (pode precisar de atualização para a versão mais recente)
$AppxUrl = "https://github.com/WhitewaterFoundry/Fedora-Remix-for-WSL/releases/latest/download/Fedora.appx"
# Diretório temporário para download
$DownloadDir = "$env:TEMP\WSL_Distro_Downloads"
$AppxFile = Join-Path $DownloadDir "Fedora.appx"

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
    Write-Host "Fedora ($DistroName) já está instalada. Nenhuma ação necessária." -ForegroundColor Green
    Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
    Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
    exit 0
}

Write-Host "Fedora ($DistroName) não encontrada. Iniciando processo de instalação..." -ForegroundColor Yellow

# 2. Tenta instalar via Winget (método preferencial)
If (Get-Command winget -ErrorAction SilentlyContinue) {
    Write-Host "Winget encontrado. Tentando instalar Fedora Remix for WSL via winget (Microsoft Store ID: $WingetId)..." -ForegroundColor Green
    try {
        winget install --id $WingetId --accept-package-agreements --accept-source-agreements --silent --disable-interactivity -ErrorAction Stop
        If ($LASTEXITCODE -eq 0) {
            Write-Host "Fedora ($DistroName) instalado com sucesso via Winget!" -ForegroundColor Green
            # Garante que a distro seja registrada no WSL (winget geralmente faz isso)
            If (-not (Test-WslDistroInstalled $DistroName)) {
                Write-Host "AVISO: Fedora instalada via Winget, mas não detectada imediatamente no WSL. Tentando iniciar para registrar..." -ForegroundColor Yellow
                Start-Process -FilePath "wsl.exe" -ArgumentList "-d $DistroName --cd /mnt/c/Users/$env:USERNAME -e exit" -Wait -NoNewWindow
            }
        } Else {
            Write-Host "AVISO: A instalação do Fedora via Winget falhou com código de saída $LASTEXITCODE. Tentando método de fallback (Appx)." -ForegroundColor Yellow
            $WingetFailed = $true
        }
    } catch {
        Write-Host "AVISO: Erro ao executar winget install. Detalhes: $($_.Exception.Message). Tentando método de fallback (Appx)." -ForegroundColor Yellow
        $WingetFailed = $true
    }
} Else {
    Write-Host "AVISO: Winget não encontrado. Tentando método de fallback (instalação direta do Appx)." -ForegroundColor Yellow
    $WingetFailed = $true
}

# 3. Fallback para instalação via Appx se Winget falhou ou não existe
If ($WingetFailed -or -not (Test-WslDistroInstalled $DistroName)) {
    Write-Host "Iniciando instalação de Fedora via pacote Appx ($AppxUrl)..." -ForegroundColor Yellow
    Ensure-DownloadDir $DownloadDir

    Write-Host "Baixando Fedora.appx para $AppxFile..." -ForegroundColor Green
    try {
        Invoke-WebRequest -Uri $AppxUrl -OutFile $AppxFile -UseBasicParsing -ErrorAction Stop
        Write-Host "Download de Fedora.appx concluído com sucesso!" -ForegroundColor Green
    } catch {
        Write-Host "ERRO: Falha ao baixar Fedora.appx. Verifique a URL, sua conexão ou as permissões. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
        Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
        exit 1
    }

    Write-Host "Instalando Fedora.appx. Esta etapa pode demorar..." -ForegroundColor Green
    try {
        # Usar Add-AppxPackage para registrar e instalar o pacote
        Add-AppxPackage -Path $AppxFile -Register -ErrorAction Stop
        Write-Host "Fedora ($DistroName) instalado com sucesso via Appx!" -ForegroundColor Green
    } catch {
        Write-Host "ERRO: Falha ao instalar Fedora.appx. O arquivo pode estar corrompido ou o pacote já está registrado. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
        Write-Host "Limpeza de arquivos temporários..." -ForegroundColor Cyan
        Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue
        exit 1
    }
}

# 4. Verifica a instalação pós-comando e configura padrão
If (Test-WslDistroInstalled $DistroName) {
    Write-Host "Fedora ($DistroName) instalado e registrado com sucesso!" -ForegroundColor Green
    Write-Host "Definindo '$DistroName' como a distro padrão do WSL..." -ForegroundColor Green
    wsl.exe --set-default $DistroName
    Write-Host "Fedora definida como a distro padrão." -ForegroundColor Green
    Write-Host "Para definir o usuário padrão no Fedora, execute: '$DistroName config --default-user seu_usuario' no PowerShell do Windows." -ForegroundColor Yellow
    Write-Host "Ou abra a distro ('wsl -d $DistroName') e siga as instruções para criar seu usuário na primeira execução." -ForegroundColor Cyan
} Else {
    Write-Host "ERRO: Fedora ($DistroName) não foi detectada como instalada. Algo pode ter falhado durante a instalação Appx ou Winget." -ForegroundColor Red
    Write-Host "Por favor, verifique manualmente o status com 'wsl --list --verbose' e considere a instalação manual pela Microsoft Store." -ForegroundColor Red
    exit 1
}

Write-Host "Limpeza de arquivos temporários de download..." -ForegroundColor Cyan
Remove-Item $DownloadDir -Recurse -Force -ErrorAction SilentlyContinue

Write-Host "Instalação de Fedora no WSL concluída!" -ForegroundColor Cyan
Write-Host "Próximo passo: Acesse sua nova distro com 'wsl -d FedoraWSL' e configure seu ambiente gráfico." -ForegroundColor Yellow
Write-Host "Lembre-se de instalar um X Server (VcXsrv) e um servidor PulseAudio no Windows antes de tentar usar a GUI." -ForegroundColor DarkYellow
```

- Bash - Fedora - Gnome

```bash
#!/bin/bash

# --- Variáveis de Configuração ---
USERNAME=$(whoami) # Deve ser o seu usuário normal no Fedora, não o root
LOCALE="en_US.UTF-8" # ou pt_BR.UTF-8
HOSTNAME="fedora-gnome-wsl"

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
print_header "Iniciando Pós-Instalação do Fedora com GNOME no WSL2"

# Verifica se é Fedora Linux
if ! grep -q "ID=fedora" /etc/os-release; then
    print_error "Este script é para Fedora Linux. A distro detectada não é Fedora."
    exit 1
fi

# Adiciona o usuário ao grupo 'wheel' para sudo (se ainda não estiver)
if ! groups $USERNAME | grep -q "wheel"; then
    print_warning "Usuário '$USERNAME' não está no grupo 'wheel'. Adicionando..."
    sudo usermod -aG wheel $USERNAME
    print_warning "Usuário '$USERNAME' adicionado ao grupo 'wheel'. Você pode precisar reiniciar o WSL para que as permissões tenham efeito."
    print_warning "Execute 'exit' e reabra a distro Fedora no WSL para continuar, ou execute como root a partir daqui se for um novo usuário."
    if [[ "$EUID" -ne 0 ]]; then
        print_error "Por favor, reinicie a distro WSL e execute o script novamente para garantir permissões de sudo."
        exit 1
    fi
fi

# --- 1. Sincronizar e Atualizar o Sistema ---
print_header "1. Sincronizando e Atualizando o Sistema"
print_info "Sincronizando espelhos e atualizando todos os pacotes..."
sudo dnf upgrade -y || { print_error "Falha ao sincronizar e atualizar o sistema. Verifique sua conexão e tente novamente."; exit 1; }
print_info "Sistema atualizado com sucesso!"

# --- 2. Configurações Essenciais ---
print_header "2. Configurações Essenciais"
print_info "Configurando Hostname para: $HOSTNAME"
echo "$HOSTNAME" | sudo tee /etc/hostname > /dev/null
echo "127.0.0.1 localhost" | sudo tee /etc/hosts > /dev/null
echo "::1       localhost" | sudo tee -a /etc/hosts > /dev/null
echo "127.0.1.1 $HOSTNAME.localdomain $HOSTNAME" | sudo tee -a /etc/hosts > /dev/null
print_info "Locale: $LOCALE"
sudo localectl set-locale LANG="$LOCALE"
print_info "Fuso Horário: Usando o do Windows/WSL por padrão (timedatectl já sincroniza)."
sudo timedatectl set-timezone America/Sao_Paulo # Define o fuso horário
print_info "Instalando NTP (systemd-timesyncd)..."
sudo dnf install -y systemd-timesyncd || print_warning "Falha ao instalar systemd-timesyncd."
sudo systemctl enable systemd-timesyncd --now || print_warning "Falha ao habilitar systemd-timesyncd."

# --- 3. Instalação do GNOME e Ferramentas Gráficas ---
print_header "3. Instalação do GNOME Desktop Environment"
if ! confirm_action "GNOME e componentes essenciais"; then
    print_info "Instalação do GNOME ignorada."
    exit 0 # Sai do script se o GNOME não for desejado
fi

print_info "Instalando GNOME Desktop Environment..."
# Use @gnome-desktop-environment para instalar o grupo GNOME
sudo dnf groupinstall -y "GNOME Desktop Environment" || print_error "Falha na instalação do grupo GNOME. Verifique as dependências."
sudo dnf install -y gnome-shell gdm gnome-tweaks gnome-extensions-app || print_error "Falha na instalação de pacotes GNOME adicionais."

# Para áudio no WSL (via PulseAudio)
print_info "Instalando PulseAudio e pavucontrol para áudio..."
sudo dnf install -y pulseaudio pavucontrol || print_warning "Falha ao instalar PulseAudio."

# Ferramentas e utilitários úteis para ambiente gráfico
print_info "Instalando ferramentas adicionais..."
sudo dnf install -y firefox nautilus gnome-terminal gedit evince eog file-roller || print_warning "Falha ao instalar pacotes adicionais do GNOME."

# Habilitar o Display Manager (GDM)
print_info "Habilitando GDM (Display Manager) para iniciar o ambiente gráfico..."
sudo systemctl enable gdm --now || print_warning "Falha ao habilitar e iniciar GDM."
print_info "GDM habilitado. Você precisará iniciar um servidor X no Windows para ver o GNOME."

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
# Para Fedora, não há AUR. Considere RPM Fusion ou Flatpak.
if confirm_action "Instalar e configurar Flatpak"; then
    print_info "Instalando Flatpak..."
    sudo dnf install -y flatpak || print_warning "Falha ao instalar flatpak."
    print_info "Adicionando repositório Flathub..."
    flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo || print_warning "Falha ao adicionar repositório Flathub."
    print_info "Flatpak configurado."
fi

# --- 6. Limpeza Final ---
print_header "6. Limpeza Final"
print_info "Limpando cache do DNF e removendo pacotes órfãos..."
sudo dnf clean all || print_warning "Falha ao limpar cache do DNF."
sudo dnf autoremove -y || print_warning "Nenhum pacote órfão para remover ou falha na remoção."
print_info "Cache e pacotes órfãos limpos."

print_header "Pós-Instalação do Fedora com GNOME Concluída!"
print_warning "Para iniciar o GNOME, certifique-se de que seu servidor X (e de áudio) está rodando no Windows."
print_warning "Então, você pode reiniciar a instância WSL ('wsl --shutdown' no PowerShell) e reabri-la."
print_warning "Ou, se GDM não iniciar automaticamente, tente 'sudo systemctl start gdm'."
print_info "Você verá a tela de login do GDM na janela do seu X Server no Windows."
```