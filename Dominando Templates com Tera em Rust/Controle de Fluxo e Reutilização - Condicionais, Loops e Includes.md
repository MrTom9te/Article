Controle de Fluxo e Reutilização - Condicionais, Loops e Includes

Até agora, você aprendeu a exibir informações e manipulá-las com filtros. Mas, e se você quiser que partes do seu template apareçam apenas sob certas condições? Ou, e se você precisar repetir um bloco de código para cada item de uma lista? É aí que entram as estruturas de controle de fluxo: condicionais e loops. Além disso, vamos explorar como reutilizar partes do seu template com *includes*.

Condicionais (If)

A estrutura condicional `if` permite que você execute um bloco de código somente se uma condição for verdadeira. A sintaxe básica é:

```html
{% if condicao %}
    <!-- Código a ser executado se a condição for verdadeira -->
{% endif %}
```

Você pode adicionar cláusulas `elif` (else if) para testar múltiplas condições, e uma cláusula `else` para executar um código caso nenhuma das condições anteriores seja verdadeira:

```html
{% if idade >= 18 %}
    <p>Você é maior de idade.</p>
{% elif idade >= 13 %}
    <p>Você é um adolescente.</p>
{% else %}
    <p>Você é uma criança.</p>
{% endif %}
```

Importante: *Variáveis indefinidas são consideradas falsas*. Isso significa que você pode testar se uma variável existe simplesmente colocando-a em um `if`:

```html
{% if usuario %}
    <p>Bem-vindo, {{ usuario.nome }}!</p>
{% else %}
    <p>Faça login para continuar.</p>
{% endif %}
```

Se a variável `usuario` não estiver definida no contexto, o bloco `else` será executado.

Loops (For)

O loop `for` permite que você itere sobre os itens de um array (ou os caracteres de uma string) e execute um bloco de código para cada item.

Iterando sobre um array:

```html
<ul>
{% for produto in produtos %}
    <li>{{ produto.nome }} - R$ {{ produto.preco }}</li>
{% endfor %}
</ul>
```

Se `produtos` for um array de objetos com os campos `nome` e `preco`, este código gerará uma lista HTML com os nomes e preços de cada produto.

Variáveis de Loop

Dentro de um loop `for`, você tem acesso a algumas variáveis especiais:

*   `loop.index`: O índice do item atual na iteração (começando em 1).
*   `loop.index0`: O índice do item atual na iteração (começando em 0).
*   `loop.first`: `true` se for a primeira iteração, `false` caso contrário.
*   `loop.last`: `true` se for a última iteração, `false` caso contrário.

Exemplo:

```html
<ul>
{% for produto in produtos %}
    <li {% if loop.first %} class="primeiro" {% endif %}>
        {{ loop.index }}. {{ produto.nome }}
    </li>
{% endfor %}
</ul>
```

Este código adiciona a classe CSS "primeiro" ao primeiro item da lista.

Iterando sobre Mapas e Structs

Você também pode iterar sobre as chaves e valores de mapas (como `HashMap` em Rust) e structs:

```html
{% for chave, valor in usuario %}
    <p>{{ chave }}: {{ valor }}</p>
{% endfor %}
```

`chave` e `valor` são nomes de variáveis que você escolhe. Eles receberão, respectivamente, a chave e o valor de cada par no mapa/struct.

Filtros em Containers de Loop

Você pode aplicar filtros diretamente ao array (ou outro container) que está sendo iterado:

```html
{% for produto in produtos | reverse %}
    <li>{{ produto.nome }}</li>
{% endfor %}
```

Isso exibirá os produtos em ordem reversa.

Iterando sobre literais de array

```html
{% for item in [1,2,3,] %}
    <p>{{item}}</p>
{% endfor %}
```

Cláusula `else` para Containers Vazios

Você pode adicionar uma cláusula `else` ao loop `for` para executar um bloco de código se o container estiver vazio:

```html
{% for produto in produtos %}
    <li>{{ produto.nome }}</li>
{% else %}
    <p>Nenhum produto encontrado.</p>
{% endfor %}
```

Controles de loop: `break` e `continue`

Dentro de um loop, você pode usar `break` para interromper a iteração e `continue` para pular para a próxima iteração.

```html
{% for item in items %}
    {% if item.condicao == true %} {% break %} {% endif %}
        <p> {{item.nome}}</p>
{% endfor %}
```
Para o loop quando a condição for verdadeira

```html
{% for item in items %}
    {% if loop.index is even %} {% continue %} {% endif %}
    {{loop.index}}. {{item.nome}}
{% endfor %}
```
Pula os itens pares.

Includes

A diretiva `include` permite que você inclua o conteúdo de outro template dentro do template atual. Isso é útil para reutilizar partes comuns do seu layout, como cabeçalhos e rodapés.

```html
{% include "cabecalho.html" %}

<main>
    <!-- Conteúdo principal da página -->
</main>

{% include "rodape.html" %}
```

Importante: O caminho para o template incluído deve ser uma *string estática*. Você não pode usar variáveis ou expressões para construir o caminho.

Contexto e Escopo em Includes

O template incluído tem acesso ao *mesmo contexto* que o template principal. No entanto, variáveis definidas com `set` dentro do template incluído *não* estarão disponíveis no template principal.  Elas ficam restritas ao escopo do include.

`ignore missing`

Você pode fazer com que o tera ignore o `include`, caso o template não exista
```
{% include "header.html" ignore missing %}
```
Incluindo uma lista de templates

Você pode fornecer uma lista de templates para incluir, o primeiro a ser encontrado vai ser utilizado
```
{% include ["custom/header.html", "header.html"] %}
{% include ["special_sidebar.html", "sidebar.html"] ignore missing %}
```
*Observação:* A versão atual do Tera não permite que você use a herança de modelos (explicada em um artigo futuro) dentro dos arquivos incluídos. Portanto, organize seus arquivos de forma consistente.

Conclusão

Com condicionais, loops e includes, você agora tem as ferramentas para criar templates muito mais flexíveis e organizados. Você pode controlar quais partes do seu template são exibidas, repetir blocos de código e reutilizar componentes. No próximo artigo, vamos explorar *macros* e *funções*, que levam a reutilização de código a um novo nível!