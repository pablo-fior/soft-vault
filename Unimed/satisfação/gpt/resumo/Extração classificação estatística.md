---
state: "[[Idea]]"
---
Claro! Abaixo estão os **scripts em TypeScript e Python** organizados por etapas da solução para um sistema de **pesquisa de satisfação de chatbot** voltado ao atendimento de clientes de um plano de saúde. Eles são responsáveis por:

1. **Extração de dados**
    
2. **Classificação (ex: por sentimento ou satisfação)**
    
3. **Cálculo estatístico**
    

---

## 🟦 TypeScript

### 1. **Extração de dados (SQLite como exemplo)**

```ts
// extract.ts
import Database from 'better-sqlite3';
import fs from 'fs';

const db = new Database('./chatbot_feedback.db');

// Extraindo dados da tabela de feedback
const rows = db.prepare('SELECT * FROM feedback').all();

// Salvando como JSON
fs.writeFileSync('./data/feedback.json', JSON.stringify(rows, null, 2));
console.log('Extração concluída.');
```

### 2. **Classificação por Sentimento (usando modelo simplificado)**

```ts
// classify.ts
import fs from 'fs';

interface Feedback {
  id: number;
  comentario: string;
  score: number;
  sentimento?: string;
}

const feedbacks: Feedback[] = JSON.parse(fs.readFileSync('./data/feedback.json', 'utf8'));

// Classificação simplificada por score
feedbacks.forEach(f => {
  if (f.score >= 4) f.sentimento = 'positivo';
  else if (f.score === 3) f.sentimento = 'neutro';
  else f.sentimento = 'negativo';
});

fs.writeFileSync('./data/classified.json', JSON.stringify(feedbacks, null, 2));
console.log('Classificação concluída.');
```

### 3. **Estatísticas básicas**

```ts
// stats.ts
import fs from 'fs';

interface Feedback {
  score: number;
  sentimento: string;
}

const data: Feedback[] = JSON.parse(fs.readFileSync('./data/classified.json', 'utf8'));

const total = data.length;
const media = data.reduce((acc, f) => acc + f.score, 0) / total;

const positivos = data.filter(f => f.sentimento === 'positivo').length;
const negativos = data.filter(f => f.sentimento === 'negativo').length;
const neutros = data.filter(f => f.sentimento === 'neutro').length;

console.log(`Total: ${total}`);
console.log(`Média de score: ${media.toFixed(2)}`);
console.log(`Distribuição: Positivos ${positivos}, Neutros ${neutros}, Negativos ${negativos}`);
```

---

## 🐍 Python

### 1. **Extração**

```python
# extract.py
import sqlite3
import pandas as pd

conn = sqlite3.connect("chatbot_feedback.db")
df = pd.read_sql_query("SELECT * FROM feedback", conn)
df.to_json("data/feedback.json", orient="records", indent=2)
print("Extração concluída.")
```

### 2. **Classificação**

```python
# classify.py
import pandas as pd

df = pd.read_json("data/feedback.json")

def classificar(score):
    if score >= 4:
        return "positivo"
    elif score == 3:
        return "neutro"
    else:
        return "negativo"

df["sentimento"] = df["score"].apply(classificar)
df.to_json("data/classified.json", orient="records", indent=2)
print("Classificação concluída.")
```

### 3. **Estatísticas**

```python
# stats.py
import pandas as pd

df = pd.read_json("data/classified.json")

media = df["score"].mean()
distribuicao = df["sentimento"].value_counts()

print(f"Total: {len(df)}")
print(f"Média de score: {media:.2f}")
print("Distribuição de sentimentos:")
print(distribuicao)
```

---

## 🔄 Observações:

- Os scripts assumem um banco SQLite com uma tabela `feedback(id, comentario, score)`.
    
- As análises e classificações são simples, mas podem ser adaptadas para usar **modelos NLP mais robustos** (como `transformers` em Python ou APIs de IA).
    

Se quiser, posso complementar com:

- Scripts de **ETL programado (CRON)**.
    
- Dashboards com Plotly/Streamlit (Python) ou Chart.js (TS).
    
- Integração com serviços de nuvem ou mensageria.
    

Deseja isso também?

[[Scripts exemplo#2. Código `analytics.ts`]]
[[Scripts exemplo#2. Código `analytics.ts`]]
[[Scripts exemplo]]
