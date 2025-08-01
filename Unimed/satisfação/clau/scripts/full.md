-- =====================================================
-- SCRIPTS DE TRANSFORMAÇÃO DE DADOS PARA [[power bi]]
-- Sistema de Satisfação Chatbot - Plano de Saúde
-- =====================================================

-- ========================================
-- 1. VIEW PRINCIPAL: [[MÉTRICAS CONSOLIDADAS]]
-- ========================================

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

-- ========================================
-- 2. VIEW [[KPIs OPERACIONAIS DIÁRIOS]]
-- ========================================

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

-- ========================================
-- 3. VIEW [[ANÁLISE DE INTENÇÕES]]
-- ========================================

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

-- ========================================
-- 4. VIEW ANÁLISE TEMPORAL (HEATMAP)
-- ========================================

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

-- ========================================
-- 5. VIEW ANÁLISE DE FALLBACKS (TOP PROBLEMAS)
-- ========================================

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

-- ========================================
-- 6. [[PROCEDURE PARA REFRESH AUTOMATIZADO]]
-- ========================================

CREATE OR REPLACE FUNCTION refresh_views_chatbot()
RETURNS void AS 
BEGIN
    -- Refresh das views materializadas se necessário
    -- (Adapte conforme sua necessidade de performance)
    
    -- Log da execução
    INSERT INTO log_refresh_views (data_refresh, status) 
    VALUES (NOW(), 'Sucesso');
    
    -- Aqui você pode adicionar validações de qualidade dos dados
    -- Por exemplo: verificar se há dados recentes, outliers, etc.
    
END;
LANGUAGE plpgsql;

-- ========================================
-- 7. TABELA DE DIMENSÕES PARA [[power bi]]
-- ========================================

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

-- ========================================
-- 8. [[QUERIES PARA VALIDAÇÃO DE DADOS]]
-- ========================================

-- Query para verificar qualidade dos dados
SELECT 
    'Conversas sem feedback' as metrica,
    COUNT(*) as valor
FROM conversas c
LEFT JOIN feedbacks f ON c.id_conversa = f.id_conversa
WHERE c.data_inicio >= CURRENT_DATE - INTERVAL '7 days'
AND f.id_feedback IS NULL

UNION ALL

SELECT 
    'Conversas com tempo zero' as metrica,
    COUNT(*) as valor
FROM conversas 
WHERE tempo_total = INTERVAL '0'

UNION ALL

SELECT 
    'Feedbacks sem conversa' as metrica,
    COUNT(*) as valor
FROM feedbacks f
LEFT JOIN conversas c ON f.id_conversa = c.id_conversa
WHERE c.id_conversa IS NULL;

-- ========================================
-- COMENTÁRIOS PARA IMPLEMENTAÇÃO NO [[power bi]]
-- ========================================

/*
INSTRUÇÕES PARA CONEXÃO COM [[power bi]]:

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
*/