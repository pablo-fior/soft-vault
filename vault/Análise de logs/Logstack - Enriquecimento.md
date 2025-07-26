
---

### Preparando o Logstash para o Futuro: Configuração, Enriquecimento e Análise Prática

O Logstash será o seu "faz-tudo" para transformar logs brutos em dados estruturados e ricos. A configuração que vamos montar aqui é um ponto de partida robusto.

---

### 1. Dados de Enriquecimento de Log Mais Relevantes para o Cenário

Para o cenário de problemas de conexão (leaks, pool exhausted, memory leak, lentidão), os dados de enriquecimento mais cruciais são aqueles que permitem:

- **Correlacionar Eventos:** Ligar o que acontece na sua aplicação com o que acontece no MySQL.
    
- **Rastrear Ciclos de Vida:** Entender o fluxo de uma conexão desde a abertura até (ou falha no) o fechamento.
    
- **Identificar Padrões:** Encontrar quais partes do código ou operações estão mais envolvidas nos problemas.
    
- **Contextualizar Erros:** Saber onde e quando algo deu errado, e em qual ambiente.
    

Aqui estão os campos de enriquecimento mais relevantes:

1. **`request_id` / `transaction_id` (CRUCIAL):** Um identificador único para cada requisição de ponta a ponta na sua aplicação. Permite rastrear todo o caminho de uma chamada.
    
2. **`connection_id` (se disponível no log da aplicação/pool):** O ID da conexão fornecido pelo pool ou pelo MySQL. Permite rastrear uma conexão específica.
    
3. **`thread_name` / `thread_id`:** A thread da aplicação que está manipulando a conexão/requisição. Útil para identificar threads problemáticas.
    
4. **`event_type`:** Um campo genérico para categorizar o evento (ex: `connection_opened`, `connection_closed`, `sql_query`, `pool_stats`, `error_db_connection`).
    
5. **`issue_type`:** Um campo específico para marcar problemas detectados (ex: `pool_exhausted`, `connection_leak_suspect`, `db_too_many_connections`, `db_slow_query`, `app_out_of_memory`).
    
6. **`query_text` (para logs SQL):** A consulta SQL que está sendo executada.
    
7. **`query_duration_ms`:** O tempo que uma consulta SQL levou para ser executada (logado pela aplicação ou extraído do Slow Query Log do MySQL).
    
8. **`class_name` / `method_name`:** O local no código da aplicação onde o log foi gerado.
    
9. **Estatísticas do Pool de Conexões (`pool_active`, `pool_idle`, `pool_waiting`, `pool_total`):** Se sua aplicação ou framework de pool logar esses dados periodicamente.
    
10. **`db_user` / `client_ip`:** O usuário do MySQL e o IP de onde a conexão se originou.
    
11. **`service_name` / `application_name`:** O nome da sua aplicação/microsserviço.
    
12. **`environment`:** (dev, qa, prod, etc.).
    
13. **`host_name`:** Nome do servidor/máquina de onde o log se origina.
    

---

### 2. Configuração Prática do Logstash (Exemplos)

Vamos criar um arquivo de configuração básico para o Logstash (`logstash.conf`). Ele terá as seções `input`, `filter` e `output`.

**Premissas:**

- **Logs da Aplicação:** Estamos assumindo que a sua aplicação está logando em arquivos de texto simples, com um formato que podemos parsear (JSON é ideal, mas usaremos `grok` para um exemplo mais abrangente). O campo `message` conterá a linha de log completa.
    
- **Logs do MySQL:** O Logstash vai ler o **Error Log** e o **Slow Query Log** do MySQL.
    
- **Filebeat:** É altamente recomendado usar o Filebeat em cada servidor para enviar os logs para o Logstash. Ele é leve, confiável e evita que o Logstash precise "puxar" os arquivos diretamente.
    

Snippet de código

```
# logstash.conf

# --- INPUTS (Onde o Logstash vai ler os logs) ---
input {
  # Input para logs da aplicação usando Filebeat
  beats {
    port => 5044
    ssl => false # Use true e configure certs em produção!
    client_inactivity_timeout => 60 # Tempo limite para inatividade do cliente
  }

  # Input para o Error Log do MySQL (se não usar Filebeat para ele)
  file {
    path => "/var/log/mysql/error.log" # Ajuste o caminho do seu MySQL Error Log
    type => "mysql_error_log"
    start_position => "beginning"
    sincedb_path => "/tmp/sincedb_mysql_error"
  }

  # Input para o Slow Query Log do MySQL (se não usar Filebeat para ele)
  file {
    path => "/var/log/mysql/mysql-slow.log" # Ajuste o caminho do seu MySQL Slow Query Log
    type => "mysql_slow_query_log"
    start_position => "beginning"
    sincedb_path => "/tmp/sincedb_mysql_slow"
  }
}

# --- FILTERS (Onde o Logstash vai processar e enriquecer os logs) ---
filter {
  # Adiciona um campo 'host' do Filebeat (ou 'path' do input file)
  # O Filebeat já adiciona 'host.name' e 'log.file.path' por padrão, mas podemos mapear.
  if [host] {
    mutate {
      add_field => { "source_host" => "%{[host][name]}" }
    }
  } else if [path] {
    mutate {
      add_field => { "source_host" => "%{path}" } # Se não tiver host do Filebeat, usa o path
    }
  }

  # Processamento de Logs da Aplicação (usando 'application' como type do Filebeat, ou 'application_log' se usar input file)
  if [type] == "application" or [type] == "application_log" {
    # Tente parsear JSON primeiro se sua aplicação puder logar em JSON
    json {
      source => "message"
      target => "json_data" # Coloca o JSON parseado em um campo 'json_data'
      on_error => "json_parse_error" # Se falhar, adiciona uma tag
    }

    # Se o JSON falhou ou se você loga em texto, use Grok
    if "json_parse_error" in [tags] or "_jsonparsefailure" in [tags] or ![json_data] {
      # Padrão Grok para logs de aplicação (adapte ao seu formato!)
      # Exemplo: 2025-07-16 10:30:00.123 ERROR [http-nio-8080-exec-1] com.example.MyService - RequestId: req-1234. Message: Connection pool exhausted.
      grok {
        match => { "message" => "%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:day} %{TIME:time} %{LOGLEVEL:log_level} \[%{DATA:thread_name}\] %{JAVACLASS:class_name} - (?:RequestId: %{NOTSPACE:request_id}\.?\s*)?Message: %{GREEDYDATA:log_message}" }
        add_tag => [ "grok_parsed" ]
      }

      # Se o Grok falhou, adicione uma tag de erro
      if "_grokparsefailure" in [tags] {
        mutate { add_tag => ["_application_log_parse_failure"] }
      }
    } else {
      # Se o JSON foi parseado com sucesso, promova campos importantes para o nível raiz
      mutate {
        rename => {
          "[json_data][timestamp]" => "timestamp"
          "[json_data][level]" => "log_level"
          "[json_data][thread]" => "thread_name"
          "[json_data][service]" => "service_name"
          "[json_data][request_id]" => "request_id"
          "[json_data][connection_id]" => "connection_id"
          "[json_data][message]" => "log_message"
          "[json_data][class]" => "class_name"
          "[json_data][method]" => "method_name"
          "[json_data][query_duration_ms]" => "query_duration_ms"
          "[json_data][pool_stats][active]" => "pool_active"
          "[json_data][pool_stats][idle]" => "pool_idle"
          "[json_data][pool_stats][waiting]" => "pool_waiting"
          "[json_data][pool_stats][total]" => "pool_total"
        }
        remove_field => [ "json_data" ] # Remove o campo JSON original
      }
    }

    # Marcação de Eventos Problemáticos (para aplicação)
    if [log_message] =~ /Connection pool exhausted|No more connections available/ {
      mutate { add_field => { "issue_type" => "pool_exhausted" } }
    } else if [log_message] =~ /Timeout acquiring connection/ {
      mutate { add_field => { "issue_type" => "connection_timeout_acquire" } }
    } else if [log_message] =~ /Failed to close connection|Error closing connection/ {
      mutate { add_field => { "issue_type" => "connection_close_error" } }
    } else if [log_message] =~ /OutOfMemoryError/ {
      mutate { add_field => { "issue_type" => "application_out_of_memory" } }
    }

    # Adiciona event_type para logs de aplicação genéricos
    mutate { add_field => { "event_type" => "application_log" } }
  }

  # Processamento do MySQL Error Log
  if [type] == "mysql_error_log" {
    # Padrão Grok para o MySQL Error Log (pode variar ligeiramente)
    grok {
      match => { "message" => "\[%{YEAR:year}-%{MONTHNUM:month}-%{MONTHDAY:day} %{TIME:time}\] \[%{WORD:log_level}\] \[%{WORD:component}\]%{SPACE}%{GREEDYDATA:log_message}" }
      add_tag => [ "grok_parsed_mysql_error" ]
    }

    # Marcação de Eventos Problemáticos (para MySQL Error Log)
    if [log_message] =~ /Too many connections/ {
      mutate { add_field => { "issue_type" => "mysql_too_many_connections" } }
    } else if [log_message] =~ /Out of memory|Failed to allocate memory/ {
      mutate { add_field => { "issue_type" => "mysql_out_of_memory" } }
    } else if [log_message] =~ /Deadlock detected/ {
      mutate { add_field => { "issue_type" => "mysql_deadlock" } }
    } else if [log_message] =~ /Lock wait timeout exceeded/ {
      mutate { add_field => { "issue_type" => "mysql_lock_timeout" } }
    }

    mutate { add_field => { "event_type" => "mysql_error_log" } }
  }

  # Processamento do MySQL Slow Query Log
  if [type] == "mysql_slow_query_log" {
    # Este Grok é complexo e sensível ao formato exato. Consulte a documentação do MySQL Slow Query Log para ajustar.
    # Exemplo simplificado:
    grok {
      # Assumindo que cada bloco de slow query começa com '# Time:' ou '# User@Host:'
      # Este padrão captura o bloco completo e depois tenta parsear os detalhes.
      # É comum usar um codec multiline no input para capturar o bloco completo primeiro.
      match => { "message" => "(?m)^# User@Host: %{USER:db_user}\[%{USER}\] @ %{IPORHOST:client_ip} \[%{IPORHOST}\]\n# Query_time: %{NUMBER:query_time_seconds:float}\s+Lock_time: %{NUMBER:lock_time_seconds:float}\s+Rows_sent: %{NUMBER:rows_sent:int}\s+Rows_examined: %{NUMBER:rows_examined:int}(?:\s+Rows_affected: %{NUMBER:rows_affected:int})?\n(?:use %{DATA:db_name};\n)?%{GREEDYDATA:query_text}" }
      add_tag => [ "grok_parsed_mysql_slow" ]
    }

    # Marca como slow query
    mutate { add_field => { "issue_type" => "mysql_slow_query" } }
    mutate { add_field => { "event_type" => "mysql_slow_query_log" } }
  }

  # Converta o timestamp extraído para o campo padrão @timestamp do Elasticsearch
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS", "yyyy-MM-dd HH:mm:ss"] # Adapte formatos conforme seus logs
    target => "@timestamp"
    remove_field => ["year", "month", "day", "time"] # Remove campos intermediários
  }

  # Adicione campos de contexto padrão
  mutate {
    add_field => {
      "environment" => "production" # Substitua pelo seu ambiente
      "service_name" => "your_application_name" # Nome da sua aplicação
      "db_instance_name" => "mysql_prod_cluster_01" # Nome da instância do seu banco de dados
    }
  }

  # Remova campos não necessários para otimizar o Elasticsearch
  mutate {
    remove_field => ["message", "@version", "ecs", "agent"] # Mantenha o que for útil
  }
}

# --- OUTPUTS (Onde o Logstash vai enviar os logs processados) ---
output {
  elasticsearch {
    hosts => ["localhost:9200"] # Endereço do seu Elasticsearch/OpenSearch
    index => "logs-%{+YYYY.MM.dd}" # Índice diário para melhor gerenciamento
    user => "elastic" # Em produção, use credenciais seguras
    password => "changeme" # Em produção, use credenciais seguras
    # ssl => true # Habilite SSL em produção
    # cacert => "/path/to/ca.crt" # Caminho para o certificado CA em produção
  }

  # Saída para depuração (visualiza no console do Logstash)
  stdout { codec => rubydebug }
}
```

Como Iniciar o Logstash:

bin/logstash -f logstash.conf --config.reload.automatic

O config.reload.automatic é útil para desenvolvimento, recarregando a configuração ao salvar.

---

### 3. Exemplos Práticos de Captura e Registro (Pela Aplicação)

A chave para o sucesso do enriquecimento é fazer com que sua **aplicação** produza os dados mais relevantes.

**Exemplo em Java (usando Logback/Log4j2 com MDC):**

1. Configuração do logback.xml (ou log4j2.xml):
    
    Configure seu appender para formatar os logs em JSON. Muitas bibliotecas (Logback-Jackson, Logstash Logback Encoder) facilitam isso.
    
    XML
    
    ```
    <configuration>
        <appender name="STDOUT_JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyNames>
                    <mdcKeyName>request_id</mdcKeyName>
                    <mdcKeyName>connection_id</mdcKeyName>
                </includeMdcKeyNames>
                <customFields>{"service_name":"my-app","environment":"production"}</customFields>
            </encoder>
        </appender>
    
        <root level="INFO">
            <appender-ref ref="STDOUT_JSON"/>
        </root>
    </configuration>
    ```
    
2. **Código da Aplicação para Enriquecimento:**
    
    Java
    
    ```
    import org.slf44j.Logger;
    import org.slf4j.LoggerFactory;
    import org.slf4j.MDC;
    
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    import java.util.UUID;
    import javax.sql.DataSource; // Ou um Connection Pool como HikariCP
    
    public class MyDataService {
    
        private static final Logger logger = LoggerFactory.getLogger(MyDataService.class);
        private final DataSource dataSource; // Seu pool de conexões
    
        public MyDataService(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    
        public void processUserData(String userId, String data) {
            String requestId = UUID.randomUUID().toString();
            MDC.put("request_id", requestId); // Adiciona o request_id ao MDC
    
            long startTime = System.currentTimeMillis();
            Connection connection = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
    
            try {
                logger.info("Starting processing for user: {}", userId);
    
                connection = dataSource.getConnection(); // Obtém a conexão do pool
                // Capture o ID da conexão se o pool ou driver permitir
                // Exemplo hipotético: logger.info("Obtained DB connection with ID: {}", connection.getMetadata().getConnectionId());
                MDC.put("connection_id", String.valueOf(connection.hashCode())); // Exemplo simples, use um ID real se possível
    
                String sql = "SELECT * FROM users WHERE user_id = ?";
                pstmt = connection.prepareStatement(sql);
                pstmt.setString(1, userId);
    
                long queryStartTime = System.currentTimeMillis();
                rs = pstmt.executeQuery();
                long queryDuration = System.currentTimeMillis() - queryStartTime;
    
                logger.info("SQL Query executed: '{}'. Duration: {}ms", sql, queryDuration);
                // Adicione a duração da query diretamente ao MDC ou ao log
                MDC.put("query_duration_ms", String.valueOf(queryDuration)); // Para logs JSON
                // Ou inclua na mensagem: logger.info("SQL Query executed. Duration: {}ms", queryDuration);
    
                // Processar resultados...
    
                logger.info("Successfully processed data for user: {}", userId);
    
            } catch (SQLException e) {
                logger.error("Database error during user data processing for user: {}. Message: {}. Stack: {}",
                             userId, e.getMessage(), e, MDC.get("request_id")); // Inclua o request_id no log de erro
                // Marque um campo específico para o tipo de erro
                MDC.put("issue_type", "database_error");
            } finally {
                // Sempre feche os recursos no bloco finally
                if (rs != null) { try { rs.close(); } catch (SQLException e) { logger.warn("Failed to close ResultSet: {}", e.getMessage()); } }
                if (pstmt != null) { try { pstmt.close(); } catch (SQLException e) { logger.warn("Failed to close PreparedStatement: {}", e.getMessage()); } }
                if (connection != null) {
                    try {
                        connection.close(); // Libera a conexão de volta para o pool
                        logger.info("Connection released to pool.");
                        MDC.remove("connection_id"); // Remove do MDC ao fechar
                    } catch (SQLException e) {
                        logger.error("Failed to close DB connection: {}", e.getMessage());
                        MDC.put("issue_type", "connection_close_error"); // Marca o erro no log
                    }
                }
                MDC.remove("request_id"); // Remova o request_id do MDC no final da requisição
                MDC.remove("query_duration_ms");
                MDC.remove("issue_type"); // Limpa o MDC
            }
    
            long totalDuration = System.currentTimeMillis() - startTime;
            logger.info("Total processing for user: {} took {}ms", userId, totalDuration);
        }
    
        // Exemplo de como logar estatísticas do pool (se o pool permitir ou se você tiver um monitor)
        public void logPoolStats(int active, int idle, int waiting, int total) {
            MDC.put("event_type", "pool_stats");
            MDC.put("pool_active", String.valueOf(active));
            MDC.put("pool_idle", String.valueOf(idle));
            MDC.put("pool_waiting", String.valueOf(waiting));
            MDC.put("pool_total", String.valueOf(total));
            logger.info("Connection pool stats - Active: {}, Idle: {}, Waiting: {}, Total: {}", active, idle, waiting, total);
            MDC.remove("event_type"); // Limpa o MDC
        }
    }
    ```
    

---

### 4. Análise e Diagnóstico (No Kibana)

Com o Logstash processando e enriquecendo seus logs, o Kibana se torna seu centro de diagnóstico.

1. **Dashboards de Monitoramento Contínuo:**
    
    - **Contagem de Erros:** Gráficos mostrando `issue_type` (ex: `pool_exhausted`, `mysql_too_many_connections`) ao longo do tempo. Um pico aqui é seu primeiro alerta.
        
    - **Uso do Pool:** Gráficos de linha para `pool_active`, `pool_idle`, `pool_waiting` (se sua aplicação logar esses dados). Você pode ver o `pool_active` subindo e `pool_idle` caindo.
        
    - **Slow Queries:** Tabela mostrando as top 10 `query_text` do `mysql_slow_query_log` por `query_time_seconds` médio.
        
    - **Conexões Ativas no MySQL:** (Se você usar Metricbeat para coletar métricas do MySQL, pode plotar `mysql.status.connections`).
        
2. **Fluxo de Diagnóstico em um Evento Futuro:**
    
    - **Passo 1: Detecção e Alerta.** Você recebe um alerta de `pool_exhausted` ou vê um pico no dashboard de erros.
        
    - **Passo 2: Investigação Preliminar (Kibana Discover).**
        
        - Acesse o Kibana, vá para a seção "Discover" (Descoberta).
            
        - Filtre pelo tempo em que o evento ocorreu.
            
        - Pesquise por `issue_type: "pool_exhausted"` ou `issue_type: "mysql_too_many_connections"`.
            
        - Observe o `request_id` e o `thread_name` dos logs de erro.
            
    - **Passo 3: Rastreamento do `request_id` (Ouro Puro).**
        
        - Pegue um dos `request_id`s dos logs de erro.
            
        - Limpe todos os filtros, exceto o período de tempo.
            
        - Pesquise por `request_id: "seu-request-id-aqui"`.
            
        - Agora você verá **TODOS os logs** relacionados a essa requisição específica: logs de info, debug, logs de abertura de conexão, logs de query, etc.
            
        - Analise a sequência: `connection_opened`, `sql_query`, `query_duration_ms` (se foi lenta), e observe se há um `connection_closed` correspondente. A ausência do `connection_closed` (ou a presença de `connection_close_error`) para aquele `request_id` é uma prova forte de um leak.
            
        - Verifique se há `OutOfMemoryError`s correlacionados, especialmente se ocorrerem após a execução de certas queries ou processamento de grandes volumes de dados.
            
    - **Passo 4: Identificação de Padrões.**
        
        - Se vários `request_id`s problemáticos apontam para o mesmo `class_name` ou `method_name`, você encontrou o provável local do bug.
            
        - Analise as `query_text` mais lentas ou as que precedem os problemas de conexão.
            
        - Observe a relação entre `pool_active` e os erros: se `pool_active` está no máximo e `pool_waiting` está alto, o pool está esgotado. Se `pool_active` está crescendo continuamente, é um leak.
            

Ao investir no enriquecimento dos logs na fonte (sua aplicação) e ter um Logstash bem configurado, você transformará a caça a bugs em um processo muito mais eficiente e menos doloroso. Seu sistema de logs se tornará um "detetive" incansável, sempre pronto para a próxima investigação!