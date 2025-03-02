# Explorando a Sintaxe Tera - Variáveis, Expressões e Estruturas de Dados

No artigo anterior, você deu os primeiros passos com o Tera, criando um template simples e entendendo os conceitos básicos. Agora, vamos aprofundar nosso conhecimento na sintaxe do Tera, explorando como usar variáveis, expressões e diferentes tipos de dados para criar templates dinâmicos e poderosos.

## Variáveis e Dados

### Tipos de Dados Literais

O Tera suporta vários tipos de dados que você pode usar diretamente nos seus templates:

*   **Booleanos:**  `true` (ou `True`) e `false` (ou `False`).  Representam valores verdadeiros ou falsos.

    ```html
    {% if usuario.logado %}
        <p>Bem-vindo de volta!</p>
    {% else %}
        <p>Faça login para continuar.</p>
    {% endif %}
    ```

*   **Inteiros:** Números inteiros (sem casas decimais).

    ```html
    <p>Você tem {{ quantidade }} itens no carrinho.</p>
    ```

*   **Floats:** Números com casas decimais.

    ```html
    <p>Preço total: R$ {{ preco_total }}</p>
    ```

*   **Strings:**  Textos delimitados por aspas duplas (`""`), aspas simples (`''`) ou crases (`````).

    ```html
    <h1>{{ titulo }}</h1>
    <p>{{ 'Olá, mundo!' }}</p>
    <p>{{ `Tera é incrível!` }}</p>
    ```

*   **Arrays:**  Listas ordenadas de valores, separados por vírgula e envoltos em colchetes (`[]`).  Os valores podem ser de diferentes tipos. A vírgula após o último elemento é opcional.

    ```html
    <ul>
    {% for item in ['maçã', 'banana', 'laranja'] %}
        <li>{{ item }}</li>
    {% endfor %}
    </ul>
    ```

### Renderizando Variáveis

Para exibir o valor de uma variável no seu template, use os delimitadores `{{ }}`:

```html
<p>Olá, {{ nome }}!</p>
```

Se a variável `nome` estiver definida no seu contexto com o valor "Ana", o resultado será:

```html
<p>Olá, Ana!</p>
```

**Importante:** Se você tentar usar uma variável que não foi definida no contexto, o Tera gerará um erro.

### Acessando Atributos

Se a sua variável for um objeto (como uma struct em Rust), você pode acessar os atributos (campos) desse objeto usando a notação de ponto (`.`) ou colchetes (`[]`).

**Notação de Ponto (`.`):**

```html
<p>Nome: {{ usuario.nome }}</p>
<p>Idade: {{ usuario.idade }}</p>
```

Se `usuario` for uma struct com os campos `nome` e `idade`, o Tera acessará os valores desses campos.

**Notação de Colchetes (`[]`):**

A notação de colchetes é mais flexível.  Você pode usar strings dentro dos colchetes:

```html
<p>Nome: {{ usuario["nome"] }}</p>
```

Isso é equivalente a `usuario.nome`.  A vantagem dos colchetes é que você pode usar *variáveis* como índices:

```html
{% set campo = "nome" %}
<p>Nome: {{ usuario[campo] }}</p>
```

Isso é muito útil quando você precisa acessar campos dinamicamente.

**Acessando elementos de arrays e tuplas:**

Para acessar elementos específicos de um array ou tupla, use a notação de ponto seguida do índice (começando em 0):

```html
{% set frutas = ['maçã', 'banana', 'laranja'] %}
<p>Primeira fruta: {{ frutas.0 }}</p>
<p>Segunda fruta: {{ frutas.1 }}</p>
```

### Variável Mágica: `__tera_context`

O Tera fornece uma variável especial chamada `__tera_context` que contém *todo* o contexto atual.  Isso pode ser útil para depuração:

```html
<pre>{{ __tera_context }}</pre>
```

Isso exibirá todo o contexto como JSON, permitindo que você veja todas as variáveis disponíveis.

## Expressões

Expressões são combinações de variáveis, literais, operadores e funções que produzem um valor.  O Tera permite usar expressões em quase todos os lugares onde você pode usar variáveis.

### Operações Matemáticas

Você pode realizar operações matemáticas básicas:

*   `+`: Adição
*   `-`: Subtração
*   `*`: Multiplicação
*   `/`: Divisão
*   `%`: Módulo (resto da divisão)

```html
<p>2 + 2 = {{ 2 + 2 }}</p>
<p>10 / 3 = {{ 10 / 3 }}</p>
<p>Resto de 10 / 3 = {{ 10 % 3 }}</p>
```

**Importante:** As operações matemáticas só funcionam com números.  Tentar usá-las com outros tipos de dados (como strings) gerará um erro.

A prioridade das operações é a padrão:

1.  `*`, `/`, `%` (maior prioridade)
2.  `+`, `-` (menor prioridade)

Use parênteses para controlar a ordem das operações:

```html
<p>{{ (2 + 3) * 4 }}</p>  <!-- Resultado: 20 -->
```

### Comparações

Você pode comparar valores usando os seguintes operadores:

*   `==`: Igual a
*   `!=`: Diferente de
*   `>`: Maior que
*   `<`: Menor que
*   `>=`: Maior ou igual a
*   `<=`: Menor ou igual a

```html
{% if idade >= 18 %}
    <p>Você é maior de idade.</p>
{% endif %}
```

### Lógica

Combine condições usando operadores lógicos:

*   `and`: E (ambas as condições devem ser verdadeiras)
*   `or`: OU (pelo menos uma condição deve ser verdadeira)
*   `not`: NÃO (inverte o valor da condição)

```html
{% if usuario.logado and usuario.admin %}
    <p>Você é um administrador logado.</p>
{% endif %}

{% if not usuario.logado %}
    <p>Você não está logado.</p>
{% endif %}
```

### Concatenação de Strings (`~`)

Use o operador `~` para concatenar (juntar) strings:

```html
{% set saudacao = "Olá" %}
{% set nome = "Mundo" %}
<p>{{ saudacao ~ ", " ~ nome ~ "!" }}</p>
```

Resultado:

```html
<p>Olá, Mundo!</p>
```

O operador `~` também funciona com números, convertendo-os para strings. Se uma variável não for uma string ou um número, ocorrerá um erro.

### Operador `in`

O operador `in` verifica se um valor está contido em outro:

*   **Em arrays:** Verifica se o valor é um elemento do array.
*   **Em strings:** Verifica se o valor é uma substring da string.
*   **Em objetos:** verifica se o valor é uma das chaves

```html
{% set numeros = [1, 2, 3, 4, 5] %}

{% if 3 in numeros %}
    <p>3 está no array.</p>
{% endif %}

{% if "Olá" in "Olá, Mundo!" %}
    <p>"Olá" é uma substring.</p>
{% endif %}
```

Você também pode usar `not in` para verificar a ausência:

```html
{% if 6 not in numeros %}
    <p>6 não está no array.</p>
{% endif %}
```

## Manipulação de Dados
### Assignments
`set` atribui um valor para uma variável, `set_global` define um valor que vai estar no escopo global

```
{% set my_var = "hello" %}
{% set my_var = 1 + 4 %}
{% set my_var = some_var %}
{% set my_var = macros::some_macro() %}
{% set my_var = global_fn() %}
{% set my_var = [1, true, some_var | round] %}
```
`set_global`
```
{% set_global my_var = "hello" %}
{% set_global my_var = 1 + 4 %}
{% set_global my_var = some_var %}
{% set_global my_var = macros::some_macro() %}
{% set_global my_var = global_fn() %}
{% set_global my_var = [1, true, some_var | round] %}
```

### Filtros

Filtros permitem que você modifique variáveis de diversas maneiras.  Você aplica um filtro usando o operador `|` (pipe).

```html
<p>{{ nome | upper }}</p>
```

Se `nome` for "ana", o resultado será "ANA".  O filtro `upper` converte a string para maiúsculas.

**Encadeamento de Filtros:**

Você pode aplicar vários filtros em sequência:

```html
<p>{{ nome | lower | replace(from="ana", to="Ana Paula") }}</p>
```

Isso primeiro converte `nome` para minúsculas (`lower`) e depois substitui "ana" por "Ana Paula" (`replace`).

**Filtros com Argumentos:**

Alguns filtros aceitam argumentos.  Os argumentos são passados entre parênteses, usando a sintaxe de *keyword arguments* (nome do argumento seguido de `=` e o valor):

```html
{{ texto | replace(from="gato", to="cachorro") }}
```

**Exemplos de Filtros Comuns:**

*   `lower`: Converte para minúsculas.
*   `upper`: Converte para maiúsculas.
*   `capitalize`: Primeira letra em maiúscula, restante em minúsculas.
*   `replace(from, to)`: Substitui uma substring por outra.
*   `trim`: Remove espaços em branco no início e no final da string.
*  `length`: Retorna comprimento do array,objeto ou string

**Filter sections**
Podemos utilizar filtros em blocos
```
{% filter upper %}
    Hello
{% endfilter %}
```
Isso transforma o texto `Hello` em `HELLO`

### Testes

Testes são usados em blocos `if` para verificar condições.  Você usa a palavra-chave `is` seguida do nome do teste.

```html
{% if numero is odd %}
    <p>O número é ímpar.</p>
{% endif %}
```

O teste `odd` verifica se um número é ímpar.

**Negação de Testes:**

Use `is not` para negar um teste:

```html
{% if numero is not even %}
    <p>O número não é par.</p>
{% endif %}
```

**Exemplos de Testes Comuns:**

*   `odd`: Verifica se um número é ímpar.
*   `even`: Verifica se um número é par.
*   `defined`: Verifica se uma variável está definida no contexto.
*   `undefined`: Verifica se uma variável *não* está definida.
*   `string`: Verifica se uma variável é uma string.
*  `number`: Verifica se é um numero
*  `iterable`: Verifica se e possivel iterar por esse objeto
* `object`: Verifica se a variável é um object

## Conclusão

Neste artigo, você explorou a sintaxe do Tera em profundidade, aprendendo sobre:

*   Tipos de dados literais.
*   Como renderizar variáveis e acessar seus atributos.
*   Uso da variável `__tera_context`.
*   Expressões: operações matemáticas, comparações, lógica e concatenação.
*   O operador `in`.
*  Assignments: `set`, `set_global`
*   Filtros e testes: como modificar e verificar variáveis.

Com esse conhecimento, você já pode criar templates bem mais dinâmicos e interessantes. No próximo artigo, vamos explorar estruturas de controle de fluxo, como condicionais (`if`) e loops (`for`), para tornar seus templates ainda mais poderosos!

Este artigo cobre:

*   **Tipos de dados literais:** Booleanos, inteiros, floats, strings, arrays.  Exemplos claros de cada um.
*   **Renderização de variáveis:**  `{{ }}`, notação de ponto, colchetes, acesso a elementos de arrays/tuplas.
*   **`__tera_context`:**  Explicação e exemplo de uso para depuração.
*   **Expressões:**  Explicação detalhada de cada tipo de expressão, com exemplos.
*   **Operador `in`:**  Explicação e exemplos para arrays, strings.
*   **Filtros:** Explicação geral, encadeamento, argumentos, exemplos de filtros comuns.
*   **Testes:** Explicação, negação de testes, exemplos de testes comuns.
* **Conclusão:**  Resumo do que foi aprendido e indicação do próximo passo.