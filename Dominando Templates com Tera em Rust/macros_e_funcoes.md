# Macros e Funções: Componentes Reutilizáveis em Tera

Até aqui, você já aprendeu a estruturar seus templates com condicionais, loops e includes. Mas, e se você pudesse criar seus próprios "blocos de construção" personalizados para tarefas repetitivas? É exatamente isso que *macros* e *funções* permitem fazer. Elas são como funções em linguagens de programação, mas dentro do seu template.

Macros

Pense em macros como "mini-templates" ou "componentes" que você pode definir e reutilizar em vários lugares. Uma macro recebe argumentos (opcionalmente) e retorna um texto (que pode ser HTML, ou qualquer outro formato).

Definindo uma Macro

Use a tag `{% macro %}` para definir uma macro.  Dentro da tag, você especifica o nome da macro e seus argumentos.  Use a tag `{% endmacro %}` para fechar a definição.

```html
{% macro input(label, tipo="text", valor="") %}
    <label>
        {{ label }}:
        <input type="{{ tipo }}" value="{{ valor }}">
    </label>
{% endmacro input %}
```

Neste exemplo:

*   `input` é o nome da macro.
*   `label`, `tipo` e `valor` são os argumentos.
*   `tipo` e `valor` têm valores padrão ("text" e "", respectivamente).  Isso significa que você pode chamar a macro sem especificar esses argumentos, e os valores padrão serão usados.

Chamando uma Macro

Para usar uma macro, use a seguinte sintaxe:

```html
{{ nome_da_macro(argumento1=valor1, argumento2=valor2) }}
```

Importante: Você *precisa* usar *keyword arguments* (nome do argumento seguido de `=` e o valor) ao chamar macros.  A ordem dos argumentos não importa.

Exemplo:

```html
{{ input(label="Nome") }}

{{ input(label="Senha", tipo="password") }}

{{ input(label="E-mail", tipo="email", valor="seu@email.com") }}
```

Importando Macros

Se você definir suas macros em um arquivo separado (o que é uma boa prática para organização), você precisa *importar* esse arquivo antes de usar as macros.

```html
{% import "macros.html" as meu_modulo %}
```

*   `macros.html` é o nome do arquivo que contém as macros.
*   `meu_modulo` é um nome que você escolhe para o "namespace" (espaço de nomes) das macros.  Você usará esse nome para chamar as macros.

Chamando macros importadas:

```html
{{ meu_modulo::input(label="Nome") }}
```

`self`: Chamando Macros no Mesmo Arquivo

Se você estiver chamando uma macro que está definida no *mesmo* arquivo, use o namespace especial `self`:

```html
{% macro mensagem(texto) %}
    <p class="mensagem">{{ texto }}</p>
{% endmacro mensagem %}

{{ self::mensagem(texto="Olá!") }}
```

Recursão em Macros

Macros podem chamar a si mesmas (recursão). Isso pode ser útil para gerar estruturas hierárquicas, como menus aninhados.

```html
{% macro menu(itens) %}
    <ul>
    {% for item in itens %}
        <li>
            {{ item.titulo }}
            {% if item.subitens %}
                {{ self::menu(itens=item.subitens) }}
            {% endif %}
        </li>
    {% endfor %}
    </ul>
{% endmacro menu %}
```

Importante: Tome cuidado com a recursão! Se a sua macro não tiver uma condição de parada clara, ela pode entrar em loop infinito e causar um erro.

Funções (Global Functions)

Funções, ou *global functions*, são funções escritas em Rust que você pode chamar diretamente dos seus templates. Elas permitem que você execute lógica mais complexa que não seria possível (ou seria muito difícil) fazer apenas com a sintaxe do Tera.

Funções em Rust

Uma função global do Tera é uma função Rust que:

1.  Recebe um `HashMap<String, Value>` como argumento. Este mapa contém os argumentos passados para a função a partir do template.
2.  Retorna um `Result<Value, Error>`.  `Value` é um tipo do Tera que representa um valor que pode ser usado no template (como um número, string, booleano, etc.). `Error` é o tipo de erro do Tera.

Exemplo (simples):

```rust
use tera::{Value, Result, Error};
use std::collections::HashMap;

// Função que retorna o dobro de um número.
fn dobrar(args: &HashMap<String, Value>) -> Result<Value> {
    // Tenta obter o argumento "numero".
    match args.get("numero") {
        Some(val) => {
            // Tenta converter o valor para um número (i64).
            match val.as_i64() {
                Some(num) => Ok(Value::from(num * 2)), // Retorna o dobro.
                None => Err(Error::msg("O argumento 'numero' deve ser um inteiro.")),
            }
        }
        None => Err(Error::msg("A função 'dobrar' requer o argumento 'numero'.")),
    }
}
```

Registrando Funções

Para usar uma função no seu template, você precisa *registrá-la* no seu motor Tera:

```rust
use tera::Tera;

// ... (código da função dobrar) ...

fn main() -> Result<(), tera::Error> {
    let mut tera = Tera::new("templates/**/*")?;
    tera.register_function("dobrar", dobrar); // Registra a função.
    Ok(())
}

```

Chamando Funções em Templates

Depois de registrar a função, você pode chamá-la no seu template:

```html
<p>O dobro de 5 é: {{ dobrar(numero=5) }}</p>
```

Funções com Captura de Variáveis Externas

Muitas vezes, você precisará que suas funções acessem variáveis externas (por exemplo, configurações, dados do banco de dados, etc.).  Você pode fazer isso usando *closures* (funções anônimas que capturam o ambiente).

Exemplo:

```rust
use tera::{Value, Result, Error, Function};
use std::collections::HashMap;

// Função que formata uma URL usando uma base de URL configurável.
fn formatar_url(base_url: String) -> impl Function {
    Box::new(move |args: &HashMap<String, Value>| -> Result<Value> {
        match args.get("caminho") {
            Some(val) => match val.as_str() {
                Some(caminho) => Ok(Value::from(format!("{}{}", base_url, caminho))),
                None => Err(Error::msg("O argumento 'caminho' deve ser uma string.")),
            }
            None => Err(Error::msg("A função 'formatar_url' requer o argumento 'caminho'.")),
        }
    })
}

// ...

fn main() -> Result<(), tera::Error>{
    let mut tera = Tera::new("templates/**/*")?;

    let base_url = "https://meusite.com".to_string();
    tera.register_function("url", formatar_url(base_url)); // A closure captura base_url.
     Ok(())
}
```

No template:

```html
<a href="{{ url(caminho="/contato") }}">Contato</a>
```

Built-in Functions

Tera já vem com diversas funções:

`range`:  Cria um array de inteiros.
`now`: Obtém a data e hora atuais.
`throw`: Gera um erro no template.
`get_random`: Obtém um número aleatório.
`get_env`: Obtém uma variável de ambiente.

Conclusão

Macros e funções expandem significativamente as capacidades dos seus templates. Com macros, você pode criar componentes reutilizáveis. Com funções, você pode integrar lógica Rust complexa.  Use essas ferramentas com sabedoria para manter seus templates limpos, organizados e poderosos!