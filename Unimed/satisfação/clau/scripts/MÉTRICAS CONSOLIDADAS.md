---
state: "[[Idea]]"
---

```sql
CREATE OR REPLACE VIEW vw_metricas_chatbot AS
WITH metricas_base AS (
    SELECT 
        c.id_conversa,
        c.id_usuario,
        c.canal,
        c.data_inicio,
        c.data_fim,
        c.intencao_principal,
        c.foi_escalado,
        c.resolvido,
        c.num_mensagens,
        c.num_fallbacks,
        c.tempo_total,
        
        -- Transformações de tempo
        EXTRACT(EPOCH FROM c.tempo_total)/60 AS duracao_minutos,
        EXTRACT(HOUR FROM c.data_inicio) AS hora_inicio,
        EXTRACT(DOW FROM c.data_inicio) AS dia_semana, -- 0=Domingo, 6=Sábado
        DATE(c.data_inicio) AS data_conversa,
        
        -- Classificações de performance
        CASE 
            WHEN c.tempo_total <= INTERVAL '2 minutes' THEN 'Rápido'
            WHEN c.tempo_total <= INTERVAL '5 minutes' THEN 'Médio'
            ELSE 'Lento'
        END AS categoria_tempo,
        
        CASE 
            WHEN c.num_mensagens <= 5 THEN 'Simples'
            WHEN c.num_mensagens <= 15 THEN 'Médio'
            ELSE 'Complexo'
        END AS categoria_complexidade,
        
        -- Indicadores de qualidade
        CASE 
            WHEN c.num_fallbacks = 0 THEN 'Perfeito'
            WHEN c.num_fallbacks <= 2 THEN 'Bom'
            WHEN c.num_fallbacks <= 5 THEN 'Regular'
            ELSE 'Problemático'
        END AS categoria_entendimento,
        
        -- Feedbacks
        f.csat,
        f.nps,
        f.comentario,
        f.sentimento,
        f.data_feedback,
        
        -- Classificação CSAT
        CASE 
            WHEN f.csat >= 4 THEN 'Satisfeito'
            WHEN f.csat = 3 THEN 'Neutro'
            WHEN f.csat <= 2 THEN 'Insatisfeito'
            ELSE 'Sem Avaliação'
        END AS categoria_csat,
        
        -- Classificação NPS
        CASE 
            WHEN f.nps >= 9 THEN 'Promotor'
            WHEN f.nps >= 7 THEN 'Neutro'
            WHEN f.nps <= 6 THEN 'Detrator'
            ELSE 'Não Avaliado'
        END AS categoria_nps
        
    FROM conversas c
    LEFT JOIN feedbacks f ON c.id_conversa = f.id_conversa
),

-- Cálculos adicionais com window functions
metricas_enriquecidas AS (
    SELECT *,
        -- Métricas de tendência por usuário
        COUNT(*) OVER (PARTITION BY id_usuario ORDER BY data_inicio 
                      RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW) AS conversas_30d,
        
        -- Ranking de satisfação por intenção
        DENSE_RANK() OVER (PARTITION BY intencao_principal ORDER BY csat DESC) AS rank_satisfacao_intencao,
        
        -- Indicador de usuário recorrente
        ROW_NUMBER() OVER (PARTITION BY id_usuario ORDER BY data_inicio) AS sequencia_usuario,
        
        -- Horário de pico
        CASE 
            WHEN EXTRACT(HOUR FROM data_inicio) BETWEEN 8 AND 11 THEN 'Manhã'
            WHEN EXTRACT(HOUR FROM data_inicio) BETWEEN 12 AND 17 THEN 'Tarde'
            WHEN EXTRACT(HOUR FROM data_inicio) BETWEEN 18 AND 22 THEN 'Noite'
            ELSE 'Madrugada'
        END AS periodo_dia
        
    FROM metricas_base
)

SELECT * FROM metricas_enriquecidas;
```

[[full]]
[[ANÁLISE DE INTENÇÕES]]
[[ANÁLISE DE FALLBACKS (TOP PROBLEMAS)]]