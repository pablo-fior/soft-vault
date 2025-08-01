---
state: "[[Idea]]"
---
Perfeito! Abaixo estão os **scripts de referência** para:

1. **Extração dos dados de interações**
    
2. **Classificação (por sentimento e satisfação)**
    
3. **Cálculo estatístico dos indicadores (CSAT, CES, NPS, PIBS etc.)**
    

---

## 📁 Estrutura dos dados (referência)

Considere que as interações estão salvas em uma tabela (SQLite ou JSON) com os campos:

```ts
{
  id: string;
  usuario_id: string;
  mensagem: string;
  resposta_bot: string;
  data: string;
  duracao_em_segundos: number;
  satisfacao_usuario: number; // 1 a 5
  tipo_feedback?: 'positivo' | 'neutro' | 'negativo';
  canal: 'web' | 'whatsapp' | 'app';
  topico: string;
}
```

---

## 🟦 TypeScript – Extração e Análise (usando Node.js + TypeORM ou SQLite3)

### 1. Instalar dependências

```bash
npm install sqlite3 csv-writer sentiment
```

### 2. Código `analytics.ts`

```ts
import sqlite3 from 'sqlite3';
import Sentiment from 'sentiment';
import { createObjectCsvWriter } from 'csv-writer';

const db = new sqlite3.Database('./chatbot.db');
const sentiment = new Sentiment();

type Registro = {
  id: string;
  mensagem: string;
  satisfacao_usuario: number;
  duracao_em_segundos: number;
  canal: string;
  topico: string;
  data: string;
};

const registros: Registro[] = [];

db.each("SELECT * FROM interacoes", (err, row) => {
  if (err) throw err;

  registros.push({
    id: row.id,
    mensagem: row.mensagem,
    satisfacao_usuario: row.satisfacao_usuario,
    duracao_em_segundos: row.duracao_em_segundos,
    canal: row.canal,
    topico: row.topico,
    data: row.data
  });
}, () => {
  const estatisticas = {
    total: registros.length,
    mediaSatisfacao: media(registros.map(r => r.satisfacao_usuario)),
    tempoMedio: media(registros.map(r => r.duracao_em_segundos)),
    sentimento: registros.map(r => ({
      id: r.id,
      sentimento: sentiment.analyze(r.mensagem).score
    }))
  };

  console.log(estatisticas);
});

function media(arr: number[]): number {
  return arr.reduce((a, b) => a + b, 0) / arr.length;
}
```

---

## 🐍 Python – Extração e Cálculo Estatístico (usando Pandas + SQLite)

### 1. Instalar dependências

```bash
pip install pandas sqlite3 textblob
python -m textblob.download_corpora
```

### 2. Código `analyze_interactions.py`

```python
import pandas as pd
import sqlite3
from textblob import TextBlob

# Conectar ao banco
conn = sqlite3.connect('chatbot.db')

# Carregar os dados
df = pd.read_sql_query("SELECT * FROM interacoes", conn)

# Análise de sentimento
def analisar_sentimento(msg):
    if not msg:
        return 0
    return TextBlob(msg).sentiment.polarity

df['sentimento'] = df['mensagem'].apply(analisar_sentimento)

# Cálculos estatísticos
total = len(df)
csat = df[df['satisfacao_usuario'] >= 4].shape[0] / total
tempo_medio = df['duracao_em_segundos'].mean()
sentimento_medio = df['sentimento'].mean()

print(f"Total de atendimentos: {total}")
print(f"CSAT: {csat:.2%}")
print(f"Tempo médio de atendimento: {tempo_medio:.2f} segundos")
print(f"Sentimento médio: {sentimento_medio:.2f}")

# Exportar CSV
df.to_csv('interacoes_classificadas.csv', index=False)
```

---

## 📍 Roadmap Técnico Integrado

### Fase 1 – Coleta & Estruturação

-  Definir schema da base (campos de interação, resposta, feedback)
    
-  Criar banco SQLite (ou Postgres/MySQL se preferir)
    
-  Criar scripts de extração
    

### Fase 2 – Classificação

-  Implementar análise de sentimento
    
-  Marcar interações como positivas/neutras/negativas
    

### Fase 3 – Indicadores

-  Calcular CSAT, NPS, PIBS, duração média, taxa de resolução
    
-  Exportar para CSV para ingestão no Power BI
    

### Fase 4 – Dashboard e Monitoramento

-  Criar painel em Power BI com os indicadores
    
-  Automatizar importação via agendamento (ex: via script Python)
    
-  Criar alertas para quedas nos índices de satisfação
    

---

Se quiser, posso montar os arquivos `.ts` e `.py` organizados para você rodar com facilidade ou expandir isso para APIs REST ou visualizações web. Deseja isso também?

[[Extração classificação estatística]]
[[Extração classificação estatística]]
[[power bi#🟦 **1. Tabelas base necessárias**]]
