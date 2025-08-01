---
state: "[[Drafting]]"
---
Para um sistema de **pesquisa de satisfação de chatbot** usado em **atendimento ao cliente de plano de saúde**, é essencial considerar tanto a **estrutura técnica e de coleta de dados**, quanto os **indicadores de performance (KPIs)** e **dados subjetivos e objetivos** que refletem a qualidade do atendimento. Abaixo está um guia completo dividido em:

---

## 🔧 **Estrutura Recomendada**

### 1. **Pontos de Coleta de Feedback**

- **Após o encerramento do atendimento:** pop-up ou mensagem com opções como:
    
    - Escala de 1 a 5 estrelas ou notas de 1 a 10.
        
    - Emoji feedback (😡 😐 🙂 😃).
        
    - Pergunta aberta opcional: “O que podemos melhorar?”
        
- **Eventos-chave:** Solicitar feedback após tarefas sensíveis, como autorização de procedimento ou agendamento de consulta.
    

### 2. **Modelo de Armazenamento e Análise**

- Banco de dados estruturado com:
    
    - ID da conversa
        
    - ID do cliente (anonimizado)
        
    - Timestamp do início e fim
        
    - Nota de satisfação
        
    - Texto do feedback
        
    - Intenções reconhecidas
        
    - Ações executadas pelo bot
        
    - Escalamento para humano (se houver)
        
    - Tempo de resposta (do bot e do cliente)
        
    - Indicador de sucesso da tarefa
        

### 3. **Segmentação**

- Classificar por:
    
    - Tipo de solicitação (boletos, reembolso, rede credenciada, etc.)
        
    - Faixa etária (se permitido)
        
    - Canal (app, web, WhatsApp)
        
    - Horário do atendimento
        

---

## 📊 **Indicadores e Estatísticas Relevantes**

### 1. **KPIs de Satisfação**

- **CSAT (Customer Satisfaction Score)**: Média das notas atribuídas.
    
- **NPS (Net Promoter Score)**: Adaptado para clientes que interagem com o bot.
    
- **Sentimento médio do feedback textual** (usando análise de sentimento NLP).
    

### 2. **KPIs de Performance do Bot**

- **Taxa de resolução sem intervenção humana** (`Self-service rate`)
    
- **Tempo médio de atendimento** (de início ao fim da sessão)
    
- **Tempo médio de resposta do bot**
    
- **Taxa de abandono** (clientes que encerraram antes de finalizar)
    
- **Taxa de redirecionamento para atendimento humano**
    

---

## 🧠 **Dados que Podem Ser Extraídos das Conversas**

### A. **Objetivos de Negócio**

- Tipos de solicitação mais frequentes
    
- Frequência por horário/dia da semana
    
- Identificação de gargalos (fluxos onde usuários travam ou desistem)
    

### B. **Indicadores de Qualidade Conversacional**

- **Número médio de turnos por sessão**
    
- **Número de mensagens mal interpretadas (fallbacks)**
    
- **Uso de palavras negativas ou positivas** (para análise de sentimento)
    
- **Trechos onde o usuário demonstrou frustração** (ex: "isso não ajudou", "quero falar com atendente")
    

### C. **Indicadores Técnicos**

- Entidades extraídas corretamente vs. erros
    
- Precisão da classificação de intenções
    
- Avaliação da coerência dos fluxos
    
- Logs de erro ou exceções
    

---

## 🔍 **Análises Qualitativas Recomendadas**

- **Clustering de tópicos frequentes em feedbacks abertos** (LDA, embedding com clustering)
    
- **Análise de sentimentos agregada por tipo de solicitação**
    
- **Heatmap de interação por fluxo (onde os usuários mais se perdem ou voltam atrás)**
    

---

## 📈 Exemplos de Dashboards

1. **Painel de Satisfação Geral**
    
    - CSAT por mês
        
    - NPS por canal
        
    - Sentimento médio por fluxo
        
2. **Painel de Performance do Bot**
    
    - % de atendimentos resolvidos sem humanos
        
    - Média de tempo por intenção
        
    - Fallbacks mais comuns
        
3. **Painel de Qualidade por Intenção**
    
    - Taxa de erro por tipo de intenção
        
    - Relação entre complexidade do pedido e nota de satisfação
        

---

## ✅ Boas Práticas

- Anonimizar os dados dos usuários sempre que possível (LGPD).
    
- Testar variações de perguntas de satisfação (formato curto x longo).
    
- Oferecer opção de feedback aberto apenas após coleta estruturada.
    
- Integrar feedback negativo a alertas ou revisão manual de conversas.
    

---

Se quiser, posso montar um exemplo de **modelo de banco de dados** ou **dashboard em JSON/SQL/Excel** com base em um cenário que você tiver em mente. Deseja isso?

Perfeito! Vou montar um exemplo completo e realista baseado em um cenário de chatbot para um **plano de saúde**, incluindo:

---

## 🗃️ **Modelo de Banco de Dados Simplificado**

### 🎯 Objetivo:

Registrar **conversas**, **feedbacks**, e dados para análises de performance e satisfação.

### 1. **Tabela: `conversas`**

|Coluna|Tipo|Descrição|
|---|---|---|
|`id_conversa`|UUID|Identificador único da conversa|
|`id_usuario`|UUID|ID do cliente (anonimizado)|
|`canal`|VARCHAR|Ex: WhatsApp, Web, App|
|`data_inicio`|TIMESTAMP|Início da conversa|
|`data_fim`|TIMESTAMP|Fim da conversa|
|`intencao_principal`|VARCHAR|Ex: Solicitar reembolso|
|`foi_escalado`|BOOLEAN|Se houve repasse para atendente humano|
|`resolvido`|BOOLEAN|Se o problema foi resolvido no atendimento|
|`num_mensagens`|INT|Total de mensagens trocadas|
|`num_fallbacks`|INT|Total de falhas de entendimento do bot|
|`tempo_total`|INTERVAL|Duração da conversa|

---

### 2. **Tabela: `feedbacks`**

|Coluna|Tipo|Descrição|
|---|---|---|
|`id_feedback`|UUID|ID único do feedback|
|`id_conversa`|UUID|Relaciona com a conversa|
|`csat`|INTEGER|Nota de 1 a 5|
|`nps`|INTEGER|De -100 a +100 (opcional)|
|`comentario`|TEXT|Texto livre opcional do usuário|
|`sentimento`|VARCHAR|Positivo, Neutro ou Negativo (calculado por NLP)|
|`data_feedback`|TIMESTAMP|Quando foi deixado o feedback|

---

### 3. **Tabela: `logs_mensagens` (opcional, para análise granular)**

|Coluna|Tipo|Descrição|
|---|---|---|
|`id_log`|UUID|ID único|
|`id_conversa`|UUID|ID da conversa|
|`emissor`|VARCHAR|"BOT" ou "USUARIO"|
|`mensagem`|TEXT|Conteúdo da mensagem|
|`timestamp`|TIMESTAMP|Momento da mensagem|
|`entidade`|TEXT|Entidade extraída (se aplicável)|
|`intencao`|TEXT|Intenção identificada (se aplicável)|
|`confiança`|FLOAT|Confiança do modelo (0 a 1)|

---

## 📊 Exemplo de Dashboard (Estrutura em JSON)

```json
{
  "kpis": {
    "csat_medio": 4.3,
    "nps": 56,
    "taxa_resolucao_sem_humano": "74%",
    "tempo_medio_resposta": "6s",
    "fallback_rate": "8%",
    "tempo_medio_conversa": "3m 25s"
  },
  "satisfacao_por_intencao": {
    "Solicitar reembolso": 4.5,
    "Agendar consulta": 4.1,
    "Reclamação": 3.2,
    "Informações plano": 4.8
  },
  "feedbacks_recentes_negativos": [
    {
      "comentario": "O robô não entendeu minha pergunta sobre reembolso.",
      "intencao": "Solicitar reembolso",
      "sentimento": "Negativo"
    },
    {
      "comentario": "Quero falar com um humano, o bot trava toda hora.",
      "intencao": "Agendar consulta",
      "sentimento": "Negativo"
    }
  ],
  "fluxos_mais_utilizados": [
    "Boletos / 34%",
    "Agendamento / 26%",
    "Reembolso / 20%",
    "Rede Credenciada / 10%",
    "Outros / 10%"
  ]
}
```

---

## 🧠 Com esse modelo, você poderá:

- Criar dashboards com ferramentas como Power BI, Grafana, Superset ou Metabase.
    
- Analisar **pontos de fricção** com base na taxa de fallbacks por intenção.
    
- Medir a evolução da satisfação mês a mês e por canal.
    
- Identificar **feedbacks que exigem ação humana imediata**.
    
- Integrar **aprendizado contínuo para treinar seu modelo NLU**.
    

Se quiser, posso te entregar:

1. O **modelo SQL para criar as tabelas**.
    
2. Um **mock de dados para simular análises em CSV ou SQLite**.
    
3. Um esboço de painel em Power BI ou um script de análise em Python/Pandas.
    

---

### 1. Guia passo a passo para criar o dashboard no Power BI Desktop

**Passo a passo básico para importar e montar o dashboard:**

1. **Abra o Power BI Desktop** no seu computador.
    
2. **Importe os dados:**
    
    - Vá em **“Obter dados” > “Texto/CSV”** e importe os arquivos CSV que você tem (`conversas.csv`, `feedbacks.csv`, `tv.csv`, `pibs.csv`).
        
    - Ou conecte direto ao banco SQLite via ODBC:
        
        - Instale o driver ODBC para SQLite (ex: [SQLite ODBC Driver](https://www.ch-werner.de/sqliteodbc/)).
            
        - No Power BI, escolha **“Obter dados” > “Banco de dados” > “ODBC”** e configure para conectar ao seu arquivo `.db`.
            
3. **Modelagem dos dados:**
    
    - Verifique se as tabelas estão relacionadas pelo campo `id_conversa` (crie relacionamento entre elas).
        
    - Configure tipos de dados corretamente (datas, números, texto).
        
4. **Criar visuais:**
    
    - Crie **cartões** para KPIs principais:
        
        - CSAT médio
            
        - Taxa de resolução
            
        - Total de interações
            
    - Crie **gráficos de barras** para intenções com mais fallbacks.
        
    - Crie **tabela ou nuvem de palavras** para comentários negativos.
        
    - Crie filtros para canal, data, sentimento, etc.
        
5. **Salve seu relatório** no formato `.pbix`.
    

---

### 2. Código Python para criar dashboards interativos fora do Power BI

Vou gerar um exemplo usando `plotly` + `pandas` que você pode rodar localmente e acessar via navegador. Esse dashboard mostrará:

- CSAT médio
    
- Taxa de resolução
    
- Gráfico de intenções com mais fallbacks
    
- Lista de comentários negativos
    

---

```python
import sqlite3
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots

# Conectar ao banco SQLite
conn = sqlite3.connect('chatbot_satisfacao.db')

# Carregar dados
df_conversas = pd.read_sql_query("SELECT * FROM conversas", conn)
df_feedbacks = pd.read_sql_query("SELECT * FROM feedbacks", conn)

conn.close()

df_conversas['foi_escalado'] = df_conversas['foi_escalado'].astype(bool)
df_conversas['resolvido'] = df_conversas['resolvido'].astype(bool)

# Calcular KPIs
csat_medio = df_feedbacks['csat'].mean()
taxa_resolucao = (df_conversas['resolvido'].sum() / len(df_conversas)) * 100

# Top intenções com mais fallbacks
fallbacks = df_conversas.groupby('intencao_principal')['num_fallbacks'].sum().sort_values(ascending=False).head(10)

# Comentários negativos
coment_neg = df_feedbacks[(df_feedbacks['sentimento'] == 'Negativo') & (df_feedbacks['comentario'].notna())]['comentario']

# Criar dashboard com Plotly
fig = make_subplots(
    rows=3, cols=1,
    specs=[[{"type": "indicator"}],
           [{"type": "bar"}],
           [{"type": "table"}]],
    subplot_titles=("CSAT Médio", "Top 10 Intenções com mais Fallbacks", "Comentários Negativos")
)

# CSAT médio como indicador
fig.add_trace(go.Indicator(
    mode="number",
    value=csat_medio,
    title={"text": "CSAT Médio"},
), row=1, col=1)

# Gráfico de barras - fallback por intenção
fig.add_trace(go.Bar(
    x=fallbacks.index,
    y=fallbacks.values,
    marker_color='crimson'
), row=2, col=1)

# Tabela de comentários negativos
fig.add_trace(go.Table(
    header=dict(values=["Comentários Negativos"], fill_color='lightgrey'),
    cells=dict(values=[coment_neg.tolist()])
), row=3, col=1)

fig.update_layout(height=800, showlegend=False, title_text="Dashboard de Satisfação do Chatbot")

fig.show()
```

---

### Como usar o dashboard Python:

1. Instale as bibliotecas necessárias:
    
    ```bash
    pip install pandas plotly
    ```
    
2. Salve o código acima em `dashboard_chatbot.py`.
    
3. Execute:
    
    ```bash
    python dashboard_chatbot.py
    ```
    
4. O navegador abrirá com o dashboard interativo.
    

---

Se precisar, posso ajudar a gerar versões mais complexas, com filtros, outros KPIs e mais gráficos. Quer seguir nessa direção?

Show! Vou preparar um código Python mais completo com:

- Filtros interativos (data, canal)
    
- KPIs (CSAT médio, taxa de resolução, total de conversas)
    
- Gráfico de barras com intenções e fallbacks
    
- Tabela com comentários negativos filtráveis
    
- Layout melhorado com títulos e instruções
    

Vou usar **Plotly Dash**, que é excelente para dashboards interativos no navegador.

---

### Código Python completo usando Dash

```python
import sqlite3
import pandas as pd
from datetime import datetime
from dash import Dash, dcc, html, Input, Output
import plotly.graph_objs as go

# Carregar dados do SQLite
def carregar_dados():
    conn = sqlite3.connect('chatbot_satisfacao.db')
    df_conversas = pd.read_sql_query("SELECT * FROM conversas", conn)
    df_feedbacks = pd.read_sql_query("SELECT * FROM feedbacks", conn)
    conn.close()

    # Ajustes de tipos
    df_conversas['foi_escalado'] = df_conversas['foi_escalado'].astype(bool)
    df_conversas['resolvido'] = df_conversas['resolvido'].astype(bool)
    df_conversas['dataInicio'] = pd.to_datetime(df_conversas['dataInicio'])
    df_feedbacks['dataFeedback'] = pd.to_datetime(df_feedbacks['dataFeedback'])

    return df_conversas, df_feedbacks

df_conversas, df_feedbacks = carregar_dados()

# Aplicativo Dash
app = Dash(__name__)
app.title = "Dashboard Satisfação Chatbot"

# Layout
app.layout = html.Div([
    html.H1("Dashboard de Satisfação do Chatbot - Plano de Saúde", style={'textAlign': 'center'}),
    
    html.Div([
        html.Label("Filtrar por Canal:"),
        dcc.Dropdown(
            id='filtro-canal',
            options=[{'label': c, 'value': c} for c in df_conversas['canal'].unique()] + [{'label': 'Todos', 'value': 'Todos'}],
            value='Todos',
            clearable=False
        ),
        html.Label("Filtrar por Período:"),
        dcc.DatePickerRange(
            id='filtro-data',
            min_date_allowed=df_conversas['dataInicio'].min(),
            max_date_allowed=df_conversas['dataInicio'].max(),
            start_date=df_conversas['dataInicio'].min(),
            end_date=df_conversas['dataInicio'].max(),
        ),
    ], style={'width': '30%', 'margin': 'auto', 'padding': '20px', 'display': 'flex', 'flexDirection': 'column', 'gap': '10px'}),
    
    html.Div([
        html.Div(id='kpi-csat', className='kpi-box'),
        html.Div(id='kpi-taxaresolucao', className='kpi-box'),
        html.Div(id='kpi-totalconversas', className='kpi-box'),
    ], style={'display': 'flex', 'justifyContent': 'space-around', 'padding': '20px'}),
    
    dcc.Graph(id='grafico-fallbacks'),
    
    html.H3("Comentários Negativos"),
    html.Div(id='lista-comentarios', style={'whiteSpace': 'pre-wrap', 'maxHeight': '200px', 'overflowY': 'scroll', 'border': '1px solid #ccc', 'padding': '10px', 'margin': '20px'}),
])

# Callbacks para atualizar KPIs e gráficos
@app.callback(
    [
        Output('kpi-csat', 'children'),
        Output('kpi-taxaresolucao', 'children'),
        Output('kpi-totalconversas', 'children'),
        Output('grafico-fallbacks', 'figure'),
        Output('lista-comentarios', 'children')
    ],
    [
        Input('filtro-canal', 'value'),
        Input('filtro-data', 'start_date'),
        Input('filtro-data', 'end_date'),
    ]
)
def atualizar_dashboard(canal, start_date, end_date):
    # Filtrar conversas
    df_filtered = df_conversas.copy()
    df_feedbacks_filtered = df_feedbacks.copy()

    if canal != 'Todos':
        df_filtered = df_filtered[df_filtered['canal'] == canal]
        ids_conversas = df_filtered['idConversa'].unique()
        df_feedbacks_filtered = df_feedbacks_filtered[df_feedbacks_filtered['idConversa'].isin(ids_conversas)]

    df_filtered = df_filtered[(df_filtered['dataInicio'] >= start_date) & (df_filtered['dataInicio'] <= end_date)]
    ids_conversas = df_filtered['idConversa'].unique()
    df_feedbacks_filtered = df_feedbacks_filtered[df_feedbacks_filtered['idConversa'].isin(ids_conversas)]

    # KPIs
    csat_medio = df_feedbacks_filtered['csat'].mean() if not df_feedbacks_filtered.empty else 0
    taxa_resolucao = (df_filtered['resolvido'].sum() / len(df_filtered) * 100) if len(df_filtered) > 0 else 0
    total_conversas = len(df_filtered)

    kpi_csat = html.Div([
        html.H4("CSAT Médio"),
        html.P(f"{csat_medio:.2f}", style={'fontSize': '28px', 'fontWeight': 'bold'})
    ], style={'textAlign': 'center', 'border': '1px solid #ccc', 'padding': '15px', 'borderRadius': '8px', 'width': '150px'})

    kpi_taxa = html.Div([
        html.H4("Taxa de Resolução (%)"),
        html.P(f"{taxa_resolucao:.2f}%", style={'fontSize': '28px', 'fontWeight': 'bold'})
    ], style={'textAlign': 'center', 'border': '1px solid #ccc', 'padding': '15px', 'borderRadius': '8px', 'width': '150px'})

    kpi_total = html.Div([
        html.H4("Total de Conversas"),
        html.P(f"{total_conversas}", style={'fontSize': '28px', 'fontWeight': 'bold'})
    ], style={'textAlign': 'center', 'border': '1px solid #ccc', 'padding': '15px', 'borderRadius': '8px', 'width': '150px'})

    # Gráfico fallbacks por intenção
    fallback_counts = df_filtered.groupby('intencaoPrincipal')['numFallbacks'].sum().sort_values(ascending=False).head(10)
    fig_fallbacks = go.Figure(data=[
        go.Bar(
            x=fallback_counts.index,
            y=fallback_counts.values,
            marker_color='crimson'
        )
    ])
    fig_fallbacks.update_layout(
        title="Top 10 Intenções com mais Fallbacks",
        xaxis_title="Intenção Principal",
        yaxis_title="Número de Fallbacks",
        margin=dict(t=50, b=150),
        xaxis_tickangle=-45
    )

    # Comentários negativos
    comentarios_negativos = df_feedbacks_filtered[
        (df_feedbacks_filtered['sentimento'] == 'Negativo') &
        (df_feedbacks_filtered['comentario'].notna())
    ]['comentario']

    comentarios_texto = "\n\n---\n\n".join(comentarios_negativos.tolist()) if not comentarios_negativos.empty else "Nenhum comentário negativo encontrado."

    return kpi_csat, kpi_taxa, kpi_total, fig_fallbacks, comentarios_texto


if __name__ == '__main__':
    app.run_server(debug=True)
```

---

### Como usar:

1. Instale as bibliotecas necessárias:
    

```bash
pip install pandas dash plotly
```

2. Salve o código em `dashboard_chatbot_dash.py`.
    
3. Coloque o banco `chatbot_satisfacao.db` na mesma pasta.
    
4. Execute o dashboard:
    

```bash
python dashboard_chatbot_dash.py
```

5. Abra o navegador no link indicado (normalmente `http://127.0.0.1:8050`).
    

---

Se quiser posso ajudar a adicionar mais filtros, outros KPIs, gráficos ou exportação de relatórios! Quer?