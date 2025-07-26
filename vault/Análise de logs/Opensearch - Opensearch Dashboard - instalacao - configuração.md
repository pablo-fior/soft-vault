

Nosso plano incluirá:

1. **Instalação e Configuração do OpenSearch:** O coração de nosso novo sistema.
    
2. **Instalação e Configuração do OpenSearch Dashboards:** A interface visual que substituirá o Kibana.
    
3. **Ajuste do Logstash:** Para que ele envie os dados para o OpenSearch.
    

---

### Preparativos Cruciais: Desinstalando o Elastic Stack Anterior

Para garantir uma instalação limpa e evitar conflitos, é vital remover o Elasticsearch e o Kibana que instalamos anteriormente.

#### 1. Pare e Desabilite os Serviços:

Bash

```
sudo systemctl stop elasticsearch
sudo systemctl disable elasticsearch
sudo systemctl stop kibana
sudo systemctl disable kibana
```

#### 2. Remova os Pacotes:

- **Para Debian/Ubuntu:**
    
    Bash
    
    ```
    sudo apt remove --purge elasticsearch kibana -y
    ```
    
- **Para RHEL/CentOS:**
    
    Bash
    
    ```
    sudo yum remove elasticsearch kibana -y
    ```
    

#### 3. Remova os Diretórios de Dados e Configuração (ATENÇÃO: Isso apagará quaisquer dados existentes do Elasticsearch!):

Bash

```
sudo rm -rf /var/lib/elasticsearch # Dados do Elasticsearch
sudo rm -rf /etc/elasticsearch     # Configurações do Elasticsearch
sudo rm -rf /var/log/elasticsearch # Logs do Elasticsearch

sudo rm -rf /etc/kibana            # Configurações do Kibana
sudo rm -rf /var/log/kibana        # Logs do Kibana
```

Agora que o terreno está limpo, podemos iniciar nossa jornada no reino do OpenSearch!

---

### Instalação e Configuração do OpenSearch

O OpenSearch, como um digno sucessor, também requer o Java. Se você já tem o OpenJDK 11 ou superior instalado para o Logstash, pode pular o primeiro passo.

#### Passo 1: Instalação do Java Development Kit (JDK) - (Se ainda não o fez)

- **Para Debian/Ubuntu:**
    
    Bash
    
    ```
    sudo apt update
    sudo apt install openjdk-11-jdk -y
    ```
    
- **Para RHEL/CentOS:**
    
    Bash
    
    ```
    sudo yum update
    sudo yum install java-11-openjdk -y
    ```
    

#### Passo 2: Instalação do OpenSearch

O OpenSearch tem seus próprios repositórios.

- **Para Debian/Ubuntu:**
    
    1. **Importe a chave GPG do OpenSearch:**
        
        Bash
        
        ```
        wget -qO - https://artifacts.opensearch.org/publickeys/opensearch.gpg | sudo gpg --dearmor -o /usr/share/keyrings/opensearch-keyring.gpg
        ```
        
    2. **Adicione o repositório OpenSearch ao seu sistema:**
        
        Bash
        
        ```
        echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring.gpg] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch-2.x.list
        ```
        
        _(**Nota:** `2.x` refere-se à série de versões. Verifique no site oficial do OpenSearch a versão mais recente se quiser uma diferente.)_
        
    3. **Atualize o índice de pacotes e instale o OpenSearch:**
        
        Bash
        
        ```
        sudo apt update
        sudo apt install opensearch -y
        ```
        
- **Para RHEL/CentOS:**
    
    1. **Crie o arquivo de repositório do OpenSearch:**
        
        Bash
        
        ```
        sudo nano /etc/yum.repos.d/opensearch.repo
        ```
        
    2. **Adicione o seguinte conteúdo ao arquivo e salve:**
        
        Ini, TOML
        
        ```
        [opensearch-2.x]
        name=OpenSearch repository for 2.x packages
        baseurl=https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/yum
        gpgcheck=1
        gpgkey=https://artifacts.opensearch.org/publickeys/opensearch.gpg
        enabled=1
        autorefresh=1
        type=rpm-md
        ```
        
    3. **Instale o OpenSearch:**
        
        Bash
        
        ```
        sudo yum install opensearch -y
        ```
        

#### Passo 3: Configuração Básica do OpenSearch

O arquivo de configuração principal do OpenSearch é o `opensearch.yml`, localizado em `/etc/opensearch/`.

1. **Abra o arquivo de configuração:**
    
    Bash
    
    ```
    sudo nano /etc/opensearch/opensearch.yml
    ```
    
2. Ajuste as configurações essenciais:
    
    Procure e configure as seguintes linhas:
    
    YAML
    
    ```
    # ---------------------------------- Cluster -----------------------------------
    cluster.name: my-opensearch-cluster
    
    # ------------------------------------ Node ------------------------------------
    node.name: node-1
    
    # ----------------------------------- Paths ------------------------------------
    # path.data e path.logs geralmente são configurados automaticamente pelo pacote.
    # path.data: /var/lib/opensearch
    # path.logs: /var/log/opensearch
    
    # ---------------------------------- Network -----------------------------------
    # Rede para escutar. "0.0.0.0" para acessível de qualquer IP.
    network.host: 0.0.0.0
    
    # Porta HTTP do OpenSearch (padrão é 9200)
    http.port: 9200
    
    # --------------------------------- Discovery ----------------------------------
    # ESSENCIAL para um único nó. Impede a busca por outros nós.
    discovery.type: single-node
    
    # ---------------------------------- Security ----------------------------------
    # A segurança (Security Plugin) vem habilitada por padrão no OpenSearch.
    # Desabilite temporariamente para a primeira inicialização para facilitar.
    # EM PRODUÇÃO, SEMPRE HABILITE E CONFIGURE ISSO.
    plugins.security.disabled: true
    ```
    
    - `plugins.security.disabled: true`: Esta linha é **crucial para a primeira inicialização** em um ambiente de teste. O OpenSearch vem com segurança habilitada por padrão. Desabilitá-la temporariamente nos permite iniciar sem a complexidade de certificados e usuários. **Lembre-se de habilitar e configurar a segurança para ambientes de produção!**
        
3. **Salve e feche o arquivo.**
    

#### Passo 4: Configuração da Memória (JVM Heap Size)

Assim como o Elasticsearch, o OpenSearch utiliza a JVM, e a alocação de memória é vital. Edite o arquivo `jvm.options`, localizado em `/etc/opensearch/`.

1. **Abra o arquivo de opções da JVM:**
    
    Bash
    
    ```
    sudo nano /etc/opensearch/jvm.options
    ```
    
2. Ajuste o tamanho do Heap (Xms e Xmx):
    
    Defina-os para o mesmo valor, aproximadamente metade da RAM total do seu servidor, mas nunca mais de 32GB.
    
    Exemplo para 8GB de RAM:
    
    ```
    -Xms4g
    -Xmx4g
    ```
    
3. **Salve e feche o arquivo.**
    

#### Passo 5: Iniciando e Habilitando o OpenSearch

Bash

```
sudo systemctl start opensearch
sudo systemctl enable opensearch
```

Para verificar o status:

Bash

```
sudo systemctl status opensearch
```

Você deve ver `active (running)`.

#### Passo 6: Verificando o Funcionamento do OpenSearch

Bash

```
curl -X GET "localhost:9200/?pretty"
```

Você deve ver uma saída JSON com informações sobre o OpenSearch.

---

### Instalação e Configuração do OpenSearch Dashboards

O OpenSearch Dashboards é a interface de visualização.

#### Passo 1: Instalação do OpenSearch Dashboards

- **Para Debian/Ubuntu:**
    
    Bash
    
    ```
    sudo apt update
    sudo apt install opensearch-dashboards -y
    ```
    
- **Para RHEL/CentOS:**
    
    Bash
    
    ```
    sudo yum install opensearch-dashboards -y
    ```
    

#### Passo 2: Configuração do OpenSearch Dashboards

O arquivo de configuração principal é o `opensearch_dashboards.yml`, localizado em `/etc/opensearch-dashboards/`.

1. **Abra o arquivo de configuração:**
    
    Bash
    
    ```
    sudo nano /etc/opensearch-dashboards/opensearch_dashboards.yml
    ```
    
2. **Ajuste as configurações essenciais:**
    
    YAML
    
    ```
    # ------------------------------------ Server ------------------------------------
    server.port: 5601
    server.host: "0.0.0.0"
    # server.name: "Meus Dashboards OpenSearch"
    
    # ---------------------------------- OpenSearch ---------------------------------
    # URLs dos nós OpenSearch aos quais o Dashboards se conectará
    opensearch.hosts: ["http://localhost:9200"]
    
    # Se a segurança do OpenSearch estiver desabilitada (plugins.security.disabled: true),
    # não precisamos de credenciais aqui. Se for habilitada, você precisaria:
    # opensearch.username: "kibanaserver" # Nome de usuário para o Dashboards se conectar ao OpenSearch
    # opensearch.password: "SUA_SENHA"
    ```
    
3. **Salve e feche o arquivo.**
    

#### Passo 3: Iniciando e Habilitando o OpenSearch Dashboards

Bash

```
sudo systemctl start opensearch-dashboards
sudo systemctl enable opensearch-dashboards
```

Para verificar o status:

Bash

```
sudo systemctl status opensearch-dashboards
```

Você deve ver `active (running)`.

#### Passo 4: Acessando o OpenSearch Dashboards

Abra seu navegador web e vá para:

```
http://SEU_IP_DO_SERVIDOR:5601
```

Você deverá ver a interface do OpenSearch Dashboards. Como desabilitamos a segurança do OpenSearch para a primeira inicialização, você não deverá ser solicitado a fazer login (em um ambiente de produção, seria necessário).

---

### Ajuste do Logstash para o OpenSearch

Agora precisamos garantir que o Logstash envie seus logs para o OpenSearch.

1. **Edite o arquivo de saída do Logstash:** `/etc/logstash/conf.d/03-output-elasticsearch.conf` (ou o nome que você deu ao seu arquivo de output).
    
    Ruby
    
    ```
    output {
      # Altere "elasticsearch" para "opensearch"
      opensearch {
        hosts => ["http://localhost:9200"]
        index => "apache-access-logs-%{+YYYY.MM.dd}"
        # Se você habilitar a segurança no OpenSearch futuramente, adicione user e password aqui:
        # user => "logstash_writer"
        # password => "SUA_SENHA_LOGSTASH"
        # ssl_enabled => false # True se seu OpenSearch usar HTTPS (recomendado para produção)
        # ssl_verification_mode => "none" # Use "full" com certificados válidos em produção
      }
      # Mantenha o stdout para depuração, se desejar:
      # stdout {
      #   codec => rubydebug
      # }
    }
    ```
    
    - A principal mudança é o nome do plugin: de `elasticsearch` para **`opensearch`**.
        
    - Os parâmetros `hosts` e `index` permanecem os mesmos.
        
2. **Reinicie o Logstash:**
    
    Bash
    
    ```
    sudo systemctl restart logstash
    ```
    

---

### Verificação Final e Próximos Passos:

1. **Verifique os logs do Logstash:** Certifique-se de que ele está inicializando sem erros e enviando dados para o OpenSearch.
    
    Bash
    
    ```
    sudo tail -f /var/log/logstash/logstash-plain.log
    ```
    
2. **Crie um "Index Pattern" no OpenSearch Dashboards:** Assim como no Kibana, no menu lateral esquerdo, vá em **"Stack Management" > "Index Patterns"** (ou similar, as categorias podem variar ligeiramente entre as versões) e crie um novo padrão (`apache-access-logs-*`) para ver seus dados na seção **"Discover"**.
    

Agora, vossa visão não estará mais restrita, e a magia do OpenSearch revelará os padrões e anomalias ocultos em vossos logs!

Para a próxima etapa, a senhora gostaria que eu a guiasse na **habilitação e configuração da segurança** para o OpenSearch e OpenSearch Dashboards (que é fundamental para um ambiente de produção), ou talvez em como **conectar o Filebeat** para enviar logs para este novo sistema OpenSearch?