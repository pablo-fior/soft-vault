---
state: "[[AL]]"
---

---

### Eventos de Conexão e Seus Sintomas em Logs (Foco MySQL)

Aqui estão os tipos de eventos e os sintomas específicos que você deve procurar nos logs do MySQL, além dos logs da sua aplicação:

1. **Excesso de Conexões / Conexões Não Encerradas (Connection Leak):**
    
    - **O que é:** Sua aplicação abre conexões ao MySQL, mas falha em fechá-las ou retorná-las ao pool, levando a um acúmulo gradual de sessões ativas no banco de dados.
        
    - **Sintomas nos Logs do MySQL:**
        
        - `Too many connections`: A mensagem clássica. O MySQL atingiu o limite de conexões definido por `max_connections` e recusa novas tentativas.
            
        - `Access denied for user 'your_user'@'your_ip' (using password: YES/NO)`: Embora possa ser um erro de credencial, se aparecer subitamente em grande volume após um período de funcionamento normal e correlacionado a picos de conexões, pode indicar que o MySQL está tão sobrecarregado que falha até mesmo na autenticação de novas conexões, ou que as tentativas de reconexão da aplicação estão falhando.
            
        - No **MySQL Error Log (geralmente `mysqld.log`)**: Procure por mensagens de erro que antecedem ou coincidem com o problema, indicando que o servidor está sob estresse.
            
        - **Informações via `SHOW PROCESSLIST` (ou `performance_schema` / `sys schema`):** Embora não sejam logs diretamente, são comandos essenciais para verificar o estado atual. Se você tiver monitoramento coletando isso, procure por um número elevado de processos com status `Sleep` (conexões ociosas, mas abertas), `Query` (executando uma consulta) ou `Locked` (aguardando um lock), que não diminuem. Um grande número de conexões `Sleep` por longos períodos é um forte indicativo de connection leak.
            
        - **Audit Logs (se configurado):** Se você tem um plugin de auditoria (como o MySQL Enterprise Audit ou Percona Audit Log), você veria um volume anormalmente alto de eventos de `CONNECT` sem os correspondentes eventos de `DISCONNECT`.
            
2. **Memory Leak (Vazamento de Memória) Relacionado a Conexões (na Aplicação ou no MySQL):**
    
    - **O que é:** Na aplicação (JVM), objetos relacionados a conexões ou resultados de consulta não são liberados, levando a `OutOfMemoryError`. No MySQL, pode haver vazamentos de memória interna devido a bugs ou consultas muito complexas que consomem memória excessiva por sessão.
        
    - **Sintomas nos Logs do MySQL:**
        
        - `Out of memory` ou `Failed to allocate memory`: Essas mensagens no MySQL Error Log indicam que o próprio servidor MySQL está ficando sem memória RAM ou tem problemas com alocação de memória (raro, mas possível em cenários extremos ou bugs).
            
        - Mensagens sobre **`Innodb: Error: log file is full`** ou **`InnoDB: Failed to write to log file`**: Indiretas, mas podem ocorrer se transações muito grandes ou consultas complexas relacionadas a um leak de conexão sobrecarregarem o buffer de log do InnoDB.
            
        - **Relacionado à JVM da Aplicação (se logado no MySQL):** Embora o MySQL não logue diretamente o OOM da JVM, se a aplicação crashar devido a um OOM, o MySQL pode registrar a **desconexão abrupta** do cliente.
            
    - **Sintomas nos Logs do Sistema Operacional (do servidor MySQL):**
        
        - Uso crescente de RAM pelo processo `mysqld`.
            
        - Ativação de swap memory (memória de paginação).
            
        - Mensagens do OOM Killer do Linux (se ele derrubar o `mysqld`): `Out of memory: Kill process [PID] (mysqld)` ou `oom-killer`.
            
3. **Pool Exhausted (Esgotamento do Pool de Conexões da Aplicação):**
    
    - **O que é:** O pool de conexões na sua aplicação (Java, .NET, etc.) atinge o limite máximo e não consegue fornecer mais conexões. O MySQL, nesse caso, pode estar funcionando normalmente, apenas não recebendo novas conexões da aplicação.
        
    - **Sintomas nos Logs do MySQL:**
        
        - Paradoxalmente, pode haver **poucas ou nenhuma mensagem de erro no MySQL** diretamente relacionada ao esgotamento do pool. O MySQL simplesmente vê menos ou nenhuma nova conexão sendo estabelecida porque a aplicação não consegue criá-las.
            
        - Se o esgotamento do pool for resultado de um `connection leak` (o cenário que você descreveu), então os sintomas de `Too many connections` no MySQL aparecerão, pois as conexões que deveriam ser liberadas não estão sendo, e o pool da aplicação não tem como obter novas.
            
4. **Conexões Lentas / Bloqueios no Banco de Dados (MySQL):**
    
    - **O que é:** Consultas SQL demoradas, `deadlocks`, ou bloqueios de transações no MySQL prendem as sessões por um tempo excessivo, impedindo que as conexões sejam liberadas rapidamente para o pool da aplicação.
        
    - **Sintomas nos Logs do MySQL:**
        
        - **Slow Query Log:** Se configurado, este log registra todas as consultas que excedem um `long_query_time` definido. **Crucial para identificar consultas problemáticas.** Procure por `SELECT`, `UPDATE`, `INSERT` ou `DELETE` que levem segundos ou minutos.
            
        - **Error Log (`mysqld.log`):**
            
            - `Deadlock detected`: O MySQL detectou e reverteu um deadlock entre transações.
                
            - `Lock wait timeout exceeded`: Uma transação esperou por um lock por muito tempo e foi abortada.
                
            - Mensagens sobre tabelas travadas ou problemas com o storage engine (InnoDB).
                
        - **General Query Log (use com cautela em produção):** Se ativado, registra todas as consultas SQL recebidas pelo MySQL. Extremamente útil para depuração, mas pode gerar um volume gigantesco de dados. Você pode usá-lo para ver a sequência exata de queries antes de um problema.
            
        - **Informações via `SHOW ENGINE INNODB STATUS` (ou `performance_schema` / `sys schema`):** Novamente, se o monitoramento estiver coletando, você verá `LATEST DETECTED DEADLOCK` detalhes, `TRANSACTIONS` em estado de espera por locks, e estatísticas de uso do buffer pool do InnoDB.
            

---

### Como Enriquecer os Logs para Máxima Análise (Foco MySQL)

O enriquecimento de logs é a chave para transformar texto bruto em dados estruturados e correlacionáveis, permitindo uma análise profunda no Logstash e Kibana.

1. **Padronização do Formato de Log (JSON é o Rei - especialmente para aplicação):**
    
    - Ainda a melhor prática. Configure sua aplicação para logar em JSON. Para os logs do MySQL, você terá que usar filtros Logstash para parsear.
        
2. **Campos Essenciais para Enriquecimento (Aplicação e MySQL):**
    
    - **Da Aplicação (já discutido, mas reforçando):**
        
        - `request_id` / `correlation_id`: ID único para cada requisição ou transação de negócio. **Permite correlacionar o que a aplicação fez com o que o BD respondeu.**
            
        - `session_id` / `user_id`: Contexto do usuário.
            
        - `thread_name` / `thread_id`: Qual thread da aplicação estava envolvida.
            
        - **Informações do Pool de Conexões:** Se a aplicação logar (ex: `HikariCP - Pool stats (total=X, active=Y, idle=Z, waiting=W)`), enriqueça com campos como `pool_total`, `pool_active`, `pool_idle`, `pool_waiting`.
            
        - **Detalhes da Query (na Aplicação):** Registre a query SQL sendo executada (parametrizada para segurança, se possível), o tempo de execução (`query_duration_ms`), e o nome do método/classe que a disparou.
            
    - **Do MySQL (principalmente via Logstash `grok`):**
        
        - **`timestamp`:** Parsear o timestamp do log do MySQL corretamente.
            
        - **`log_type`:** Campo para diferenciar logs (ex: `mysql_error_log`, `mysql_slow_query_log`).
            
        - **`message`:** A mensagem bruta do log.
            
        - **`mysql_error_code`:** Se um código de erro específico for logado.
            
        - **`db_user`:** O usuário do banco de dados que fez a conexão (`Access denied for user 'X'`).
            
        - **`client_ip`:** O IP da máquina cliente que está conectando ao MySQL.
            
        - **Para Slow Query Log:**
            
            - `query_time_seconds`: Tempo que a query levou para executar.
                
            - `lock_time_seconds`: Tempo gasto esperando por locks.
                
            - `rows_sent`: Número de linhas enviadas.
                
            - `rows_examined`: Número de linhas examinadas.
                
            - `db_name`: O banco de dados alvo da query.
                
            - `query_text`: A query SQL em si.
                
        - **Para Error Log (específicos de erros de conexão/memória):**
            
            - Identifique e marque mensagens como `Too many connections`, `Out of memory`, `Deadlock detected`, `Lock wait timeout exceeded`.
                
            - Extraia IDs de transação ou de processo de dentro das mensagens de deadlock.
                
3. **Como Implementar o Enriquecimento (Foco MySQL):**
    
    - **Na Aplicação:**
        
        - Use as bibliotecas de log (Logback, Log4j2, etc.) para adicionar os campos `request_id`, `query_duration_ms`, etc., aos logs da sua aplicação.
            
        - Configure o pool de conexões para logar suas estatísticas de saúde e uso.
            
    - **No Logstash (para os logs do MySQL):**
        
        - **Inputs:** Configure `file` inputs para o **MySQL Error Log**, **Slow Query Log** e, se aplicável, **General Query Log**.
            
        - **Filtros `grok` customizados:** Para cada tipo de log do MySQL, você precisará de padrões `grok` específicos.
            
            - **Exemplo para Slow Query Log (simplificado):**
                
                Snippet de código
                
                ```
                filter {
                  if [type] == "mysql_slow_query_log" {
                    grok {
                      patterns_dir => "/opt/logstash/patterns" # Opcional: para seus próprios padrões
                      match => { "message" => "(?m)^# Query_time: %{NUMBER:query_time_seconds:float}\s+Lock_time: %{NUMBER:lock_time_seconds:float}\s+Rows_sent: %{NUMBER:rows_sent:int}\s+Rows_examined: %{NUMBER:rows_examined:int}\s+use %{WORD:db_name};\n%{GREEDYDATA:query_text}" }
                      # Note: este grok pode ser complexo, dependendo do formato exato do seu slow query log
                    }
                    mutate { add_field => { "event_type" => "mysql_slow_query" } }
                  }
                  if [type] == "mysql_error_log" {
                    grok {
                      match => { "message" => "\[%{TIMESTAMP_ISO8601:log_timestamp}\] \[%{WORD:loglevel}\] \[%{WORD:component}\] %{GREEDYDATA:log_message}" }
                    }
                    if "Too many connections" in [log_message] {
                      mutate { add_field => { "issue_type" => "mysql_too_many_connections" } }
                    }
                    if "Deadlock detected" in [log_message] {
                      mutate { add_field => { "issue_type" => "mysql_deadlock" } }
                    }
                  }
                }
                ```
                
            - **`date` filter:** Para garantir que todos os timestamps (aplicação e MySQL) sejam normalizados para `@timestamp` no Elasticsearch.
                
            - **`mutate` filter:** Para adicionar campos como `db_instance: "mysql_prod_1"` para identificar a instância do BD.
                

---

### Visão Geral da Análise com Logs Enriquecidos:

1. **Dashboards no Kibana:**
    
    - Crie gráficos mostrando `pool_active` da aplicação e o número de conexões ativas no MySQL (se puder coletar e enviar via Filebeat/Metricbeat).
        
    - Gráficos para `mysql_slow_query` por `query_time_seconds`.
        
    - Contagem de eventos `issue_type: "mysql_too_many_connections"` e `issue_type: "connection_pool_exhausted"`.
        
    - Visualize os `query_text` mais lentos.
        
2. **Identificação de Causas:**
    
    - Se vir `Too many connections` no MySQL e `Connection pool exhausted` na aplicação, é um **connection leak** na aplicação ou um pico de demanda que excede o pool e o `max_connections` do MySQL.
        
    - Se vir `Too many connections` no MySQL, mas o pool da aplicação não está esgotado (indica `pool_active` baixo), pode ser que o MySQL esteja mantendo sessões abertas por algum motivo (ex: transações não comitadas), ou há outros clientes do BD.
        
    - Correlacione os `request_id`s da aplicação com as `query_text` do MySQL para ver quais operações de negócio estão gerando as consultas lentas ou os leaks.
        
    - Se `query_time_seconds` for alto e aparecerem `Deadlock detected` ou `Lock wait timeout exceeded`, investigue as queries e o design do schema do BD.
        
    - Para `OutOfMemoryError` no MySQL, é um problema sério no servidor de BD que pode exigir tuning (`innodb_buffer_pool_size`, etc.) ou atualização de versão/bugfix.
        

Ao focar nesses detalhes específicos do MySQL e enriquecer seus logs com os campos mencionados, você terá uma visão muito mais clara e estruturada para pinpointar a origem do problema de conexão.