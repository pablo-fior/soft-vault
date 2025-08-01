---
state: "[[Pronto]]"
---
# Roteiro de Apresentação: Linguagem de Marcação Markdown

_Duração: 40 minutos | Apresentação para colegas_

## 1. Introdução (5 minutos)

### Abertura

- "Quantos de vocês já escreveram documentação e ficaram perdidos com formatação HTML?"
- "Hoje vou mostrar como o Markdown pode simplificar isso drasticamente"

### O que é Markdown?

- Linguagem de marcação leve criada por John Gruber em 2004
- Objetivo: facilitar a escrita e leitura de texto formatado
- Converte texto simples em HTML válido
- Filosofia: "fácil de escrever, fácil de ler"

### Por que usar Markdown?

- Simplicidade e legibilidade
- Compatibilidade universal
- Controle de versão amigável
- Foco no conteúdo, não na formatação

---

## 2. Sintaxe Básica - Demonstração Prática (15 minutos)

### Cabeçalhos (2 min)

```markdown
# Título Principal (H1)
## Subtítulo (H2)
### Seção (H3)
#### Subseção (H4)
```

### Formatação de Texto (3 min)

```markdown
**Texto em negrito**
*Texto em itálico*
***Texto em negrito e itálico***
~~Texto riscado~~
`Código inline`
```

### Listas (3 min)

```markdown
# Lista não ordenada
- Item 1
- Item 2
  - Subitem 2.1
  - Subitem 2.2

# Lista ordenada
1. Primeiro item
2. Segundo item
3. Terceiro item
```

### Links e Imagens (3 min)

```markdown
[Texto do link](https://exemplo.com)
[Link com título](https://exemplo.com "Título do link")

![Texto alternativo](caminho/para/imagem.jpg)
![Imagem com título](imagem.jpg "Título da imagem")
```

### Citações e Código (4 min)

````markdown
> Esta é uma citação
> Pode ter múltiplas linhas
> > Citação aninhada

```python
def exemplo():
    print("Bloco de código")
    return "com syntax highlighting"
```
````


---

## 3. Recursos Avançados (10 minutos)

### Tabelas (3 min)
```markdown
| Coluna 1 | Coluna 2 | Coluna 3 |
|----------|----------|----------|
| Dados 1  | Dados 2  | Dados 3  |
| A        | B        | C        |

# Alinhamento
| Esquerda | Centro | Direita |
|:---------|:------:|--------:|
| Texto    | Texto  | Texto   |
```

| Coluna 1 | Coluna 2 | Coluna 3 |
|----------|----------|----------|
| Dados 1  | Dados 2  | Dados 3  |
| A        | B        | C        |


| Esquerda | Centro | Direita |
|:---------|:------:|--------:|
| Texto    | Texto  | Texto   |

### Linha Horizontal (1 min)

```markdown
---
***
___
```

### Escape de Caracteres (2 min)

```markdown
\*Asterisco literal\*
\# Hashtag literal
\[Colchetes literais\]
```

### Links de Referência (2 min)

```markdown
[Texto do link][1]
[Outro link][referencia]

[1]: https://exemplo.com
[referencia]: https://site.com "Título opcional"
```

### Listas de Tarefas (2 min)

```markdown
- [x] Tarefa concluída
- [ ] Tarefa pendente
- [ ] Outra tarefa
```

---

## 4. Ferramentas e Aplicações Práticas (7 minutos)

### Editores Recomendados (2 min)

- **Typora**: Editor visual em tempo real
- **Mark Text**: Interface limpa e intuitiva
- **Obsidian**: Para anotações e conhecimento
- **VS Code**: Com extensões Markdown
- **Notion**: Suporte nativo ao Markdown

### Onde Usar Markdown (3 min)

- **GitHub**: README, issues, pull requests
- **Documentação**: GitBook, MkDocs
- **Blogs**: Jekyll, Hugo, Ghost
- **Notas**: Obsidian, Notion, Joplin
- **Emails**: Alguns clientes suportam
- **Apresentações**: Marp, reveal.js

### Demonstração Rápida (2 min)

- Mostrar conversão em tempo real
- Exemplo de README no GitHub
- Visualizar HTML gerado

---

## 5. Dicas Práticas e Melhores Práticas (5 minutos)

### Organização de Documentos

- Use hierarquia clara de cabeçalhos
- Mantenha linhas em branco entre seções
- Seja consistente com formatação

### Dicas de Produtividade

- Aprenda os atalhos do seu editor
- Use templates para documentos recorrentes
- Automatize a conversão com scripts

### Armadilhas Comuns

- Cuidado com espaços extras
- Nem todos os sabores de Markdown são iguais
- Teste a renderização em diferentes plataformas

### Exemplo Prático

```markdown
# Projeto XYZ

## Descrição
Este projeto tem como objetivo...

## Instalação
1. Clone o repositório
2. Execute `npm install`
3. Configure as variáveis de ambiente

## Uso
Para usar o sistema:
- Faça login com suas credenciais
- Acesse o painel principal
- Selecione a opção desejada

## Contribuição
Contribuições são bem-vindas! Veja nosso [guia](CONTRIBUTING.md).
```

---

## 6. Exercício Prático (5 minutos)

### Desafio para os Colegas

"Vamos criar juntos um documento Markdown básico!"

**Tarefa**: Criar um README para um projeto fictício contendo:

- Título do projeto
- Descrição em 2-3 linhas
- Lista de funcionalidades
- Instruções de instalação
- Seção de contato com link

### Exemplo de Solução

```markdown
# Sistema de Gerenciamento de Tarefas

Um aplicativo simples para organizar suas tarefas diárias.
Desenvolvido com foco na simplicidade e produtividade.

## Funcionalidades
- [x] Criar tarefas
- [x] Marcar como concluída
- [ ] Definir prioridades
- [ ] Lembretes por email

## Instalação
1. Baixe o arquivo ZIP
2. Extraia em uma pasta
3. Execute o arquivo `install.bat`

## Contato
Para dúvidas, entre em contato: [email@exemplo.com](mailto:email@exemplo.com)
```

---

## 7. Conclusão e Perguntas (3 minutos)

### Resumo dos Benefícios

- Simplicidade na escrita
- Legibilidade do código fonte
- Compatibilidade universal
- Facilita colaboração e versionamento

### Próximos Passos

- Experimentem com seus projetos atuais
- Migrem documentações existentes
- Explorem editores especializados
- Considerem para apresentações e blogs

### Recursos Adicionais

- [Guia oficial do Markdown](https://daringfireball.net/projects/markdown/)
- [CommonMark Spec](https://commonmark.org/)
- [Markdown Cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

### Perguntas e Discussão

- Alguma dúvida sobre a sintaxe?
- Querem ver algum exemplo específico?
- Como pretendem usar em seus projetos?

---

## Dicas para o Apresentador

### Material Necessário

- Projetor/tela para demonstrações
- Editor de Markdown instalado
- Exemplos práticos preparados
- Acesso à internet para mostrar GitHub/sites

### Interação com a Audiência

- Faça perguntas durante a apresentação
- Incentive participação no exercício prático
- Relacione com projetos/problemas conhecidos da equipe

### Timing Flexível

- Se houver muitas perguntas, reduza o exercício prático
- Prepare exemplos extras caso sobre tempo
- Mantenha ritmo dinâmico nas demonstrações

### Encerramento

- Ofereça-se para ajudar individualmente
- Sugira formar um grupo de estudo
- Compartilhe recursos e links úteis