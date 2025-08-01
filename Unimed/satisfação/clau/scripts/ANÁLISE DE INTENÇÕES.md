---
state: "[[Idea]]"
---
```sql
CREATE OR REPLACE VIEW vw_analise_intencoes AS
WITH stats_intencao AS (
    SELECT 
        c.intencao_principal,
        COUNT(*) as total_conversas,
        COUNT(CASE WHEN c.resolvido = true THEN 1 END) as conversas_resolvidas,
        COUNT(CASE WHEN c.foi_escalado = true THEN 1 END) as conversas_escaladas,
        AVG(EXTRACT(EPOCH FROM c.tempo_total)/60) as tma_medio,
        AVG(c.num_mensagens) as msgs_media,
        AVG(c.num_fallbacks) as fallbacks_medio,
        
        -- Estatísticas de satisfação
        COUNT(f.csat) as feedbacks_recebidos,
        AVG(f.csat) as csat_medio,
        COUNT(CASE WHEN f.csat >= 4 THEN 1 END) as feedbacks_positivos,
        
        -- Análise de sentimento
        COUNT(CASE WHEN f.sentimento = 'Positivo' THEN 1 END) as sentimento_positivo,
        COUNT(CASE WHEN f.sentimento = 'Negativo' THEN 1 END) as sentimento_negativo
        
    FROM conversas c
    LEFT JOIN feedbacks f ON c.id_conversa = f.id_conversa
    WHERE c.data_inicio >= CURRENT_DATE - INTERVAL '30 days'  -- Últimos 30 dias
    GROUP BY c.intencao_principal
)

SELECT 
    intencao_principal,
    total_conversas,
    
    -- Taxas de Performance
    ROUND(conversas_resolvidas * 100.0 / total_conversas, 2) as taxa_resolucao_pct,
    ROUND(conversas_escaladas * 100.0 / total_conversas, 2) as taxa_escalonamento_pct,
    
    -- Métricas Operacionais
    ROUND(tma_medio, 2) as tma_minutos,
    ROUND(msgs_media, 1) as mensagens_media,
    ROUND(fallbacks_medio, 1) as fallbacks_media,
    
    -- Satisfação
    feedbacks_recebidos,
    ROUND(csat_medio, 2) as csat_medio,
    CASE 
        WHEN feedbacks_recebidos > 0 THEN 
            ROUND(feedbacks_positivos * 100.0 / feedbacks_recebidos, 1)
        ELSE NULL 
    END as pct_csat_positivo,
    
    -- Sentimento
    CASE 
        WHEN (sentimento_positivo + sentimento_negativo) > 0 THEN
            ROUND(sentimento_positivo * 100.0 / (sentimento_positivo + sentimento_negativo), 1)
        ELSE NULL
    END as pct_sentimento_positivo,
    
    -- Ranking de prioridade para melhoria (quanto maior, mais crítico)
    ROUND(
        (conversas_escaladas * 100.0 / total_conversas) * 0.4 +  -- 40% peso escalonamento
        (COALESCE(100 - (feedbacks_positivos * 100.0 / NULLIF(feedbacks_recebidos, 0)), 50)) * 0.3 + -- 30% peso insatisfação
        (fallbacks_medio * 10) * 0.3, 2  -- 30% peso dificuldade de entendimento
    ) as score_prioridade_melhoria
    
FROM stats_intencao
ORDER BY total_conversas DESC;
```

[[KPIs OPERACIONAIS DIÁRIOS]]
[[MÉTRICAS CONSOLIDADAS]]
[[ANÁLISE DE FALLBACKS (TOP PROBLEMAS)]]
[[QUERIES PARA VALIDAÇÃO DE DADOS]]
