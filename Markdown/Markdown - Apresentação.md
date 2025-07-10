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

---
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
````

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