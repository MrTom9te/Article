# Introdução ao Tera - Seu Motor de Templates para Rust

Se você está começando no mundo da programação Rust e quer criar aplicações web, interfaces de linha de comando (CLIs) mais sofisticadas, ou gerar qualquer tipo de texto de forma dinâmica, você veio ao lugar certo!  Neste artigo, vamos mergulhar no mundo dos *templates* e conhecer o Tera, um motor de templates poderoso e fácil de usar, feito sob medida para Rust.

## O que são Templates?

Imagine que você precisa enviar um e-mail de boas-vindas para cada novo usuário que se cadastra no seu site.  O texto do e-mail é quase sempre o mesmo, exceto pelo nome do usuário.  Você *poderia* escrever um código Rust que monta essa mensagem, concatenando strings:

```rust
let nome = "Ana"; // Imagine que isso vem do cadastro
let mensagem = format!("Olá, {}! Seja bem-vinda ao nosso site!", nome);
println!("{}", mensagem);
```

Isso funciona, mas imagine fazer isso para e-mails mais complexos, com HTML, formatação, links, etc.  Fica complicado e difícil de manter.  É aí que entram os *templates*.

Um template é como um *formulário* com "lacunas" a serem preenchidas.  Você define a estrutura geral do texto (e-mail, página HTML, arquivo de configuração) e deixa espaços reservados para os dados variáveis.  O motor de templates (no nosso caso, o Tera) combina o template com os dados, preenchendo as lacunas e gerando o resultado final.

**Analogia:** Pense em um template como uma receita de bolo. A receita descreve os ingredientes e o modo de preparo (a estrutura).  Os ingredientes específicos (os dados) você pode variar: chocolate, morango, baunilha.  O resultado final é sempre um bolo, mas com sabores diferentes.

**Benefícios dos Templates:**

*   **Separação de Preocupações:**  O template cuida da *apresentação* (como as coisas são exibidas), enquanto o código Rust cuida da *lógica* (obter os dados, fazer cálculos).  Isso torna seu código mais limpo, organizado e fácil de entender.
*   **Reutilização:**  Você pode usar o mesmo template para gerar diferentes saídas, apenas mudando os dados.  Por exemplo, um template de página de produto pode ser usado para exibir todos os produtos do seu site.
*   **Manutenção:**  Se você precisar mudar o layout do seu site, basta alterar o template, sem mexer no código Rust.
*   **Edição Facilitada:**  Designers podem trabalhar nos templates (HTML, CSS) sem precisar saber Rust.

## Por que usar o Tera?

Existem vários motores de templates disponíveis, mas o Tera se destaca por alguns motivos:

1.  **Integração com Rust:** O Tera foi criado especificamente para Rust.  Ele aproveita as vantagens da linguagem, como a segurança de tipos e a performance.  A integração é natural e fácil.
2.  **Sintaxe Familiar:** Se você já usou Jinja2 (Python) ou Django templates, vai se sentir em casa com o Tera.  A sintaxe é muito parecida, o que facilita o aprendizado.
3.  **Recursos Poderosos:** O Tera não é apenas um simples substituidor de variáveis.  Ele oferece:
    *   **Filtros:** Modificam os dados (por exemplo, converter para maiúsculas, formatar datas).
    *   **Testes:** Verificam condições (por exemplo, se um número é par).
    *   **Macros:** Funções de template reutilizáveis.
    *   **Herança de Templates:**  Crie um layout base e estenda-o em outras páginas.
    *   E muito mais!

## Instalação e Configuração Inicial

Vamos colocar a mão na massa!  Para usar o Tera, você precisa adicioná-lo como dependência no seu projeto Rust.

1.  **Abra o arquivo `Cargo.toml`:**  Este arquivo contém as informações do seu projeto, incluindo as dependências.

2.  **Adicione o Tera:**  Dentro da seção `[dependencies]`, adicione a seguinte linha:

    ```toml
    tera = "1"
    ```
    Isso diz ao Cargo (o gerenciador de pacotes do Rust) para baixar e usar a versão mais recente do Tera (versão 1.x.x).

3. **Dependências Opcionais (Features)**:
   Tera has some optional features that bring in additional dependencies. If you don't need specific filters like `truncate`, `date`, or `filesizeformat`, you can disable the default features to reduce the size of your project:

    ```toml
    tera = { version = "1", default-features = false }
    ```

4.  **`extern crate tera;` (Rust pré-2018):**  Se você estiver usando uma versão do Rust anterior a 2018, precisa adicionar a seguinte linha no início do seu arquivo `src/lib.rs` ou `src/main.rs`:

    ```rust
    extern crate tera;
    ```

    Isso importa a biblioteca Tera para o seu código.  Nas versões mais recentes do Rust (edição 2018 em diante), isso não é mais necessário.

## Conceitos Fundamentais e Primeiro Exemplo

Antes de criarmos nosso primeiro template, vamos entender alguns conceitos básicos:

*   **Delimitadores:**  O Tera usa símbolos especiais para identificar as partes do template que devem ser processadas:
    *   `{{ ... }}`:  Para exibir o valor de uma variável ou expressão.
    *   `{% ... %}`:  Para executar comandos (como loops e condicionais).
    *   `{# ... #}`:  Para comentários (não aparecem no resultado final).

*  **Raw**
Para criar um bloco onde o texto não interpretado pelo tera, útil para quando precisar usar os delimitadores do tera, podemos usar o bloco `{raw%}`
```
{% raw %}
  Hello {{ name }}
{% endraw %}
```
irá mostrar o texto  `Hello {{ name }}`

* **Espaços em branco**
Em tera temos como remover os espaços em branco desnecessários com `{%-` remove os espaços antes e `-%}` remove os espaços depois, e também funciona com os outros delimitadores `{{-`,`-}}`,`{#-`,`-#}`

*   **Contexto:**  O contexto é um conjunto de dados que você passa para o Tera.  Ele contém as variáveis que serão usadas no template.  Você pode criar um contexto de duas formas principais:
    *   Usando a struct `tera::Context`.
    *   Usando uma struct que implemente a trait `Serialize` do `serde_json`.

*   **Renderização:**  É o processo de combinar o template com o contexto para gerar o resultado final.  Você faz isso usando os métodos `Tera::new` (para criar o motor de templates) e `tera.render` (para renderizar um template específico).

**Exemplo "Olá, Mundo!"**

Vamos criar um exemplo simples para ver o Tera em ação.  Crie uma pasta chamada `templates` no seu projeto (no mesmo nível do `src`). Dentro da pasta `templates`, crie um arquivo chamado `hello.html` com o seguinte conteúdo:

```html
<h1>Olá, {{ name }}!</h1>
```

Agora, no seu arquivo `src/main.rs`, escreva o seguinte código:

```rust
use tera::{Tera, Context};

fn main() -> Result<(), tera::Error> {
    // 1. Cria o motor de templates.
    //    "templates/**/*" significa: procure todos os arquivos
    //    dentro da pasta "templates" e suas subpastas.
    let mut tera = Tera::new("templates/**/*")?;

    // 2. Cria um contexto.
    let mut context = Context::new();
    context.insert("name", "Mundo"); // Adiciona a variável "name".

    // 3. Renderiza o template.
    //    "hello.html": o nome do arquivo dentro da pasta "templates".
    let rendered = tera.render("hello.html", &context)?;

    // 4. Imprime o resultado.
    println!("{}", rendered);

    Ok(())
}
```

**Explicação Detalhada:**

1.  **`Tera::new("templates/**/*")?`:**
    *   `Tera::new` cria uma instância do motor de templates Tera.
    *   `"templates/**/*"` é um *glob pattern*.  Ele diz ao Tera para procurar todos os arquivos (`*`) dentro da pasta `templates` e de todas as suas subpastas (`**`).  O `?` no final é para tratamento de erros (se o Tera não conseguir encontrar a pasta, o programa para).

2.  **`let mut context = Context::new();`:**
    *   Cria um novo contexto vazio.

3.  **`context.insert("name", "Mundo");`:**
    *   Adiciona a variável `name` ao contexto, com o valor "Mundo".  É essa variável que será usada no template `hello.html`.

4.  **`let rendered = tera.render("hello.html", &context)?;`:**
    *   `tera.render` faz a mágica acontecer!  Ele pega o template `hello.html` e o contexto, e combina os dois.
    *   `&context` é uma referência ao contexto (o Tera precisa do contexto para saber os valores das variáveis).
    *   O resultado é armazenado na variável `rendered`.

5.  **`println!("{}", rendered);`:**
    *   Imprime o resultado da renderização no console.

**Executando o Exemplo:**

1.  Salve os arquivos.
2.  No terminal, navegue até a pasta do seu projeto.
3.  Execute o comando `cargo run`.

Você deverá ver a seguinte saída:

```
<h1>Olá, Mundo!</h1>
```

**Exemplo com `serde_json::Serialize`:**

Em vez de usar `tera::Context`, você pode usar uma struct que derive a trait `Serialize` do `serde_json`:

```rust
use tera::{Tera, Context};
use serde::Serialize;

#[derive(Serialize)]
struct Pessoa {
    nome: String,
    idade: u32,
}

fn main() -> Result<(), tera::Error> {
    let mut tera = Tera::new("templates/**/*")?;

    // Cria uma instância da struct Pessoa.
    let pessoa = Pessoa {
        nome: "Alice".to_string(),
        idade: 30,
    };

    // Usa Context::from_serialize para criar o contexto.
    let context = Context::from_serialize(pessoa)?;


    let rendered = tera.render("pessoa.html", &context)?;
    println!("{}", rendered);

    Ok(())
}
```

Crie um arquivo `templates/pessoa.html`:

```html
<p>Nome: {{ nome }}</p>
<p>Idade: {{ idade }}</p>
```

Ao executar, você verá:

```
<p>Nome: Alice</p>
<p>Idade: 30</p>
```

**Conclusão e Próximos Passos**

Parabéns! Você deu os primeiros passos com o Tera.  Você aprendeu o que são templates, por que usar o Tera, como instalá-lo e como criar um exemplo simples.

No próximo artigo, vamos explorar a sintaxe do Tera em mais detalhes: variáveis, expressões, filtros e testes.  Prepare-se para mergulhar fundo!

Este artigo cobre os seguintes pontos:

*   **Introdução clara e acessível a templates.**
*   **Analogia com formulários e receitas.**
*   **Explicação dos benefícios dos templates.**
*   **Motivação para usar o Tera.**
*   **Guia passo a passo para instalação e configuração.**
*   **Explicação detalhada dos conceitos fundamentais.**
*   **Exemplo "Olá, Mundo!" completo e comentado.**
*   **Exemplo com `serde_json::Serialize`.**
*   **Conclusão e indicação dos próximos passos.**
