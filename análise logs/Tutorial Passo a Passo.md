---
state: "[[AL]]"
---

### Tutorial Passo a Passo: Implementação Rápida de um Ambiente ELK/OpenSearch para Logs de Conexão

Este guia assume que você tem servidores Linux para a aplicação, o MySQL e para o ELK/OpenSearch.

---

### Passo 1: Prepare Sua Aplicação (Onde a Mágica Começa)

Esta é a fase mais crítica, pois a qualidade dos seus logs depende dela.

1. **Configure o Logging Estruturado (JSON):**
    
    - **Ação:** Edite a configuração da sua biblioteca de logging (ex: `logback.xml` para Java, `appsettings.json` para .NET, ou configuração do seu logger para Python/Node.js) para que ela produza logs em **formato JSON**.
        
    - **Por quê:** Facilita imensamente o parsing automático e a estrutura dos dados no ELK/OpenSearch.
        
2. **Implemente o MDC para `request_id` e `connection_id`:**
    
    - **Ação:** No código da sua aplicação, use o **MDC (Mapped Diagnostic Context)** para adicionar um `request_id` único no início de cada requisição (HTTP, mensagem de fila, etc.) e **remova-o no `finally`**. Se possível, adicione também o `connection_id` quando uma conexão de banco de dados for obtida do pool.
        
    - **Por quê:** Permite correlacionar todos os logs de uma única operação em todos os sistemas envolvidos. O `connection_id` rastreia o ciclo de vida da conexão.
        
3. **Adicione Métricas do Pool de Conexões:**
    
    - **Ação:** Configure seu pool de conexões (ex: HikariCP, c3p0) para logar periodicamente suas estatísticas de uso (ativo, ocioso, esperando, total).
        
    - **Por quê:** Fornece uma visão contínua da saúde do seu pool, essencial para identificar esgotamento ou leaks.
        
4. **Logue Eventos de Conexão (Opcional, mas Útil):**
    
    - **Ação:** Se seu framework não fizer, adicione logs explícitos de `connection_opened` e `connection_closed` com o `connection_id`.
        
    - **Por quê:** Oferece visibilidade granular sobre o ciclo de vida de cada conexão.
        

---

### Passo 2: Configure o MySQL (A Visão do Banco de Dados)

1. **Ative o Slow Query Log:**
    
    - **Ação:** Edite o arquivo de configuração do MySQL (`my.cnf` ou `my.ini`).
        
        Ini, TOML
        
        ```
        # my.cnf
        [mysqld]
        slow_query_log = 1
        slow_query_log_file = /var/log/mysql/mysql-slow.log
        long_query_time = 1  # Loga queries que demoram mais de 1 segundo
        log_queries_not_using_indexes = 1 # Opcional: loga queries sem índice
        ```
        
    - **Por quê:** Identifica consultas demoradas que podem prender conexões.
        
2. **Monitore o Error Log:**
    
    - **Ação:** Garanta que o `log_error` esteja configurado para um local acessível.
        
        Ini, TOML
        
        ```
        # my.cnf
        [mysqld]
        log_error = /var/log/mysql/error.log
        ```
        
    - **Por quê:** Contém mensagens críticas como `Too many connections`, `Deadlock detected`, e erros de memória.
        

---

### Passo 3: Implemente os Coletores de Logs (Filebeat)

Instale o Filebeat em **todos os servidores** que geram logs (aplicação e MySQL).

1. **Instalação do Filebeat:**
    
    - **Ação:** Siga a documentação oficial para instalar o Filebeat no seu sistema operacional (geralmente via pacotes `apt` ou `yum`).
        
2. **Configuração do Filebeat (`filebeat.yml`):**
    
    - **Ação:** Configure os `inputs` para os logs da sua aplicação e do MySQL.
        
        YAML
        
        ```
        # /etc/filebeat/filebeat.yml
        filebeat.inputs:
        - type: filestream
          enabled: true
          paths:
            - /var/log/minha_aplicacao/*.log # Log da aplicação
          fields:
            service_name: "minha-api"
            environment: "production"
            log_type: "application" # Usado no Logstash para identificar
          json.keys_under_root: true # Se seus logs forem JSON
          json.overwrite_keys: true
          json.add_error_key: true
        
        - type: filestream
          enabled: true
          paths:
            - /var/log/mysql/error.log # MySQL Error Log
          fields:
            service_name: "mysql"
            environment: "production"
            log_type: "mysql_error"
          # MySQL logs são geralmente texto, então não use json.* aqui
        
        - type: filestream
          enabled: true
          paths:
            - /var/log/mysql/mysql-slow.log # MySQL Slow Query Log
          fields:
            service_name: "mysql"
            environment: "production"
            log_type: "mysql_slowlog"
          multiline.pattern: '^# User@Host:' # Crucial para logs de múltiplas linhas
          multiline.negate: true
          multiline.match: after
        
        output.logstash:
          hosts: ["<IP_DO_SERVIDOR_LOGSTASH>:5044"]
          # ssl.enable: true # Habilite em produção
          # ssl.certificate_authorities: ["/etc/filebeat/certs/ca.crt"] # CA do Logstash
        ```
        
    - **Por quê:** Garante que todos os logs relevantes sejam coletados de forma confiável e com metadados básicos. O `multiline` para o Slow Query Log é vital para agrupar entradas.
        
3. **Inicie o Filebeat:**
    
    - **Ação:** `sudo systemctl enable filebeat && sudo systemctl start filebeat`
        

---

### Passo 4: Configure o Logstash (O Coração do Processamento)

1. **Instalação do Logstash:**
    
    - **Ação:** Siga a documentação oficial para instalar o Logstash no seu servidor central.
        
2. **Crie o Arquivo de Configuração (`logstash.conf`):**
    
    - **Ação:** Use o exemplo fornecido na resposta anterior, ajustando os caminhos, IPs e os padrões `grok` para corresponderem exatamente aos seus formatos de log.
        
    - **Principais pontos a verificar:**
        
        - **`input`:** Porta do Beats (5044).
            
        - **`filter`:**
            
            - **`json` filter:** Se seus logs de aplicação forem JSON.
                
            - **`grok` filters:** Para logs de texto (aplicação fallback, MySQL Error Log, MySQL Slow Query Log). **Teste seus padrões `grok` com um Grok Debugger online para garantir que funcionam.**
                
            - **`mutate` filters:** Para adicionar `issue_type`, `event_type` e promover campos para o nível raiz.
                
            - **`date` filter:** Para normalizar todos os timestamps para `@timestamp`.
                
        - **`output`:** Endereço do Elasticsearch/OpenSearch e nome do índice (`logs-%{+YYYY.MM.dd}`). Configure SSL e credenciais para produção.
            
    - **Por quê:** É aqui que os logs são transformados em dados ricos e estruturados, prontos para análise.
        
3. **Inicie o Logstash:**
    
    - **Ação:** `sudo systemctl enable logstash && sudo systemctl start logstash`
        
    - **Depuração:** Use `sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/main.conf --config.test_and_exit` para testar sua configuração antes de iniciar o serviço.
        

---

### Passo 5: Configure o Elasticsearch / OpenSearch (O Armazém Inteligente)

1. **Instalação e Configuração:**
    
    - **Ação:** Instale o Elasticsearch ou OpenSearch (e Kibana/OpenSearch Dashboards) em seu servidor dedicado. Siga a documentação oficial.
        
    - **Segurança:** **IMPRESCINDÍVEL em produção:** Configure SSL/TLS e autenticação (usuários e senhas).
        
    - **Melhor Prática:** Alocar recursos adequados (CPU, RAM, disco rápido) e configurar políticas de **ILM (Index Lifecycle Management)** para Elasticsearch ou **ISM (Index State Management)** para OpenSearch.
        
        JSON
        
        ```
        # Exemplo de política ILM/ISM (configure via Kibana/Dashboards ou API)
        PUT _ilm/policy/my_log_policy
        {
          "policy": {
            "phases": {
              "hot": {
                "min_age": "0ms",
                "actions": {
                  "rollover": { "max_age": "1d", "max_docs": 10000000, "max_size": "50gb" }
                }
              },
              "warm": {
                "min_age": "7d",
                "actions": {
                  "set_priority": { "priority": 50 },
                  "forcemerge": { "max_num_segments": 1 }
                }
              },
              "cold": {
                "min_age": "30d",
                "actions": {
                  "set_priority": { "priority": 0 },
                  "freeze": {}
                }
              },
              "delete": {
                "min_age": "90d",
                "actions": {
                  "delete": {}
                }
              }
            }
          }
        }
        ```
        
    - **Template de Índice:** Crie um template para seus índices `logs-*` para garantir que os campos numéricos sejam mapeados corretamente. Faça isso no Kibana/Dashboards em "Stack Management" -> "Index Management" -> "Index Templates".
        
    - **Por quê:** Garante que seus dados sejam armazenados de forma eficiente, pesquisável e que o volume de dados seja gerenciado automaticamente, evitando problemas de disco cheio.
        

---

### Passo 6: Configure o Kibana / OpenSearch Dashboards (A Janela para Seus Dados)

1. **Acesse a Interface:**
    
    - **Ação:** Acesse o Kibana/OpenSearch Dashboards via navegador (`http://<IP_DO_SERVIDOR_KIBANA>:5601`).
        
2. **Crie Padrões de Índice:**
    
    - **Ação:** Em "Stack Management" (ou "OpenSearch Dashboards Management") -> "Index Patterns", crie um padrão `logs-*` e selecione `@timestamp` como seu campo de tempo.
        
    - **Por quê:** Permite que o Kibana reconheça e visualize seus índices de log.
        
3. **Crie Dashboards Essenciais para Conexões:**
    
    - **Ação:** Vá para a seção "Dashboard" e comece a criar visualizações:
        
        - **Visualização de Gráfico de Linha:**
            
            - **Métrica:** Contagem de documentos.
                
            - **Eixo X:** `@timestamp` (intervalo automático).
                
            - **Quebrar série por:** `issue_type` (tipo de erro).
                
            - **Por quê:** Visualize picos de `pool_exhausted`, `mysql_too_many_connections`, `application_out_of_memory` ao longo do tempo.
                
        - **Visualização de Gráfico de Linha/Área:**
            
            - **Métrica:** Média de `pool_active`, `pool_idle`, `pool_waiting`.
                
            - **Eixo X:** `@timestamp`.
                
            - **Por quê:** Mostra a saúde do pool de conexões da aplicação.
                
        - **Tabela de Dados:**
            
            - **Colunas:** `query_text`, `query_duration_ms`, `db_user`, `client_ip`.
                
            - **Filtro:** `event_type: "mysql_slow_query_log"`.
                
            - **Ordenação:** Por `query_duration_ms` (descendente).
                
            - **Por quê:** Identifica rapidamente as queries mais lentas.
                
    - **Por quê:** Fornecem uma visão imediata da saúde do sistema e alertam sobre anomalias.
        
4. **Explore os Dados (Aba Discover):**
    
    - **Ação:** Use a aba "Discover" para buscar, filtrar e inspecionar logs individuais.
        
        - **Filtre por `issue_type`:** Para ver apenas os logs de problemas (`issue_type: "pool_exhausted"`).
            
        - **Rastreie por `request_id`:** O **passo mais poderoso**. Digite `request_id: "seu-id-unico-aqui"` na barra de busca para ver todos os logs de uma requisição.
            
    - **Por quê:** Permite mergulhar nos detalhes para correlacionar eventos e encontrar a causa raiz.
        

---

Ufa! Com este roteiro, você terá uma base sólida para monitorar e diagnosticar problemas de conexão de forma proativa. O investimento inicial vale a pena para a paz de espírito e a agilidade na resolução de incidentes.

Agora sim, pode respirar aliviado! 😉