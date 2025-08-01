---
state: "[[Idea]]"
---
1. Use as views criadas como fonte de dados:
   - vw_metricas_chatbot: Dados principais para análises detalhadas
   - vw_kpis_diarios: Para gráficos de tendência e dashboards executivos
   - vw_analise_intencoes: Para [[Análise de Performance]] por tipo de solicitação
   - vw_analise_temporal: Para heatmaps de demanda por horário
   - vw_top_fallbacks: Para identificar oportunidades de melhoria

2. Relacionamentos sugeridos no [[power bi]]:
   - dim_tempo[data] → vw_kpis_diarios[data] (1:N)
   - dim_canal[canal] → vw_metricas_chatbot[canal] (1:N)

3. Medidas DAX recomendadas:
   - Taxa Resolução = DIVIDE([Conversas Resolvidas], [Total Conversas])
   - CSAT Médio = AVERAGE(vw_metricas_chatbot[csat])
   - Tendência CSAT = [CSAT Atual] - [CSAT Período Anterior]

4. Configurar refresh automático:
   - Agendar refresh das views a cada 1 hora durante horário comercial
   - Refresh completo diário durante madrugada

5. Performance:
   - Considere materializar as views mais pesadas se o volume for alto
   - Implemente particionamento por data se necessário
   - Use índices nas colunas de data e id_conversa

[[full]]
[[full]]
[[power bi#📊 **3. Sugestão de visuais para o Dashboard**]]
