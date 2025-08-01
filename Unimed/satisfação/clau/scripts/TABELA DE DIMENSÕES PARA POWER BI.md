---
state: "[[Idea]]"
---
```sql
-- Dimensão Tempo (para facilitar análises temporais)
CREATE OR REPLACE VIEW dim_tempo AS
SELECT DISTINCT
    DATE(data_inicio) as data,
    EXTRACT(YEAR FROM data_inicio) as ano,
    EXTRACT(MONTH FROM data_inicio) as mes,
    EXTRACT(DAY FROM data_inicio) as dia,
    EXTRACT(DOW FROM data_inicio) as dia_semana_num,
    CASE EXTRACT(DOW FROM data_inicio)
        WHEN 0 THEN 'Domingo'
        WHEN 1 THEN 'Segunda-feira'
        WHEN 2 THEN 'Terça-feira'  
        WHEN 3 THEN 'Quarta-feira'
        WHEN 4 THEN 'Quinta-feira'
        WHEN 5 THEN 'Sexta-feira'
        WHEN 6 THEN 'Sábado'
    END as dia_semana_nome,
    CASE 
        WHEN EXTRACT(DOW FROM data_inicio) IN (0,6) THEN 'Fim de semana'
        ELSE 'Dia útil'
    END as tipo_dia
FROM conversas
WHERE data_inicio >= CURRENT_DATE - INTERVAL '365 days'
ORDER BY data;

-- Dimensão Canal
CREATE OR REPLACE VIEW dim_canal AS
SELECT DISTINCT
    canal,
    CASE canal
        WHEN 'WhatsApp' THEN 'Mensagem'
        WHEN 'Web' THEN 'Site'
        WHEN 'App' THEN 'Aplicativo'
        ELSE 'Outros'
    END as categoria_canal
FROM conversas;
```

[[ANÁLISE TEMPORAL (HEATMAP)]]
[[KPIs OPERACIONAIS DIÁRIOS]]
[[MÉTRICAS CONSOLIDADAS]]
