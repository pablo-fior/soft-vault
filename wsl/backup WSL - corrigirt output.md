output do export nao exibindo

```powershell
# Script de Backup Automático do WSL - Múltiplas Distribuições

# Autor: Backup WSL Script

  

# Configurações - AJUSTE CONFORME NECESSÁRIO

$BACKUP_DIR = 'C:\WSL\backups'

$WSL_DISTROS = @('Ubuntu', 'FedoraLinux-42', 'archlinux')

$MAX_BACKUPS = 7

$LOG_FILE = "$BACKUP_DIR\backup_log.txt"

$SHOW_EXPORT_OUTPUT = $true  # Definir como $false para ocultar output do export

  

# Função para escrever no log

function Write-Log {

    param([string]$Message)

    $timestamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'

    $logMessage = "[$timestamp] $Message"

    Write-Host $logMessage

    Add-Content -Path $LOG_FILE -Value $logMessage

}

  

# Função para obter distribuições WSL disponíveis

function Get-WSLDistributions {

    try {

        $wslOutput = wsl --list --quiet 2>$null

        if ($LASTEXITCODE -eq 0) {

            return $wslOutput | Where-Object { $_ -and $_.Trim() -ne '' }

        } else {

            return @()

        }

    } catch {

        Write-Log "Erro ao obter lista de distribuições WSL: $($_.Exception.Message)"

        return @()

    }

}

  

# Função para executar comando e capturar output em tempo real

function Invoke-CommandWithOutput {

    param(

        [string]$Command,

        [string[]]$Arguments,

        [string]$WorkingDirectory = $null

    )

    $processInfo = New-Object System.Diagnostics.ProcessStartInfo

    $processInfo.FileName = $Command

    $processInfo.RedirectStandardOutput = $true

    $processInfo.RedirectStandardError = $true

    $processInfo.UseShellExecute = $false

    $processInfo.CreateNoWindow = $true

    if ($Arguments) {

        $processInfo.Arguments = ($Arguments -join ' ')

    }

    if ($WorkingDirectory) {

        $processInfo.WorkingDirectory = $WorkingDirectory

    }

    $process = New-Object System.Diagnostics.Process

    $process.StartInfo = $processInfo

    # Eventos para capturar output em tempo real

    $outputBuilder = New-Object System.Text.StringBuilder

    $errorBuilder = New-Object System.Text.StringBuilder

    $outputAction = {

        param($sender, $e)

        if (-not [string]::IsNullOrEmpty($e.Data)) {

            $outputBuilder.AppendLine($e.Data)

            if ($SHOW_EXPORT_OUTPUT) {

                Write-Host "  [WSL] $($e.Data)" -ForegroundColor Cyan

                Add-Content -Path $LOG_FILE -Value "  [WSL] $($e.Data)"

            }

        }

    }

    $errorAction = {

        param($sender, $e)

        if (-not [string]::IsNullOrEmpty($e.Data)) {

            $errorBuilder.AppendLine($e.Data)

            if ($SHOW_EXPORT_OUTPUT) {

                Write-Host "  [WSL-ERR] $($e.Data)" -ForegroundColor Yellow

                Add-Content -Path $LOG_FILE -Value "  [WSL-ERR] $($e.Data)"

            }

        }

    }

    Register-ObjectEvent -InputObject $process -EventName OutputDataReceived -Action $outputAction | Out-Null

    Register-ObjectEvent -InputObject $process -EventName ErrorDataReceived -Action $errorAction | Out-Null

    try {

        $process.Start() | Out-Null

        $process.BeginOutputReadLine()

        $process.BeginErrorReadLine()

        $process.WaitForExit()

        return @{

            ExitCode = $process.ExitCode

            StandardOutput = $outputBuilder.ToString()

            StandardError = $errorBuilder.ToString()

        }

    } finally {

        Get-EventSubscriber | Where-Object { $_.SourceObject -eq $process } | Unregister-Event

        if (!$process.HasExited) {

            $process.Kill()

        }

        $process.Dispose()

    }

}

  

# Função para fazer backup de uma distribuição específica

function Backup-WSLDistribution {

    param(

        [string]$DistroName,

        [string]$BackupDirectory

    )

    Write-Log "--- Iniciando backup da distribuição: $DistroName ---"

    $timestamp = Get-Date -Format 'yyyyMMdd_HHmmss'

    $backupFileName = "${DistroName}_backup_${timestamp}.tar"

    $backupFilePath = Join-Path $BackupDirectory $backupFileName

    Write-Log "Arquivo de destino: $backupFilePath"

    try {

        # Parar a distribuição WSL antes do backup

        Write-Log "Parando a distribuição $DistroName..."

        wsl --terminate $DistroName 2>$null

        Start-Sleep -Seconds 2

        # Verificar se a distribuição está realmente parada

        $runningDistros = wsl --list --running --quiet 2>$null

        if ($runningDistros -contains $DistroName) {

            Write-Log "Aguardando $DistroName parar completamente..."

            Start-Sleep -Seconds 3

        }

        Write-Log "Executando export da distribuição $DistroName..."

        if ($SHOW_EXPORT_OUTPUT) {

            Write-Log "--- Output do comando wsl --export ---"

        }

        # Executar o comando de export com captura de output

        $result = Invoke-CommandWithOutput -Command 'wsl' -Arguments @('--export', $DistroName, $backupFilePath)

        if ($SHOW_EXPORT_OUTPUT) {

            Write-Log "--- Fim do output do comando wsl --export ---"

        }

        # Verificar resultado

        if ($result.ExitCode -eq 0 -and (Test-Path $backupFilePath)) {

            $backupSize = (Get-Item $backupFilePath).Length

            $backupSizeMB = [math]::Round($backupSize / 1MB, 2)

            Write-Log "SUCCESS: Backup de $DistroName concluído!"

            Write-Log "  Tamanho do arquivo: $backupSizeMB MB"

            Write-Log "  Localização: $backupFilePath"

            # Opcional: Comprimir o backup (descomente se desejar)

            # Write-Log "Comprimindo backup de $DistroName..."

            # Compress-Archive -Path $backupFilePath -DestinationPath "$backupFilePath.zip" -Force

            # Remove-Item $backupFilePath -Force

            # Write-Log "Backup comprimido: $backupFilePath.zip"

            return $true

        } else {

            Write-Log "ERROR: Falha ao criar backup de $DistroName"

            Write-Log "  Código de saída: $($result.ExitCode)"

            if (-not [string]::IsNullOrEmpty($result.StandardError)) {

                Write-Log "  Erro: $($result.StandardError.Trim())"

            }

            # Remover arquivo parcial se existir

            if (Test-Path $backupFilePath) {

                Write-Log "  Removendo arquivo parcial..."

                Remove-Item $backupFilePath -Force -ErrorAction SilentlyContinue

            }

            return $false

        }

    } catch {

        Write-Log "ERROR: Exceção durante backup de ${DistroName}: $($_.Exception.Message)"

        # Remover arquivo parcial se existir

        if (Test-Path $backupFilePath) {

            Remove-Item $backupFilePath -Force -ErrorAction SilentlyContinue

        }

        return $false

    }

}

  

# Função para limpar backups antigos

function Clean-OldBackups {

    param(

        [string]$DistroName,

        [string]$BackupDirectory,

        [int]$MaxBackups

    )

    Write-Log "Limpando backups antigos de $DistroName..."

    $backupPattern = "${DistroName}_backup_*.tar"

    $backupFiles = Get-ChildItem -Path $BackupDirectory -Filter $backupPattern -ErrorAction SilentlyContinue | Sort-Object CreationTime -Descending

    if ($backupFiles.Count -gt $MaxBackups) {

        $filesToDelete = $backupFiles | Select-Object -Skip $MaxBackups

        $deletedSize = 0

        foreach ($file in $filesToDelete) {

            $fileSize = [math]::Round($file.Length / 1MB, 2)

            $deletedSize += $fileSize

            Write-Log "  Removendo: $($file.Name) ($fileSize MB)"

            Remove-Item $file.FullName -Force -ErrorAction SilentlyContinue

        }

        Write-Log "Limpeza de $DistroName OK. Removidos $($filesToDelete.Count) backups ($deletedSize MB)"

        Write-Log "  Mantidos $MaxBackups backups mais recentes"

    } else {

        Write-Log "  Sem limpeza necessária para $DistroName. Total: $($backupFiles.Count)"

    }

}

  

# Função para mostrar progresso visual

function Show-Progress {

    param(

        [int]$Current,

        [int]$Total,

        [string]$Activity

    )

    $percent = [math]::Round(($Current / $Total) * 100, 0)

    $progressBar = '=' * [math]::Floor($percent / 5)

    $emptyBar = ' ' * (20 - [math]::Floor($percent / 5))

    Write-Log "[$progressBar$emptyBar] $percent% - $Activity"

}

  

# INÍCIO DO SCRIPT PRINCIPAL

Write-Log '========================================'

Write-Log '=== BACKUP MÚLTIPLO WSL - INICIANDO ==='

Write-Log '========================================'

Write-Log "Configurações:"

Write-Log "  Diretório: $BACKUP_DIR"

Write-Log "  Max backups por distro: $MAX_BACKUPS"

Write-Log "  Mostrar output do export: $SHOW_EXPORT_OUTPUT"

Write-Log ""

  

# Criar diretório de backup

if (!(Test-Path $BACKUP_DIR)) {

    New-Item -ItemType Directory -Path $BACKUP_DIR -Force | Out-Null

    Write-Log "Diretório criado: $BACKUP_DIR"

}

  

# Verificar WSL

Write-Log "Verificando disponibilidade do WSL..."

try {

    $wslVersion = wsl --version 2>$null

    if ($LASTEXITCODE -eq 0) {

        Write-Log "WSL disponível e funcionando"

    } else {

        Write-Log 'ERROR: WSL não está disponível'

        exit 1

    }

} catch {

    Write-Log "ERROR: Falha ao verificar WSL: $($_.Exception.Message)"

    exit 1

}

  

# Obter distribuições disponíveis

Write-Log "Obtendo lista de distribuições..."

$availableDistros = Get-WSLDistributions

Write-Log "Distribuições WSL disponíveis: $($availableDistros -join ', ')"

  

if ($availableDistros.Count -eq 0) {

    Write-Log 'ERROR: Nenhuma distribuição WSL encontrada'

    Write-Log 'Verifique se o WSL está instalado e configurado corretamente'

    exit 1

}

  

# Validar distribuições configuradas

Write-Log "Validando distribuições configuradas..."

$validDistros = @()

$invalidDistros = @()

  

foreach ($distro in $WSL_DISTROS) {

    if ($availableDistros -contains $distro) {

        $validDistros += $distro

    } else {

        $invalidDistros += $distro

    }

}

  

if ($invalidDistros.Count -gt 0) {

    Write-Log "WARNING: Distribuições configuradas não encontradas: $($invalidDistros -join ', ')"

}

  

if ($validDistros.Count -eq 0) {

    Write-Log 'ERROR: Nenhuma das distribuições configuradas foi encontrada!'

    Write-Log "  Configuradas: $($WSL_DISTROS -join ', ')"

    Write-Log "  Disponíveis: $($availableDistros -join ', ')"

    exit 1

}

  

Write-Log "Distribuições que serão processadas: $($validDistros -join ', ')"

Write-Log ""

  

# Contadores e estatísticas

$totalDistros = $validDistros.Count

$successfulBackups = 0

$failedBackups = 0

$startTime = Get-Date

  

Write-Log "=== INICIANDO PROCESSO DE BACKUP ==="

  

# Loop principal - fazer backup de cada distribuição

for ($i = 0; $i -lt $validDistros.Count; $i++) {

    $distro = $validDistros[$i]

    $currentNum = $i + 1

    Show-Progress -Current $currentNum -Total $totalDistros -Activity "Processando $distro"

    $distroStartTime = Get-Date

    $backupResult = Backup-WSLDistribution -DistroName $distro -BackupDirectory $BACKUP_DIR

    $distroEndTime = Get-Date

    $distroElapsed = $distroEndTime - $distroStartTime

    Write-Log "Tempo decorrido para $distro`: $($distroElapsed.ToString('mm\:ss'))"

    if ($backupResult) {

        $successfulBackups++

        Clean-OldBackups -DistroName $distro -BackupDirectory $BACKUP_DIR -MaxBackups $MAX_BACKUPS

    } else {

        $failedBackups++

    }

    # Pausa entre backups (exceto no último)

    if ($i -lt ($validDistros.Count - 1)) {

        Write-Log "Aguardando 3 segundos antes do próximo backup..."

        Start-Sleep -Seconds 3

    }

    Write-Log ""

}

  

# Calcular tempo total

$endTime = Get-Date

$totalElapsed = $endTime - $startTime

  

# Relatório final detalhado

Write-Log '========================================'

Write-Log '=== RELATÓRIO FINAL DETALHADO ==='

Write-Log '========================================'

Write-Log "Tempo total de execução: $($totalElapsed.ToString('hh\:mm\:ss'))"

Write-Log "Distribuições processadas: $totalDistros"

Write-Log "Backups bem-sucedidos: $successfulBackups"

Write-Log "Backups com falha: $failedBackups"

Write-Log "Taxa de sucesso: $([math]::Round(($successfulBackups / $totalDistros) * 100, 1))%"

  

if ($failedBackups -eq 0) {

    Write-Log 'RESULTADO: TODOS OS BACKUPS CONCLUÍDOS COM SUCESSO!'

} else {

    Write-Log 'RESULTADO: ALGUNS BACKUPS FALHARAM - Verifique os detalhes acima'

    Write-Log "Distribuições com falha: $failedBackups de $totalDistros"

}

  

# Informações do diretório de backup

try {

    $backupFiles = Get-ChildItem -Path $BACKUP_DIR -Filter "*.tar" -ErrorAction SilentlyContinue

    $totalBackupSize = ($backupFiles | Measure-Object -Property Length -Sum).Sum

    $totalBackupSizeGB = [math]::Round($totalBackupSize / 1GB, 2)

    Write-Log "Informações do diretório de backup:"

    Write-Log "  Total de arquivos .tar: $($backupFiles.Count)"

    Write-Log "  Tamanho total dos backups: $totalBackupSizeGB GB"

} catch {

    Write-Log "Não foi possível calcular informações do diretório de backup"

}

  

# Verificar espaço em disco

try {

    $drive = (Get-Item $BACKUP_DIR).Root

    $freeSpace = Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='$($drive.Name.TrimEnd('\'))'" | Select-Object -ExpandProperty FreeSpace

    $freeSpaceGB = [math]::Round($freeSpace / 1GB, 2)

    Write-Log "Espaço livre no disco: $freeSpaceGB GB"

    if ($freeSpaceGB -lt 5) {

        Write-Log 'WARNING: Pouco espaço em disco disponível!'

    } elseif ($freeSpaceGB -lt 10) {

        Write-Log 'INFO: Espaço em disco moderado - monitore o crescimento dos backups'

    }

} catch {

    Write-Log 'WARNING: Não foi possível verificar espaço em disco'

}

  

# Reiniciar distribuições (opcional)

Write-Log ""

Write-Log "Reiniciando distribuições WSL..."

$restartErrors = 0

foreach ($distro in $validDistros) {

    try {

        $result = wsl --distribution $distro --exec echo "Reiniciado" 2>$null

        if ($LASTEXITCODE -eq 0) {

            Write-Log "  $distro`: OK"

        } else {

            Write-Log "  $distro`: Aviso - pode não ter reiniciado corretamente"

            $restartErrors++

        }

    } catch {

        Write-Log "  $distro`: Erro no restart"

        $restartErrors++

    }

}

  

if ($restartErrors -eq 0) {

    Write-Log "Todas as distribuições foram reiniciadas com sucesso"

} else {

    Write-Log "Algumas distribuições podem não ter reiniciado corretamente ($restartErrors erros)"

}

  

Write-Log ""

Write-Log '========================================'

Write-Log 'SCRIPT DE BACKUP CONCLUÍDO'

Write-Log "Verifique o log completo em: $LOG_FILE"

Write-Log '========================================'

  

# Código de saída baseado no resultado

if ($failedBackups -eq 0) {

    exit 0

} else {

    exit 1

}
```