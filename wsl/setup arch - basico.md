---
state: "[[Volte aqui]]"
---
# Script de Pós-Instalação para Arch Linux (WSL)

O script a seguir irá:

1. Atualizar completamente o sistema.
    
2. Instalar pacotes de desenvolvimento essenciais (`base-devel`, `git`, etc.).
    
3. Configurar o `sudo` para não solicitar senha para o seu usuário (uma conveniência comum no WSL).
    
4. Clonar, compilar e instalar o `yay`, um popular AUR helper.
    
5. Limpar os arquivos de instalação ao final.

### Com sudo

```bash
#!/bin/bash
# ==============================================================================
# Script de Pós-Instalação para Arch Linux no WSL
#
# Autor: Gemini
# Data: 25/07/2025
#
# DESCRIÇÃO:
# Este script automatiza os passos iniciais de configuração de uma nova
# instância do Arch Linux no WSL. Ele atualiza o sistema, instala
# ferramentas essenciais e configura o AUR helper 'yay'.
#
# USO:
# 1. Salve este código em um arquivo, por exemplo: setup_arch.sh
# 2. Dê permissão de execução: chmod +x setup_arch.sh
# 3. Execute o script: ./setup_arch.sh
# ==============================================================================

# -- Início da Execução do Script --
# Imprime uma mensagem inicial para informar o usuário sobre o que está acontecendo.
echo "🚀 Iniciando a configuração pós-instalação para Arch Linux no WSL..."
echo "--------------------------------------------------------------------"

# --- PASSO 1: ATUALIZAÇÃO COMPLETA DO SISTEMA ---
# A primeira e mais crucial etapa em um novo sistema.
# O comando 'pacman -Syu' sincroniza os bancos de dados de pacotes com os servidores
# e atualiza todos os pacotes que estão desatualizados.
# A flag '--noconfirm' responde 'sim' automaticamente a todas as prompts de confirmação,
# permitindo que o script rode sem interrupção.
echo "🔄 Sincronizando bancos de dados e atualizando o sistema com pacman..."
sudo pacman -Syu --noconfirm

# --- PASSO 2: INSTALAÇÃO DE PACOTES ESSENCIAIS ---
# Instala um conjunto de pacotes fundamentais.
# 'base-devel': Contém ferramentas como make, gcc, etc., necessárias para compilar pacotes (como o yay).
# 'git': Essencial para controle de versão e para clonar repositórios da AUR.
# 'sudo': Garante que o utilitário sudo esteja presente e configurado.
# 'wget', 'curl': Ferramentas de linha de comando para download de arquivos.
# 'neovim': Um editor de texto moderno e poderoso.
# A flag '--needed' impede que pacotes já instalados e atualizados sejam reinstalados.
echo "📦 Instalando pacotes essenciais (base-devel, git, sudo, curl, wget, neovim)..."
sudo pacman -S --needed --noconfirm base-devel git sudo wget curl neovim

# --- PASSO 3: CONFIGURAÇÃO DO SUDO SEM SENHA ---
# Esta etapa é uma conveniência para ambientes de desenvolvimento como o WSL.
# ATENÇÃO: Isso representa um risco de segurança em sistemas de produção ou multiusuário.
# No WSL, onde você é o único usuário, o risco é consideravelmente menor.
echo "🔑 Configurando o sudo para não exigir senha para o usuário atual..."
# Obtém o nome do usuário que está executando o script.
USERNAME=$(whoami)
# Cria um novo arquivo de configuração dentro de /etc/sudoers.d/.
# Esta é a maneira mais segura e recomendada de estender as regras do sudo,
# pois evita a edição direta do arquivo /etc/sudoers, que é mais arriscada.
echo "$USERNAME ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/99-nopasswd-$USERNAME > /dev/null

# --- PASSO 4: INSTALAÇÃO DO YAY (AUR HELPER) ---
# O yay não pode ser compilado pelo usuário 'root'. Portanto, todo o processo
# é feito com as permissões do usuário padrão.
echo "✨ Instalando o AUR helper 'yay'..."

# Navega para o diretório home do usuário para garantir permissões de escrita.
cd ~

# Clona o repositório 'yay' a partir do Arch User Repository (AUR).
echo "    -> Clonando o repositório do yay..."
git clone https://aur.archlinux.org/yay.git

# Entra no diretório recém-clonado.
cd yay

# Compila e instala o pacote.
# 'makepkg' é a ferramenta usada para construir pacotes no Arch.
# A flag '-s' instala automaticamente as dependências necessárias.
# A flag '-i' instala o pacote após a compilação bem-sucedida.
echo "    -> Compilando e instalando o yay (isso pode levar alguns minutos)..."
makepkg -si --noconfirm

# --- PASSO 5: LIMPEZA ---
# Após a instalação bem-sucedida, não precisamos mais dos arquivos de compilação.
echo "🧹 Limpando os arquivos de instalação do yay..."
# Retorna ao diretório anterior (home).
cd ..
# Remove a pasta do repositório 'yay' de forma recursiva e forçada.
rm -rf yay

# --- FINALIZAÇÃO ---
echo "--------------------------------------------------------------------"
echo "✅ Script finalizado com sucesso!"
echo ""
echo "O seu ambiente Arch Linux no WSL foi atualizado e configurado."
echo "O helper 'yay' está instalado e pronto para uso."
echo ""
echo "Para sincronizar todos os pacotes (oficiais e da AUR) no futuro, use o comando:"
echo "   yay -Syu"
echo "--------------------------------------------------------------------"
```

### Sem sudo

```bash
#!/bin/bash
# ==============================================================================
# Script de Pós-Instalação para Arch Linux no WSL (Versão sem sudo interno)
#
# Autor: Gemini
# Data: 25/07/2025
#
# DESCRIÇÃO:
# Este script deve ser executado com privilégios de root (ex: 'sudo ./script.sh').
# Ele automatiza os passos iniciais de configuração de uma nova instância
# do Arch Linux no WSL, atualizando o sistema, instalando ferramentas
# essenciais e configurando o AUR helper 'yay'.
# ==============================================================================

# -- Início da Execução do Script --
# Imprime uma mensagem inicial para informar o usuário sobre o que está acontecendo.
echo "🚀 Iniciando a configuração pós-instalação para Arch Linux no WSL..."
echo "--------------------------------------------------------------------"

# --- VERIFICAÇÃO DE PRIVILÉGIOS ---
# O script agora verifica se está sendo executado como root.
# Se o ID do usuário (UID) não for 0 (que é o UID do root), ele exibe um erro e sai.
if [ "$EUID" -ne 0 ]; then
  echo "❌ Erro: Este script precisa ser executado com privilégios de root."
  echo "Por favor, execute-o usando: sudo $0"
  exit 1
fi

# --- PASSO 1: ATUALIZAÇÃO COMPLETA DO SISTEMA ---
# O comando 'pacman -Syu' sincroniza os bancos de dados de pacotes com os servidores
# e atualiza todos os pacotes que estão desatualizados.
# Como o script inteiro já está rodando como root, o 'sudo' não é mais necessário aqui.
echo "🔄 Sincronizando bancos de dados e atualizando o sistema com pacman..."
pacman -Syu --noconfirm

# --- PASSO 2: INSTALAÇÃO DE PACOTES ESSENCIAIS ---
# Instala pacotes fundamentais. Como o script já tem privilégios de root,
# o pacman pode instalar os pacotes diretamente.
echo "📦 Instalando pacotes essenciais (base-devel, git, sudo, curl, wget, neovim)..."
pacman -S --needed --noconfirm base-devel git sudo wget curl neovim

# --- PASSO 3: INSTALAÇÃO DO YAY (AUR HELPER) ---
# ATENÇÃO: A compilação de pacotes da AUR não deve ser feita como root por segurança.
# Para contornar isso, o script identifica o usuário padrão do WSL (ou o usuário que usou sudo)
# e executa a parte de compilação como esse usuário.
echo "✨ Instalando o AUR helper 'yay'..."

# Identifica o usuário não-root que invocou o sudo ou o usuário padrão do WSL.
# A variável de ambiente $SUDO_USER contém o nome do usuário original.
REAL_USER=${SUDO_USER:-$(logname)}
if [ -z "$REAL_USER" ]; then
    echo "❌ Não foi possível determinar o usuário não-root. Impossível instalar o yay."
    exit 1
fi
echo "    -> Trocando para o usuário '$REAL_USER' para compilar o yay..."
HOME_DIR=$(eval echo ~$REAL_USER)

# Executa a clonagem, compilação e instalação como o usuário não-root.
# O comando 'runuser' ou 'su' é usado para rebaixar os privilégios temporariamente.
runuser -l $REAL_USER -c "cd $HOME_DIR && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si --noconfirm"

# --- PASSO 4: LIMPEZA ---
# Remove a pasta de compilação que foi criada no diretório home do usuário.
echo "🧹 Limpando os arquivos de instalação do yay..."
rm -rf $HOME_DIR/yay

# --- FINALIZAÇÃO ---
echo "--------------------------------------------------------------------"
echo "✅ Script finalizado com sucesso!"
echo ""
echo "O seu ambiente Arch Linux no WSL foi atualizado e configurado."
echo "O helper 'yay' está instalado e pronto para uso."
echo ""
echo "Para sincronizar todos os pacotes no futuro, use o comando como seu usuário normal:"
echo "   yay -Syu"
echo "--------------------------------------------------------------------"
```