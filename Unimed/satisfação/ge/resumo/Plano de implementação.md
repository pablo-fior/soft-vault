---
state: "[[Drafting]]"
---
### Plano de Implementação da Solução de Avaliação do Chatbot

#### Fase 1: Fundação e Coleta de Dados Básicos (Duração: 1-2 meses)

Esta fase inicial foca em implementar o mínimo viável para começar a coletar dados valiosos imediatamente, com o objetivo de ter as primeiras métricas para análise.

1. **Implementação da Pesquisa Pós-Atendimento (CSAT)**
    
    - **Ação:** Disparar uma pesquisa simples e de baixo esforço imediatamente ao final de cada interação do usuário com o chatbot1.
        
    - **Pergunta-chave:** "De 1 a 5, quão satisfeito(a) você ficou com este atendimento?"2.
        
    - **Formato:** Usar um formato visual (estrelas ou emojis) diretamente na janela de chat para maximizar a taxa de resposta3.
        
    - **Dados a Coletar:** A nota da pesquisa e o tempo de conversa44.
        
2. **Registro de Dados Operacionais**
    
    - **Ação:** Configurar o registro automatizado de dados brutos das conversas.
        
    - **Dados a Coletar:** Tempo total da conversa, número de interações, data e hora da solicitação e, crucialmente, se houve transferência para um humano5555.
        
3. **Desenvolvimento do Módulo de Análise e Visualização (MVP)**
    
    - **Ação:** Criar um painel simples de Business Intelligence (BI) para visualizar as métricas coletadas.
        
    - **Indicadores-chave:** Taxa de Resolução no Primeiro Contato (FCR) e Taxa de Escalonamento para atendimento humano66666666666.
        

**Valor desta Fase:** A implementação do CSAT fornece um indicador direto da satisfação do paciente7. A coleta de dados operacionais permitirá calcular métricas de eficiência do bot, como a taxa de resolução de problemas, já na primeira interação8888.

---

#### Fase 2: Enriquecimento da Análise e Feedback Qualitativo (Duração: 2-3 meses)

Esta fase aprofunda a coleta de dados, adicionando camadas de análise qualitativa para entender o "porquê" das notas e das ineficiências identificadas na Fase 1.

1. **Expansão da Pesquisa Pós-Atendimento**
    
    - **Ação:** Adicionar um campo opcional para comentários livres após a avaliação inicial de 1 a 59999.
        
    - **Ação:** Implementar uma pergunta de múltipla escolha para entender a causa raiz da insatisfação se a nota for neutra ou negativa (por exemplo, "O assistente não entendeu minha pergunta" 10101010).
        
    - **Ação:** Adicionar a pesquisa de Esforço do Cliente (CES), "De 1 a 5, quão fácil foi resolver sua solicitação?" para correlacionar a satisfação com a complexidade da demanda11.
        
2. **Implementação da Análise de Conteúdo**
    
    - **Ação:** Integrar uma ferramenta de Análise de Sentimento (NLP) para categorizar as mensagens do usuário como positivas, neutras ou negativas12.
        
    - **Ação:** Desenvolver um processo para listar e agrupar as frases que o chatbot não conseguiu compreender (intenções não reconhecidas)13.
        

**Valor desta Fase:** A coleta de comentários livres e a análise de sentimento capturam as nuances e problemas não previstos14141414. Ao identificar as intenções não reconhecidas, a equipe terá uma fonte direta para treinar e melhorar o bot15.

---

#### Fase 3: Tomada de Decisão e Ciclo de Melhoria Contínua (Duração: Contínua)

Nesta última fase, o foco é transformar os dados e _insights_ coletados em ações concretas, criando um ciclo de melhoria contínua.

1. **Criação do [[Dashboard Executivo]] Completo**
    
    - **Ação:** Ampliar o painel da Fase 1 para incluir todos os KPIs16.
        
    - **Dados a exibir:** CSAT, CES, Taxa de Resolução (FCR), Taxa de Escalonamento e NPS17171717171717171717171717171717.
        
    - **Funcionalidades:** Incluir filtros por período, assunto da conversa e status de transbordo 18, e alertas para métricas fora do padrão19.
        
2. **Análises Preditivas e de Causa Raiz**
    
    - **Ação:** Realizar análises de correlação entre variáveis (por exemplo, "chats com mais de 10 mensagens têm CSAT 20% menor") para identificar pontos de falha na resolução rápida20.
        
    - **Ação:** Usar a mineração de comentários e a análise de tópicos para agrupar e identificar as principais causas de insatisfação, como "Problemas com credenciado" ou "Demora no reembolso"21.
        
3. **Estabelecimento de um Ciclo de Melhoria**
    
    - **Ação:** Implementar reuniões semanais entre as equipes de TI, SAC e operações para revisar as métricas e ajustar os fluxos do chatbot com base nos dados22.
        
    - **Ação:** Usar os dados de intenções não reconhecidas para o retreinamento contínuo do bot, personalizando fluxos para diferentes perfis de cliente23.
        

**Valor desta Fase:** Esta fase garante que os dados não apenas sejam coletados, mas também traduzidos em ações concretas que impactam diretamente a satisfação do cliente e a eficiência operacional24.