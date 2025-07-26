Este script instalará o **Ubuntu** usando o método mais direto e oficial da Microsoft: `wsl --install -d Ubuntu`, com um fallback para `winget`.

```powershell
# SCRIPT POWERSHELL (EXECUTAR NO WINDOWS COMO ADMINISTRADOR)
# Nome do arquivo: Install-UbuntuWSL.ps1

Write-Host "Iniciando a instalação de Ubuntu para WSL2..." -ForegroundColor Cyan

# Define o nome da distro no WSL
$DistroName = "Ubuntu"
# ID do pacote Ubuntu na Microsoft Store / Winget
$WingetId = "Canonical.Ubuntu"

# --- Funções Auxiliares ---
Function Test-WslDistroInstalled {
    Param (
        [string]$Distro
    )
    $wslList = wsl.exe --list --quiet
    return ($wslList | Select-String -Pattern "^$Distro$") -ne $null
}

# --- Início do Script Principal ---

# 1. Verifica se a distro já está instalada
If (Test-WslDistroInstalled $DistroName) {
    Write-Host "Ubuntu ($DistroName) já está instalada. Nenhuma ação necessária." -ForegroundColor Green
    exit 0
}

Write-Host "Ubuntu ($DistroName) não encontrada. Iniciando processo de instalação..." -ForegroundColor Yellow

# 2. Tenta instalar via 'wsl --install -d Ubuntu' (método mais oficial e direto)
Write-Host "Tentando instalar Ubuntu via 'wsl --install -d Ubuntu'..." -ForegroundColor Green
try {
    wsl.exe --install -d Ubuntu -ErrorAction Stop
    If ($LASTEXITCODE -eq 0) {
        Write-Host "Ubuntu ($DistroName) instalado com sucesso via 'wsl --install'!" -ForegroundColor Green
    } Else {
        Write-Host "AVISO: 'wsl --install -d Ubuntu' falhou com código de saída $LASTEXITCODE. Tentando método de fallback (Winget)." -ForegroundColor Yellow
        $WslInstallFailed = $true
    }
} catch {
    Write-Host "AVISO: Erro ao executar 'wsl --install -d Ubuntu'. Detalhes: $($_.Exception.Message). Tentando método de fallback (Winget)." -ForegroundColor Yellow
    $WslInstallFailed = $true
}

# 3. Fallback para instalação via Winget se 'wsl --install' falhou
If ($WslInstallFailed -or -not (Test-WslDistroInstalled $DistroName)) {
    If (Get-Command winget -ErrorAction SilentlyContinue) {
        Write-Host "Winget encontrado. Tentando instalar Ubuntu via winget (Microsoft Store ID: $WingetId)..." -ForegroundColor Green
        try {
            winget install --id $WingetId --accept-package-agreements --accept-source-agreements --silent --disable-interactivity -ErrorAction Stop
            If ($LASTEXITCODE -eq 0) {
                Write-Host "Ubuntu ($DistroName) instalado com sucesso via Winget!" -ForegroundColor Green
            } Else {
                Write-Host "ERRO: A instalação do Ubuntu via Winget falhou com código de saída $LASTEXITCODE." -ForegroundColor Red
                Write-Host "Pode ser necessário aceitar os termos de licença manualmente ou o pacote já está instalado." -ForegroundColor Yellow
            }
        } catch {
            Write-Host "ERRO: Erro ao executar winget install para Ubuntu. Detalhes: $($_.Exception.Message)" -ForegroundColor Red
        }
    } Else {
        Write-Host "ERRO: Nem 'wsl --install' nem 'winget' foram capazes de instalar Ubuntu." -ForegroundColor Red
        Write-Host "Por favor, instale Ubuntu manualmente pela Microsoft Store." -ForegroundColor Yellow
        exit 1
    }
}

# 4. Verifica a instalação pós-comando e configura padrão
If (Test-WslDistroInstalled $DistroName) {
    Write-Host "Ubuntu ($DistroName) instalado e registrado com sucesso!" -ForegroundColor Green
    Write-Host "Definindo '$DistroName' como a distro padrão do WSL..." -ForegroundColor Green
    wsl.exe --set-default $DistroName
    Write-Host "Ubuntu definida como a distro padrão." -ForegroundColor Green
    Write-Host "Na primeira execução de 'wsl -d Ubuntu', você será solicitado a criar um usuário e senha." -ForegroundColor Cyan
} Else {
    Write-Host "ERRO: Ubuntu ($DistroName) não foi detectada como instalada. Algo pode ter falhado durante a instalação." -ForegroundColor Red
    Write-Host "Por favor, verifique manualmente o status com 'wsl --list --verbose' e considere a instalação manual pela Microsoft Store." -ForegroundColor Red
    exit 1
}

Write-Host "Instalação de Ubuntu no WSL concluída!" -ForegroundColor Cyan
Write-Host "Próximo passo: Acesse sua nova distro com 'wsl -d Ubuntu' e execute o script de pós-instalação para ambiente gráfico." -ForegroundColor Yellow
Write-Host "Lembre-se de instalar um X Server (VcXsrv) e um servidor PulseAudio no Windows antes de tentar usar a GUI." -ForegroundColor DarkYellow
```