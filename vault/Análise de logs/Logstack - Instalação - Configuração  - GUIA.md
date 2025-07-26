
---

## Guia de Instalação e Configuração do Logstash

### Pré-requisitos

Antes de começar, certifique-se de que seu servidor atende aos seguintes requisitos:

1. **Sistema Operacional:** Este guia assume um sistema operacional baseado em Debian/Ubuntu ou RHEL/CentOS. (Se for outro SO, por favor, me avise para eu adaptar).
    
2. **Java Development Kit (JDK):** O Logstash é uma aplicação baseada em Java, então o JDK é essencial. Recomenda-se a versão 11 ou superior.
    
3. **Acesso Root/Sudo:** Você precisará de permissões de administrador para instalar pacotes e configurar serviços.
    
4. **Conexão com a Internet:** Para baixar os pacotes necessários.
    

---

### Passo 1: Instalação do Java Development Kit (JDK)

É crucial ter o Java instalado e configurado corretamente.

#### Para sistemas baseados em Debian/Ubuntu:

Bash

```
sudo apt update
sudo apt install openjdk-11-jdk -y
```

#### Para sistemas baseados em RHEL/CentOS:

Bash

```
sudo yum update
sudo yum install java-11-openjdk -y
```

Após a instalação, verifique a versão do Java:

Bash

```
java -version
```

Você deve ver uma saída indicando a versão 11 (ou superior) do OpenJDK.

---

### Passo 2: Instalação do Logstash

A Elastic (empresa por trás do Logstash, Elasticsearch e Kibana) fornece repositórios oficiais, o que facilita muito a instalação e futuras atualizações.

#### Para sistemas baseados em Debian/Ubuntu:

1. **Importar a chave GPG da Elastic:**
    
    Bash
    
    ```
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
    ```
    
2. **Adicionar o repositório Elastic ao seu sistema:**
    
    Bash
    
    ```
    echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
    ```
    
    _(Nota: A URL `/packages/8.x/` indica a versão principal. Verifique no site oficial da Elastic a versão mais recente se quiser uma diferente da série 8.x.)_
    
3. **Atualizar o índice de pacotes e instalar o Logstash:**
    
    Bash
    
    ```
    sudo apt update
    sudo apt install logstash -y
    ```
    

#### Para sistemas baseados em RHEL/CentOS:

1. **Criar o arquivo de repositório da Elastic:**
    
    Bash
    
    ```
    sudo nano /etc/yum.repos.d/logstash.repo
    ```
    
2. **Adicione o seguinte conteúdo ao arquivo e salve (Ctrl+X, Y, Enter para nano):**
    
    Ini, TOML
    
    ```
    [logstash-8.x]
    name=Elastic repository for 8.x packages
    baseurl=https://artifacts.elastic.co/packages/8.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    ```
    
    _(Nota: Novamente, a URL `/packages/8.x/` indica a versão principal. Verifique no site oficial da Elastic a versão mais recente se quiser uma diferente da série 8.x.)_
    
3. **Instalar o Logstash:**
    
    Bash
    
    ```
    sudo yum install logstash -y
    ```
    

---

### Passo 3: Configuração Básica do Logstash

O Logstash utiliza arquivos de configuração com uma sintaxe específica (Pipelines) para definir suas entradas (inputs), filtros (filters) e saídas (outputs).

O diretório padrão para as configurações do Logstash é `/etc/logstash/conf.d/`.

1. Crie um arquivo de configuração de exemplo:
    
    Vamos criar um pipeline simples para testar. Este pipeline irá ler do stdin (entrada padrão), aplicar um filtro básico de data e enviar para o stdout (saída padrão).
    
    Bash
    
    ```
    sudo nano /etc/logstash/conf.d/01-simple.conf
    ```
    
2. **Adicione o seguinte conteúdo:**
    
    Ruby
    
    ```
    input {
      stdin { }
    }
    
    filter {
      date {
        match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
        target => "@timestamp"
      }
    }
    
    output {
      stdout {
        codec => rubydebug
      }
    }
    ```
    
    - **Input (`stdin`):** O Logstash vai esperar por entrada de texto no terminal.
        
    - **Filter (`date`):** Tenta parsear um campo chamado `timestamp` para o `@timestamp` padrão do Logstash. (Você pode remover este filtro se não tiver um campo `timestamp` no seu teste inicial).
        
    - **Output (`stdout` com `rubydebug` codec):** Exibe o evento processado de forma legível no terminal.
        
3. **Salve e feche o arquivo.**
    

---

### Passo 4: Testando a Configuração do Logstash

Antes de iniciar o serviço, é bom testar a sintaxe da sua configuração.

Bash

```
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/01-simple.conf --config.test_and_exit
```

Se a configuração estiver correta, você verá uma mensagem como `Configuration OK`.

---

### Passo 5: Iniciando e Habilitando o Logstash

Após o teste, você pode iniciar e habilitar o serviço Logstash para que ele inicie automaticamente com o sistema.

Bash

```
sudo systemctl start logstash
sudo systemctl enable logstash
```

Para verificar o status do serviço:

Bash

```
sudo systemctl status logstash
```

Você deve ver `active (running)`.

---

### Passo 6: Verificando o Logstash em Funcionamento (com o exemplo simples)

Já que configuramos o Logstash para ler do `stdin` e escrever no `stdout`, podemos interagir com ele.

1. **Comece um teste de input:**
    
    Bash
    
    ```
    sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/01-simple.conf
    ```
    
    O Logstash vai iniciar e esperar por sua entrada.
    
2. **Digite algo e pressione Enter:**
    
    ```
    Hello Logstash!
    ```
    
3. **Você deverá ver uma saída JSON formatada similar a esta:**
    
    JSON
    
    ```
    {
           "message" => "Hello Logstash!",
          "@version" => "1",
        "@timestamp" => "2025-07-16T17:56:19.000Z",
              "host" => "your-server-hostname"
    }
    ```
    
    Pressione `Ctrl+C` para sair do teste.
    

---

### Próximos Passos e Considerações Importantes:

- **Pipelines Reais:** O pipeline de exemplo é muito básico. Para investigação de logs, você vai querer configurar **inputs** para ler logs de arquivos (`file`), Syslog (`syslog`), ou outras fontes. Os **filters** serão usados para parsear e enriquecer seus logs (como `grok` para logs não estruturados, `json` para logs JSON, etc.). As **outputs** geralmente serão para o Elasticsearch.
    
- **Elasticsearch e Kibana:** O Logstash é uma peça do **ELK Stack** (Elasticsearch, Logstash, Kibana). Para ter uma solução completa de centralização e análise de logs, você precisará instalar e configurar o Elasticsearch (para armazenar e indexar os dados) e o Kibana (para visualizar e explorar os dados).
    
- **Segurança:** Em ambientes de produção, é fundamental configurar a segurança para o Logstash, incluindo comunicação TLS/SSL e autenticação, especialmente ao se comunicar com o Elasticsearch.
    
- **Performance:** Monitore o uso de CPU e memória do Logstash. Para grandes volumes de logs, pode ser necessário ajustar as configurações de JVM e alocar mais recursos.
    
- **Monitoramento de Logs do Logstash:** Os próprios logs do Logstash estão em `/var/log/logstash/`. Verifique-os em caso de problemas.
    

Este guia deve te dar uma base sólida para começar a trabalhar com o Logstash. Se tiver dúvidas sobre a configuração de inputs/filters/outputs mais específicos ou sobre a integração com Elasticsearch, é só perguntar!