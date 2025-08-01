---
state: "[[Drafting]]"
---

## Pontos de Maior Atenção

**1. Complexidade Emocional no Contexto de Saúde**

- CSAT pode ser enganoso quando o bot funciona bem tecnicamente mas entrega "notícias ruins" (ex: procedimento não coberto)
- [Inferência] Pacientes podem avaliar negativamente informações corretas mas desfavoráveis [Inferência Fim]
- Necessário distinguir entre falhas técnicas e insatisfação com políticas do plano

**2. Momento Crítico de Coleta**

- Timing inadequado da [[Pesquisa]] pode comprometer a qualidade dos dados
- Risco de fadiga de [[Pesquisa]] se aplicada excessivamente
- Necessidade de capturar feedback em pontos específicos de falha (pré-escalonamento)

**3. Análise Correlacional Deficiente**

- Dados isolados (CSAT, CES, NPS) não fornecem visão completa
- Falta de conexão entre métricas de satisfação e KPIs operacionais
- Ausência de análise de causa raiz dos problemas identificados

## Implementações Prioritárias

**Primeira Prioridade - Sistema Híbrido de Métricas (Fases 1-3 meses):**

- CSAT + CES pós-interação imediata
- Pergunta qualitativa obrigatória após notas baixas: "Poderia nos contar o motivo da sua nota?"
- NPS trimestral/semestral para medir impacto na lealdade à marca

**Segunda Prioridade - KPIs Operacionais Essenciais:**

- Taxa de Sucesso da Tarefa: `((Conversas Resolvidas)/(Total de Conversas com Intenção Clara))×100`
- Taxa de Escalonamento: `((Conversas Escaladas)/(Total de Conversas))×100`
- Taxa de Abandono com análise de pontos específicos de saída

**Terceira Prioridade - Análise Forense dos Logs:**

- Mapeamento de eventos "Não Entendido" (fallbacks)
- Identificação de pontos de abandono por etapa da conversa
- [Não Verificado] Análise de sentimento em tempo real para detectar urgência/frustração [Não Verificado Fim]

**Quarta Prioridade - Sistema de Circuito Fechado:**

- Processo formal de 5 passos: Coletar → Analisar → Agir → Informar → Monitorar
- Matriz de correlação integrando todas as fontes de dados
- Squad dedicado de "Melhoria do Chatbot" (CX + TI + Operações)

O documento enfatiza que a medição deve ser uma função estratégica contínua, não um projeto pontual, visando transformar insights em melhorias tangíveis na experiência do paciente.

[[Resumo Técnico Insight]]
[[Resumo Técnico Insight]]
[[Resumo Técnico#Parte IV: A Estrutura Estratégica para a Melhoria Contínua]]