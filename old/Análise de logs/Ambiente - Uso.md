

---

### Preparando o Ambiente ELK/OpenSearch para o Futuro: Captura, Enriquecimento e Análise de Eventos de Conexão

Para garantir que seus logs estejam sempre prontos para a próxima análise de problemas de conexão (leaks, pool exhausted, memory leak, lentidão), você precisará de uma solução robusta que cubra desde a coleta até a visualização. O **ELK Stack** (Elasticsearch, Logstash, Kibana) ou o **OpenSearch Stack** (OpenSearch, OpenSearch Dashboards) são excelentes para isso.

Vamos detalhar a configuração, os dados de enriquecimento mais relevantes e as melhores práticas para cada componente.

---

### 1. Dados de Enriquecimento de Log Mais Relevantes para o Cenário

A chave para um diagnóstico eficiente está em transformar logs brutos em dados estruturados e correlacionáveis. Para problemas de conexão, os seguintes campos são cruciais para adicionar em seus logs:

- **`request_id` / `correlation_id` (CRUCIAL):** Um identificador único para cada requisição HTTP ou transação de negócio que passa pela sua aplicação. Ele permite rastrear o fluxo completo de uma operação através de todos os logs envolvidos (aplicação, banco de dados, microsserviços).
    
- **`connection_id`:** O ID interno da conexão, fornecido pelo pool de conexões da sua aplicação ou pelo próprio banco de dados (MySQL). Permite seguir o ciclo de vida de uma conexão específica.
    
- **`thread_name` / `thread_id`:** A thread da aplicação que está manipulando a conexão/requisição. Útil para identificar threads que estão presas ou consumindo recursos excessivamente.
    
- **`event_type`:** Um campo para categorizar o evento de log (ex: `connection_opened`, `connection_closed`, `sql_query_start`, `sql_query_end`, `pool_stats`).
    
- **`issue_type`:** Um campo para marcar problemas detectados (ex: `pool_exhausted`, `connection_leak_suspect`, `db_too_many_connections`, `db_slow_query`, `app_out_of_memory`, `db_deadlock`).
    
- **`query_text`:** A consulta SQL completa que foi executada.
    
- **`query_duration_ms`:** O tempo, em milissegundos, que uma consulta SQL levou para ser executada. (Logado pela aplicação ou extraído do Slow Query Log do MySQL).
    
- **`class_name` / `method_name`:** O local exato no código da aplicação (classe e método) onde o log foi gerado.
    
- **Estatísticas do Pool de Conexões (`pool_active`, `pool_idle`, `pool_waiting`, `pool_total`):** Se sua aplicação ou framework de pool logar esses dados periodicamente, são métricas vitais.
    
- **`db_user` / `client_ip`:** O usuário do banco de dados MySQL e o endereço IP de onde a conexão se originou.
    
- **`service_name` / `application_name`:** O nome da sua aplicação/microsserviço.
    
- **`environment`:** O ambiente de implantação (dev, qa, prod, etc.).
    
- **`host_name`:** O nome do servidor/máquina de onde o log se origina.
    

---

### 2. Melhores Práticas para Captura e Registro (Pela Aplicação e MySQL)

A melhor forma de enriquecer os logs é **na fonte**, ou seja, na sua aplicação e, no caso do MySQL, configurando seus logs para serem o mais detalhados possível.

#### a) Na Aplicação (Recomendado)

- **Logging Estruturado (JSON):**
    
    - **Melhor Prática:** Configure sua aplicação para logar em **formato JSON**. Isso transforma logs de texto livre em objetos estruturados, eliminando a necessidade de parsers complexos como `grok` no Logstash e garantindo a correta extração de campos.
        
    - **Ferramentas:** Utilize bibliotecas de log que suportam JSON nativamente (ex: Logback-Jackson ou Logstash Logback Encoder para Java; Serilog para .NET; Winston para Node.js; `structlog` para Python).
        
    - **Exemplo de Log JSON:**
        
        JSON
        
        ```
        {
          "timestamp": "2025-07-16T15:30:00.123Z",
          "level": "ERROR",
          "thread": "http-nio-8080-exec-5",
          "service": "minha-api",
          "environment": "prod",
          "request_id": "abc-123-xyz",
          "connection_id": "conn-42",
          "class": "com.example.app.MyDao",
          "method": "saveData",
          "message": "Connection pool exhausted. Max: 50, Active: 50, Idle: 0, Waiting: 10",
          "issue_type": "pool_exhausted",
          "query_duration_ms": 1500,
          "pool_active": 50,
          "pool_idle": 0
        }
        ```
        
- **Contexto Dinâmico (MDC - Mapped Diagnostic Context):**
    
    - **Melhor Prática:** Utilize MDC (disponível em frameworks como Logback/Log4j2 para Java) para injetar `request_id`, `connection_id` e outros identificadores em _todos_ os logs gerados por uma thread durante o ciclo de vida de uma requisição. Você define o `request_id` no início da requisição e o remove no final.
        
    - **Exemplo de Código (Java com Logback/MDC):**
        
        Java
        
        ```
        import org.slf4j.MDC;
        import java.util.UUID;
        // ... (resto do seu código)
        
        public void handleRequest() {
            String requestId = UUID.randomUUID().toString();
            MDC.put("request_id", requestId); // Adiciona o ID ao contexto da thread
        
            try {
                // Lógica de negócio que usa a conexão
                logger.info("Starting processing for request.");
                Connection conn = dataSource.getConnection();
                MDC.put("connection_id", String.valueOf(conn.hashCode())); // Exemplo, use um ID real se disponível
                // ... usar a conexão ...
                logger.info("Query executed successfully.");
            } catch (Exception e) {
                logger.error("Error during request processing.", e);
            } finally {
                // Sempre limpe o MDC no final da requisição!
                MDC.remove("request_id");
                MDC.remove("connection_id");
            }
        }
        ```
        
- **Log de Eventos de Conexão:**
    
    - **Melhor Prática:** Logue explicitamente a **abertura** e o **fechamento/liberação** de conexões, incluindo o `connection_id`. Se o pool de conexões logar suas estatísticas de uso (ativos, ociosos, esperando), capture essas mensagens.
        

#### b) No MySQL

- **Error Log:** O MySQL automaticamente registra erros, avisos e notas importantes aqui. Ele é crucial para `Too many connections`, `Out of memory`, `Deadlock detected`, e `Lock wait timeout exceeded`.
    
    - **Melhor Prática:** Monitore este log de perto. Ele é sua primeira linha de defesa para problemas no lado do banco de dados.
        
- **Slow Query Log:** Registra consultas que excedem um tempo limite (`long_query_time`).
    
    - **Melhor Prática:** Ative e configure o `long_query_time` para um valor razoável (ex: 1 segundo ou menos) para capturar queries problemáticas. É sua principal ferramenta para identificar queries que prendem conexões.
        
- **General Query Log (Cuidado em Produção!):** Registra _todas_ as consultas SQL executadas.
    
    - **Melhor Prática:** Não use em produção a menos que para depuração pontual de um problema específico, devido ao imenso volume de dados que ele gera. Em ambientes de desenvolvimento/teste, pode ser muito útil para ver a sequência exata de queries.
        
- **Audit Logs (Plugins):** Se você precisa rastrear quem fez o quê no BD, considere plugins de auditoria (MySQL Enterprise Audit, Percona Audit Log). Eles podem registrar eventos de conexão/desconexão.
    

---

### 3. Configuração do Ambiente ELK/OpenSearch: Componente a Componente

Aqui está a estrutura de configuração para cada peça do ELK/OpenSearch para o seu cenário:

#### a) Filebeat (Coleta de Logs nos Servidores)

- **Função:** Agente leve que monitora arquivos de log e envia os dados para o Logstash (ou diretamente para o Elasticsearch/OpenSearch, se o processamento for mínimo).
    
- **Melhor Prática:** Instale o Filebeat em cada servidor (da aplicação e do MySQL) que gera logs. Use módulos Filebeat se disponíveis (ex: `mysql` module para MySQL logs).
    

YAML

```
# /etc/filebeat/filebeat.yml (Exemplo para servidor de aplicação)
filebeat.inputs:
- type: filestream
  enabled: true
  paths:
    - /var/log/minha_aplicacao/*.log
  fields: # Adiciona campos personalizados (útil se a aplicação não loga JSON)
    service_name: "minha-api"
    environment: "production"
  json.keys_under_root: true # Se seus logs forem JSON, ele "desempacota" o JSON
  json.overwrite_keys: true # Permite que campos JSON sobrescrevam campos do Filebeat
  json.add_error_key: true # Adiciona um campo de erro se o JSON for inválido

# Exemplo de configuração do módulo MySQL (se você usá-lo)
# filebeat.modules:
# - module: mysql
#   error:
#     enabled: true
#     var.paths: ["/var/log/mysql/error.log"]
#   slowlog:
#     enabled: true
#     var.paths: ["/var/log/mysql/mysql-slow.log"]

output.logstash:
  hosts: ["logstash_host:5044"] # Endereço do seu servidor Logstash
  # ssl.enable: true # Habilitar SSL em produção
  # ssl.certificate_authorities: ["/etc/filebeat/certs/ca.crt"]
```

#### b) Logstash (Processamento e Enriquecimento)

- **Função:** Recebe logs do Filebeat, parseia, enriquece e envia para o Elasticsearch/OpenSearch.
    
- **Melhor Prática:** Centralize o Logstash. Configure os filtros para extrair todos os campos de enriquecimento definidos.
    

Snippet de código

```
# /etc/logstash/conf.d/main.conf (Exemplo de configuração Logstash)

input {
  beats {
    port => 5044
    ssl => false # Habilite e configure certificados em produção!
    # ssl_certificate_authorities => ["/etc/logstash/certs/ca.crt"]
    # ssl_certificate => "/etc/logstash/certs/logstash.crt"
    # ssl_key => "/etc/logstash/certs/logstash.key"
  }
}

filter {
  # Primeiro, tente o parsing de JSON (ideal para logs de aplicação)
  json {
    source => "message"
    target => "json_parsed" # Coloca o JSON parseado em um campo temporário
    remove_field => ["message"] # Remove a mensagem original depois de parsear
    on_error => "json_parse_error" # Tag para erros de parsing JSON
  }

  # Se o JSON foi parseado com sucesso, promova campos importantes
  if ![json_parsed][@metadata][json_parse_error] {
    # Promove campos do JSON para o nível raiz do documento
    mutate {
      add_field => {
        "log_level" => "%{[json_parsed][level]}"
        "thread_name" => "%{[json_parsed][thread]}"
        "service_name" => "%{[json_parsed][service]}"
        "environment" => "%{[json_parsed][environment]}"
        "request_id" => "%{[json_parsed][request_id]}"
        "connection_id" => "%{[json_parsed][connection_id]}"
        "class_name" => "%{[json_parsed][class]}"
        "method_name" => "%{[json_parsed][method]}"
        "log_message" => "%{[json_parsed][message]}"
        "query_text" => "%{[json_parsed][query_text]}"
        "query_duration_ms" => "%{[json_parsed][query_duration_ms]}"
        "pool_active" => "%{[json_parsed][pool_active]}"
        "pool_idle" => "%{[json_parsed][pool_idle]}"
        "pool_waiting" => "%{[json_parsed][pool_waiting]}"
        "pool_total" => "%{[json_parsed][pool_total]}"
        "event_type" => "%{[json_parsed][event_type]}" # Se a app já categorizar
        "issue_type" => "%{[json_parsed][issue_type]}" # Se a app já identificar
      }
    }
    # Remova o campo temporário
    mutate { remove_field => ["json_parsed"] }
  } else {
    # Se não for JSON ou o JSON falhou, use Grok (para logs de texto da aplicação ou logs do MySQL)
    # Exemplo: Logs de Aplicação (se não forem JSON ou se o parsing JSON falhar)
    if [fields][service_name] == "minha-api" { # Identifica o log pela field que o Filebeat adicionou
      grok {
        match => { "message" => "%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:day} %{TIME:time} %{LOGLEVEL:log_level} \[%{DATA:thread_name}\] %{JAVACLASS:class_name} - (?:RequestId: %{NOTSPACE:request_id}\.?\s*)?(?:ConnectionId: %{NOTSPACE:connection_id}\.?\s*)?Message: %{GREEDYDATA:log_message}" }
        add_tag => [ "grok_parsed_app_log" ]
      }
      # Marcação de Eventos Problemáticos (para logs de aplicação que não são JSON)
      if [log_message] =~ /Connection pool exhausted|No more connections available/ { mutate { add_field => { "issue_type" => "pool_exhausted" } } }
      else if [log_message] =~ /Timeout acquiring connection/ { mutate { add_field => { "issue_type" => "connection_timeout_acquire" } } }
      else if [log_message] =~ /Failed to close connection|Error closing connection/ { mutate { add_field => { "issue_type" => "connection_close_error" } } }
      else if [log_message] =~ /OutOfMemoryError/ { mutate { add_field => { "issue_type" => "application_out_of_memory" } } }
      # Defina um event_type se a aplicação não fez isso
      if ! [event_type] { mutate { add_field => { "event_type" => "application_log" } } }
    }
    # Processamento do MySQL Error Log (se o Filebeat o enviou com 'type' ou 'fields.log_type')
    else if [fields][log_type] == "mysql_error" {
      grok {
        match => { "message" => "\[%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:day} %{TIME:time}\] \[%{WORD:log_level}\] \[%{WORD:component}\]%{SPACE}%{GREEDYDATA:log_message}" }
        add_tag => [ "grok_parsed_mysql_error" ]
      }
      if [log_message] =~ /Too many connections/ { mutate { add_field => { "issue_type" => "mysql_too_many_connections" } } }
      else if [log_message] =~ /Out of memory|Failed to allocate memory/ { mutate { add_field => { "issue_type" => "mysql_out_of_memory" } } }
      else if [log_message] =~ /Deadlock detected/ { mutate { add_field => { "issue_type" => "mysql_deadlock" } } }
      else if [log_message] =~ /Lock wait timeout exceeded/ { mutate { add_field => { "issue_type" => "mysql_lock_timeout" } } }
      mutate { add_field => { "event_type" => "mysql_error_log" } }
    }
    # Processamento do MySQL Slow Query Log
    else if [fields][log_type] == "mysql_slowlog" {
      # Use um codec multiline no Filebeat para este tipo de log, pois as queries podem ter múltiplas linhas.
      grok {
        match => { "message" => "(?m)^# User@Host: %{USER:db_user}\[%{USER}\] @ %{IPORHOST:client_ip} \[%{IPORHOST}\]\n# Query_time: %{NUMBER:query_time_seconds:float}\s+Lock_time: %{NUMBER:lock_time_seconds:float}\s+Rows_sent: %{NUMBER:rows_sent:int}\s+Rows_examined: %{NUMBER:rows_examined:int}(?:\s+Rows_affected: %{NUMBER:rows_affected:int})?\n(?:use %{DATA:db_name};\n)?%{GREEDYDATA:query_text}" }
        add_tag => [ "grok_parsed_mysql_slow" ]
      }
      mutate { add_field => { "issue_type" => "mysql_slow_query" } }
      mutate { add_field => { "event_type" => "mysql_slow_query_log" } }
    }
  }

  # Normaliza o timestamp (se ainda não for @timestamp)
  # Prioriza o timestamp que vem do JSON ou Grok, senão usa o do Filebeat.
  if [timestamp] {
    date {
      match => ["timestamp", "ISO8601", "yyyy-MM-dd HH:mm:ss.SSS", "yyyy-MM-dd HH:mm:ss"] # Adapte aos seus formatos
      target => "@timestamp"
      remove_field => ["timestamp"]
    }
  } else if ![event_type] { # Se não foi parseado de forma alguma, atribua um tipo default
    mutate { add_field => { "event_type" => "unparsed_log" } }
  }

  # Adiciona campos de contexto genéricos se não vierem do Filebeat/aplicação
  if ![service_name] { mutate { add_field => { "service_name" => "%{[fields][service_name]}" } } }
  if ![environment] { mutate { add_field => { "environment" => "%{[fields][environment]}" } } }
  if ![source_host] { mutate { add_field => { "source_host" => "%{[agent][hostname]}" } } }

  # Limpeza final de campos que não são necessários no Elasticsearch
  mutate {
    remove_field => ["@version", "agent", "ecs", "input", "log", "message", "host", "path", "prospector"]
    # Remova 'message' se tudo estiver em 'log_message' ou 'query_text'
  }
}

output {
  elasticsearch {
    hosts => ["https://your_elasticsearch_or_opensearch_host:9200"] # Altere para seu host
    index => "logs-%{+YYYY.MM.dd}" # Índice diário
    user => "elastic" # Ou usuário do OpenSearch
    password => "your_secure_password" # Use segredos, não hardcode!
    ssl => true
    ssl_certificate_verification => true
    cacert => "/etc/logstash/certs/ca.crt" # Caminho para o certificado CA
  }
  # Saída para console para depuração
  stdout { codec => rubydebug }
}
```

#### c) Elasticsearch / OpenSearch (Armazenamento e Indexação)

- **Função:** Banco de dados de busca e análise, otimizado para grandes volumes de dados de log.
    
- **Melhor Prática:**
    
    - **Templates de Índice:** Defina templates de índice para pré-definir os tipos de dados e como eles devem ser indexados. Isso garante que seus campos numéricos (como `query_duration_ms`, `pool_active`) sejam indexados como números, permitindo agregações e gráficos.
        
    - **Gerenciamento de Ciclo de Vida do Índice (ILM/ISM):** Configure políticas para automaticamente mover índices antigos para armazenamento mais lento (warm/cold) e deletá-los após um certo período. Isso otimiza o uso de disco e a performance.
        

#### d) Kibana / OpenSearch Dashboards (Visualização e Análise)

- **Função:** Interface gráfica para explorar, visualizar e criar dashboards a partir dos dados no Elasticsearch/OpenSearch.
    
- **Melhor Prática:**
    
    - **Dashboards de Visão Geral:**
        
        - **Contagem de Eventos por Tipo (`issue_type`):** Gráfico de barras ou linha mostrando picos de `pool_exhausted`, `mysql_too_many_connections`, `application_out_of_memory` ao longo do tempo.
            
        - **Métricas do Pool de Conexões:** Gráficos de linha para `pool_active`, `pool_idle`, `pool_waiting` para identificar rapidamente a saúde do pool.
            
        - **Top N Queries Lentas:** Tabela mostrando as `query_text` mais lentas do `mysql_slow_query_log`, ordenadas por `query_duration_ms`.
            
        - **Conexões Ativas no MySQL:** (Se estiver usando Metricbeat para coletar métricas do MySQL).
            
    - **Fluxo de Diagnóstico em um Evento Futuro (no Kibana Discover):**
        
        1. **Detecção:** Identifique o problema através dos seus dashboards ou alertas.
            
        2. **Filtro por `issue_type`:** Na aba "Discover", filtre os logs pelo `issue_type` (ex: `issue_type: "pool_exhausted"`). Isso o levará diretamente aos logs de erro.
            
        3. **Rastreamento pelo `request_id`:** Pegue um `request_id` de um log de erro. Limpe os filtros de `issue_type` e pesquise por `request_id: "seu-request-id-aqui"`. Você verá **todos os logs correlacionados** a essa requisição específica, incluindo abertura/fechamento de conexões, queries SQL, e outros eventos.
            
        4. **Análise de Ciclo de Vida:** Examine a sequência de logs para aquele `request_id`.
            
            - Você verá a mensagem de `connection_opened` (com `connection_id` e `thread_name`).
                
            - Depois, a `sql_query_start` e `sql_query_end` (com `query_text` e `query_duration_ms`).
                
            - Procure pela mensagem de `connection_closed` ou `connection_released` para o mesmo `connection_id`. Se ela estiver faltando, você tem um **connection leak**.
                
            - Se houver um `application_out_of_memory` no meio, verifique os logs imediatamente anteriores para ver qual operação ou consulta o precedeu.
                
        5. **Identificação da Causa Raiz:** A ausência de logs de fechamento, a presença de erros específicos (`Failed to close connection`), ou a correlação com queries lentas e `OutOfMemoryError`s, apontarão para a parte do código ou configuração que precisa ser ajustada.
            
        6. **Visualização de Padrões:** Use a funcionalidade de "Visualize" do Kibana para criar gráficos que mostram, por exemplo, quais `class_name` ou `method_name` estão mais associados a `issue_type: "connection_leak_suspect"`.
            

---

Ao implementar essa estrutura completa, você terá um ecossistema de logs que não apenas armazena seus dados, mas os enriquece e os torna facilmente pesquisáveis e analisáveis, garantindo que você esteja sempre preparado para diagnosticar rapidamente qualquer evento de conexão futuro.