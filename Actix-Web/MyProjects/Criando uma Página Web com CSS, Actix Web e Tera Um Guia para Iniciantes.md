# Criando uma Página Web com CSS, Actix Web e Tera: Um Guia para Iniciantes

Se você está começando no mundo da programação web e quer aprender a criar páginas com um visual atraente, este guia é para você! Vamos explorar como combinar o poder do Rust com o framework Actix Web, o sistema de templates Tera e, claro, o CSS para dar estilo à sua página.

## O Que Vamos Aprender?

Neste artigo, você aprenderá:

1.  **Conceitos Fundamentais**: O que são HTML, CSS, Actix Web e Tera.
2.  **Configuração do Projeto**: Como preparar o ambiente de desenvolvimento.
3.  **Estrutura HTML Básica**: Como criar o esqueleto da sua página.
4.  **Estilizando com CSS**: Como adicionar cores, fontes e layout.
5.  **Integração com Actix Web**: Como servir sua página HTML.
6.  **Usando Templates Tera**: Como criar páginas dinâmicas.
7.  **Dicas e Melhores Práticas**: Como organizar seu código e evitar erros comuns.

## 1. Fundamentos: As Peças do Quebra-Cabeça

Antes de mergulharmos no código, vamos entender o papel de cada ferramenta:

-   **HTML (HyperText Markup Language)**: É a linguagem que define a estrutura da sua página. Pense nele como o esqueleto do seu site, organizando o conteúdo em cabeçalhos, parágrafos, imagens, etc.
-   **CSS (Cascading Style Sheets)**: É a linguagem que define o estilo da sua página. Imagine-o como a "roupa" do seu site, controlando cores, fontes, espaçamento e layout.
-   **Actix Web**: É um framework web para Rust que nos permite criar servidores web de forma rápida e eficiente. Ele será o "motor" que faz nosso site funcionar.
-   **Tera**: É um sistema de templates para Rust. Ele nos ajuda a criar páginas HTML dinâmicas, onde podemos inserir dados do nosso código Rust.

## 2. Configurando o Projeto: Preparando o Terreno

Para começar, você precisa ter o Rust instalado no seu computador. Se ainda não tiver, siga as instruções em [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).

Agora, vamos criar um novo projeto Rust:

1.  Abra o terminal (ou prompt de comando).
2.  Digite:

    ```bash
    cargo new minha_pagina_web
    cd minha_pagina_web
    ```

Isso cria uma nova pasta chamada `minha_pagina_web` com a estrutura básica de um projeto Rust.

1.  Adicione as dependências:

Abra o arquivo `Cargo.toml` e adicione as seguintes linhas na seção `[dependencies]`:

```toml
actix-web = "4"  # Versão mais recente do Actix Web
tera = "1"       # Versão mais recente do Tera
serde = { version = "1.0", features = ["derive"] } #Para serializar os dados para o template
```

## 3. Estrutura HTML Básica: O Esqueleto da Página

Crie uma pasta chamada `templates` na raiz do seu projeto (no mesmo nível do `src`). Dentro de `templates`, crie um arquivo chamado `index.html`:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minha Página Web</title>
    <link rel="stylesheet" href="static/style.css">
</head>
<body>
    <h1>Olá, Mundo!</h1>
    <p>Bem-vindo à minha primeira página web com Actix, Tera e CSS!</p>
</body>
</html>
```

**Explicação**:

-   `<!DOCTYPE html>`: Informa ao navegador que estamos usando HTML5.
-   `<html lang="pt-br">`: Define o idioma da página.
-   `<head>`: Contém informações sobre a página, como o título e links para arquivos CSS.
-   `<meta charset="UTF-8">`: Define a codificação de caracteres (para exibir acentos corretamente).
-   `<meta name="viewport" ...>`: Configura a exibição em dispositivos móveis.
-   `<title>`: O título que aparece na aba do navegador.
-   `<link rel="stylesheet" ...>`: Importa nosso arquivo CSS (que criaremos em breve).
-   `<body>`: Contém o conteúdo visível da página.
-   `<h1>`: Um cabeçalho principal.
-   `<p>`: Um parágrafo.

## 4. Estilizando com CSS: Dando Vida à Página

Crie uma pasta chamada `static` na raiz do projeto. Dentro de `static`, crie um arquivo chamado `style.css`:

```css
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    text-align: center;
}

h1 {
    color: #333;
}

p {
    color: #666;
}
```

**Explicação**:

-   `body`: Aplica estilos a todo o corpo da página.
    -   `font-family`: Define a fonte do texto.
    -   `background-color`: Define a cor de fundo.
    -   `text-align`: Alinha o texto ao centro.
-   `h1`: Aplica estilos ao cabeçalho `<h1>`.
    -   `color`: Define a cor do texto.
-   `p`: Aplica estilos ao parágrafo `<p>`.
    -   `color`: Define a cor do texto.

## 5. Integração com Actix Web: Servindo a Página

Agora, vamos fazer o Actix Web servir nossa página. Modifique o arquivo `src/main.rs` para o seguinte:

```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};
use tera::Tera;
use serde::Serialize;

//criamos um struct que representa os dados que serão enviados para o template
#[derive(Serialize)]
struct Context {
    mensagem: String,
}

#[get("/")]
async fn index(tera: web::Data<Tera>) -> impl Responder {
    //criamos uma instância do Context com uma mensagem
    let context = Context {
        mensagem: "Bem-vindo à minha página dinâmica!".to_string(),
    };
  
    //renderizamos o template, passando o context como parametro
    let rendered = tera.render("index.html", &tera::Context::from_serialize(&context).unwrap())
        .unwrap();
    
    // Retorna o HTML renderizado como resposta
    HttpResponse::Ok().body(rendered)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Inicializa o Tera e carrega os templates da pasta "templates"
    let tera = Tera::new("templates/**/*").unwrap();

    // Inicia o servidor Actix Web
    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(tera.clone())) // Registra o Tera no app
            .service(index) // Configura a rota "/"
            .service(actix_files::Files::new("/static", "./static")) // Serve arquivos estáticos (CSS, JS, etc.)
    })
    .bind(("127.0.0.1", 8080))? // Define o endereço e a porta
    .run() // Inicia o servidor
    .await
}
```

**Explicação detalhada**:

1.  **Importações**:
    -   `actix_web`: As funções e estruturas básicas do Actix Web.
    -   `tera::Tera`: O motor de templates Tera.
    -  `serde::Serialize`: Para passar os dados de forma serializada.

2.  **Struct Context**: Define os dados que vamos passar para o template.
    - `@derive(Serialize)`: Permite que a estrutura seja convertida em um formato que o Tera entende.

3.  **Função `index`**:
    -   `#[get("/")]`: Define que esta função será executada quando alguém acessar a raiz do site (`/`).
    -   `tera: web::Data<Tera>`: Recebe uma instância do Tera como argumento.
    -   `let context = ...`: Cria os dados que o template vai usar.
    -   `tera.render(...)`: Renderiza o template `index.html` com os dados do `context`.
    -   `HttpResponse::Ok().body(...)`: Retorna o HTML renderizado como resposta.

4.  **Função `main`**:
    -   `#[actix_web::main]`: Indica que esta é a função principal (o ponto de entrada do programa).
    -   `Tera::new("templates/**/*").unwrap()`: Cria uma instância do Tera e carrega todos os arquivos HTML da pasta `templates`.
    -   `HttpServer::new(...)`: Configura o servidor Actix Web.
    -   `.app_data(web::Data::new(tera.clone()))`: Registra a instância do Tera para que possamos usá-la nas nossas funções de rota.
    -   `.service(index)`: Associa a função `index` à rota `/`.
    -   `.service(actix_files::Files::new("/static", "./static"))`: Configura o Actix para servir arquivos estáticos (como CSS e JavaScript) da pasta `static`. Isso é crucial para que nosso arquivo `style.css` seja carregado.
    -   `.bind(("127.0.0.1", 8080))?`: Define que o servidor vai rodar no endereço `localhost` (127.0.0.1) na porta 8080.
    -   `.run()`: Inicia o servidor.

## 6. Usando Templates Tera: Páginas Dinâmicas

Agora, vamos modificar nosso `index.html` para usar o Tera:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Minha Página Web</title>
    <link rel="stylesheet" href="static/style.css">
</head>
<body>
    <h1>Olá, Mundo!</h1>
    <p>{{ context.mensagem }}</p> </body>
</html>
```

**O que mudou?**

-   `<p>{{ context.mensagem }}</p>`: Usamos a sintaxe `{{ ... }}` do Tera para inserir o valor da variável `mensagem` que definimos no nosso código Rust.

## 7. Executando e Testando

1.  No terminal, execute o comando:

    ```bash
    cargo run
    ```

2.  Abra o seu navegador e acesse: `http://127.0.0.1:8080`

Você deverá ver a sua página com o título "Olá, Mundo!" e a mensagem "Bem-vindo à minha página dinâmica!", tudo estilizado com o CSS que criamos!

## Dicas e Melhores Práticas

-   **Organização**: Mantenha seus arquivos HTML em `templates`, seus arquivos CSS em `static/css`, e seus arquivos JavaScript em `static/js`.
-   **Nomes Descritivos**: Use nomes de arquivos e variáveis que indiquem claramente o que eles fazem.
-   **Comentários**: Adicione comentários ao seu código para explicar o que ele faz. Isso é útil para você e para outros desenvolvedores.
-   **Validação HTML/CSS**: Use ferramentas online (como o validador do W3C) para verificar se seu HTML e CSS estão corretos.
-   **Reutilização**: Se você tiver partes de código que se repetem em várias páginas (como um cabeçalho ou rodapé), crie templates separados para eles e inclua-os nos seus templates principais.

## Próximos Passos

-   **Aprenda mais sobre CSS**: Explore propriedades como `display`, `position`, `flexbox` e `grid` para criar layouts mais complexos.
-   **Adicione Interatividade com JavaScript**: Use JavaScript para manipular elementos da página, responder a eventos do usuário e fazer requisições ao servidor.
-   **Explore o Actix Web**: Aprenda a criar rotas mais complexas, lidar com formulários, usar bancos de dados e muito mais.
-   **Aprofunde-se no Tera**: Explore filtros, loops, condicionais e outras funcionalidades do Tera para criar templates mais poderosos.

Com este guia, você deu os primeiros passos para criar páginas web dinâmicas e estilosas com Rust, Actix Web, Tera e CSS. Continue praticando, experimentando e explorando as possibilidades!
