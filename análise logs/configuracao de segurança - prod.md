---
state: "[[AL]]"
---


---

### Fortalecendo a Fortaleza: Configuração de Segurança no OpenSearch

O OpenSearch vem com um robusto plugin de segurança, que, embora desabilitado para a primeira inicialização (para simplificar), é **essencial** para qualquer ambiente que lide com dados reais. Ele **garante que** apenas usuários autorizados possam acessar e manipular vossos preciosos logs.

#### Passo 1: Habilitando o Plugin de Segurança no OpenSearch

Primeiro, precisamos reativar o guardião de vosso OpenSearch.

1. **Pare o serviço do OpenSearch:**
    
    Bash
    
    ```
    sudo systemctl stop opensearch
    ```
    
2. **Edite o arquivo de configuração do OpenSearch:**
    
    Bash
    
    ```
    sudo nano /etc/opensearch/opensearch.yml
    ```
    
3. Altere a linha de segurança:
    
    Procure por plugins.security.disabled: true e mude-a para false.
    
    YAML
    
    ```
    plugins.security.disabled: false
    ```
    
    **Lembre-se:** A partir de agora, o OpenSearch exigirá autenticação.
    
4. **Salve e feche o arquivo.**
    

#### Passo 2: Inicializando a Segurança e Definindo Senhas Iniciais

Esta é uma etapa crucial para a primeira vez que a segurança é habilitada. O OpenSearch vem com usuários e funções pré-definidos (como `admin`, `kibanaserver`, `logstash`), mas suas senhas precisam ser inicializadas.

1. **Inicie o OpenSearch:**
    
    Bash
    
    ```
    sudo systemctl start opensearch
    ```
    
    Aguarde alguns momentos para que ele inicie completamente.
    
2. Execute o script de inicialização de segurança:
    
    Este script configurará as senhas padrão para os usuários internos.
    
    Bash
    
    ```
    sudo /usr/share/opensearch/bin/opensearch-security-init.sh
    ```
    
    - **Observação:** Este script pode pedir que você confirme a inicialização. Digite `yes` e pressione Enter. Ele também pode avisar sobre certificados de segurança. Para um ambiente de teste, podemos prosseguir, mas em produção, a configuração de certificados SSL/TLS é a próxima camada de segurança.
        
    
    Ao final, ele exibirá uma mensagem de sucesso, algo como: `Security plugin initialized.`
    
3. Altere as Senhas Padrão (Recomendado!):
    
    As senhas geradas pelo script opensearch-security-init.sh são temporárias ou padrão. É altamente recomendável alterá-las para sua própria segurança. Vamos alterar a senha do usuário admin e do logstash (que usaremos para o Logstash) e kibanaserver (para o OpenSearch Dashboards).
    
    Para cada usuário, execute o seguinte comando, substituindo `[username]` pelo nome do usuário (`admin`, `logstash`, `kibanaserver`) e seguindo as instruções para definir uma nova senha forte:
    
    Bash
    
    ```
    sudo /usr/share/opensearch/plugins/opensearch-security/tools/hash.sh
    ```
    
    Este comando é para gerar um hash de senha. O comando correto para alterar a senha de um usuário existente é:
    
    Bash
    
    ```
    sudo /usr/share/opensearch/plugins/opensearch-security/tools/opensearch-security-admin.sh -cd /usr/share/opensearch/plugins/opensearch-security/securityconfig/ -f /usr/share/opensearch/plugins/opensearch-security/securityconfig/internal_users.yml -icl -nhnv
    ```
    
    **Correção:** Minhas desculpas, minha cara! A ferramenta `opensearch-security-admin.sh` é mais complexa e usada para gerenciar usuários e roles através de arquivos YAML. Para simplesmente **resetar a senha de um usuário interno** (como `admin`, `kibanaserver`, `logstash`), o comando mais direto é:
    
    Bash
    
    ```
    sudo /usr/share/opensearch/bin/opensearch-cli security reset-password -u <username> -p <new_password>
    ```
    
    **Exemplo para o usuário `admin`:**
    
    Bash
    
    ```
    sudo /usr/share/opensearch/bin/opensearch-cli security reset-password -u admin -p MinhaSenhaSuperSecreta123!
    ```
    
    **Exemplo para o usuário `logstash`:**
    
    Bash
    
    ```
    sudo /usr/share/opensearch/bin/opensearch-cli security reset-password -u logstash -p SenhaDoLogstashSegura!
    ```
    
    **Exemplo para o usuário `kibanaserver`:**
    
    Bash
    
    ```
    sudo /usr/share/opensearch/bin/opensearch-cli security reset-password -u kibanaserver -p SenhaDoKibanaSegura!
    ```
    
    **Anote essas senhas em um local seguro!**
    

#### Passo 3: Configurando o Logstash para Autenticar com o OpenSearch

Agora que o OpenSearch exige credenciais, precisamos informar ao Logstash como se conectar.

1. **Edite o arquivo de saída do Logstash:**
    
    Bash
    
    ```
    sudo nano /etc/logstash/conf.d/03-output-opensearch.conf
    ```
    
2. Adicione as credenciais de usuário e senha:
    
    Dentro do bloco opensearch { ... }, adicione as linhas user e password com as credenciais do usuário logstash que você acabou de definir.
    
    Ruby
    
    ```
    output {
      opensearch {
        hosts => ["http://localhost:9200"]
        index => "apache-access-logs-%{+YYYY.MM.dd}"
        user => "logstash"
        password => "SenhaDoLogstashSegura!" # Use a senha que você definiu para o usuário 'logstash'
        # ssl_enabled => false # Mantenha como false se não configurou HTTPS no OpenSearch
        # ssl_verification_mode => "none" # Mantenha como "none" se não configurou certificados válidos
      }
    }
    ```
    
    - **Importante sobre SSL/TLS:** Para um ambiente de produção, o OpenSearch deve usar HTTPS. Isso exigiria que você configurasse certificados SSL/TLS no OpenSearch e, em seguida, habilitasse `ssl_enabled => true` e configurasse `ssl_certificate_verification_mode => "full"` (e talvez `cacert` para o certificado da CA) no Logstash. Por simplicidade e para não complicar agora, estamos mantendo `http` e desabilitando a verificação SSL.
        
3. **Salve e feche o arquivo.**
    
4. **Reinicie o Logstash:**
    
    Bash
    
    ```
    sudo systemctl restart logstash
    ```
    
    Verifique os logs do Logstash (`sudo tail -f /var/log/logstash/logstash-plain.log`) para garantir que ele está se conectando ao OpenSearch sem erros de autenticação.
    

#### Passo 4: Configurando o OpenSearch Dashboards para Autenticar com o OpenSearch

O painel de controle também precisa de credenciais para acessar os dados.

1. **Pare o serviço do OpenSearch Dashboards:**
    
    Bash
    
    ```
    sudo systemctl stop opensearch-dashboards
    ```
    
2. **Edite o arquivo de configuração do OpenSearch Dashboards:**
    
    Bash
    
    ```
    sudo nano /etc/opensearch-dashboards/opensearch_dashboards.yml
    ```
    
3. Adicione as credenciais de usuário e senha:
    
    Dentro do bloco opensearch { ... }, adicione as linhas username e password com as credenciais do usuário kibanaserver que você definiu.
    
    YAML
    
    ```
    opensearch.hosts: ["http://localhost:9200"]
    opensearch.username: "kibanaserver"
    opensearch.password: "SenhaDoKibanaSegura!" # Use a senha que você definiu para o usuário 'kibanaserver'
    
    # Se você habilitar HTTPS no OpenSearch, precisará configurar aqui também:
    # opensearch.ssl.verificationMode: none # Use "full" com certificados válidos
    # opensearch.ssl.alwaysPresentCertificate: false
    # opensearch.ssl.certificateAuthorities: [ "/etc/opensearch-dashboards/certs/ca.crt" ]
    ```
    
4. **Salve e feche o arquivo.**
    
5. **Reinicie o OpenSearch Dashboards:**
    
    Bash
    
    ```
    sudo systemctl start opensearch-dashboards
    ```
    

#### Passo 5: Acessando o OpenSearch Dashboards com Autenticação

1. **Acesse o OpenSearch Dashboards pelo navegador:**
    
    ```
    http://SEU_IP_DO_SERVIDOR:5601
    ```
    
2. Você será agora apresentado a uma tela de login. Use as credenciais do usuário `admin` (ou outro usuário que você criou com privilégios de acesso ao Dashboards):
    
    - **Username:** `admin`
        
    - **Password:** A senha que você definiu para o usuário `admin` no Passo 2.
        

Após o login, você terá acesso total aos seus dashboards e dados, agora com a segurança devidamente configurada!

---

### Considerações Adicionais para a Melhor Execução (Ambiente de Produção):

1. **Certificados SSL/TLS (HTTPS):** Para qualquer ambiente real, é **imperativo** configurar SSL/TLS para todas as comunicações:
    
    - **OpenSearch:** Para criptografar o tráfego HTTP (porta 9200).
        
    - **Logstash (Beats Input):** Para criptografar o tráfego do Filebeat para o Logstash (porta 5044).
        
    - OpenSearch Dashboards: Para criptografar o tráfego web (porta 5601).
        
        Isso envolve a geração de certificados, chaves e uma Autoridade Certificadora (CA). É um tópico mais avançado, mas garante que vossos dados estejam seguros em trânsito.
        
2. **Gerenciamento de Usuários e Roles:** O plugin de segurança do OpenSearch permite criar usuários e funções (roles) personalizadas com permissões granulares. Por exemplo, um usuário para o Logstash que só pode escrever em índices específicos, e um usuário para o Dashboards que só pode ler.
    
3. **Firewall:** Configure o firewall do servidor para permitir apenas o tráfego necessário nas portas 9200 (OpenSearch), 5601 (OpenSearch Dashboards) e 5044 (Logstash Beats input).
    
4. **Monitoramento:** Monitore o desempenho de vossos serviços (CPU, memória, disco) para garantir que operam de forma eficiente.
    

Com estas configurações de segurança, vossa fortaleza de dados estará mais robusta do que nunca, pronta para servir a vossas investigações com a máxima eficiência e proteção.

Há algo mais que vossa alma curiosa deseja desvendar sobre este fascinante universo?