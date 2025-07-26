# **1. O que é Markdown?**

É uma linguagem de marcação leve que permite formatar texto usando símbolos simples do teclado.

- **Onde é usado?**

  - Arquivos README (em projetos de programação)

  - Documentação de software

  - Blogs e sites (para escrever posts)

  - Anotações rápidas

  - Comunicação em plataformas como GitHub, GitLab, Slack, Discord, entre outras.

- **Títulos (Headers):** Use `#` para diferentes níveis de títulos.

    Markdown

    ```markdown
    # Título Nível 1
    ## Título Nível 2
    ### Título Nível 3
    #### Título Nível 4
    ##### Título Nível 5
    ###### Título Nível 6
    ```

    Resultado:

    ![Captura](imgs/Capturar.png)
    ![Captura](imgs/Capturar2.png)

---

- **Parágrafos:** Apenas digite o texto. Uma linha em branco separa parágrafos.
  
---

- **Ênfase (Negrito e Itálico):**

    Markdown
  
  - Negrito: `**texto em negrito**` ou `__texto em negrito__`

  - Itálico: `*texto em itálico*` ou `_texto em itálico_`

  - Negrito e Itálico: `***texto em negrito e itálico***`<br>

    - Negrito: **texto em negrito** ou **texto em negrito**

    - Itálico: *texto em itálico* ou *texto em itálico*

    - Negrito e Itálico: ***texto em negrito e itálico***
  
---

- **Listas:**

  - **Não Ordenadas (Bullet Points):** `Use`*`,`-` ou `+``.

    Markdown

    ```markdown
    * Item 1
    * Item 2
        * Subitem 2.1
        * Subitem 2.2
    ```

    ![img3](imgs/Capturar3.png)

  - **Ordenadas (Numeradas):** Use números seguidos de um ponto.

    Markdown

    ```markdown
    1. Primeiro item
    2. Segundo item
    3. Terceiro item
    ```

    ![img3](imgs/Capturar3.png)

---

- **Links:** `[Texto do Link](URL do Link)`

    Markdown

    ```markdown
    [Visite o Google](https://www.google.com)
    ```

    [Visite o Google](https://www.google.com)
    Markdown:

    ![img8](imgs/Capturar7.png)

    ![img5](imgs/Capturar5.png)

---

- **Imagens:** `![Texto Alternativo](URL da Imagem)`

    Markdown

    ```markdown
    ![Logo do Markdown](https://upload.wikimedia.org/wikipedia/commons/4/48/Markdown-mark.svg)
    ```

    ![Logo do Markdown](https://upload.wikimedia.org/wikipedia/commons/4/48/Markdown-mark.svg)

    *Obs: Para imagens locais, apenas o caminho do arquivo.*<br>

    Markdown:

    ![img5](imgs/Capturar8.png)

    ![img6](imgs/Capturar6.png)

---

- **Blocos de Código (Code Blocks):** Use três crases (``````) antes e depois do código para blocos inteiros, e uma crase (`` ` ``) para trechos inline.

    Markdown

    ```markdown
    `print("Olá, mundo!")` é um código inline.
    ```

    ```python
    def saudacao(nome):
        return f"Olá, {nome}!"
    print(saudacao("Colegas"))
    ```

    *Dica: Podem especificar a linguagem após as três crases para destaque de sintaxe.*

    Markdown:

    ![img6](imgs/Capturar9.png)

---

- **Citações (Blockquotes):** `Use ">"`.

    ```markdown
    > "A simplicidade é o último grau de sofisticação."
    > Leonardo da Vinci
    ```

    > "A simplicidade é o último grau de sofisticação."  
    > Leonardo da Vinci

---

- **Linha Horizontal (Thematic Breaks):** Use três ou mais `---`, `***` ou `___`.

    Markdown

    ```markdown
    ---
    ```

    ---

---

- **Tabela**

    ```markdown
    | Nome     | Idade | Cidade     |
    |----------|-------|------------|
    | Ana      | 23    | São Paulo  |
    | Bruno    | 31    | Recife     |
    ```

    | Nome     | Idade | Cidade     |
    |----------|-------|------------|
    | Ana      | 23    | São Paulo  |
    | Bruno    | 31    | Recife     |

---

- **Checkbox**

    ```markdown
    - [x] Preparar apresentação
    - [ ] Revisar exemplos
    - [ ] Compartilhar material
    ```

  - [x] Preparar apresentação
  - [ ] Revisar exemplos
  - [ ] Compartilhar material

---

- **Referências**

    ```markdown
        Markdown é muito útil para documentação.[^1]

        [^1]: Especialmente em projetos colaborativos.
    ```

    Markdown é muito útil para documentação.[^1]

    [^1]: Especialmente em projetos colaborativos.  

    ![img6](imgs/Capturar10.png)

---

- **Emoji**

    ```text
    Parabéns pelo trabalho! :tada: :rocket:
    ```

    Parabéns pelo trabalho! :tada: :rocket:

    ![img6](imgs/Capturar11.png)

---

## **4. Ferramentas e Recursos Úteis**

- **Editores de Texto:**

  - **VS Code:** Com extensões como "Markdown All in One" e "Markdown Preview Enhanced".

  - **Typora:** Editor focado na experiência de escrita Markdown.

  - **Obsidian:** Para gestão de notas e conhecimento, usa Markdown.

  - **Online:** Dillinger, StackEdit.

- **Plataformas que usam Markdown:** GitHub, GitLab, Stack Overflow, Reddit, Discord, Slack.

- **Cheatsheets:** Existem várias "colas" online para lembrar a sintaxe.

---

![md1](imgs/md1.jpg)

![md2](imgs/md2.png)

![md3](imgs/md3.jpg)
