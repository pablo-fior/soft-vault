---
state: "[[Idea]]"
---
```sql
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
```

[[MÉTRICAS CONSOLIDADAS]]
[[ANÁLISE DE INTENÇÕES]]
[[KPIs OPERACIONAIS DIÁRIOS]]
