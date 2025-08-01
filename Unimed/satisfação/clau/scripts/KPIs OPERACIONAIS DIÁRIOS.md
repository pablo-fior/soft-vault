---
state: "[[Idea]]"
---
```sql
CREATE OR REPLACE VIEW vw_kpis_diarios AS
SELECT 
    DATE(data_inicio) AS data,
    canal,
    
    -- Volume
    COUNT(*) AS total_conversas,
    COUNT(DISTINCT id_usuario) AS usuarios_unicos,
    
    -- Taxa de Sucesso
    ROUND(
        COUNT(CASE WHEN resolvido = true THEN 1 END) * 100.0 / COUNT(*), 2
    ) AS taxa_resolucao_pct,
    
    -- Taxa de Escalonamento
    ROUND(
        COUNT(CASE WHEN foi_escalado = true THEN 1 END) * 100.0 / COUNT(*), 2
    ) AS taxa_escalonamento_pct,
    
    -- Taxa de Retenção (inverso do escalonamento)
    ROUND(
        COUNT(CASE WHEN foi_escalado = false THEN 1 END) * 100.0 / COUNT(*), 2
    ) AS taxa_retencao_pct,
    
    -- Métricas de Performance
    ROUND(AVG(EXTRACT(EPOCH FROM tempo_total)/60), 2) AS tma_minutos,
    ROUND(AVG(num_mensagens), 1) AS media_mensagens,
    ROUND(AVG(num_fallbacks), 1) AS media_fallbacks,
    
    -- Distribuição de Complexidade
    ROUND(
        COUNT(CASE WHEN num_mensagens <= 5 THEN 1 END) * 100.0 / COUNT(*), 1
    ) AS pct_conversas_simples,
    
    -- Satisfação (onde disponível)
    ROUND(AVG(
        (SELECT csat FROM feedbacks f WHERE f.id_conversa = c.id_conversa)
    ), 2) AS csat_medio,
    
    COUNT(
        (SELECT 1 FROM feedbacks f WHERE f.id_conversa = c.id_conversa AND f.csat >= 4)
    ) * 100.0 / NULLIF(COUNT(
        (SELECT 1 FROM feedbacks f WHERE f.id_conversa = c.id_conversa AND f.csat IS NOT NULL)
    ), 0) AS pct_csat_positivo

FROM conversas c
GROUP BY DATE(data_inicio), canal
ORDER BY data DESC, canal;
```


[[ANÁLISE DE INTENÇÕES]]
[[MÉTRICAS CONSOLIDADAS]]
[[ANÁLISE TEMPORAL (HEATMAP)]]
