---
state: "[[Drafting]]"
---
```table-of-contents
```

# Estrutura Abrangente para a Mensuração da Satisfação do Paciente com Chatbots no Setor de Planos de Saúde

## Parte I: As Métricas Fundamentais da Satisfação do Paciente: Uma Tríade de Perspectivas

A implementação de um sistema robusto para medir a satisfação com um chatbot de atendimento em um plano de saúde exige uma abordagem multifacetada. Não se trata apenas de perguntar "você está satisfeito?", mas de compreender as diferentes dimensões da experiência do paciente. Para isso, o mercado consolidou uma tríade de métricas principais: Customer Satisfaction Score (CSAT), Customer Effort Score (CES) e Net Promoter Score (NPS). Cada uma oferece uma perspectiva única e, quando utilizadas em conjunto, fornecem uma visão holística e acionável do desempenho do chatbot e do seu impacto na relação com o beneficiário.1

### Seção 1.1: CSAT (Customer Satisfaction Score): Capturando o Sentimento Imediato Pós-Interação

O Customer Satisfaction Score (CSAT) é a métrica mais direta para avaliar a satisfação transacional. Ele responde à pergunta fundamental: "Quão satisfeito o paciente ficou com _esta interação específica_?".3

**Aplicação para o Chatbot de Saúde:** O CSAT é ideal para ser acionado imediatamente após a conclusão de uma tarefa específica pelo chatbot. Exemplos de pontos de gatilho incluem a confirmação de um agendamento de consulta, a verificação da cobertura de um procedimento, a localização de um prestador na rede credenciada ou a resposta a uma dúvida sobre um sinistro. A sua força reside no feedback em tempo real e na sua natureza binária, que sinaliza rapidamente se a interação foi bem-sucedida ou não do ponto de vista do usuário.1

**Metodologia:** A pesquisa de CSAT deve ser curta e de fácil resposta. As abordagens mais comuns incluem uma escala numérica (por exemplo, de 1 a 5), estrelas, ou ícones intuitivos como emojis (polegar para cima/baixo).5 A pergunta deve ser direta e inequívoca, como: "Qual foi o seu nível de satisfação com o atendimento que acabou de receber?".4 O cálculo geralmente expressa o resultado como uma porcentagem de clientes que responderam positivamente (por exemplo, notas 4 e 5 em uma escala de 5).3

**A Nuance Crítica no Contexto da Saúde:** Embora o CSAT seja excelente para medir a _utilidade_ e a _eficiência_ do chatbot, ele pode ser uma ferramenta imprecisa para capturar a complexidade emocional de uma interação de saúde. Um paciente pode estar perfeitamente satisfeito com a rapidez e a clareza do chatbot, mas profundamente insatisfeito com a informação recebida. Por exemplo, um chatbot que informa de forma rápida e correta que um determinado procedimento não tem cobertura está funcionando perfeitamente do ponto de vista técnico. No entanto, o paciente, ao receber a notícia negativa, provavelmente atribuirá uma nota baixa de CSAT.

Sem um contexto adicional, essa nota baixa poderia levar a uma interpretação equivocada de que o chatbot está falhando. A equipe de desenvolvimento poderia investir tempo e recursos tentando "consertar" um fluxo conversacional que, na verdade, já é ótimo. A falha não está no canal de atendimento, mas na percepção do paciente sobre a política do plano. Por isso, é absolutamente mandatório que a pergunta quantitativa do CSAT seja seguida por uma pergunta qualitativa opcional, como: "Para entendermos melhor a sua avaliação, poderia nos contar o motivo da sua nota?".6 É nesta resposta aberta que residem os insights mais valiosos para distinguir entre falhas de serviço e insatisfação com a política.

### Seção 1.2: CES (Customer Effort Score): Quantificando a Facilidade de Resolução das Demandas do Paciente

O Customer Effort Score (CES) mede o nível de esforço que um cliente precisou despender para ter sua solicitação resolvida.7 A premissa é simples: quanto mais fácil for para o cliente interagir com a empresa, maior será sua satisfação e lealdade.

**Aplicação para o Chatbot de Saúde:** No contexto de um plano de saúde, o CES é, possivelmente, a métrica transacional mais crítica. Os pacientes frequentemente interagem com o plano em momentos de estresse, doença ou confusão sobre processos administrativos complexos. Reduzir o esforço cognitivo e processual não é um luxo, mas um componente central de um atendimento empático e eficaz.9 O CES é a métrica perfeita para avaliar a fluidez de jornadas como encontrar um especialista na rede, entender os detalhes de uma fatura, solicitar o reembolso de uma despesa ou submeter um pedido de pré-autorização.10

**Metodologia:** A pesquisa de CES também deve ser simples e direta, com uma pergunta como: "Quão fácil foi resolver o seu problema com a nossa empresa?" ou "O quanto você se esforçou para obter a informação que precisava?".9 A resposta é geralmente coletada em uma escala de 1 a 7, variando de "Muito Difícil" a "Muito Fácil".

**A Conexão com a Confiança e a Eficiência Operacional:** Um CES alto (indicando alto esforço) é um preditor direto do abandono do canal e do escalonamento para o atendimento humano. Se um paciente considera o chatbot difícil de usar, ele perde a confiança na ferramenta digital e recorre a canais mais caros e trabalhosos, como o telefone. Isso não apenas frustra o paciente, mas também anula um dos principais objetivos do chatbot: a eficiência operacional.

Uma experiência de alto esforço, como tentar encontrar um médico especialista e o chatbot não entender sinônimos ("derma" para "dermatologista") ou exigir o nome exato do procedimento, impacta diretamente outros KPIs operacionais. O paciente frustrado abandona a conversa (aumentando a **Taxa de Abandono**) e liga para a central de atendimento (aumentando a **Taxa de Escalonamento**).11 O CES elevado é a causa raiz da falha desses outros indicadores. Portanto, otimizar para um baixo esforço (baixo CES) não é apenas uma questão de satisfação, mas uma estratégia direta para melhorar a eficiência operacional e reduzir custos.

### Seção 1.3: NPS (Net Promoter Score): Medindo a Lealdade e a Confiança Geral no Plano de Saúde

O Net Promoter Score (NPS) é uma métrica que transcende a interação individual para medir a lealdade do cliente a longo prazo e sua disposição de recomendar a marca.6 Ele responde à pergunta estratégica: "Qual a probabilidade de você recomendar nosso plano de saúde a um amigo ou familiar?".

**Aplicação para o Chatbot de Saúde:** É crucial entender que o NPS é uma métrica _relacional_, não _transacional_.2 Portanto, não deve ser utilizado para avaliar uma única conversa com o chatbot. O NPS mede a percepção geral sobre o plano de saúde, e o chatbot é um dos muitos pontos de contato que contribuem para essa percepção. A sua aplicação correta é através de pesquisas periódicas (por exemplo, trimestrais ou semestrais) enviadas a uma amostra de beneficiários, incluindo aqueles que interagiram recentemente com o chatbot.

**Metodologia:** A metodologia do NPS é padronizada globalmente. Utiliza-se a pergunta "Em uma escala de 0 a 10, o quanto você indicaria nossa empresa para um amigo ou colega?". As respostas classificam os clientes em três categorias: Detratores (notas 0-6), Neutros (notas 7-8) e Promotores (notas 9-10).3 O score final é calculado pela fórmula:

NPS=(% de Promotores)−(% de Detratores), resultando em um número que pode variar de -100 a +100.

**Conectando os Pontos para uma Análise Estratégica:** O verdadeiro poder do NPS para avaliar o chatbot vem da segmentação das respostas. Ao cruzar os dados da pesquisa de NPS com os registros de uso do chatbot, é possível obter insights estratégicos profundos. A análise deve comparar o NPS de beneficiários que utilizam o chatbot com frequência com o daqueles que não o utilizam.

Se o grupo de usuários do chatbot apresentar um NPS significativamente mais alto, isso constitui uma forte evidência de que o chatbot é um ativo valioso, que melhora a experiência geral e fortalece o relacionamento com a marca. Por outro lado, se o NPS desse grupo for consistentemente mais baixo, é um sinal de alerta de que o chatbot, mesmo que apresente scores de CSAT aceitáveis em interações isoladas, pode estar, na verdade, erodindo a lealdade e prejudicando a reputação da marca a longo prazo. Esta análise fornece um argumento poderoso e baseado em dados para justificar investimentos na otimização da plataforma de IA ou para realizar uma reformulação completa da experiência conversacional.

### Seção 1.4: Um Modelo Híbrido Estratégico: Integrando CSAT, CES e NPS para uma Visão Holística

Nenhuma métrica isolada é capaz de contar a história completa da experiência do paciente. A abordagem mais robusta e estratégica é a integração das três métricas, criando uma visão multidimensional e complementar.1 O CSAT oferece o pulso transacional imediato, o CES mede o atrito na jornada do paciente, e o NPS monitora a saúde do relacionamento a longo prazo.

**Estratégia de Implementação Integrada:**

1. **Pós-Interação Transacional:** Imediatamente após a conclusão de uma tarefa ou a resolução de uma dúvida, aplique uma pesquisa curta que combine CSAT e CES. Por exemplo: "Em uma escala de 1 a 5, qual o seu nível de satisfação com a solução oferecida pelo nosso assistente virtual? (CSAT)" seguida por "E quão fácil foi obter essa solução? (CES)".
    
2. **Pesquisa Relacional Periódica:** Utilize o NPS em campanhas separadas, via e-mail ou SMS, para avaliar o impacto do chatbot e de outros canais na lealdade geral do beneficiário, conforme descrito na seção anterior.
    
3. **Análise Correlacional:** O objetivo final é correlacionar os dados. Por exemplo, é possível analisar se interações com um CES alto (muito esforço) estão consistentemente associadas a scores de NPS mais baixos ao longo do tempo? Ou se um CSAT baixo está ligado a uma jornada específica? Esta análise integrada transforma a medição em insight estratégico.
    

A tabela a seguir resume as características, aplicações e nuances de cada métrica, servindo como um guia de referência para a tomada de decisão.

**Tabela 1: Comparativo das Métricas Fundamentais de Satisfação (CSAT, CES, NPS)**

|Métrica|Pergunta Central|O que Mede|Aplicação Ideal no Chatbot de Saúde|Vantagens|Desvantagens|
|---|---|---|---|---|---|
|**CSAT** (Customer Satisfaction Score)|"Qual o seu nível de satisfação com esta interação?" 4|Satisfação transacional e imediata com um ponto de contato específico.|Imediatamente após a resolução de uma dúvida, agendamento de consulta ou conclusão de um fluxo.|Simples, rápido, fornece feedback em tempo real sobre a eficácia da interação.1|Pode ser influenciado por fatores externos à performance do chatbot (ex: políticas do plano). Não mede lealdade.3|
|**CES** (Customer Effort Score)|"Quão fácil foi resolver o seu problema?" 7|O esforço percebido pelo cliente para obter uma solução ou realizar uma tarefa.|Após jornadas processuais como encontrar um prestador, entender uma fatura ou solicitar uma autorização.|Forte preditor de lealdade e de comportamento futuro (recompra, abandono). Foco na simplicidade da experiência.7|Não captura a dimensão emocional da interação; uma experiência pode ser fácil, mas impessoal.|
|**NPS** (Net Promoter Score)|"Em uma escala de 0 a 10, o quanto você recomendaria nosso plano de saúde?" 6|Lealdade do cliente e saúde geral do relacionamento com a marca.|Em pesquisas periódicas (não transacionais) para avaliar o impacto do chatbot na percepção geral da marca.|Métrica padronizada, fácil de comparar (benchmarking). Foco no crescimento e na lealdade a longo prazo.2|Não fornece feedback específico sobre pontos de contato individuais. É um indicador macro.3|

## Parte II: Arquitetando o Sistema de Pesquisa de Satisfação: Melhores Práticas para o Contexto da Saúde

Com as métricas fundamentais definidas, o próximo passo é desenhar e implementar o sistema de pesquisa de forma eficaz. No setor de saúde, onde a clareza, a empatia e o respeito pelo tempo do paciente são primordiais, a arquitetura da pesquisa é tão importante quanto as métricas que ela mede.

### Seção 2.1: Desenhando Perguntas Eficazes: Clareza, Empatia e Neutralidade

A qualidade dos dados coletados é diretamente proporcional à qualidade das perguntas formuladas. Uma pesquisa mal desenhada pode gerar dados inúteis ou, pior, enganosos.

- **Clareza e Concisão:** As perguntas devem ser formuladas em linguagem simples, direta e livre de jargões médicos ou de seguros.5 É preciso presumir que o paciente pode estar estressado, doente ou realizando múltiplas tarefas. Pesquisas longas e complexas levam à fadiga e ao abandono.14 A recomendação é limitar as pesquisas a poucas perguntas essenciais, idealmente não mais do que 10-15 para pesquisas mais longas e 2-3 para pesquisas transacionais.5
    
- **Fraseado Neutro:** É fundamental evitar perguntas tendenciosas que possam influenciar a resposta do usuário. Em vez de perguntar "Quão excelente foi o nosso assistente virtual?", uma formulação neutra e mais eficaz seria "Como você avaliaria a assistência que recebeu?".14 A neutralidade garante que a avaliação reflita a verdadeira percepção do paciente, e não uma sugestão implícita na pergunta.
    
- **Empatia e Tom de Voz:** O tom da pesquisa deve ser consistente com a persona do chatbot, que no contexto da saúde deve ser empática, profissional e solícita.15 A forma como o feedback é solicitado, especialmente após uma interação sensível, pode impactar tanto a taxa de resposta quanto a qualidade do feedback.
    
- **O Poder da Pergunta Aberta:** Como já mencionado, é crucial incluir sempre uma pergunta aberta opcional após a avaliação quantitativa. Questões como "Poderia nos contar um pouco mais sobre sua avaliação?" ou "Há algo que poderíamos ter feito melhor?" são fontes inestimáveis de insights qualitativos.5 É nesse campo de texto livre que os pacientes explicam o "porquê" por trás de suas notas, revelando pontos de atrito, sugestões de melhoria e elogios específicos que os dados quantitativos jamais capturariam.
    

### Seção 2.2: Timing e Gatilhos Conversacionais Ideais para a Aplicação da Pesquisa

O momento em que a pesquisa é apresentada ao usuário é um fator crítico para a sua eficácia. O objetivo é capturar o feedback quando a experiência está fresca na memória do paciente, sem ser intrusivo.

- **Pós-Resolução Imediata:** O momento mais eficaz para solicitar feedback transacional (CSAT/CES) é imediatamente após o chatbot confirmar que a solicitação do usuário foi resolvida ou quando o próprio usuário indica o fim da conversa.7 A memória da interação está vívida, o que leva a um feedback mais preciso.14
    
- **Gatilho Pré-Escalonamento:** Quando um usuário solicita explicitamente falar com um atendente humano, este é um momento de falha do chatbot e uma oportunidade de ouro para coletar dados. Antes de efetuar a transferência, o chatbot pode perguntar de forma útil: "Entendido. Para que eu possa direcionar seu atendimento da melhor forma, poderia me dizer com qual questão o assistente virtual não conseguiu te ajudar?". Esta abordagem captura o ponto de falha específico no exato momento em que ele ocorre.
    
- **Timeout por Inatividade:** Se uma conversa é abandonada devido à inatividade do usuário, pode-se configurar um gatilho para um acompanhamento posterior (por exemplo, via SMS ou e-mail, se houver consentimento prévio). Uma mensagem como "Notamos que nossa conversa não foi concluída. Houve algum problema?" pode ajudar a recuperar feedback sobre pontos de atrito que levaram ao abandono.17
    
- **Evitar a Fadiga de Pesquisa:** É contraproducente pesquisar cada interação de um mesmo usuário. Devem ser implementadas regras de negócio para limitar a frequência das pesquisas (por exemplo, no máximo uma pesquisa por usuário a cada 24 horas ou por semana). Respeitar o tempo do paciente é fundamental para manter o engajamento e evitar que ele ignore futuras solicitações de feedback.14
    

### Seção 2.3: Melhores Práticas de Canal e Formato

A escolha do canal e do formato da pesquisa deve priorizar a conveniência do paciente e a redução do atrito.

- **Pesquisas Dentro do Chat:** Para feedback imediato, as pesquisas devem ocorrer dentro da própria janela de conversação. O uso de botões de resposta rápida, escalas de clique ou emojis torna a avaliação instantânea e intuitiva, exigindo esforço mínimo do usuário.5
    
- **Engajamento Pós-Chat (E-mail/SMS/WhatsApp):** Para pesquisas mais detalhadas ou para as pesquisas relacionais como o NPS, um canal assíncrono como e-mail ou SMS é mais apropriado. Isso permite que o paciente responda no momento que lhe for mais conveniente. O uso de canais onde os clientes já estão confortáveis, como o WhatsApp, pode aumentar significativamente as taxas de resposta, pois demonstra que a empresa valoriza o tempo deles.5
    
- **Acessibilidade:** As pesquisas devem ser acessíveis a todos os beneficiários, o que pode incluir a oferta em múltiplos idiomas, dependendo da demografia da base de clientes. O design deve ser simples e compatível com leitores de tela para garantir que pessoas com deficiência visual também possam fornecer seu feedback.14
    
- **Anonimato e Confidencialidade:** É vital comunicar de forma explícita que o feedback é confidencial e será usado exclusivamente para a melhoria dos serviços. No setor de saúde, onde a privacidade é uma preocupação primordial, garantir o anonimato encoraja respostas mais honestas e críticas, pois os pacientes podem temer que um feedback negativo possa, de alguma forma, impactar seu atendimento ou cobertura.14
    

## Parte III: Além das Pesquisas – Desvendando Insights a Partir de Dados Brutos de Conversação

As pesquisas de satisfação fornecem o "o quê". A análise dos dados brutos de conversação revela o "porquê". Os logs do chatbot são uma fonte riquíssima e não filtrada de informações sobre o comportamento do usuário, a performance da IA e os pontos de atrito da jornada. Esta análise avançada é o que diferencia um sistema de medição bom de um sistema de medição verdadeiramente estratégico.

### Seção 3.1: KPIs Operacionais Essenciais: O Painel de Desempenho do Chatbot

Embora não sejam medidas diretas de satisfação, os Indicadores-Chave de Desempenho (KPIs) operacionais são _proxies_ poderosos. Um chatbot que apresenta bom desempenho nestas métricas tem uma probabilidade muito maior de satisfazer os usuários. Eles fornecem uma linha de base objetiva e quantitativa da eficiência e eficácia do chatbot.12

**Principais KPIs a serem monitorados:**

- **Taxa de Sucesso da Tarefa (ou Taxa de Resolução):** A porcentagem de interações em que o chatbot conseguiu resolver a consulta do usuário com sucesso, sem necessidade de intervenção humana. Este é, sem dúvida, o KPI de performance mais importante.12
    
- **Resolução no Primeiro Contato (FCR - First Contact Resolution):** Similar à Taxa de Sucesso, mas com a nuance de medir se o problema foi resolvido na _primeira conversa_, sem que o usuário precise retornar pelo mesmo motivo. Um FCR alto indica eficiência e resolutividade.11
    
- **Taxa de Escalonamento (ou Taxa de Transbordo):** A porcentagem de conversas que são transferidas para um atendente humano. Uma taxa elevada é um forte indicador de que a base de conhecimento do chatbot é insuficiente, seu design conversacional é falho ou os problemas dos usuários são mais complexos do que o previsto.12
    
- **Taxa de Contenção:** O inverso da Taxa de Escalonamento. Representa a porcentagem de conversas totalmente gerenciadas pelo bot, do início ao fim.
    
- **Taxa de Abandono:** A porcentagem de usuários que encerram a conversa antes de chegar a uma resolução. É um sinal claro de atrito, frustração ou longos tempos de espera.11
    
- **Tempo Médio de Atendimento (TMA):** A duração média de uma conversa com o chatbot. Um TMA muito longo pode indicar um fluxo conversacional confuso ou ineficiente, enquanto um TMA muito curto pode indicar que o bot não está engajando o usuário para resolver problemas complexos.11
    

A tabela a seguir apresenta um painel de controle sugerido para o monitoramento contínuo da saúde operacional do chatbot.

**Tabela 2: Indicadores de Desempenho (KPIs) Essenciais para um Chatbot de Saúde**

|KPI|Definição|Fórmula Sugerida|Por que Importa para a Satisfação|Meta Exemplo|
|---|---|---|---|---|
|**Taxa de Sucesso da Tarefa**|% de conversas em que o objetivo do usuário foi alcançado pelo bot.|((Conversas Resolvidas)/(Total de Conversas com Intenc¸​a˜o Clara))×100|Indica diretamente a eficácia e utilidade do bot. Usuários que resolvem seus problemas ficam mais satisfeitos.|> 80%|
|**Taxa de Escalonamento**|% de conversas transferidas para um atendente humano.|((Conversas Escaladas)/(Total de Conversas))×100|Um baixo escalonamento significa que o bot é autossuficiente. Escalonamento excessivo gera frustração e custos.|< 15%|
|**Taxa de Contenção**|% de conversas tratadas inteiramente pelo bot.|100%−Taxa de Escalonamento|Mede a eficiência e o ROI do chatbot. Uma alta contenção demonstra a capacidade do bot de resolver problemas de ponta a ponta.|> 85%|
|**Taxa de Abandono**|% de usuários que desistem da conversa antes da resolução.|((Conversas Abandonadas)/(Total de Conversas Iniciadas))×100|Sinaliza atrito, confusão ou demora. Pacientes frustrados abandonam o canal e ficam insatisfeitos.|< 10%|
|**Tempo Médio de Atendimento (TMA)**|Duração média das interações completas com o bot.|(Tempo Total de Todas as Conversas)/(Nuˊmero Total de Conversas)|Um TMA equilibrado sugere eficiência. Tempos muito longos podem indicar um fluxo confuso.|Varia por complexidade (ex: 3-5 min)|

### Seção 3.2: Análise Forense dos Logs de Conversação: Identificando Atritos, Falhas e Pontos de Abandono

Os KPIs oferecem uma visão macro. A análise forense dos logs de conversação fornece o detalhe micro para identificar exatamente onde e por que o chatbot está falhando.21

**Técnicas de Análise:**

- **Análise de Eventos "Não Entendido" (Fallbacks):** Identificar as perguntas e frases que o chatbot mais falha em compreender. Uma alta concentração de fallbacks em um tópico específico (ex: "reembolso de vacina") aponta para uma lacuna clara na base de conhecimento ou no treinamento do modelo de Processamento de Linguagem Natural (PLN).23
    
- **Mapeamento de Pontos de Abandono (Drop-off):** Analisar os fluxos de conversação para identificar em qual etapa específica os usuários mais desistem. Isso pode revelar uma pergunta confusa, um bug técnico ou um passo que exige um esforço desproporcional do paciente.23
    
- **Análise de "Motivos de Conclusão":** Rastrear sistematicamente _por que_ as conversas terminam. Plataformas de chatbot mais avançadas categorizam o fim das conversas (ex: fluxo concluído com sucesso, inatividade do usuário, excesso de tentativas incorretas, escalonamento para humano). Uma alta taxa de "tentativas incorretas" em um campo de entrada de dados é um sinal inequívoco de um ponto de atrito que precisa ser redesenhado.25
    

Essa análise forense permite uma resolução de problemas proativa. Em vez de esperar que um paciente reclame de um processo confuso, a equipe pode identificar o passo problemático ao observar o comportamento agregado de centenas de usuários nos logs e corrigi-lo para todos. Por exemplo, se o painel de KPIs mostra uma alta **Taxa de Abandono** e a análise dos logs revela que 70% desses abandonos ocorrem após o chatbot solicitar o "número da carteirinha", o diagnóstico é claro: os pacientes podem não ter esse número em mãos facilmente. A correção estratégica é redesenhar o fluxo para permitir a autenticação por outros meios, como o CPF ou a data de nascimento. Isso reduz o atrito, diminui a taxa de abandono e melhora diretamente o CES.

### Seção 3.3: O Poder da Análise de Sentimento: Monitorando a Emoção do Paciente em Tempo Real

A análise de sentimento utiliza IA e PLN para classificar automaticamente a linguagem escrita do usuário como positiva, negativa ou neutra.26 Isso funciona como um barômetro emocional em tempo real para cada conversa, adicionando uma camada de dados emocionais aos KPIs operacionais.

**Aplicações para o Chatbot de Saúde:**

- **Sistema de Alerta Precoce:** Uma mudança brusca de sentimento de neutro para negativo durante uma conversa é um forte sinal de alerta. Isso pode acionar um alerta para uma equipe de monitoramento ou até mesmo uma oferta proativa de transferência para um atendente humano, permitindo intervir e potencialmente reverter uma experiência ruim antes que ela termine.29
    
- **Contextualização de KPIs:** Uma conversa pode ser tecnicamente bem-sucedida (o bot resolveu a questão), mas ter um sentimento altamente negativo. Isso remete ao cenário da "notícia ruim" (ex: procedimento não autorizado). A análise de sentimento ajuda a diferenciar entre uma falha do chatbot e a decepção do paciente com uma decisão do plano, permitindo uma análise mais precisa da performance do canal.
    

Análises de sentimento mais sofisticadas podem ir além da simples polaridade (positivo/negativo) para detectar emoções mais granulares como confusão, urgência ou frustração. Treinar um modelo para reconhecer frases como "não estou entendendo", "preciso de ajuda urgente" ou "estou tentando há horas" permite criar um sistema de roteamento e análise muito mais inteligente. Por exemplo, um modelo treinado para o contexto da saúde pode diferenciar "Onde fica a clínica da Rua X?" (sentimento neutro) de "Estou com muita dor, preciso de um pronto-socorro perto da Rua X AGORA" (sentimento de alta urgência). Essa tag de "urgência" pode ser usada para contornar os fluxos padrão e oferecer imediatamente informações de emergência ou um botão de chamada, uma aplicação que pode ter um impacto significativo no bem-estar do paciente.

### Seção 3.4: Mineração de Texto Avançada: Descobrindo o "Porquê" por Trás das Notas Através da Análise de Tópicos e Intenções

A mineração de texto (text mining) vai um passo além da análise de sentimento. Em vez de apenas medir a emoção, ela busca identificar automaticamente os _tópicos_ e as _intenções_ do usuário dentro das conversas.31 Ela responde à pergunta: "Sobre o que nossos pacientes estão realmente falando?".

**Aplicações para o Chatbot de Saúde:**

- **Identificação de Problemas Emergentes:** Ao analisar os tópicos mais frequentes em conversas escaladas ou com sentimento negativo, o plano de saúde pode identificar problemas em larga escala quase em tempo real. Por exemplo, um aumento súbito de perguntas sobre uma nova política de coparticipação indica que a comunicação sobre essa mudança foi falha e está gerando confusão.
    
- **Otimização da Base de Conhecimento:** A mineração de texto pode revelar as perguntas mais frequentes que o chatbot _não consegue_ responder. Isso cria uma lista de prioridades clara e orientada por dados para a criação de novos conteúdos e o desenvolvimento de novas funcionalidades no chatbot.33
    

O maior valor da mineração de texto está em descobrir a "necessidade não dita". Os pacientes nem sempre usam a terminologia correta ou articulam seu problema de forma clara. A mineração de texto pode agrupar frases semanticamente relacionadas. Por exemplo, ela pode aprender que "cobrança indevida", "minha fatura veio errada" e "não reconheço este valor no meu boleto" se referem ao mesmo tópico de "Disputas de Faturamento". Ao monitorar o volume deste tópico, o plano de saúde pode identificar a causa raiz de possíveis erros sistêmicos de faturamento. Isso representa uma mudança de paradigma: de simplesmente melhorar o chatbot para melhorar o _processo de negócio_ que origina a dúvida do paciente, gerando um impacto muito mais profundo na satisfação e na eficiência.

## Parte IV: A Estrutura Estratégica para a Melhoria Contínua

Coletar dados é apenas o primeiro passo. O valor real é gerado quando esses dados são sintetizados em uma estratégia coesa e acionável, criando um sistema de aprendizado que melhora continuamente a experiência do paciente e a eficiência operacional.

### Seção 4.1: A Matriz de Correlação: Vinculando Notas de Pesquisa a KPIs Operacionais e Tópicos de Conversa

O objetivo final é criar uma visão unificada que conecte todas as fontes de dados. Isso pode ser materializado em um painel de controle ou relatório que correlaciona as métricas de satisfação (CSAT, CES, NPS) com os KPIs operacionais e os insights da análise de conversas.

**Exemplos de Correlações a serem Investigadas:**

- Cruzar os scores de **CSAT** com a **Taxa de Sucesso da Tarefa**. Existe uma correlação positiva clara? Se não, por quê?
    
- Analisar o score médio de **CES** para conversas que foram **escaladas** versus aquelas que foram **contidas**. A diferença é estatisticamente significativa?
    
- Identificar os 5 principais **tópicos** (via mineração de texto) que aparecem em conversas com os scores de **CSAT** mais baixos.
    
- Monitorar o **NPS** de usuários que experimentaram uma alta **Taxa de Escalonamento** em suas interações com o chatbot.
    

Essa matriz funciona como uma ferramenta de diagnóstico. Um score de CSAT baixo é um sintoma. A matriz de correlação ajuda a encontrar a doença. Por exemplo, se scores de CSAT baixos estão altamente correlacionados com o tópico "Status da Pré-autorização" e com uma alta Taxa de Escalonamento para esse mesmo tópico, o diagnóstico é claro: o chatbot está falhando em fornecer informações sobre pré-autorizações, e essa falha está causando diretamente a insatisfação do paciente e sobrecarregando o atendimento humano.

### Seção 4.2: Estabelecendo um Sistema de Feedback de Circuito Fechado (Closed-Loop)

Dados sem ação são inúteis. Um sistema de feedback de circuito fechado é um processo formal para transformar insights em melhorias tangíveis e mensuráveis.

**O Processo de 5 Passos:**

1. **Coletar:** Reunir dados de todas as fontes de forma contínua (Pesquisas, KPIs, Logs, Sentimento, Mineração de Texto).
    
2. **Analisar:** Utilizar a Matriz de Correlação para identificar e priorizar os problemas mais críticos (aqueles com maior impacto negativo na satisfação e nos custos operacionais).
    
3. **Agir:** Atribuir o problema identificado à equipe responsável. É uma falha de design do chatbot (requer ajuste no fluxo conversacional)? Uma lacuna de conhecimento (precisa adicionar conteúdo à base de dados)? Um problema no processo de negócio (requer alerta ao departamento de sinistros ou faturamento)?
    
4. **Informar:** Comunicar as melhorias aos pacientes. Se um processo que frustrava muitos usuários foi corrigido, uma simples mensagem proativa como "Ótima notícia! Agora você pode consultar o status do seu reembolso diretamente pelo nosso assistente virtual" pode reconstruir a confiança e incentivar a readoção do canal.
    
5. **Monitorar:** Acompanhar os KPIs e as métricas de satisfação relevantes após a implementação da mudança para confirmar que a correção foi eficaz e medir o impacto da melhoria.
    

### Seção 4.3: Estudos de Caso Ilustrativos: Lições de Implementações em Saúde e Seguros

Para tornar as recomendações mais tangíveis, é útil analisar exemplos do mundo real que demonstram o impacto dessas estratégias.

- **General Health Clinic (MarkiTech):** Este caso reportou uma redução de 80% nos tempos de espera e um aumento de 60% na satisfação do paciente. Ele ilustra como a automação de consultas de rotina impacta diretamente a percepção de qualidade do serviço e a eficiência.35
    
- **Fairfax Colon & Rectal Surgery (Voiceoc):** Com 95% de satisfação do paciente e uma redução de 33% na necessidade de pessoal de atendimento, este exemplo conecta diretamente a eficiência do chatbot à satisfação do paciente e a uma redução de custos operacionais significativa.36
    
- **AG2R La Mondiale (Inbenta):** Este estudo de caso de uma seguradora francesa é um exemplo perfeito da aplicação da mineração de texto. Ao analisar as conversas do chatbot, a empresa descobriu que uma grande frustração dos clientes era o curto período de tempo que os documentos ficavam disponíveis online. Ao estender esse período, a empresa observou um aumento quantificável na satisfação geral. Isso demonstra como o chatbot pode ser um sensor para identificar e corrigir falhas em processos de negócio adjacentes.37
    
- **Sensely ("Molly") e Babylon Health:** Estes são exemplos de aplicações mais avançadas, focadas no suporte a pacientes com doenças crônicas e em consultas iniciais. Eles indicam a trajetória futura dos chatbots na saúde, movendo-se de tarefas puramente administrativas para funções de suporte clínico.38
    

### Seção 4.4: Recomendações Conclusivas: Um Roteiro Estratégico para Implementação

A implementação de um sistema tão completo deve ser feita em fases, permitindo que a organização desenvolva maturidade analítica e gere valor em cada etapa.

- **Fase 1: Medição Fundamental (Meses 1-3)**
    
    - Implementar pesquisas de **CSAT** e **CES** imediatamente após as interações transacionais.
        
    - Configurar um painel de controle para os **KPIs operacionais** essenciais (Taxa de Sucesso, Taxa de Escalonamento, etc.).
        
    - Iniciar revisões manuais semanais de uma amostra de conversas fracassadas ou escaladas para obter os primeiros insights qualitativos.
        
- **Fase 2: Análise Avançada (Meses 4-6)**
    
    - Implementar uma ferramenta de **análise de sentimento** para começar a rastrear a emoção do paciente em tempo real.
        
    - Começar a usar **mineração de texto** para categorizar os tópicos das conversas escaladas e com baixa satisfação.
        
    - Lançar a primeira pesquisa relacional de **NPS** para estabelecer uma linha de base da lealdade geral e começar a segmentar os resultados.
        
- **Fase 3: Otimização Estratégica (A partir do Mês 7)**
    
    - Construir a **Matriz de Correlação** integrada para conectar todas as fontes de dados em um único painel analítico.
        
    - Formalizar o processo de **feedback de circuito fechado (closed-loop)**, criando um comitê ou squad de "Melhoria do Chatbot" com membros das áreas de Experiência do Cliente (CX), TI e Operações.
        
    - Utilizar os insights não apenas para melhorar o chatbot, mas para recomendar ativamente mudanças em processos de negócio subjacentes que causam atrito para o paciente.
        

Em conclusão, a medição da satisfação com o chatbot de um plano de saúde não é um projeto com início, meio e fim, mas uma função estratégica e contínua. Quando executada corretamente, ela se torna um canal direto para a voz do paciente, permitindo que a organização construa confiança, reduza custos operacionais e, o mais importante, ofereça uma experiência de cuidado mais empática, eficiente e humana.
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

| Coluna        | Tipo      | Descrição                            |
| ------------- | --------- | ------------------------------------ |
| `id_log`      | UUID      | ID único                             |
| `id_conversa` | UUID      | ID da conversa                       |
| `emissor`     | VARCHAR   | "BOT" ou "USUARIO"                   |
| `mensagem`    | TEXT      | Conteúdo da mensagem                 |
| `timestamp`   | TIMESTAMP | Momento da mensagem                  |
| `entidade`    | TEXT      | Entidade extraída (se aplicável)     |
| `intencao`    | TEXT      | Intenção identificada (se aplicável) |
| `confiança`   | FLOAT     | Confiança do modelo (0 a 1)          |

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