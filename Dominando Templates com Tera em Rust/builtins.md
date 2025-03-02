 # Built-ins do Tera: Filtros, Testes e Funções Integrados

O Tera vem com um conjunto de *filtros*, *testes* e *funções* pré-definidos que você pode usar diretamente nos seus templates, sem precisar escrever código Rust adicional. Esses *built-ins* (recursos integrados) cobrem uma ampla gama de tarefas comuns, tornando seus templates mais poderosos e concisos.

Filtros (Built-in Filters)

Filtros são usados para modificar variáveis. Você já viu alguns exemplos, como `upper` e `lower`. Aqui está uma lista mais completa, com explicações e exemplos de uso:

*   **`lower`**: Converte uma string para minúsculas.

    ```html
    {{ "Olá Mundo" | lower }}  <!-- Resultado: olá mundo -->
    ```

*   **`upper`**: Converte uma string para maiúsculas.

    ```html
    {{ "Olá Mundo" | upper }}  <!-- Resultado: OLÁ MUNDO -->
    ```

*   **`wordcount`**: Conta o número de palavras em uma string.

    ```html
    {{ "Esta frase tem cinco palavras" | wordcount }}  <!-- Resultado: 5 -->
    ```

*   **`capitalize`**: Converte a primeira letra da string para maiúscula e as demais para minúsculas.

    ```html
    {{ "olá mundo" | capitalize }}  <!-- Resultado: Olá mundo -->
    ```

*   **`replace(from, to)`**: Substitui todas as ocorrências de uma substring por outra. *Requer* os argumentos nomeados `from` e `to`.

    ```html
    {{ "Eu gosto de gatos" | replace(from="gatos", to="cachorros") }}
    <!-- Resultado: Eu gosto de cachorros -->
    ```

*   **`addslashes`**: Adiciona barras invertidas antes de aspas (simples e duplas). Útil para escapar strings que serão usadas em JavaScript, por exemplo.

    ```html
    {{ "I'm using Tera" | addslashes }}  <!-- Resultado: I\'m using Tera -->
    ```

*   **`slugify`**: (Requer a feature `builtins`) Converte uma string em um "slug" (formato amigável para URLs): remove caracteres especiais, converte para minúsculas e substitui espaços por hifens.

    ```html
    {{ "Meu Artigo Incrível!" | slugify }}  <!-- Resultado: meu-artigo-incrivel -->
    ```

*   **`title`**: Converte a primeira letra de cada palavra para maiúscula.

    ```html
    {{ "um título de exemplo" | title }} <!-- Resultado: Um Título De Exemplo -->
    ```

*    **`trim`**:Remove espaços do começo e final da string

     ```html
     {{ "  um título de exemplo    " | trim }} <!-- Resultado: um título de exemplo -->
     ```
*   **`trim_start`**: Remove espaços do começo da string

    ```html
    {{ "  um título de exemplo    " | trim_start }} <!-- Resultado: um título de exemplo    -->
    ```
*   **`trim_end`**: Remove espaços do final da string
     ```html
      {{ "  um título de exemplo    " | trim_end }} <!-- Resultado:   um título de exemplo -->
     ```
* **`trim_start_matches(pat)`**: Remove caracteres que derem match no padrão
    ```html
     {{ "//a/b/c//" | trim_start_matches(pat="//") }} <!-- Resultado: a/b/c// -->
    ```
*   **`trim_end_matches(pat)`**: Remove caracteres que derem match no padrão do final da string
    ```html
    {{ "//a/b/c//" | trim_end_matches(pat="//") }} <!-- Resultado: //a/b/c -->
    ```

*   **`truncate(length, end?)`**: (Requer a feature `builtins`) Trunca uma string para um determinado comprimento. O argumento `length` é obrigatório. O argumento opcional `end` especifica o que adicionar ao final se a string for truncada (o padrão é "...").

    ```html
    {{ "Esta é uma frase longa" | truncate(length=10) }}
    <!-- Resultado: Esta é u... -->

    {{ "Esta é uma frase longa" | truncate(length=10, end="") }}
    <!-- Resultado: Esta é u -->
    ```

*   **`linebreaksbr`**: Converte quebras de linha (`\n` ou `\r\n`) em tags `<br>` HTML.

    ```html
    {{ "Olá\nMundo" | linebreaksbr }}  <!-- Resultado: Olá<br>Mundo -->
    ```
    *Observação*: Se autoescaping estiver habilitado, use `| safe` após `linebreaksbr`.

*   **`spaceless`**: Remove espaços em branco *entre* tags HTML.

    ```html
    {{ " <p> Olá </p>  <p>Mundo</p> " | spaceless }}
    <!-- Resultado: <p>Olá</p><p>Mundo</p> -->
    ```
    *Observação*: Se autoescaping estiver habilitado, use `| safe` após `spaceless`.

*  **`indent(prefix?)`**: Adiciona espaços em branco no começo de cada linha
    ```html
    {{ "ola\nmundo" | indent }} <!-- Resultado: '    ola\n    mundo' -->
    ```
    Também temos os atributos opcionais
    `first`: para indentar a primeira linha também, por default e `false`
    ```html
    {{ "ola\nmundo" | indent(first=true) }} <!-- Resultado: '    ola\n    mundo' -->
    ```
    `blank`: indentar linhas em branco, por default e `false`

*   **`striptags`**: Remove *tags* HTML de uma string. *Atenção*: Não use isso para segurança (sanitização de HTML)! É apenas para remover formatação básica.

    ```html
    {{ "<h1>Olá</h1><p>Mundo</p>" | striptags }}  <!-- Resultado: OláMundo -->
    ```
     *Observação*: Se autoescaping estiver habilitado, use `| safe` após `striptags`.

*   **`first`**: Retorna o primeiro elemento de um array. Retorna uma string vazia se o array estiver vazio.

    ```html
    {{ [1, 2, 3] | first }}  <!-- Resultado: 1 -->
    ```

*   **`last`**: Retorna o último elemento de um array. Retorna uma string vazia se o array estiver vazio.

    ```html
    {{ [1, 2, 3] | last }}  <!-- Resultado: 3 -->
    ```

*   **`nth(n)`**: Retorna o n-ésimo elemento de um array (índice baseado em 0). Requer o argumento nomeado `n`.

    ```html
    {{ ['a', 'b', 'c'] | nth(n=1) }} <!-- Resultado: 'b' -->
    ```

*   **`join(sep)`**: Junta os elementos de um array em uma string, usando um separador. Requer o argumento nomeado `sep`.

    ```html
    {{ [1, 2, 3] | join(sep=", ") }}  <!-- Resultado: 1, 2, 3 -->
    ```

*   **`length`**: Retorna o comprimento de um array, string ou objeto.

    ```html
    {{ "Olá" | length }}  <!-- Resultado: 3 -->
    {{ [1, 2, 3] | length }}  <!-- Resultado: 3 -->
    ```

*   **`reverse`**: Inverte um array ou string.

    ```html
    {{ "Olá" | reverse }}  <!-- Resultado: álO -->
    {{ [1, 2, 3] | reverse }}  <!-- Resultado: [3, 2, 1] -->
    ```

*   **`sort`**: Ordena um array em ordem crescente.

    ```html
    {{ [3, 1, 2] | sort }}  <!-- Resultado: [1, 2, 3] -->
    ```
    Também podemos ordenar uma lista de structs ou tuplas com o argumento `attribute`

*   **`unique`**:Remove itens duplicados
    ```html
    {{ [1,1,2,3,4,4,4] | unique }}  <!-- Resultado: [1,2,3,4] -->
    ```
    Também pode utilizar o argumento `attribute`

*   **`slice(start?, end?)`**: Extrai uma fatia de um array.  `start` (inclusivo, padrão 0) e `end` (exclusivo, padrão: comprimento do array) são opcionais. Índices negativos contam a partir do final.

    ```html
    {{ [1, 2, 3, 4, 5] | slice(start=1, end=3) }}  <!-- Resultado: [2, 3] -->
    {{ [1, 2, 3, 4, 5] | slice(end=-2) }}  <!-- Resultado: [1, 2, 3] -->
    ```

*    **`group_by(attribute)`**: Agrupa um array usando o atributo, discarta valores `null`

*   **`filter(attribute, value?)`**: Filtra um array, mantendo apenas os elementos onde o atributo especificado tem o valor dado. O argumento `attribute` é obrigatório. Se `value` for omitido, filtra elementos onde o atributo é `null`.

    ```html
    {{ produtos | filter(attribute="categoria", value="livros") }}
    ```
*   **`map(attribute)`**:Retorna um array que contem os atributos de cada item

*   **`concat(with)`**: Concatena um array com outro array ou com um único valor. Requer o argumento nomeado `with`.

    ```html
    {{ [1, 2] | concat(with=[3, 4]) }}  <!-- Resultado: [1, 2, 3, 4] -->
    ```

*   **`urlencode`**: (Requer a feature `builtins`) Codifica uma string para uso em URLs (percent-encoding).

    ```html
    {{ "https://meusite.com/pesquisa?q=Olá Mundo" | urlencode }}
    ```

*   **`urlencode_strict`**: (Requer a feature `builtins`) Similar a `urlencode`, mas codifica *todos* os caracteres não alfanuméricos (incluindo `/`).

*   **`abs`**: Retorna o valor absoluto de um número.

    ```html
    {{ -5 | abs }}  <!-- Resultado: 5 -->
    ```

*   **`pluralize(singular?, plural?)`**: Retorna um sufixo plural (padrão "s") se o valor não for 1 ou -1.  Você pode especificar sufixos diferentes com `singular` e `plural`.

    ```html
    Você tem {{ num_itens }} item{{ num_itens | pluralize }}.
    ```

*   **`round(method?, precision?)`**: Arredonda um número.  `method` pode ser "common" (padrão), "ceil" (para cima) ou "floor" (para baixo). `precision` (padrão 0) especifica o número de casas decimais.

    ```html
    {{ 3.14159 | round(precision=2) }}  <!-- Resultado: 3.14 -->
    ```

*   **`filesizeformat`**: (Requer a feature `builtins`) Formata um número de bytes como um tamanho de arquivo legível (ex: "1.2 MB").

    ```html
    {{ 1234567 | filesizeformat }}  <!-- Resultado: 1.2 MB (aproximadamente) -->
    ```

*   **`date(format?, timezone?, locale?)`**: (Requer a feature `builtins` e `date-locale` para o locale) Formata um timestamp (segundos desde a época UNIX) como uma data/hora.  O argumento `format` usa a sintaxe do `strftime` (veja a documentação do chrono). Se você tiver uma data/hora em string ISO 8601, pode passá-la diretamente. Você pode especificar um timezone.

    ```html
    {{ 1678886400 | date(format="%Y-%m-%d %H:%M:%S") }}
    {{ "2023-03-15T12:00:00Z" | date(format="%Y-%m-%d", timezone="America/Sao_Paulo") }}
    ```

*   **`escape`**: Escapa caracteres HTML especiais (`&`, `<`, `>`, `"`, `'`, `/`).

    ```html
    {{ "<script>alert('oi');</script>" | escape }}
    ```

*   **`escape_xml`**: Escapes XML special characters.
    ```html
    {{ "<script>alert('oi');</script>" | escape_xml }}
    ```

*   **`safe`**: Marca uma string como "segura", indicando que ela *não* deve ser escapada. *Use com extrema cautela!*  Só use `safe` se você tiver certeza absoluta de que o conteúdo da string é seguro.  `safe` deve ser o *último* filtro em uma cadeia.

    ```html
    {{ conteudo_html | safe }}
    ```

*    **`get(key, default?)`**: Acessa um valor de um objeto pela chave

*    **`split(pat)`**: Separa uma string pelo padrão
* 
    ```html
    {{ "ola,mundo,feliz" | split(pat=",") }} <!-- result: ["ola","mundo","feliz"] -->
    ```
*   **`int(default?, base?)`**: Converte um valor para inteiro. Os argumentos `default` e `base` são opcionais.

*   **`float(default?)`**: Converte um valor para ponto flutuante (f64). O argumento `default` é opcional.
*   **`json_encode(pretty?)`**: Converte um valor para uma representação JSON. O argumento opcional pretty (booleano) habilita a formatação pretty-printed (múltiplas linhas e indentação). Use com | safe se autoescaping estiver habilitado.

    ```html
    {{ dados | json_encode() | safe }}
    ```
*   **`as_str`**: Converte qualquer valor para sua representação como string.

*   **`default(value)`**:  Retorna o valor informado, se a variável não estiver definida no contexto.

    ```html
    {{ variavel_opcional | default(value="Valor Padrão") }}
    ```

Testes (Built-in Tests)

Testes são usados em blocos `if` para verificar condições.

*   **`defined`**: Verifica se uma variável está definida.

    ```html
    {% if usuario is defined %}...{% endif %}
    ```

*   **`undefined`**: Verifica se uma variável *não* está definida.

    ```html
    {% if usuario is undefined %}...{% endif %}
    ```

*   **`odd`**: Verifica se um número é ímpar.

    ```html
    {% if numero is odd %}...{% endif %}
    ```

*   **`even`**: Verifica se um número é par.

    ```html
    {% if numero is even %}...{% endif %}
    ```

*   **`string`**: Verifica se um valor é uma string.

    ```html
    {% if variavel is string %}...{% endif %}
    ```

*   **`number`**: Verifica se um valor é um número (inteiro ou ponto flutuante).

    ```html
    {% if variavel is number %}...{% endif %}
    ```

*   **`divisibleby(number)`**: Verifica se um número é divisível por outro.

    ```html
    {% if numero is divisibleby(3) %}...{% endif %}
    ```

*   **`iterable`**: Verifica se um valor é iterável (array, tupla ou objeto).

    ```html
    {% if variavel is iterable %}...{% endif %}
    ```
*   **`object`**: Verifica se um valor é um objeto

*   **`starting_with(prefix)`**: Verifica se uma string começa com um determinado prefixo.

    ```html
    {% if texto is starting_with("Olá") %}...{% endif %}
    ```

*   **`ending_with(suffix)`**: Verifica se uma string termina com um determinado sufixo.

    ```html
    {% if texto is ending_with("!") %}...{% endif %}
    ```

*   **`containing(value)`**: Verifica se uma string, array ou mapa *contém* um determinado valor.

    ```html
    {% if "mundo" in texto %}...{% endif %}
    {% if 5 in numeros %}...{% endif %}
    ```
*   **`matching(regex)`**: Verifica se uma string corresponde a uma expressão regular (regex). A sintaxe de regex é a do crate `regex` do Rust.

    ```html
     {% if "123-456-7890" is matching(r"^\d{3}-\d{3}-\d{4}$") %}...{% endif %}
    ```

Funções (Built-in Functions)

Funções integradas que você pode chamar diretamente nos seus templates.

*   **`range(end, start?, step_by?)`**: Gera um array de números. `end` (exclusivo) é obrigatório. `start` (padrão 0) e `step_by` (padrão 1) são opcionais.

    ```html
    {% for i in range(end=5) %}...{% endfor %}
    ```

*   **`now(timestamp?, utc?)`**: (Requer a feature `builtins`) Retorna a data e hora atuais.  `timestamp` (padrão `false`) retorna um timestamp UNIX (segundos desde a época). `utc` (padrão `false`) retorna a hora UTC em vez da hora local.

    ```html
    {{ now() }}
    {{ now(timestamp=true) }}
    ```

*   **`throw(message)`**: Lança um erro com a mensagem especificada. Útil para sinalizar erros em macros ou em situações específicas.

    ```html
    {% if condicao_invalida %}{{ throw(message="Erro: Condição inválida!") }}{% endif %}
    ```

*   **`get_random(start?, end)`**: (Requer a feature `builtins`) Retorna um número inteiro aleatório entre `start` (inclusivo, padrão 0) e `end` (exclusivo). `end` é obrigatório.

    ```html
    {{ get_random(end=100) }}
    ```

*   **`get_env(name, default?)`**: Obtém o valor de uma variável de ambiente. `name` é obrigatório.  `default` é um valor opcional a ser retornado se a variável de ambiente não estiver definida.

    ```html
    {{ get_env(name="DATABASE_URL") }}
    {{ get_env(name="PORT", default=8080) }}
    ```

Conclusão

Este artigo forneceu uma visão geral abrangente dos filtros, testes e funções integrados do Tera. Com esses recursos, você pode manipular dados, verificar condições e executar tarefas comuns diretamente nos seus templates, sem precisar escrever código Rust extra para muitas operações simples. Explore esses built-ins e use-os para tornar seus templates mais expressivos e eficientes!