---
state: "[[Idea]]"
---
```sql

CREATE OR REPLACE VIEW vw_analise_temporal AS
SELECT 
    EXTRACT(DOW FROM data_inicio) as dia_semana,
    CASE EXTRACT(DOW FROM data_inicio)
        WHEN 0 THEN 'Domingo'
        WHEN 1 THEN 'Segunda'
        WHEN 2 THEN 'Terça'
        WHEN 3 THEN 'Quarta'
        WHEN 4 THEN 'Quinta'
        WHEN 5 THEN 'Sexta'
        WHEN 6 THEN 'Sábado'
    END as nome_dia_semana,
    
    EXTRACT(HOUR FROM data_inicio) as hora,
    
    COUNT(*) as total_conversas,
    COUNT(DISTINCT id_usuario) as usuarios_unicos,
    
    -- Métricas de qualidade por horário
    ROUND(AVG(CASE WHEN foi_escalado = false THEN 1.0 ELSE 0.0 END) * 100, 1) as taxa_retencao,
    ROUND(AVG(EXTRACT(EPOCH FROM tempo_total)/60), 1) as tma_medio,
    
    -- Densidade relativa (para heatmap)
    COUNT(*) * 1.0 / (
        SELECT COUNT(*) FROM conversas 
        WHERE data_inicio >= CURRENT_DATE - INTERVAL '30 days'
    ) as densidade_relativa

FROM conversas
WHERE data_inicio >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 
    EXTRACT(DOW FROM data_inicio),
    EXTRACT(HOUR FROM data_inicio)
ORDER BY dia_semana, hora;
```

[[TABELA DE DIMENSÕES PARA POWER BI]]
[[MÉTRICAS CONSOLIDADAS]]
[[Análise custo query timeout#Recomendações Holísticas para Otimização em Servidores Próprios]]
[[KPIs OPERACIONAIS DIÁRIOS]]
