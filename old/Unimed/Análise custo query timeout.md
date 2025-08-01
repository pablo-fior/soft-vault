# Análise Comparativa de Custo Computacional: Consulta SQL Complexa vs. Agendamento de Timers em Node.js em Servidores Próprios

## 1. Introdução

### Contextualização da Importância da Otimização de Custo Computacional em Ambientes de Servidores Próprios

Em ambientes de servidores próprios (on-premise), o custo computacional não se traduz diretamente em uma fatura de provedor de nuvem, mas em consumo direto de recursos de hardware – CPU, memória, disco e rede. Este consumo impacta diretamente a capacidade de processamento do sistema, o tempo de resposta das aplicações e a escalabilidade geral. Uma gestão ineficiente desses recursos pode rapidamente levar a gargalos de performance, degradação da experiência do usuário e a necessidade de investimentos prematuros e desnecessários em hardware mais potente.

Para um DBA, a otimização é uma disciplina contínua, crucial para garantir a sustentabilidade, a robustez e a performance de aplicações críticas, especialmente aquelas que lidam com alto volume de dados ou operações de agendamento intensivas. A capacidade de identificar e mitigar fontes de alto consumo de recursos é fundamental para a saúde e longevidade de qualquer sistema de informação.

### Apresentação dos Dois Cenários a Serem Analisados

Serão examinados dois cenários distintos, mas com potencial impacto significativo no custo computacional em um ambiente de servidores próprios:

1. **Cenário SQL:** Uma consulta SQL complexa, executada a cada 3 minutos, em uma tabela que cresce em 5 mil linhas diárias. Esta consulta envolve múltiplos `INNER JOIN`s, condições `WHERE` complexas (incluindo busca por `NULL`, um `flag string` e um `id` de parâmetro), uma subquery `NOT IN` com `COALESCE`, e uma condição `AND` que utiliza a função `TIMESTAMPDIFF(MINUTE)`.

2. **Cenário Node.js:** O agendamento de 5 mil `setTimeout` em JavaScript por dia, cada um com um limite de 60 minutos, executados em um backend Node.js.

O objetivo desta análise é desmistificar o "custo computacional" para cada cenário, detalhando os recursos específicos consumidos, identificando os principais gargalos e, finalmente, comparando a eficiência de cada abordagem para determinar qual impõe a maior carga e por quê.

## 2. Custo Computacional no Cenário SQL

### 2.1. Definição de Custo Computacional em Banco de Dados

Ao avaliar o custo computacional de uma consulta em banco de dados, a atenção se volta para três pilares fundamentais de consumo de recursos que impactam diretamente o desempenho e a capacidade do sistema. O entendimento desses pilares é essencial para qualquer estratégia de otimização:

- **Tempo de CPU:** Este é o tempo que a Unidade Central de Processamento (CPU) gasta executando as operações associadas à consulta. Isso inclui cálculos, iterações sobre conjuntos de dados, ordenações, agregações e o processamento de junções. O tempo de CPU é diretamente influenciado pela complexidade do plano de execução escolhido pelo otimizador do banco de dados e pela eficiência dos algoritmos utilizados para processar os dados.1 Consultas com muitos

    `JOIN`s, `GROUP BY`s ou `ORDER BY`s tendem a consumir mais CPU.

- **Uso de Memória:** Refere-se ao volume de dados que o banco de dados precisa manter na memória RAM para processar a consulta. Consultas que exigem grandes ordenações (por exemplo, para cláusulas `ORDER BY` ou `GROUP BY`), agregações complexas ou cálculos intermediários que não cabem no cache podem consumir quantidades significativas de memória.1 O uso eficiente de caches, como o

    _buffer pool_, é vital para reduzir a necessidade de I/O de disco, mas consultas mal otimizadas podem sobrecarregar a memória disponível, levando a _spills_ para o disco, o que aumenta o I/O.

- **I/O de Disco (Entrada/Saída):** Frequentemente, este é o fator de custo mais significativo para consultas em banco de dados. Ele está diretamente relacionado ao número de leituras e gravações realizadas no armazenamento secundário (disco).1 Consultas que acessam grandes volumes de dados, que não se beneficiam de índices ou que realizam varreduras completas de tabelas, resultam em alto I/O de disco. A latência do disco, combinada com o número de páginas lidas, pode aumentar drasticamente o tempo de execução da consulta.1

O papel do plano de execução de consultas é crucial. Esta ferramenta, obtida através de comandos como `EXPLAIN` (no MySQL) ou `EXPLAIN ANALYZE` (no PostgreSQL), é a "receita" que o otimizador do banco de dados escolhe para executar a consulta, fornecendo métricas detalhadas sobre como os recursos (CPU, memória, I/O) serão consumidos.1 Ele revela se os índices estão sendo utilizados, se varreduras de tabela completas estão ocorrendo, ou se operações custosas como ordenações em disco são necessárias. O custo total da consulta é uma soma ponderada desses fatores, com I/O de disco geralmente desempenhando o papel mais significativo.1 Isso implica que, ao otimizar, a prioridade frequentemente recai sobre a redução de I/O, mesmo que isso possa implicar em um ligeiro aumento de CPU ou memória para operações como

_hashing_ ou ordenação em memória. A meta é sempre o menor custo total, o que geralmente se alinha com a minimização de acessos ao disco.

### 2.2. Análise Detalhada da Consulta SQL Proposta

A consulta SQL em questão apresenta várias características que podem levar a um alto custo computacional, especialmente em um cenário de crescimento contínuo da tabela principal.

#### Impacto das 5 mil linhas novas/dia e Crescimento da Tabela

Um crescimento diário de 5 mil novas linhas em uma tabela principal é um volume considerável. Ao longo do tempo, isso transformará uma tabela de tamanho gerenciável em uma tabela grande, potencialmente massiva. Tabelas grandes exigem estratégias de indexação robustas e manutenção contínua para evitar a degradação de desempenho.3 O aumento do volume de dados impacta diretamente o tempo de varredura de tabela (se índices não forem usados) e a eficiência dos índices existentes devido à fragmentação lógica e física.2

O crescimento contínuo sem manutenção adequada, como reorganização ou reconstrução de índices, levará a uma fragmentação crescente dos índices e dos dados físicos no disco.2 Essa fragmentação faz com que o banco de dados precise ler mais páginas de disco para encontrar os dados desejados, aumentando o I/O e, consequentemente, o tempo de CPU para processar esses dados. A fragmentação é uma "dívida técnica" que se acumula e degrada a performance de forma insidiosa, tornando as operações de busca mais lentas e mais custosas em termos de I/O.1

#### Custo de `SELECT` em 6 campos e `INNER JOIN` com tabelas pequenas de configuração

Um `SELECT` em 6 campos é, por si só, uma operação de projeção relativamente leve. No entanto, seu custo é amplificado exponencialmente se a operação subjacente for uma varredura de tabela completa. Os `INNER JOIN`s com tabelas pequenas de configuração podem ser eficientes se as colunas de junção estiverem adequadamente indexadas em ambas as tabelas (principal e configuração).6 O otimizador de consultas pode então utilizar

`Index Seek` ou algoritmos de junção mais eficientes como `Hash Join` ou `Merge Join`.8

O custo potencial surge se a tabela principal não tiver índices adequados nas colunas de junção. Nesse caso, o otimizador pode ser forçado a realizar varreduras completas da tabela principal ou a construir grandes tabelas hash/ordenar dados em memória ou disco para realizar a junção.8 Isso resultaria em um aumento significativo no consumo de CPU e memória, mesmo que as tabelas de configuração sejam pequenas. O "calcanhar de Aquiles" reside na falta de índices nas colunas de junção da

_tabela principal_. Isso pode forçar o otimizador a escanear a tabela grande repetidamente ou a construir estruturas de dados temporárias custosas, aumentando significativamente o consumo de CPU e memória, transformando uma operação aparentemente simples em um gargalo.6

#### Custo da Cláusula `WHERE` (busca por `NULL`, flag string de um caracter e ID de parâmetro)

A condição `WHERE buscando um NULL` (`IS NULL`) é particularmente problemática para a maioria dos índices B-TREE (o tipo mais comum de índice em SGBDs relacionais), pois eles geralmente não armazenam entradas para chaves totalmente `NULL`.9 Isso significa que, para encontrar registros com valores

`NULL`, a consulta pode ser forçada a realizar uma varredura de tabela completa (`Full Table Scan`).9 Esta é uma operação intensiva em I/O e CPU, especialmente em tabelas grandes.1

Para a `flag string de um caracter e um ID de parâmetro`, se estas colunas estiverem adequadamente indexadas, a busca será eficiente, utilizando `Index Seek` para localizar rapidamente os registros.10 Se não houver índices ou se os índices forem inadequados, a consulta pode resultar em varredura de tabela ou de índice completo, aumentando o custo. A combinação de

`IS NULL` com outras condições no `WHERE` pode inviabilizar o uso de índices compostos que incluam a coluna `NULL`, ou forçar o otimizador a escolher um plano menos eficiente que envolva varreduras. A sugestão de usar valores padrão em vez de `NULL` 9 é uma prática recomendada para otimização de índices e melhor performance de consultas, pois valores não-NULL são mais eficientemente indexados.

#### Custo da Subquery `NOT IN` que tem como parâmetro um `SELECT COALESCE` em outra tabela

Subqueries, especialmente quando usadas com `NOT IN`, são conhecidas por serem extremamente custosas. Se a subquery for _correlacionada_ (ou seja, ela referencia colunas da consulta externa e, portanto, é executada uma vez para _cada linha_ processada pela consulta externa), o custo computacional dispara exponencialmente.11 A função

`COALESCE` em si é relativamente barata e não gera linhas.14 No entanto, sua presença dentro da subquery não mitiga o custo principal da execução repetitiva da subquery. O problema reside na "multiplicação" das operações de CPU e I/O conforme a subquery é reavaliada para cada linha da consulta principal.9

A presença de `NOT IN` com uma subquery, especialmente se for correlacionada, é um dos maiores indicadores de problemas de performance em SQL. O custo não é apenas o da subquery em si, mas a _multiplicação_ desse custo pelo número de linhas na tabela externa, levando a um consumo exponencial de CPU e I/O.12 Isso pode transformar uma consulta em um "monstro" de performance. A recomendação é sempre evitar subqueries correlacionadas em

`NOT IN`. Alternativas como `LEFT JOIN` com `IS NULL` na coluna da tabela juntada, ou `NOT EXISTS` com uma subquery, são geralmente muito mais otimizáveis e permitem um uso mais eficiente de índices.13

#### Custo da Função `TIMESTAMPDIFF(MINUTE)` na Cláusula `WHERE`

A função `TIMESTAMPDIFF` é uma ferramenta poderosa e eficiente para calcular diferenças de tempo entre duas expressões de data/hora.15 O grande problema surge quando

`TIMESTAMPDIFF` (ou qualquer outra função) é aplicada a uma coluna na cláusula `WHERE` (e.g., `TIMESTAMPDIFF(MINUTE, coluna_data, NOW()) > outro_campo`). Isso impede que o otimizador de consultas utilize qualquer índice existente na `coluna_data`.16

Sem a capacidade de usar um índice, o banco de dados é forçado a realizar uma varredura completa da tabela para cada execução da consulta, avaliando a função para cada linha.16 Esta é uma operação altamente ineficiente em tabelas grandes, resultando em alto I/O e CPU.1 O problema não é a complexidade computacional da função

`TIMESTAMPDIFF` em si, que é baixa. O problema é que ela torna a condição _non-sargable_. Isso significa que o banco de dados não consegue usar um índice para "buscar" diretamente as linhas que satisfazem a condição, tendo que processar cada registro. Isso aumenta drasticamente o custo de I/O e CPU, transformando uma busca indexada em uma varredura completa.

A Tabela 1 resume os fatores de custo computacional da consulta SQL:

**Tabela 1: Fatores de Custo Computacional da Consulta SQL**

|Componente da Consulta|Impacto Potencial na CPU|Impacto Potencial na Memória|Impacto Potencial no I/O de Disco|Observações/Justificativa|
|---|---|---|---|---|
|Crescimento de 5k linhas/dia|Médio a Alto|Médio|Alto|Aumenta o volume de dados a serem processados; causa fragmentação de índices, degradando leituras.2|
|`SELECT` em 6 campos|Baixo|Baixo|Baixo (se indexado)|Custo baixo se a seleção for otimizada por índices; alto se resultar em varredura de tabela completa.1|
|`INNER JOIN`s (tabelas pequenas)|Médio a Alto|Médio|Médio a Alto|Eficiente com índices nas colunas de junção; custoso se forçar varreduras ou _hash joins_ grandes na tabela principal.6|
|`WHERE IS NULL`|Médio a Alto|Baixo|Alto|Impede o uso eficiente de índices B-TREE, forçando varreduras de tabela completa.9|
|`WHERE flag string` e `ID`|Baixo a Médio|Baixo|Baixo (se indexado)|Eficiente com índices; alto se forçar varredura de tabela.10|
|`NOT IN` Subquery com `COALESCE`|Alto|Médio a Alto|Alto|Subquery correlacionada executa para cada linha da consulta externa, multiplicando o custo de CPU e I/O.12|
|`TIMESTAMPDIFF` em `WHERE`|Médio a Alto|Baixo|Alto|Torna a condição _non-sargable_, impedindo o uso de índices e forçando varredura de tabela completa.16|

### 2.3. Fatores Chave de Otimização e Recomendações para o SQL

Para mitigar os altos custos computacionais identificados na consulta SQL, diversas estratégias de otimização podem ser aplicadas.

#### Estratégias de Indexação Adequada

A indexação é a ferramenta mais poderosa para otimização de consultas. É absolutamente fundamental criar índices nas colunas usadas nas condições `ON` dos `INNER JOIN`s em ambas as tabelas (principal e de configuração).6 Isso permite que o otimizador encontre e combine registros de forma muito mais eficiente, reduzindo drasticamente o tempo de execução. Para as colunas

`flag string` e `id de parâmetro` na cláusula `WHERE`, a criação de índices permitirá que o banco de dados realize `Index Seek`s rápidos em vez de varreduras completas.10

Para a condição `IS NULL`, considere alterar o esquema para usar um valor padrão (e.g., `0`, uma string vazia, ou uma data mínima) em vez de `NULL` se a lógica de negócio permitir. Valores não-NULL podem ser indexados de forma mais eficiente.9 Se

`NULL` for inevitável, um índice parcial/filtrado (se suportado pelo SGBD) pode ser uma alternativa, mas a performance ainda pode ser subótima.

Para otimizar a condição `TIMESTAMPDIFF`, a melhor abordagem é transformá-la em uma condição "sargable", ou seja, uma condição que o otimizador pode usar para buscar diretamente no índice. Em vez de `TIMESTAMPDIFF(MINUTE, coluna_data, NOW()) > outro_campo`, reescreva para algo como `coluna_data < DATE_SUB(NOW(), INTERVAL outro_campo MINUTE)`. Isso permite que um índice na `coluna_data` seja utilizado eficientemente.16 É importante lembrar que a indexação não é uma bala de prata, e o excesso de índices pode prejudicar a performance de escrita e consumir espaço em disco. O foco deve ser em indexar seletivamente as colunas mais frequentemente usadas em

`WHERE` clauses, `JOIN`s e `ORDER BY` para obter o maior benefício com o menor custo de manutenção.10

#### Análise e Otimização do Plano de Execução (`EXPLAIN`)

Sempre utilize `EXPLAIN` (ou `EXPLAIN ANALYZE` para ver o tempo real de execução) para entender como o otimizador está processando a consulta.1 Ele é a "verdade" sobre o custo da sua consulta e revelará se os índices estão sendo usados, se há varreduras de tabela completas, ordenações dispendiosas em disco ou

`Hash Joins` grandes que consomem muitos recursos. Uma consulta que "parece" simples ou que foi "otimizada" no código pode, na realidade, ser um problema de performance no plano de execução devido à falta de índices, estatísticas desatualizadas ou anti-padrões de consulta. A leitura deste plano permite identificar os gargalos mais críticos.

#### Gerenciamento da Fragmentação de Índices

Com 5 mil novas linhas por dia, a tabela principal e seus índices sofrerão fragmentação ao longo do tempo. Monitore ativamente os níveis de fragmentação e agende operações regulares de reorganização ou reconstrução de índices para manter a eficiência de acesso aos dados.2 A manutenção de índices é um processo contínuo e essencial para bancos de dados com alta taxa de inserção. Ignorar a fragmentação é permitir que a performance se degrade lentamente ao longo do tempo, como uma "dívida técnica" de performance que se acumula e afeta a experiência do usuário de forma gradual, mas perceptível.

#### Alternativas para `IS NULL` e `NOT IN` com Subquery

Se a lógica de negócio permitir, evite `NULL` para colunas que serão frequentemente filtradas. Em vez disso, use valores padrão que possam ser indexados de forma eficiente.9 Para

`NOT IN` com Subquery, prefira `LEFT JOIN` com `IS NULL` na coluna da tabela juntada (que seria `NULL` se não houvesse correspondência) ou `NOT EXISTS` com uma subquery. Essas abordagens são geralmente mais eficientes e permitem melhor otimização pelo SGBD, evitando a execução repetitiva da subquery.13 Reescrever consultas para evitar anti-padrões é uma habilidade crucial; muitas vezes, a mesma lógica de negócio pode ser expressa de maneiras que são ordens de magnitude mais eficientes para o otimizador do banco de dados, transformando uma consulta lenta em uma rápida.

#### Considerações sobre Particionamento de Tabelas

Para tabelas que crescem muito rapidamente e atingem volumes massivos, o particionamento pode ser uma estratégia avançada para melhorar a eficiência de consultas e a manutenção. Ele divide a tabela lógica em segmentos físicos menores e mais gerenciáveis, o que pode reduzir o volume de dados que o otimizador precisa escanear para uma dada consulta.20 O particionamento é uma solução de escalabilidade que vai além da indexação. Ele pode reduzir o volume de dados que o otimizador precisa considerar para uma dada consulta, melhorando o desempenho de I/O e CPU em cenários de dados massivos, e facilitando operações de manutenção e backup.6

A Tabela 2 apresenta as estratégias de otimização SQL e seus benefícios:

**Tabela 2: Estratégias de Otimização SQL e seus Benefícios**

|Estratégia de Otimização|Benefício Primário|Impacto no Custo Computacional|Observações|
|---|---|---|---|
|Indexação de Colunas de Junção|Redução de I/O, Redução de CPU, Melhor Tempo de Resposta|Alto|Essencial para `JOIN`s eficientes.6|
|Indexação de Colunas `WHERE`|Redução de I/O, Redução de CPU, Melhor Tempo de Resposta|Alto|Crucial para filtros rápidos.10|
|Reescrever `TIMESTAMPDIFF` para Sargable|Redução de I/O, Redução de CPU, Melhor Tempo de Resposta|Alto|Permite uso de índice na coluna de data; requer reescrita da query.16|
|Evitar `NOT IN` Correlacionado|Redução de CPU, Redução de I/O, Melhor Tempo de Resposta|Alto|Substituir por `LEFT JOIN` + `IS NULL` ou `NOT EXISTS`.13|
|Manutenção de Índices|Melhor Tempo de Resposta, Redução de I/O|Médio|Reduz fragmentação, mantém a eficiência do índice.2|
|Particionamento de Tabelas|Melhor Escalabilidade, Redução de I/O|Alto|Para tabelas muito grandes; facilita manutenção e otimiza consultas em subconjuntos de dados.6|
|Evitar `NULL` em colunas filtradas|Melhor Tempo de Resposta, Redução de I/O|Médio|Permite melhor indexação; requer alteração de esquema.9|

## 3. Custo Computacional no Cenário Node.js

### 3.1. Entendimento do Event Loop do Node.js e Operações Assíncronas

O Node.js opera de forma fundamentalmente diferente de um SGBD tradicional. Ele é construído em torno de um modelo de I/O não-bloqueante e uma arquitetura _single-threaded_ para o JavaScript principal.21 Essa arquitetura é a chave para sua capacidade de lidar com alta concorrência.

O Event Loop é o coração do Node.js, uma "máquina de estado" que gerencia operações assíncronas. Ele opera em um loop contínuo, verificando a fila de tarefas (callbacks) e as executando quando a _call stack_ principal (o thread JavaScript) está vazia.21 Este mecanismo é o que permite ao Node.js performar operações de I/O não-bloqueantes. O Event Loop possui fases distintas para processar diferentes tipos de callbacks (e.g., Timers, Pending Callbacks, Poll, Check, Close Callbacks).21 Funções como

`setTimeout` e `setInterval` são processadas especificamente na fase de "Timers".21

Quando uma operação assíncrona (como uma chamada de I/O, uma requisição de rede ou o agendamento de um timer) é iniciada, o Node.js a "descarrega" para o kernel do sistema operacional (ou para o pool de threads do libuv), liberando imediatamente o thread principal JavaScript para processar outras tarefas.21 O callback associado a essa operação é então enfileirado para ser executado

_apenas_ quando a operação assíncrona é concluída e o Event Loop está livre. A principal vantagem do Node.js é sua capacidade de lidar com alta concorrência de I/O sem a sobrecarga de múltiplos threads. No entanto, essa vantagem se torna uma desvantagem crítica se o código JavaScript _dentro_ dos callbacks for CPU-intensivo e síncrono, pois isso bloqueará o único thread principal, impedindo que o Event Loop processe outras tarefas e levando à degradação da responsividade da aplicação.

### 3.2. Análise do Cenário de 5 mil `setTimeout` por Dia

O cenário de 5 mil `setTimeout` por dia no backend Node.js apresenta considerações específicas de custo computacional.

#### Overhead de Agendamento de Timers (CPU e Memória)

Cada chamada a `setTimeout` cria um objeto timer que é gerenciado internamente pelo Node.js (através da biblioteca `libuv`) e, em última instância, pelo sistema operacional. Este objeto consome uma pequena quantidade de memória.25 O agendamento em si, ou seja, a chamada para

`setTimeout`, é uma operação leve e assíncrona. O Node.js simplesmente "entrega" a tarefa de agendamento ao sistema operacional e continua processando o código JavaScript subsequente no thread principal.21

Considerando 5 mil `setTimeout` por dia, distribuídos ao longo das 24 horas, isso significa uma taxa de criação de timers de aproximadamente 3-4 timers por minuto. Esta é uma carga de agendamento relativamente baixa e gerenciável em termos de CPU para a operação de _criação_ do timer. O custo computacional de `setTimeout` não está primariamente no _agendamento_ do timer, que é uma operação de I/O _offloaded_ para o kernel e, portanto, não bloqueia o thread JavaScript. O custo real e o risco de performance residem na _execução da função de callback_ quando o timer expira. Se essa função for CPU-bound e síncrona, ela bloqueará o Event Loop, causando lentidão.

#### Comportamento Não-Bloqueante do `setTimeout` e o que isso realmente significa

O comportamento não-bloqueante de `setTimeout` significa que a chamada da função retorna imediatamente, permitindo que o Node.js continue a executar o código JavaScript subsequente sem interrupção.21 O callback associado ao timer só será executado

_após_ o tempo especificado _e quando o Event Loop estiver livre_ para processá-lo.21

É importante notar que o tempo especificado no `setTimeout` (e.g., 60 minutos no cenário) é um _limite mínimo_ ou um _limiar_.22 A execução real do callback pode ser atrasada se houver outras tarefas na fila do Event Loop que precisam ser processadas primeiro, ou devido ao agendamento do sistema operacional.22 A confusão comum é que "não-bloqueante" significa que a operação inteira é mágica e não consome recursos. Na verdade, o "não-bloqueante" se refere à chamada inicial da função

`setTimeout`. No entanto, se o callback for uma operação síncrona e de longa duração (CPU-intensiva), ele _irá_ bloquear o Event Loop durante sua execução, causando latência e tornando a aplicação não responsiva para outras requisições.21

#### Potencial para "Event Loop Starvation" e "Memory Leaks"

O risco mais significativo no cenário Node.js é o "Event Loop Starvation". Se os callbacks dos 5 mil `setTimeout`s forem CPU-intensivos (realizando cálculos complexos, processamento de dados em larga escala) ou se um grande número de callbacks for agendado para executar em um curto período (por exemplo, muitos timers expirando simultaneamente), o Event Loop pode ficar sobrecarregado.21 Isso leva a um aumento na latência de resposta da aplicação e a torna "não responsiva" para outras requisições.29 O cenário de 5 mil

`setTimeout`s por dia, com limite de 60 minutos, sugere que muitos podem estar ativos simultaneamente aguardando sua execução. Se cada callback for pesado e síncrono, a probabilidade de "Event Loop Starvation" é alta, impactando a performance geral do backend Node.js.

Embora o objeto timer em si seja relativamente pequeno, se os callbacks de `setTimeout` criarem e mantiverem referências a grandes estruturas de dados que não são liberadas após a execução, ou se os timers não forem explicitamente cancelados quando não são mais necessários (via `clearTimeout`), isso pode levar a vazamentos de memória.25 Um acúmulo de 5000 objetos timer (e seus contextos de

_closure_) pode, com o tempo, impactar o uso de memória, levando a um aumento gradual no `heapUsed` e `rss`.25

#### Limitações do `setTimeout` (limite de 32 bits, persistência em memória)

A implementação de `setTimeout` em Node.js (e navegadores) tem um limite máximo de atraso, que é aproximadamente 24.8 dias (2^31 - 1 milissegundos), devido ao uso de um inteiro de 32 bits para armazenar o valor do tempo.30 Crucialmente, os timers agendados com

`setTimeout` são armazenados apenas na memória do processo Node.js. Se o servidor for reiniciado, o processo Node.js cair ou for encerrado, todos os timers agendados serão perdidos e não serão executados.30 Isso o torna inadequado para agendamento de tarefas críticas e de longo prazo que precisam de persistência ou garantia de execução. A natureza efêmera e o limite de tempo do

`setTimeout` indicam que ele não é uma solução de agendamento robusta para cenários de produção que exigem garantia de execução (mesmo após falhas do servidor) ou agendamentos muito distantes no futuro. Para tais necessidades, ferramentas de agendamento externas ou bibliotecas mais sofisticadas com mecanismos de persistência seriam recomendadas.30

## 4. Comparativo de Custo Computacional e Eficiência

### 4.1. Comparação Direta dos Cenários

Ao comparar os dois cenários, é fundamental entender a natureza fundamental das operações envolvidas. O cenário SQL envolve processamento de dados em disco, manipulação de conjuntos de dados potencialmente grandes e otimização de consultas por um SGBD. O cenário Node.js envolve agendamento e execução de funções JavaScript em um ambiente de Event Loop.

A consulta SQL, conforme descrita, possui diversos "anti-padrões" de performance que, se não otimizados, resultarão em alto consumo de CPU, memória e, principalmente, I/O de disco. A cada execução (a cada 3 minutos), ela tem o potencial de varrer tabelas inteiras, executar subqueries correlacionadas repetidamente e realizar cálculos em tempo de execução que impedem o uso de índices. O crescimento diário da tabela agrava esses problemas, pois o volume de dados a ser processado aumenta continuamente, levando a tempos de execução cada vez maiores e maior consumo de recursos.1

Por outro lado, o agendamento de 5 mil `setTimeout` no Node.js é, em sua essência, uma operação de agendamento assíncrona. O custo de _agendar_ um timer é baixo e não bloqueia o Event Loop principal.21 O custo computacional real no Node.js depende quase que inteiramente da

_natureza da função de callback_ que é executada quando o timer expira. Se essa função for leve e não-CPU-intensiva (e.g., uma simples chamada a uma API externa, uma operação de I/O que é descarregada), o impacto no Event Loop e nos recursos será mínimo. No entanto, se o callback for uma operação síncrona e pesada em CPU (e.g., processamento de dados complexo, criptografia), ele bloqueará o Event Loop, causando "Event Loop Starvation" e degradando a responsividade da aplicação.21

### 4.2. Qual tem o Maior Custo Computacional e Por Quê?

Considerando as descrições dos cenários e as características inerentes de cada tecnologia, o **cenário SQL tem o maior custo computacional**.

As razões para isso são multifacetadas:

- **I/O de Disco Intenso:** A consulta SQL, com a condição `WHERE IS NULL` e a função `TIMESTAMPDIFF` na cláusula `WHERE`, provavelmente forçará varreduras completas da tabela principal.9 Além disso, a subquery

    `NOT IN` com `COALESCE`, se correlacionada, executará para cada linha da consulta externa, resultando em leituras de disco repetitivas e ineficientes.12 O I/O de disco é frequentemente o fator de custo mais significativo em bancos de dados 1, e a consulta proposta parece maximizá-lo.

- **Consumo de CPU Exponencial:** A subquery correlacionada em `NOT IN` é um dos maiores consumidores de CPU em SQL, pois sua execução é multiplicada pelo número de linhas da consulta externa.12 Operações de

    `JOIN` sem índices adequados também podem levar a algoritmos de junção mais caros em termos de CPU, como `Hash Joins` que exigem construção de tabelas hash em memória ou disco.8

- **Sensibilidade ao Crescimento de Dados:** O custo da consulta SQL é diretamente proporcional ao tamanho da tabela. Com 5 mil novas linhas por dia, a degradação do desempenho será contínua e acentuada, exigindo cada vez mais recursos para processar o mesmo tipo de consulta.3

No cenário Node.js, embora 5 mil `setTimeout` por dia pareça um número grande, o custo por timer é baixo. O agendamento é assíncrono, e a maior parte do tempo o Event Loop estará ocioso ou processando outras tarefas.21 O risco de alto custo no Node.js só se materializa se os callbacks dos timers forem

_intrinsecamente_ CPU-intensivos e síncronos, o que levaria a um bloqueio do Event Loop.21 No entanto, mesmo nesse caso, a natureza do problema é diferente: é um gargalo de processamento único no thread principal, enquanto a consulta SQL pode estar esgotando múltiplos recursos (CPU, memória, I/O) em paralelo devido à complexidade das operações de banco de dados.

### 4.3. Qual tem a Maior Eficiência e Por Quê?

A **maior eficiência é do cenário Node.js**, _assumindo que as funções de callback dos `setTimeout` são operações leves ou que descarregam tarefas CPU-intensivas para Worker Threads_.

As razões para a maior eficiência do Node.js são:

- **Modelo de I/O Não-Bloqueante:** O Node.js é inerentemente eficiente para lidar com um grande número de operações assíncronas, como agendamento de timers. Ele não espera que uma operação de I/O (como a configuração de um timer pelo sistema operacional) seja concluída antes de prosseguir para a próxima tarefa.21 Isso permite que um único thread JavaScript gerencie milhares de operações concorrentes sem a sobrecarga de múltiplos threads de sistema operacional.

- **Recursos sob Demanda (para callbacks):** O custo computacional dos `setTimeout` é primariamente incorrido _apenas_ quando o callback é executado. Se esses callbacks forem leves, o consumo de CPU e memória será mínimo.21 Se forem CPU-intensivos, o Node.js oferece Worker Threads para descarregar essas tarefas para threads separados, evitando o bloqueio do Event Loop principal e mantendo a responsividade da aplicação.21

- **Escalabilidade Horizontal:** Embora o Node.js seja single-threaded para o JavaScript principal, a aplicação como um todo pode ser escalada horizontalmente (executando múltiplas instâncias em diferentes núcleos de CPU ou servidores) para lidar com maior carga.

Em contraste, a eficiência da consulta SQL é severamente comprometida pelos anti-padrões de consulta e pela falta de otimização. Mesmo que o banco de dados seja executado em hardware poderoso, uma consulta mal otimizada consumirá recursos desproporcionalmente, levando a tempos de resposta elevados e limitando a capacidade do sistema de lidar com outras requisições. A eficiência do banco de dados depende criticamente da otimização do plano de execução e do uso adequado de índices, algo que a consulta proposta não parece estar aproveitando ao máximo.1

Portanto, enquanto o Node.js pode ser eficiente para o agendamento e execução de tarefas assíncronas (especialmente se os callbacks forem otimizados), a consulta SQL, em sua forma atual, representa um dreno de recursos muito maior e menos eficiente, com um custo computacional que escalará de forma insustentável com o crescimento dos dados.

## 5. Conclusões e Recomendações Finais

### Síntese dos Principais Achados

A análise comparativa entre a consulta SQL complexa e o agendamento de `setTimeout` em Node.js revela que o **cenário SQL, em sua configuração atual, impõe o maior custo computacional e demonstra menor eficiência**. Este custo elevado é atribuível a múltiplos fatores, incluindo a provável ocorrência de varreduras completas de tabela devido a condições `WHERE` que impedem o uso de índices (`IS NULL` e `TIMESTAMPDIFF` aplicado à coluna indexada), e o impacto exponencial de uma subquery `NOT IN` potencialmente correlacionada. O crescimento diário de 5 mil linhas na tabela principal agrava esses problemas, resultando em maior I/O de disco e consumo de CPU, e uma degradação contínua da performance.

Em contrapartida, o cenário Node.js, com 5 mil `setTimeout` por dia, é inerentemente mais eficiente para operações de agendamento e I/O não-bloqueante. O custo de agendar os timers é baixo. O principal risco reside na natureza das funções de callback: se forem CPU-intensivas e síncronas, podem levar a "Event Loop Starvation", bloqueando o único thread JavaScript e impactando a responsividade da aplicação. No entanto, o Node.js oferece mecanismos como Worker Threads para mitigar esse risco, permitindo que tarefas pesadas sejam descarregadas para threads separados. Além disso, a natureza dos timers `setTimeout` os torna não persistentes, o que é uma limitação para tarefas críticas.

### Recomendações Holísticas para Otimização em Servidores Próprios

Com base nesta análise, as seguintes recomendações são cruciais para otimizar o custo computacional em ambientes de servidores próprios:

1. **Otimização Profunda da Consulta SQL:**

    - **Indexação Estratégica:** Crie índices nas colunas usadas em `JOIN`s e nas condições `WHERE` (`flag string`, `id de parâmetro`).

    - **Reescrita de Condições `WHERE`:** Transforme a condição `TIMESTAMPDIFF` em uma condição _sargable_ (e.g., `coluna_data < DATE_SUB(NOW(), INTERVAL outro_campo MINUTE)`).

    - **Alternativas para `NULL` e `NOT IN`:** Se possível, evite `NULL` em colunas frequentemente filtradas, usando valores padrão. Substitua `NOT IN` com subquery por `LEFT JOIN... IS NULL` ou `NOT EXISTS` para evitar subqueries correlacionadas e melhorar drasticamente o desempenho.

    - **Análise do Plano de Execução:** Utilize `EXPLAIN ANALYZE` (ou equivalente) para cada alteração na consulta. Este é o guia definitivo para entender como o SGBD está processando a consulta e para identificar os gargalos remanescentes.

    - **Manutenção de Índices:** Com o crescimento diário de 5 mil linhas, implemente uma rotina regular de monitoramento e manutenção de índices (reorganização/reconstrução) para combater a fragmentação e manter a eficiência das buscas.

    - **Considerar Particionamento:** Para a tabela principal, que cresce rapidamente, avalie a implementação de particionamento. Isso pode melhorar a performance de consultas ao reduzir o volume de dados a serem escaneados e facilitar a manutenção e o gerenciamento de dados históricos.

2. **Otimização do Backend Node.js:**

    - **Análise dos Callbacks `setTimeout`:** Monitore o tempo de execução e o consumo de CPU das funções de callback associadas aos `setTimeout`. Se forem CPU-intensivas, considere refatorá-las.

    - **Worker Threads para Tarefas CPU-Intensivas:** Para qualquer callback que execute cálculos complexos ou processamento de dados em larga escala, utilize Worker Threads. Isso descarregará a carga do Event Loop principal, mantendo a aplicação responsiva e evitando "Event Loop Starvation".

    - **Alternativas para Agendamento Persistente:** Dado que `setTimeout` não é persistente e tem um limite de tempo, para tarefas críticas que exigem garantia de execução ou agendamentos de longo prazo, utilize ferramentas de agendamento externas (e.g., Cron Jobs no sistema operacional, serviços de mensageria com _dead-letter queues_) ou bibliotecas Node.js de agendamento robustas que ofereçam persistência (e.g., Agenda.js, BullMQ).

    - **Monitoramento de Memória:** Monitore o uso de memória do processo Node.js (RSS, heapTotal, heapUsed) para identificar potenciais vazamentos de memória associados ao acúmulo de objetos timer ou contextos de closure.

Em resumo, a otimização de custo computacional em servidores próprios exige uma abordagem multifacetada. No cenário apresentado, a prioridade máxima deve ser dada à reengenharia e otimização da consulta SQL, pois ela representa o maior risco de degradação de performance e consumo de recursos. Concomitantemente, a gestão eficiente dos timers no Node.js, focando na natureza dos callbacks e na robustez do agendamento, garantirá que o backend permaneça responsivo e eficiente.

<deep-research-source-lists _ngcontent-ng-c3093917948="" _nghost-ng-c3489336990=""
    class="ng-tns-c3489336990-206 ng-tns-c3093917948-203 ng-star-inserted"><collapsible-button
        _ngcontent-ng-c3489336990="" expandtooltip="" collapsetooltip="" aria-controls="used-sources-list"
        data-test-id="used-sources-button" class="ng-tns-c3489336990-206 ng-star-inserted"><button mat-button=""
            class="mdc-button mat-mdc-button-base mat-mdc-tooltip-trigger button mat-mdc-button mat-unthemed"
            mat-ripple-loader-uninitialized="" mat-ripple-loader-class-name="mat-mdc-button-ripple"
            aria-expanded="true"><span class="mat-mdc-button-persistent-ripple mdc-button__ripple"></span><span
                class="mdc-button__label"><span _ngcontent-ng-c3489336990="" class="gds-title-m">Fontes usadas no
                    relatório</span></span><mat-icon role="img" iconpositionend="" data-test-id="icon"
                class="mat-icon notranslate google-symbols mat-ligature-font mat-icon-no-color" aria-hidden="true"
                data-mat-icon-type="font" data-mat-icon-name="expand_less" fonticon="expand_less"></mat-icon><span
                class="mat-focus-indicator"></span><span
                class="mat-mdc-button-touch-target"></span></button><!----></collapsible-button>
    <div _ngcontent-ng-c3489336990=""
        class="source-list used-sources ng-tns-c3489336990-206 ng-trigger ng-trigger-expandCollapse ng-star-inserted"
        aria-expanded="true" style="opacity: 1;"><browse-web-item _ngcontent-ng-c3489336990="" _nghost-ng-c910368191=""
            class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://pt.scribd.com/document/864584647/ADMINISTRACAO-EM-BANCO-DE-DADOS-3"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://pt.scribd.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                pt.scribd.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">ADMINISTRAÇÃO EM
                                BANCO DE DADOS 3 | PDF - Scribd</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://learn.microsoft.com/pt-br/sql/relational-databases/indexes/reorganize-and-rebuild-indexes?view=sql-server-ver17"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://learn.microsoft.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                learn.microsoft.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Manter índices
                                de forma ideal para melhorar o desempenho e reduzir a utilização de recursos - SQL
                                Server | Microsoft Learn</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.datacamp.com/pt/tutorial/sql-server-index"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://www.datacamp.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                datacamp.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Índice do SQL
                                Server: Aumentar o desempenho do banco de dados - DataCamp</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://stackoverflow.com/questions/2065754/is-there-any-general-rule-on-sql-query-complexity-vs-performance"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://stackoverflow.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                stackoverflow.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Is there any
                                general rule on SQL query complexity Vs performance? - Stack Overflow</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://stackoverflow.com/questions/8950978/in-node-js-does-settimeout-block-the-event-loop"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://stackoverflow.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                stackoverflow.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">In Node.js, does
                                setTimeout() block the event loop? - Stack Overflow</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://stackoverflow.com/questions/57387513/long-settimeout-on-nodejs-gets-ignored-by-event-queue"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://stackoverflow.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                stackoverflow.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Long setTimeout
                                on NodeJs gets ignored by event queue - Stack Overflow</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://awari.com.br/como-otimizar-o-plano-de-execucao-no-sql-server-dicas-essenciais-para-melhorar-o-desempenho/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://awari.com.br/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                awari.com.br</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Como Otimizar o
                                Plano de Execução no Sql Server: Dicas Essenciais para Melhorar o Desempenho - Awari
                            </div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.datacamp.com/pt/blog/sql-query-optimization"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://www.datacamp.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                datacamp.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Otimização de
                                consultas SQL: 15 técnicas para melhorar o desempenho - DataCamp</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://nodejs.org/en/learn/diagnostics/memory/understanding-and-tuning-memory"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://nodejs.org/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">nodejs.org
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Understanding
                                and Tuning Memory - Node.js</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://cloud.google.com/compute/docs/instances/sql-server/best-practices?hl=pt-br"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://cloud.google.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                cloud.google.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Práticas
                                recomendadas para instâncias do SQL Server | Compute Engine Documentation</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.reddit.com/r/node/comments/1b9j4jy/can_node_timers_cause_memory_leaks/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">reddit.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Can Node Timers
                                cause memory leaks? - Reddit</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.createse.com.br/post/a-cria%C3%A7%C3%A3o-de-%C3%ADndices-para-consultas-sql-em-ambientes-de-alta-disponibilidade"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.createse.com.br/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                createse.com.br</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">A Criação de
                                Índices para Consultas SQL em Ambientes de Alta Disponibilidade</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.reddit.com/r/SQL/comments/15g9ak7/coalesce_with_subquery_not_working/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">reddit.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Coalesce() with
                                subquery not working : r/SQL - Reddit</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.reddit.com/r/SQL/comments/8opvwr/how_do_subqueries_inside_of_coalesce_in_ms_sql/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">reddit.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">How do
                                subqueries inside of COALESCE in MS SQL work? Having a hard time processing.</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://dbplus.tech/en/2024/03/20/or-and-is-null-might-be-holding-you-back/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://dbplus.tech/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">dbplus.tech
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">'OR' and 'IS
                                NULL' Might Be Holding You Back - - DBPLUS Better Performance</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://nodejs.org/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">nodejs.org
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">The Node.js
                                Event Loop</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.geeksforgeeks.org/node-js/node-js-event-loop/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://www.geeksforgeeks.org/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                geeksforgeeks.org</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">NodeJS Event
                                Loop - GeeksforGeeks</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://ukpaiudoprecious0.medium.com/how-to-boost-node-js-performance-optimize-event-loop-and-async-operations-9e16ffb6d0b3"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://ukpaiudoprecious0.medium.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                ukpaiudoprecious0.medium.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">How to Boost
                                Node.js Performance: Optimize Event Loop and Async Operations.</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.createse.com.br/post/como-otimizar-joins-em-consultas-sql-para-evitar-gargalos-de-performance"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.createse.com.br/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                createse.com.br</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Como Otimizar
                                Joins em Consultas SQL para Evitar Gargalos de Performance - CreateSe</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://last9.io/blog/understanding-worker-threads-in-node-js/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://last9.io/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">last9.io
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Node.js Worker
                                Threads Explained (Without the Headache) - Last9</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://appmaster.io/pt/blog/recuperacao-complexa-de-dados-com-juncoes-de-banco-de-dados"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://appmaster.io/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                appmaster.io</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Dominando a
                                recuperação de dados complexos com junções de banco de dados</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.w3resource.com/mysql/date-and-time-functions/mysql-timestampdiff-function.php"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.w3resource.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                w3resource.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">MySQL
                                TIMESTAMPDIFF() function - w3resource</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://blog.devart.com/mysql-timestampdiff.html"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://blog.devart.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                blog.devart.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">MySQL
                                TIMESTAMPDIFF() Function: Syntax, Examples &amp; Use Cases - Devart Blog</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.datacamp.com/doc/mysql/mysql-monitoring-index-usage"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://www.datacamp.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                datacamp.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">MySQL Monitoring
                                Index Usage Indexes - DataCamp</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://medium.com/@prathik.codes/correlated-subqueries-in-sql-some-not-too-simple-examples-a4ce231a54df"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://medium.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">medium.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Correlated
                                Subqueries in SQL: Some Not-Too-Simple Examples | by Prathik C - Medium</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0" href="https://sqlenlight.com/support/help/sa0128/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://sqlenlight.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                sqlenlight.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">SA0128 : Avoid
                                using correlated subqueries. Consider using JOIN instead - SQL Enlight</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://developer.mozilla.org/en-US/docs/Web/API/Window/setTimeout"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://developer.mozilla.org/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                developer.mozilla.org</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Window:
                                setTimeout() method - Web APIs | MDN</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://nodejs.org/en/learn/asynchronous-work/discover-javascript-timers"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t1.gstatic.com/faviconV2?url=https://nodejs.org/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">nodejs.org
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Discover
                                JavaScript Timers - Node.js</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://2coffee.dev/en/articles/understanding-the-event-loop-in-nodejs"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://2coffee.dev/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">2coffee.dev
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Understanding
                                the Event Loop in node.js</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://medium.com/draftkings-engineering/event-loop-starvation-in-nodejs-a19901e26b41"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://medium.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">medium.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Event Loop
                                Starvation in NodeJS. This article details the concept of… | by Connor Stevens |
                                DraftKings Engineering | May, 2025 | Medium</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.reddit.com/r/dataengineering/comments/1axd7cy/what_are_your_top_sql_query_optimization_tips/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t2.gstatic.com/faviconV2?url=https://www.reddit.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">reddit.com
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">What are your
                                Top SQL Query Optimization tips? : r/dataengineering - Reddit</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://five.co/blog/sql-multiple-where-clauses/"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://five.co/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">five.co
                            </div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">SQL Multiple
                                WHERE Clauses: How to Guide - Five</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><!----></div>
    <!----><collapsible-button _ngcontent-ng-c3489336990="" expandtooltip="" collapsetooltip=""
        aria-controls="unused-sources-list" data-test-id="unused-sources-button"
        class="ng-tns-c3489336990-206 ng-star-inserted"><button mat-button=""
            class="mdc-button mat-mdc-button-base mat-mdc-tooltip-trigger button mat-mdc-button mat-unthemed"
            mat-ripple-loader-class-name="mat-mdc-button-ripple" aria-expanded="true"><span
                class="mat-mdc-button-persistent-ripple mdc-button__ripple"></span><span class="mdc-button__label"><span
                    _ngcontent-ng-c3489336990="" class="gds-title-m">Fontes lidas, mas não usadas no
                    relatório</span></span><mat-icon role="img" iconpositionend="" data-test-id="icon"
                class="mat-icon notranslate mat-icon-no-color google-symbols mat-ligature-font" aria-hidden="true"
                data-mat-icon-type="font" data-mat-icon-name="expand_less" fonticon="expand_less"></mat-icon><span
                class="mat-focus-indicator"></span><span class="mat-mdc-button-touch-target"></span><span
                class="mat-ripple mat-mdc-button-ripple"></span></button><!----></collapsible-button>
    <div _ngcontent-ng-c3489336990=""
        class="source-list unused-sources ng-tns-c3489336990-206 ng-trigger ng-trigger-expandCollapse ng-star-inserted"
        aria-expanded="true" style="opacity: 1;"><browse-web-item _ngcontent-ng-c3489336990="" _nghost-ng-c910368191=""
            class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://www.dreamhost.com/blog/pt/bun-vs-node/"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://www.dreamhost.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                dreamhost.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">Bun vs. Node:
                                Mais difícil, Melhor, Mais rápido, Mais forte? - DreamHost Blog</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://codificar.com.br/php-vs-nodejs-comparativo-das-linguagens-de-desenvolvimento-back-end/"><!----><span
                    _ngcontent-ng-c2322918752="" data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t0.gstatic.com/faviconV2?url=https://codificar.com.br/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                codificar.com.br</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">PHP vs NodeJs:
                                comparativo das linguagens de desenvolvimento back-end - Codificar</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><browse-web-item
            _ngcontent-ng-c3489336990="" _nghost-ng-c910368191="" class="ng-tns-c3489336990-206 ng-star-inserted"
            jslog="243335;track:generic_click;BardVeMetadataKey:[[&quot;r_76a54a081d1bd303&quot;,&quot;c_16ec9b389ebb91e0&quot;,null,&quot;c_16ec9b389ebb91e0_d985fdaf-3ebb-4e5b-807f-7feed3235c0e&quot;,null,null,null,null,1,null,null,1,0]]"
            style=""><a _ngcontent-ng-c910368191="" target="_blank" rel="noopener" externallink=""
                _nghost-ng-c2322918752="" tabindex="0"
                href="https://hightouch.com/sql-dictionary/sql-is-null"><!----><span _ngcontent-ng-c2322918752=""
                    data-test-id="content" class="link-content ng-star-inserted">
                    <div _ngcontent-ng-c910368191="" matripple="" class="mat-ripple browse-item gds-body-m">
                        <div _ngcontent-ng-c910368191="" class="title-container"><img _ngcontent-ng-c910368191=""
                                role="presentation" class="favicon"
                                src="https://t3.gstatic.com/faviconV2?url=https://hightouch.com/&amp;client=BARD&amp;type=FAVICON&amp;size=256&amp;fallback_opts=TYPE,SIZE,URL">
                            <div _ngcontent-ng-c910368191="" data-test-id="domain-name" class="display-name">
                                hightouch.com</div>
                            <div _ngcontent-ng-c910368191="" data-test-id="sub-title" class="sub-title">SQL IS NULL -
                                Syntax, Use Cases, and Examples - Hightouch</div>
                        </div>
                    </div>
                </span><span _ngcontent-ng-c2322918752="" class="cdk-visually-hidden ng-star-inserted">Abre em uma nova
                    janela</span><!----><!----><!----></a></browse-web-item><!----><!----><!----></div><!---->
</deep-research-source-lists>