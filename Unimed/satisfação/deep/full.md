---
state: "[[Drafting]]"
---
Para um sistema de pesquisa de satisfação de um chatbot de plano de saúde, recomendo a seguinte estrutura, indicadores e dados para análise:

### **Estrutura Recomendada:**
1. **Pesquisa Pós-Atendimento (CSAT):**
   - Disparada ao final de cada interação.
   - Pergunta-chave:  
     *"Em uma escala de 1 a 5, quanto você avalia o atendimento recebido?"*  
     (1 = Muito insatisfeito, 5 = Muito satisfeito).
   - Campo opcional para comentários livres.

2. **Avaliação por Etapa (Micro-Feedback):**
   - Botões 👍/👎 em respostas específicas do chatbot.
   - Permite identificar pontos de falha técnicos (ex.: intenções não reconhecidas).

3. **Pesquisa de Esforço do Cliente (CES):**
   - *"Foi fácil resolver sua solicitação? (1 = Muito difícil, 5 = Muito fácil)"*.
   - Correlaciona satisfação com complexidade da demanda.

4. **Análise de Sentimento em Tempo Real:**
   - NLP para detectar frustração/felicidade nas mensagens (ex.: palavras como "frustrado", "demorado", "ótimo").

5. **Follow-up para Casos Críticos:**
   - Se usuário der nota ≤ 2, disparar contato humano em 24h.

---

### **Indicadores e Estatísticas Chave (KPIs):**
| **KPI**                     | **Cálculo**                                  | **Objetivo**                             |
|-----------------------------|----------------------------------------------|------------------------------------------|
| **CSAT**                    | (% notas 4-5) / Total de respostas           | Satisfação geral                         |
| **Taxa de Resolução (FCR)** | (Chats sem escalonamento) / Total de chats  | Eficiência do bot                       |
| **NPS**                     | (% Promotores) - (% Detratores)              | Lealdade do cliente                     |
| **CES**                     | Média de notas (1-5) de "facilidade"         | Simplicidade da experiência             |
| **Taxa de Escalonamento**   | (Chats transferidos para humano) / Total    | Identificar limites do bot              |
| **TMA (Tempo Médio de Atendimento)** | Soma tempos de resolução / Total de chats | Eficiência operacional                  |

---

### **Dados Relevantes para Extrair das Conversas:**
1. **Metadados Operacionais:**
   - Tempo total da conversa
   - Número de interações
   - Hora/dia da solicitação
   - Taxa de transferência para humano

2. **Dados Técnicos:**
   - Intenções do usuário não reconhecidas
   - Respostas com baixa confiança do NLP
   - Erros de integração (ex.: falha ao acessar API de beneficiário)

3. **Indicadores de Qualidade:**
   - Sentimento médio (análise de NLP)
   - Taxa de engajamento com feedback (ex.: 30% aceitam responder pesquisa)
   - Tópicos com maior insatisfação (via mineração de comentários livres)

4. **Dados Contextuais:**
   - Tipo de demanda (reembolso, agendamento, dúvida cobertura)
   - Perfil do usuário (se idoso, gestante, etc. – se consentido)
   - Canais prévios usados (ex.: usuário veio do app, site ou WhatsApp)

---

### **Análises Recomendadas:**
1. **Correlação entre Variáveis:**
   - Ex.: Chats com >10 mensagens têm CSAT 20% menor → Indicador de falha na resolução rápida.

2. **Root Cause de Insatisfação:**
   - Agrupar comentários livres por tópicos (ex.:  
     *"Demora reembolso"*, *"Problemas com credenciado"*).

3. **Benchmark por Intenção:**
   - Comparar CSAT de "agendamento" vs. "solicitar reembolso" para priorizar melhorias.

4. **Impacto no Custo:**
   - Calcular redução de custos por chat não escalonado vs. CSAT.

---

### **Ferramentas Úteis:**
- **NLP:** IBM Watson, Google Dialogflow (para análise de sentimento/intenção)
- **Dashboards:** [[power bi]] + Azure Bot Service (integrar métricas em tempo real)
- **Pesquisas:** Typeform, SurveyMonkey (integração via webhook)

### **Boas Práticas:**
1. **LGPD:** Anonimizar dados sensíveis (ex.: CPF, nome do usuário).
2. **Amostragem:** Para volumes altos, analisar 15-20% das conversas com foco em outliers.
3. **Ação Contínua:** Reuniões semanais entre TI, SAC e operações para ajustar fluxos com base em CSAT.

Esta estrutura permite transformar dados brutos em insights acionáveis, priorizando melhorias que impactam diretamente a satisfação do cliente e redução de custos.