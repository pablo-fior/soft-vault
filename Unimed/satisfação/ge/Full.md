---
state: "[[Drafting]]"
---
### Parte 1: Estrutura Recomendada para o Sistema de Pesquisa

O ideal é uma abordagem minimalista e integrada à experiência do usuário para maximizar a taxa de adesão.

#### 1. Gatilho e Momento da Pesquisa

- **Quando:** A pesquisa deve ser disparada **imediatamente** após o encerramento da interação do usuário com o chatbot, seja por resolução do problema ou por transbordo para um atendente humano. A experiência ainda estará fresca na mente do cliente.
    
- **Onde:** Diretamente na janela do chat. Evite enviar por e-mail ou SMS dias depois, pois a taxa de resposta será muito menor.
    

#### 2. Formato da Pesquisa (Abordagem em Etapas)

A ideia é começar com uma pergunta de baixo esforço e, dependendo da resposta, aprofundar um pouco mais.

- **Etapa 1: A Métrica Principal (Baixo Esforço)**
    
    - Use uma pergunta clara e direta para medir a satisfação geral. As mais recomendadas são **CSAT** ou **CES**.
        
    - **Exemplo (CSAT - Customer Satisfaction Score):**
        
        > "De 1 a 5, quão satisfeito(a) você ficou com este atendimento?"
        > 
        > (Use estrelas, emojis de carinhas ou números. É visual e rápido.)
        
    - **Exemplo (CES - Customer Effort Score):**
        
        > "De 1 a 5, quão fácil foi resolver sua questão com nosso assistente virtual?"
        > 
        > (Onde 1 = Muito Difícil e 5 = Muito Fácil)
        
- **Etapa 2: O Motivo (Pergunta Categórica)**
    
    - Se a avaliação for neutra ou negativa (ex: notas 1, 2 ou 3), apresente uma pergunta de múltipla escolha para entender a causa raiz da insatisfação sem exigir que o usuário digite.
        
    - **Exemplo (após uma nota baixa):**
        
        > "O que podemos melhorar? (selecione uma ou mais opções)"
        > 
        > - [ ] O assistente não entendeu minha pergunta.
        >     
        > - [ ] A resposta não resolveu meu problema.
        >     
        > - [ ] Demorou muito para eu ser atendido(a).
        >     
        > - [ ] Eu precisava falar com um humano.
        >     
        > - [ ] Outro motivo.
        >     
        
- **Etapa 3: O Feedback Aberto (Opcional, mas valioso)**
    
    - Independentemente da nota, ofereça um campo de texto opcional para comentários.
        
    - **Exemplo:**
        
        > "Gostaria de deixar um comentário? Sua opinião é muito importante para nós."
        > 
        > [Caixa de texto opcional]
        

**Por que essa estrutura?**

- **Minimiza o Abandono:** A primeira pergunta é extremamente fácil de responder.
    
- **Dados Estruturados:** A segunda etapa categoriza os problemas, facilitando a análise quantitativa.
    
- **Insights Profundos:** O campo aberto captura nuances e problemas que você não previu.
    
- [Inferência] Uma pesquisa curta e objetiva tende a gerar uma taxa de resposta mais alta do que formulários longos. [Inferência Fim]
    

---

### Parte 2: Estatísticas e Indicadores (KPIs) Fundamentais

Aqui estão os indicadores-chave de desempenho que você deve acompanhar.

#### Indicadores de Satisfação do Cliente

1. **CSAT (Customer Satisfaction Score)**
    
    - **O que é:** Mede a satisfação geral com o atendimento.
        
    - **Como calcular:** `(Número de respostas 'Satisfeito' e 'Muito Satisfeito' (notas 4 e 5) / Número total de respostas) * 100`
        
    - **Meta:** Acompanhar a evolução do percentual de clientes satisfeitos ao longo do tempo.
        
2. **CES (Customer Effort Score)**
    
    - **O que é:** Mede o quão fácil foi para o cliente resolver seu problema. É um forte preditor de lealdade.
        
    - **Como calcular:** Média simples das notas recebidas (de 1 a 5 ou 1 a 7).
        
    - **Meta:** Manter a média de esforço a mais baixa possível (ou a mais alta, dependendo da escala, como no exemplo de 1 a 5 onde 5 é "muito fácil").
        
3. **NPS (Net Promoter Score)**
    
    - **O que é:** Mede a lealdade do cliente. Pode ser usado de forma mais ampla, não apenas para o chatbot.
        
    - **Pergunta:** "Em uma escala de 0 a 10, o quanto você recomendaria nosso atendimento via chatbot a um amigo ou familiar?"
        
    - **Como calcular:**
        
        - **Promotores:** Notas 9-10
            
        - **Neutros:** Notas 7-8
            
        - **Detratores:** Notas 0-6
            
        - **Fórmula:** `$$NPS = (\% \text{ de Promotores}) - (\% \text{ de Detratores})$$`
            
    - **Meta:** Aumentar o NPS ao longo do tempo.
        

#### Indicadores de Performance do Chatbot

1. **Taxa de Resolução no Primeiro Contato (FCR - First Contact Resolution)**
    
    - **O que é:** Percentual de interações em que o chatbot resolveu a questão do cliente sem necessidade de qualquer outro contato ou escalonamento.
        
    - **Como medir:** Pode ser inferido pela ausência de transbordo e confirmado com uma pergunta simples ao final: "Sua dúvida foi resolvida? (Sim/Não)".
        
2. **Taxa de Retenção (Containment Rate)**
    
    - **O que é:** Percentual de conversas totalmente gerenciadas pelo chatbot, sem a necessidade de transferir para um atendente humano.
        
    - **Fórmula:** `(Total de interações - Interações transferidas para humano) / Total de interações * 100`
        
3. **Taxa de Abandono**
    
    - **O que é:** Percentual de usuários que iniciam uma conversa mas a encerram antes de obter uma resolução.
        
    - **Análise:** É crucial entender _em que ponto_ da conversa o abandono ocorre.
        

---

### Parte 3: Dados Relevantes a Extrair das Conversas

As próprias conversas são uma mina de ouro. Analisá-las revela o "porquê" por trás dos números.

1. **Intenções Não Reconhecidas ("Não entendi, pode repetir?")**
    
    - **O que extrair:** Liste todas as frases que o chatbot não conseguiu compreender. Agrupe-as por similaridade.
        
    - **Relevância:** Isso mostra exatamente quais novos fluxos de conversa e conhecimentos precisam ser adicionados ao bot. É a sua principal fonte para treinamento e melhoria.
        
2. **Gatilhos de Transbordo/Escalonamento**
    
    - **O que extrair:** Analise as últimas interações do usuário antes dele solicitar um atendente humano. Quais palavras ou frases ele usou? (ex: "falar com atendente", "humano", "não resolveu").
        
    - **Relevância:** Ajuda a identificar os pontos de falha do chatbot. A solicitação de transbordo pode indicar uma intenção mal configurada ou um fluxo de conversa frustrante.
        
3. **Análise de Tópicos e Assuntos**
    
    - **O que extrair:** Categorize todas as conversas por assunto (ex: "2ª via de boleto", "cobertura de exame", "rede credenciada", "reembolso", "cancelamento de plano").
        
    - **Relevância:**
        
        - Identifica os serviços mais procurados, que devem ter os fluxos mais robustos e eficientes.
            
        - Permite cruzar dados: **qual assunto gera a menor nota de CSAT?** Por exemplo, se o CSAT para "reembolso" é consistentemente baixo, o fluxo de conversa para esse tópico precisa ser revisado com urgência.
            
4. **Análise de Sentimento**
    
    - **O que extrair:** Use ferramentas de processamento de linguagem natural (NLP) para analisar a evolução do sentimento do cliente ao longo da conversa (de neutro para positivo, ou de neutro para negativo).
        
    - **Relevância:** Identifica pontos de fricção. Se o sentimento do cliente piora consistentemente após uma determinada resposta do bot, você encontrou um problema.
        
5. **Análise dos Comentários Abertos**
    
    - **O que extrair:** Leia e categorize manualmente (ou com ajuda de IA) todos os feedbacks escritos. Crie uma nuvem de palavras com os termos mais frequentes.
        
    - **Relevância:** Captura problemas, elogios e sugestões inesperadas que as métricas quantitativas não mostram. É a voz direta do seu cliente.
        

### Sugestão de Dashboard de Análise

Para visualizar tudo isso, considere criar um painel de Business Intelligence (BI) com:

- **Visão Geral:** KPIs principais (CSAT, CES, FCR, Taxa de Retenção) com gráficos de tendência (semanal/mensal).
    
- **Ranking de Problemas:** Um gráfico de barras com os motivos de insatisfação mais selecionados na Etapa 2 da [[Pesquisa]].
    
- **Voz do Cliente:** Uma nuvem de palavras dos comentários abertos e uma lista dos feedbacks mais recentes (positivos e negativos).
    
- **Oportunidades de Melhoria:** Uma tabela com as "intenções não reconhecidas" mais frequentes.
    
- **Filtros:** Permita que os dados sejam filtrados por período, assunto da conversa e se houve ou não transbordo.
    

Ao combinar a medição estruturada da satisfação com a análise profunda do conteúdo das conversas, você terá um ciclo contínuo e poderoso de melhoria para o seu chatbot.

[[Resumo - IA#Parte 1: Estrutura Recomendada para o Sistema de Pesquisa]]
[[Resumo - IA#2. Formato da Pesquisa (Abordagem em Etapas)]]
[[Resumo - IA#1. Gatilho e Momento da Pesquisa]]
