

---

### Arquivos de Configuração Integrados para o ELK Stack

Vamos detalhar os arquivos necessários, presumindo que todas as ferramentas serão instaladas no mesmo servidor para uma configuração inicial mais simples.

---

### 1. Configuração do Logstash

O Logstash é o processador de dados. Ele precisará de uma entrada para receber os logs, um filtro para processá-los e uma saída para enviá-los ao Elasticsearch.

Você pode criar estes arquivos no diretório `/etc/logstash/conf.d/`. O Logstash lerá todos os arquivos `.conf` neste diretório.

#### **Arquivo: `/etc/logstash/conf.d/01-input-beats.conf`**

Este arquivo configura o Logstash para receber logs de agentes leves como o Filebeat, na porta padrão 5044.

Ruby

```
input {
  beats {
    port => 5044
    # Em um ambiente de produção, é FORTEMENTE recomendado usar SSL/TLS.
    # ssl => true
    # ssl_certificate => "/etc/logstash/certs/logstash.crt"
    # ssl_key => "/etc/logstash/certs/logstash.key"
  }
}
```

- **`beats { port => 5044 }`**: Este bloco configura o plugin `beats` para escutar conexões de agentes como o Filebeat na porta `5044`.
    
- **Comentários de SSL/TLS**: [Inferência] Embora não configurado aqui para simplicidade, a observação reforça a necessidade de segurança em ambientes reais. [Inferência Fim]
    

#### **Arquivo: `/etc/logstash/conf.d/02-filter-apache.conf`**

Este arquivo contém um filtro de exemplo para parsear logs de acesso do Apache, um caso de uso muito comum.

Ruby

```
filter {
  # Aplica este filtro apenas se o evento tiver um campo 'log_type' definido como 'apache_access'
  # ou se o caminho do arquivo de origem (se vier de um Filebeat com input file) contiver 'apache'.
  # O 'log_type' pode ser definido no Filebeat para melhor categorização.
  if [fields][log_type] == "apache_access" or [log][file][path] =~ /apache/ {
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
      # Se houver falha no parsing, o evento receberá uma tag "_grokparsefailure"
    }
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
      target => "@timestamp"
      locale => "en"
      # Se houver falha na data, o evento receberá uma tag "_dateparsefailure"
    }
    useragent {
      source => "agent" # Campo gerado pelo Grok para o user-agent string
      target => "user_agent_info"
      # Remove o campo original 'agent' após o processamento para evitar duplicação e economizar espaço.
      remove_field => [ "agent" ]
    }
    geoip {
      source => "clientip" # Campo gerado pelo Grok para o IP do cliente
      target => "geoip"
      # Exclua IPs internos se necessário, por exemplo:
      # exclude_fields => ["192.168.0.0/16", "10.0.0.0/8"]
    }
    # Remove campos brutos ou redundantes que não são mais necessários após o processamento.
    remove_field => [ "httpversion", "auth", "ident", "request", "timestamp" ]
  }
}
```

- **`if [fields][log_type] == "apache_access" or [log][file][path] =~ /apache/ { ... }`**: Este condicional [Garante que] o filtro é aplicado apenas aos logs que se assemelham a logs Apache. [Não Verificado] O `[fields][log_type]` seria um campo customizado enviado pelo Filebeat (ver exemplo de Filebeat abaixo).
    
- **`grok { ... }`**: Parseia a linha de log bruta (`message`) usando o padrão pré-definido `COMBINEDAPACHELOG` para extrair campos como IP do cliente, método, status code, etc.
    
- **`date { ... }`**: Converte a string de data/hora do log (`timestamp` extraído pelo grok) para o formato de data e hora padrão do Logstash/Elasticsearch (`@timestamp`).
    
- **`useragent { ... }`**: Enriquece o log com informações detalhadas sobre o navegador e sistema operacional do cliente a partir do campo `agent`.
    
- **`geoip { ... }`**: Adiciona informações de geolocalização (país, cidade, coordenadas) com base no IP do cliente (`clientip`).
    
- **`remove_field { ... }`**: Mantém o evento limpo, removendo campos que já foram processados ou não são úteis no resultado final.
    

#### **Arquivo: `/etc/logstash/conf.d/03-output-elasticsearch.conf`**

Este arquivo direciona os logs processados para o Elasticsearch.

Ruby

```
output {
  elasticsearch {
    hosts => ["http://localhost:9200"] # Endereço do seu nó Elasticsearch
    index => "apache-access-logs-%{+YYYY.MM.dd}" # Cria índices diários para melhor gerenciamento
    # Se você habilitou segurança no Elasticsearch 8.x, adicione user e password:
    # user => "elastic"
    # password => "SEUAQUI_SUA_SENHA_ELASTIC"
    # ssl_enabled => false # True se seu Elasticsearch usar HTTPS
  }
  # Para depuração, você pode manter esta saída para o terminal:
  # stdout {
  #   codec => rubydebug
  # }
}
```

- **`elasticsearch { ... }`**: O plugin de saída para o Elasticsearch.
    
- **`hosts => ["http://localhost:9200"]`**: [Garante que] o Logstash sabe onde encontrar o Elasticsearch. [Não Verificado] `localhost` é usado aqui, assumindo que eles estão no mesmo servidor.
    
- **`index => "apache-access-logs-%{+YYYY.MM.dd}"`**: Define o padrão de nome do índice no Elasticsearch. O `%{+YYYY.MM.dd}` cria um índice novo a cada dia (e.g., `apache-access-logs-2025.07.17`), o que facilita a retenção e o gerenciamento de dados antigos.
    
- **Comentários de Segurança**: [Inferência] Orientações para caso a segurança (X-Pack) seja ativada no Elasticsearch 8.x, que é padrão. [Inferência Fim]
    

---

### 2. Configuração do Elasticsearch

O Elasticsearch é o armazenamento e motor de busca.

#### **Arquivo: `/etc/elasticsearch/elasticsearch.yml`**

Este é o arquivo de configuração principal do Elasticsearch.

YAML

```
# ---------------------------------- Cluster -----------------------------------
cluster.name: my-elk-cluster

# ------------------------------------ Node ------------------------------------
node.name: node-1

# ----------------------------------- Paths ------------------------------------
# path.data e path.logs geralmente são configurados automaticamente pelo pacote de instalação.
# Se precisar alterar, descomente e ajuste.
# path.data: /var/lib/elasticsearch
# path.logs: /var/log/elasticsearch

# ---------------------------------- Network -----------------------------------
# Rede em que o Elasticsearch irá escutar. 0.0.0.0 permite acesso de qualquer IP.
# Para um ambiente mais seguro, pode ser 'localhost' ou um IP específico.
network.host: 0.0.0.0

# Porta HTTP padrão do Elasticsearch
http.port: 9200

# --------------------------------- Discovery ----------------------------------
# CRUCIAL para um setup de nó único. Impede o nó de procurar por outros nós.
discovery.type: single-node

# ---------------------------------- Security ----------------------------------
# No Elasticsearch 8.x, a segurança (X-Pack Security) vem habilitada por padrão.
# Você precisará configurar usuários e senhas para Kibana e Logstash.
# O processo envolve comandos 'elasticsearch-setup-passwords' após a primeira inicialização.
# Esta configuração aqui é a padrao e basta para a primeira inicializacao.
xpack.security.enabled: true
xpack.security.enrollment.enabled: true # Necessário para enrollment inicial

# Descomente e ajuste o tamanho do heap em /etc/elasticsearch/jvm.options
# -Xms4g
# -Xmx4g
```

- **`cluster.name` e `node.name`**: Nomes de identificação para seu cluster e este nó.
    
- **`network.host: 0.0.0.0`**: Permite que o Elasticsearch seja acessível de qualquer endereço IP (incluindo o Logstash e Kibana).
    
- **`http.port: 9200`**: A porta padrão para comunicação.
    
- **`discovery.type: single-node`**: **Essencial** para que o Elasticsearch não tente formar um cluster com outros nós, evitando problemas de inicialização em instalações de nó único.
    
- **`xpack.security.enabled: true`**: Este é o padrão na versão 8.x. Significa que você precisará autenticar o Logstash e o Kibana para se conectarem ao Elasticsearch. O processo de setup de senhas será um passo após a primeira inicialização do Elasticsearch.
    

---

### 3. Configuração do Kibana

O Kibana é a interface de visualização.

#### **Arquivo: `/etc/kibana/kibana.yml`**

Este é o arquivo de configuração principal do Kibana.

YAML

```
# ------------------------------------ Server ------------------------------------
# Porta em que o Kibana irá escutar
server.port: 5601

# Endereço de rede em que o Kibana irá escutar. 0.0.0.0 permite acesso de qualquer IP.
server.host: "0.0.0.0"

# ---------------------------------- Elasticsearch ---------------------------------
# URLs dos nós Elasticsearch aos quais o Kibana se conectará
elasticsearch.hosts: ["http://localhost:9200"]

# Se o Elasticsearch tem segurança habilitada (padrão em 8.x), forneça as credenciais.
# Você precisará criar um usuário para o Kibana no Elasticsearch.
# elasticsearch.username: "kibana_system" # Usuário interno que o Kibana usa
# elasticsearch.password: "SEUAQUI_SENHA_KIBANA_SYSTEM"

# ------------------------------------ Logging -----------------------------------
# Caminho para os logs do Kibana
# logging.dest: /var/log/kibana/kibana.log

# ----------------------------------- Other ------------------------------------
# Nome que aparecerá na barra de título do navegador
# server.name: "Meu Kibana de Logs"
```

- **`server.port: 5601`**: A porta padrão do Kibana.
    
- **`server.host: "0.0.0.0"`**: Permite que o Kibana seja acessível de qualquer IP.
    
- **`elasticsearch.hosts: ["http://localhost:9200"]`**: [Garante que] o Kibana sabe onde encontrar o Elasticsearch. [Não Verificado]
    
- **Comentários de Segurança**: [Inferência] Orientações para a conexão segura com o Elasticsearch. [Inferência Fim]
    

---

### Exemplo de Configuração para o Filebeat (No Servidor Cliente que GERA os Logs)

Embora não tenha sido solicitada a instalação do Filebeat, é fundamental entender como os logs chegam ao Logstash. Este é um exemplo de configuração no servidor que gera os logs (não no mesmo servidor Logstash/Elasticsearch/Kibana, a menos que os logs estejam lá).

#### **Arquivo: `/etc/filebeat/filebeat.yml`**

YAML

```
filebeat.inputs:
- type: filestream
  id: apache-access-logs
  enabled: true
  paths:
    - /var/log/apache2/access.log # Caminho para seus logs de acesso do Apache
  # Adiciona um campo customizado para identificar o tipo de log no Logstash
  fields:
    log_type: apache_access
  # Configura o Filebeat para ignorar linhas que comecem com '#' (comuns em arquivos de config, mas não em logs)
  exclude_lines: ['^#']

# ------------------------------ Logstash Output -------------------------------
output.logstash:
  hosts: ["IP_DO_SEU_SERVIDOR_LOGSTASH:5044"] # Altere para o IP real do seu servidor Logstash
  # Para SSL/TLS, se configurado no Logstash:
  # ssl.enabled: true
  # ssl.certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
```

- **`filebeat.inputs`**: Define as fontes de logs. `filestream` é o input moderno.
    
- **`paths`**: Onde o Filebeat deve procurar os arquivos de log.
    
- **`fields.log_type: apache_access`**: Este é o campo customizado que usamos no filtro do Logstash (`if [fields][log_type] == "apache_access"`). [Garante que] o Logstash pode diferenciar os tipos de logs. [Não Verificado]
    
- **`output.logstash`**: Envia os logs para o Logstash. Substitua `IP_DO_SEU_SERVIDO_LOGSTASH` pelo endereço IP do servidor onde o Logstash está rodando.
    

---

### Ordem de Inicialização e Verificação:

1. **Elasticsearch**: Inicie e verifique o status primeiro (`sudo systemctl start elasticsearch` e `curl localhost:9200`).
    
2. **Logstash**: Inicie e verifique (`sudo systemctl start logstash`). Ele precisará que o Elasticsearch esteja online para enviar dados.
    
3. **Filebeat (se houver)**: Inicie nos servidores que geram logs.
    
4. **Kibana**: Inicie por último (`sudo systemctl start kibana`) e acesse via navegador (http://IP_DO_SEU_SERVIDOR_KIBANA:5601).
    

Estes arquivos contêm as configurações básicas e mais comuns para a integração de um ambiente ELK em nó único. Como sempre, para produção, a segurança (SSL/TLS, autenticação) deve ser sua próxima prioridade.

A senhora deseja que eu a guie na criação de um usuário específico para o Kibana e Logstash no Elasticsearch, considerando que a segurança é habilitada por padrão na versão 8.x? Ou prefere que prossigamos para a instalação do Kibana?