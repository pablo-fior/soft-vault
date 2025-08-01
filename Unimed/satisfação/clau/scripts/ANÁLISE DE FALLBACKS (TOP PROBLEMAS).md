---
state: "[[Idea]]"
---
```sql
CREATE OR REPLACE VIEW vw_top_fallbacks AS
WITH fallbacks_detalhados AS (
    SELECT 
        l.id_conversa,
        l.mensagem as mensagem_usuario,
        l.intencao,
        l.confiança,
        c.intencao_principal,
        c.foi_escalado,
        f.csat,
        f.sentimento
    FROM logs_mensagens l
    JOIN conversas c ON l.id_conversa = c.id_conversa
    LEFT JOIN feedbacks f ON l.id_conversa = f.id_conversa
    WHERE l.emissor = 'USUARIO' 
    AND (l.confiança < 0.5 OR l.intencao = 'fallback' OR l.intencao IS NULL)
    AND l.timestamp >= CURRENT_DATE - INTERVAL '7 days'  -- Últimos 7 dias
),

patterns_fallback AS (
    SELECT 
        -- Normalização básica da mensagem para agrupamento
        LOWER(TRIM(REGEXP_REPLACE(mensagem_usuario, '[^a-záàâãéèêíïóôõöúçñ\s]', '', 'g'))) as mensagem_normalizada,
        COUNT(*) as frequencia,
        COUNT(CASE WHEN foi_escalado = true THEN 1 END) as escalamentos,
        AVG(csat) as csat_medio,
        COUNT(CASE WHEN sentimento = 'Negativo' THEN 1 END) as sentimentos_negativos,
        
        -- Amostra de mensagens originais para contexto
        STRING_AGG(mensagem_usuario, ' | ' ORDER BY id_conversa LIMIT 3) as exemplos_mensagens
        
    FROM fallbacks_detalhados
    GROUP BY LOWER(TRIM(REGEXP_REPLACE(mensagem_usuario, '[^a-záàâãéèêíïóôõöúçñ\s]', '', 'g')))
    HAVING COUNT(*) >= 2  -- Só padrões que aparecem pelo menos 2x
)

SELECT 
    mensagem_normalizada,
    frequencia,
    escalamentos,
    ROUND(escalamentos * 100.0 / frequencia, 1) as taxa_escalonamento_pct,
    ROUND(csat_medio, 2) as csat_medio,
    sentimentos_negativos,
    ROUND(sentimentos_negativos * 100.0 / frequencia, 1) as pct_sentimento_negativo,
    
    -- Score de prioridade para correção
    ROUND(
        frequencia * 0.4 +  -- 40% frequência
        (escalamentos * 100.0 / frequencia) * 0.35 +  -- 35% taxa escalonamento
        (sentimentos_negativos * 100.0 / frequencia) * 0.25, 2  -- 25% sentimento negativo
    ) as score_prioridade,
    
    exemplos_mensagens
    
FROM patterns_fallback
ORDER BY score_prioridade DESC
LIMIT 50;
```

[[ANÁLISE DE INTENÇÕES]]
[[MÉTRICAS CONSOLIDADAS]]
[[KPIs OPERACIONAIS DIÁRIOS]]
[[QUERIES PARA VALIDAÇÃO DE DADOS]]
[[Análise custo query timeout#Recomendações Holísticas para Otimização em Servidores Próprios]]