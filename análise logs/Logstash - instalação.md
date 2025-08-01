---
state: "[[AL]]"
---

---

Para usar o **Logstash** na busca por evidências de uma possível causa para o evento de excesso de conexões de banco de dados não encerradas, o foco principal será na **coleta, parsing (análise) e filtragem** dos seus logs. O Logstash atua como o motor de pré-processamento que prepara seus dados para análise no Elasticsearch e visualização no Kibana.

Aqui estão os recursos do Logstash e como você os utilizaria:

---

### 1. Configuração de Inputs (Entradas)

Você precisa dizer ao Logstash de onde ele deve ler os logs.

- **`file` input:** O mais comum. O Logstash pode ler logs diretamente dos arquivos de log no seu servidor de aplicação e do banco de dados.
    
    - **Uso para seu caso:** Configure um `file` input para ler os logs da sua aplicação (onde o código está tentando abrir/fechar conexões) e os logs do banco de dados (que registram as conexões).
        
    
    Snippet de código
    
    ```
    input {
      file {
        path => "/var/log/minha_aplicacao/*.log" # Caminho para os logs da sua aplicação
        start_position => "beginning"
        sincedb_path => "/tmp/sincedb_app" # Garante que ele não leia logs já processados
        type => "application_log"
      }
      file {
        path => "/var/log/mysql/*.log" # Exemplo para logs do MySQL
        start_position => "beginning"
        sincedb_path => "/tmp/sincedb_db"
        type => "database_log"
      }
      # Adicione outros inputs conforme necessário, ex: syslog para logs do SO
    }
    ```
    
- **`beats` input:** Se você usa o **Filebeat** (recomendado para enviar logs do servidor para o Logstash), configure um input `beats`. O Filebeat é mais eficiente para coletar logs e envia-os de forma confiável para o Logstash.
    
    Snippet de código
    
    ```
    input {
      beats {
        port => 5044 # Porta padrão para o Filebeat
        # Você pode adicionar SSL/TLS aqui para segurança
      }
    }
    ```
    

---

### 2. Filtros para Parsing e Enriquecimento

Esta é a parte crucial do Logstash para sua investigação. Os filtros permitem que você analise as mensagens de log (que geralmente são strings de texto livre) e extraia informações estruturadas em campos separados.

- **`grok` filter:** O mais poderoso para parsear logs não estruturados. Ele usa padrões predefinidos ou personalizados para identificar e extrair dados de strings de texto.
    
    - **Uso para seu caso:**
        
        - **Extrair timestamps:** Certifique-se de que o timestamp do log seja parseado corretamente para que você possa correlacionar eventos.
            
        - **Identificar e extrair o nível do log (INFO, WARN, ERROR):** Isso ajuda a filtrar rapidamente por mensagens de erro.
            
        - **Extrair IDs de thread/processo:** Essencial para rastrear o fluxo de execução que levou ao problema de conexão.
            
        - **Extrair mensagens específicas:** Por exemplo, `Failed to close connection`, `Connection pool exhausted`, `Timeout acquiring connection`, `Too many connections`. Você pode criar padrões `grok` para identificar essas mensagens e marcá-las com um campo `event_type: connection_issue`.
            
        - **Extrair nomes de classes/métodos:** Se seus logs incluírem essa informação, ela pode apontar para o código problemático.
            
        - **Extrair IPs de origem/destino e portas:** Para logs de rede ou de banco de dados, pode ajudar a ver de onde as conexões estão vindo.
            
        - **Exemplo de `grok` para log de aplicação:**
            
        
        Snippet de código
        
        ```
        filter {
          if [type] == "application_log" {
            grok {
              match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} \[%{DATA:thread_name}\] %{DATA:logger_name} - %{GREEDYDATA:log_message}" }
              add_field => { "received_at" => "%{@timestamp}" }
            }
            # Tentar extrair um connection_id se presente na mensagem
            grok {
              match => { "log_message" => ".*connection ID: %{NUMBER:connection_id}.*" }
              tag_on_failure => ["_connection_id_not_found"]
            }
            # Se encontrar mensagens de pool esgotado
            if "Connection pool exhausted" in [log_message] {
              mutate { add_field => { "issue_type" => "connection_pool_exhausted" } }
            }
            # Se encontrar mensagens de erro ao fechar conexão
            if "Failed to close connection" in [log_message] or "Error closing connection" in [log_message] {
              mutate { add_field => { "issue_type" => "connection_close_error" } }
            }
          }
          # Exemplo de grok para log do DB (adapte ao formato do seu DB)
          if [type] == "database_log" {
            grok {
              match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{NUMBER:process_id}\] %{WORD:db_event}: %{GREEDYDATA:db_message}" }
            }
            if "Too many connections" in [db_message] {
              mutate { add_field => { "issue_type" => "db_max_connections_reached" } }
            }
          }
        }
        ```
        
- **`date` filter:** Converte o campo de timestamp extraído pelo `grok` (ou outro filtro) em um campo `@timestamp`, que é o formato padrão do Elasticsearch. Fundamental para visualizações temporais.
    
    Snippet de código
    
    ```
    filter {
      if [timestamp] { # Se o grok extraiu um campo 'timestamp'
        date {
          match => ["timestamp", "ISO8601"] # Exemplo para formato ISO8601
          target => "@timestamp"
        }
      }
    }
    ```
    
- **`mutate` filter:** Usado para modificar campos existentes, adicionar novos campos, renomear campos, remover campos, etc.
    
    - **Uso para seu caso:**
        
        - **Adicionar tags:** Adicione tags como `_connection_issue_` ou `_db_error_` para facilitar a filtragem no Kibana.
            
        - **Adicionar campos de contexto:** Por exemplo, `service_name: "minha_aplicacao"`, `environment: "producao"`.
            
        - **Remover campos desnecessários:** Para otimizar o armazenamento.
            
    
    Snippet de código
    
    ```
    filter {
      mutate {
        add_tag => ["logstash_processed"]
      }
    }
    ```
    
- **`json` filter:** Se seus logs estão em formato JSON, este filtro pode parseá-los automaticamente em campos estruturados. Muito mais fácil do que `grok`.
    
    - **Uso para seu caso:** Se sua aplicação gerar logs em JSON, use isso para simplificar o parsing.
        
    
    Snippet de código
    
    ```
    filter {
      json {
        source => "message" # Assume que a mensagem de log é um JSON
        target => "json_data" # Opcional: coloca o JSON parseado em um sub-campo
      }
    }
    ```
    

---

### 3. Configuração de Outputs (Saídas)

Depois de processar os logs, o Logstash precisa enviá-los para algum lugar.

- **`elasticsearch` output:** O destino principal para seus logs processados.
    
    Snippet de código
    
    ```
    output {
      elasticsearch {
        hosts => ["localhost:9200"] # Endereço do seu Elasticsearch/OpenSearch
        index => "logs-%{+YYYY.MM.dd}" # Cria um índice diário para organização
      }
      stdout { codec => rubydebug } # Útil para depuração do Logstash
    }
    ```
    

---

### Como Buscar Evidências da Causa com o Logstash (e Kibana):

1. **Ingestão de Logs Relevantes:** Certifique-se de que todos os logs pertinentes (aplicação, banco de dados, servidor de aplicação) estejam sendo coletados pelo Logstash.
    
2. **Extração de Campos Chave:** Use filtros `grok` para extrair:
    
    - **IDs de Sessão/Transação/Requisição:** Se sua aplicação gera esses IDs, o Logstash pode extraí-los. Isso é CRUCIAL para rastrear o ciclo de vida completo de uma requisição e ver onde a conexão pode ter sido perdida.
        
    - **Eventos de Abertura/Fechamento de Conexão:** Crie padrões `grok` ou `mutate` para identificar e marcar mensagens que indicam abertura (`Obtained connection`, `Opening new DB connection`) e fechamento (`Closing connection`, `Connection released to pool`).
        
    - **Mensagens de Erro/Exceção:** Extraia stack traces e mensagens de erro relacionadas a conexões, pools ou timeouts.
        
3. **Marcação (Tagging) de Eventos Problemáticos:** Use `mutate { add_tag => [...] }` quando o Logstash identificar mensagens como "Connection pool exhausted", "Failed to close connection" ou "Too many connections". Isso cria um campo facilmente pesquisável no Kibana.
    
4. **Enriquecimento com Contexto:** Adicione campos como nome da aplicação, ambiente, nome do host. Isso ajuda a agrupar e filtrar os logs no Kibana.
    
5. **Análise no Kibana:** Uma vez que os dados estão no Elasticsearch (via Logstash):
    
    - **Dashboards Temporais:** Crie um gráfico de linhas no Kibana que mostre o **número de eventos de "conexão aberta"** ao longo do tempo. Se você também tiver logs de "conexão fechada", pode plotar ambos para ver se o número de aberturas excede significativamente o de fechamentos.
        
    - **Filtros Rápidos:** Use os campos que você criou no Logstash (ex: `issue_type: "connection_pool_exhausted"`) para filtrar rapidamente todos os logs relacionados ao problema.
        
    - **Correlação de Eventos:** Pesquise por um `transaction_id` ou `thread_id` específico que apareceu em um log de erro. Isso pode te mostrar toda a sequência de eventos (onde a conexão foi aberta, o que aconteceu depois, e onde ela falhou ao fechar) para aquela requisição específica.
        
    - **Top N Threads/Métodos:** Se você extraiu nomes de threads ou métodos, pode criar visualizações que mostram quais threads ou partes do código estavam mais ativas ou associadas a erros de conexão.
        

O Logstash é a peça que transforma seus logs brutos em dados estruturados e pesquisáveis, pavimentando o caminho para uma investigação eficiente no Kibana. É o seu pré-processador de dados para essa missão.