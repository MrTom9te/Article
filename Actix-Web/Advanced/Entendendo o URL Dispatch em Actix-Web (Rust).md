## Introdução

O *URL Dispatch* (Despacho de URL) é um mecanismo fundamental em frameworks web como o Actix-Web. Ele permite mapear URLs para funções específicas (handlers) que processam as requisições.  Em outras palavras, é como um sistema de "roteamento" que direciona o tráfego web para o código correto. Este artigo explica como o URL Dispatch funciona no Actix-Web, usando uma linguagem clara e exemplos práticos.

## Fundamentos

Imagine um sistema de correios. Cada carta (requisição) tem um endereço (URL). O carteiro (URL Dispatch) olha o endereço e entrega a carta na casa certa (handler).  No Actix-Web, o "endereço" é a URL, e a "casa" é uma função que vai gerar a resposta (como uma página HTML, dados JSON, etc.).

## Configuração de Recursos (Resource Configuration)

Em Actix-Web, chamamos as "casas" (handlers) de *recursos*. Configurar um recurso significa definir:

1.  **Nome:** Um identificador único para gerar URLs posteriormente.
2.  **Padrão (Pattern):**  Uma "máscara" que define quais URLs se encaixam nesse recurso.
3.  **Rotas (Routes):**  Combinações de métodos HTTP (GET, POST, etc.) e handlers.

### Usando `App::route()`

A maneira mais simples de configurar uma rota é usar `App::route()`. Você passa o padrão da URL, o método HTTP e a função handler.

```rust
use actix_web::{web, App, HttpResponse, HttpServer};

async fn greet() -> HttpResponse {
    HttpResponse::Ok().body("Olá!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(greet)) // Rota para GET na raiz ("/")
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `App::new()`: Cria uma nova aplicação Actix-Web.
-   `.route("/", web::get().to(greet))`:
    -   `"/":` O padrão da URL. Neste caso, a raiz do site.
    -   `web::get()`:  Especifica que esta rota lida com requisições GET.
    -   `.to(greet)`:  Diz que a função `greet` será chamada quando essa rota for acionada.
-   `greet()`: Uma função *assíncrona* (por causa do `async`) que retorna uma `HttpResponse`.
    -   `HttpResponse::Ok().body("Olá!")`: Cria uma resposta HTTP com código 200 (OK) e o corpo "Olá!".

### Usando `App::service()`

`App::service()` oferece mais controle.  Ele permite agrupar várias rotas dentro de um mesmo recurso.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}


#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::resource("/app") //Define um padrão de URL 
                    .route(web::get().to(hello)) //Define uma rota dentro do padrão /app que so aceita o metodo GET
                    .route(web::post().to(echo))//Define uma rota dentro do padrão /app que so aceita o metodo POST
                    .route(web::head().to(manual_hello)), //Define uma rota dentro do padrão /app que so aceita o metodo HEAD
            )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

**Explicação:**

-   `web::resource("/app")`:  Cria um recurso com o padrão "/app".  Todas as rotas dentro dele usarão esse prefixo.
-   `.route(web::get().to(hello))`:  Uma rota dentro do recurso "/app" que lida com GET e chama `hello()`.
-    `.route(web::post().to(echo))`: Uma rota dentro do recurso "/app" que lida com POST e chama `echo()`.
-    `.route(web::head().to(manual_hello))`: Uma rota dentro do recurso "/app" que lida com HEAD e chama `manual_hello()`.

Se nenhuma rota corresponder, o Actix-Web retorna automaticamente um erro 404 (Not Found).

### Configurando Rotas Detalhadamente (Route Configuration)

Dentro de um recurso, você pode usar `Resource::route()` para criar rotas e configurar *guards* (filtros).

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, http::header};

async fn index() -> impl Responder {
    HttpResponse::Ok().body("index")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().service(
            web::resource("/path")
                .route(
                    web::route() //Cria a rota.
                        .guard(web::guard::Get()) //So ativa a rota se o method do http for GET
                        .guard(web::guard::Header("content-type", "text/plain")) // So ativa a rota se o header do http conter "content-type" e "text/plain"
                        .to(index), //Define o handler
                ),
        )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
**Explicação:**

- `web::resource("/path")`: Cria um recurso em `/path`.
-   `.route(web::route() ...)`: Cria uma nova rota.
    -   `.guard(web::guard::Get())`:  Um *guard* que só permite requisições GET.
    -   `.guard(web::guard::Header("content-type", "text/plain"))`: Um *guard* que verifica se o cabeçalho `Content-Type` é `text/plain`.
    -   `.to(index)`:  O handler `index` só será chamado se *ambos* os guards forem verdadeiros.

`Resource::route()` retorna um objeto `Route` que oferece métodos como:

-   `guard()`: Adiciona um guard.
-   `method()`:  Adiciona um guard de método HTTP (como `web::get()`).
-   `to()`:  Define o handler (deve ser o último passo).

## Como o Roteamento Funciona (Route Matching)

4.  **Ordem Importa:** O Actix-Web verifica as rotas na ordem em que foram definidas. A primeira que corresponder é usada.
5.  **Caminho (Path):** O Actix-Web compara o caminho da URL (ex: `/usuarios/123`) com os padrões definidos.
6.  **Guards:** Se houver guards, todos devem ser `true` para a rota ser considerada.
7.  **Handler:** Se uma rota corresponder, seu handler é executado.
8.  **Not Found:** Se nenhuma rota corresponder, o Actix-Web retorna 404.

## Sintaxe dos Padrões de URL (Resource Pattern Syntax)

-   `/`:  Uma barra no início é opcional (mas recomendada para clareza).
-   `{nome}`:  Um *segmento variável*.  Captura qualquer texto até a próxima `/`.  Ex: `/usuarios/{id}` captura o ID do usuário.
-  `{nome:regex}`:  Um segmento variável com uma expressão regular. Ex: `{id:\d+}` captura apenas números.
-   `*`: Coringa, mas geralmente é melhor usar expressões regulares para maior controle.
- `/foo/{bar}`: O padrão `foo/La%20Pe%C3%B1a` retorna `Params {'bar': 'La Peña'}`.

Exemplo de um padrão que captura "o resto" da URL:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
async fn index(info: web::Path<(String, String)>) -> impl Responder {
     format!("bar value {} tail value: {}!", info.0,info.1)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().service(
            web::resource("foo/{bar}/{tail:.*}")
                .route(
                    web::route() //Cria a rota.
                        .guard(web::guard::Get()) //So ativa a rota se o method do http for GET
                        .to(index), //Define o handler
                ),
        )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
- `foo/1/2/` retorna `Params {'bar': '1', 'tail': '2/'}`
-  `foo/abc/def/a/b/c` retorna `Params {'bar': 'abc', 'tail': 'def/a/b/c'}`

## Escopos (Scoping Routes)

Escopos permitem organizar rotas com prefixos comuns.  É como criar "subpastas" de URLs.

```rust
use actix_web::{web, App, HttpResponse, HttpServer};

async fn list_users() -> HttpResponse {
   HttpResponse::Ok().body("Lista de Usuários")
}

async fn show_user(info: web::Path<String>) -> HttpResponse {
   HttpResponse::Ok().body(format!("Mostrando usuário {}", info))
}


#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().service(
            web::scope("/users") //Define o escopo principal
                .route("", web::get().to(list_users))  // Rota para /users
                .route("/show/{id}", web::get().to(show_user)),  // Rota para /users/show/{id}
        )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `web::scope("/users")`:  Todas as rotas dentro desse escopo terão o prefixo `/users`.
-   `.route("", ...)`:  A rota vazia "" dentro do escopo corresponde a `/users`.
-  `.route("/show/{id}", ...)`: Corresponde a `/users/show/{id}`.

Você pode ter escopos aninhados (escopos dentro de escopos).

## Informações do Casamento (Match Information)

O Actix-Web armazena os valores capturados dos segmentos variáveis (como `{id}`) em `HttpRequest::match_info()`.

```rust
use actix_web::{web, App, HttpRequest, HttpServer, Responder};

async fn index(req: HttpRequest) -> impl Responder {
    let v1: String =  req.match_info().get("v1").unwrap().parse().unwrap();
    let v2: String =  req.match_info().query("v2").parse().unwrap();
    format!("Valores: {} e {}", v1, v2)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
           .route("/a/{v1}/{v2}/", web::get().to(index)) // NOTE: trailing slash!
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```
-   `req.match_info().get("v1")`:  Obtém o valor do segmento chamado "v1".
-   `req.match_info().query("v2")`: Obtém o valor de um parâmetro *query string* (parte depois do `?` na URL) chamado "v2".

### Extratores de Informações do Path (Path Information Extractor)

Para extrair informações de forma *type-safe* (com tipos definidos), use `web::Path`.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// Extraindo para uma tupla
async fn with_tuple(info: web::Path<(u32, String)>) -> impl Responder {
    format!("ID: {}, Nome: {}", info.0, info.1)
}
// Extraindo para uma struct
#[derive(serde::Deserialize)]
struct Info {
   username: String,
}

async fn with_struct(info: web::Path<Info>) -> impl Responder {
     format!("Usuário: {}", info.username)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
           .route("/tuple/{id}/{name}", web::get().to(with_tuple))
            .route("/struct/{username}", web::get().to(with_struct))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

-   `web::Path<(u32, String)>`:  Extrai dois valores: um `u32` e uma `String`. A ordem importa!
-  `web::Path<Info>`:  Extrai para a struct `Info`, que *deve* implementar `serde::Deserialize`. Os nomes dos campos da struct devem corresponder aos nomes dos segmentos variáveis.

`web::Query` funciona de forma similar para parâmetros de query string.

## Gerando URLs (Generating Resource URLs)

Use `HttpRequest::url_for()` para gerar URLs a partir dos nomes dos recursos.

```rust
//No código do actix-web este é um exemplo, mas ele não funciona, pois a função url_for não existe mais 
//use actix_web::{web, App, HttpRequest, HttpServer, Responder};
//
//async fn index(req: HttpRequest) -> impl Responder {
//    let url = req.url_for("foo", &["a", "b", "c"]); // Gera a URL para o recurso "foo"
//    match url {
//       Ok(url) => format!("URL: {}", url),
//        Err(_) => "Erro ao gerar URL".to_string(),
//    }
//}

//#[actix_web::main]
//async fn main() -> std::io::Result<()> {
//    HttpServer::new(|| {
//        App::new()
//            .service(web::resource("/test/{a}/{b}/{c}").name("foo").route(web::get().to(|| async {
//                HttpResponse::Ok()
//            })))
//            .route("/", web::get().to(index))
//    })
//    .bind(("127.0.0.1", 8080))?
//    .run()
//    .await
//}
```

-   `req.url_for("foo", &["a", "b", "c"])`: Gera a URL para o recurso chamado "foo", substituindo os segmentos variáveis pelos valores fornecidos.
- `url_for()` só funciona para recursos *nomeados* (usando `.name()`).

## Recursos Externos (External Resources)

Você pode registrar URLs externas (como links para outros sites) para usar em `url_for()`.

```rust
use actix_web::{web, App, HttpRequest, HttpResponse, HttpServer, Responder};

//No código do actix-web este é um exemplo, mas ele não funciona, pois a função url_for não existe mais 
//async fn index(req: HttpRequest) -> impl Responder {
//    let url = req.url_for("youtube", &[""]); //Não vai funcionar pois a função url_for não existe mais
//    match url {
//        Ok(url) => format!("youtube URL: {}", url),
//        Err(_) => "Erro ao gerar URL".to_string(),
//    }
//}

//#[actix_web::main]
//async fn main() -> std::io::Result<()> {
//    HttpServer::new(|| {
//        App::new()
//            .external_resource("youtube", "https://youtube.com/watch/{v}") //Define um recurso externo
//            .route("/", web::get().to(index))
//    })
//    .bind(("127.0.0.1", 8080))?
//    .run()
//    .await
//}
```
- `external_resource("youtube", "https://youtube.com/watch/{v}")`: Define um recurso externo chamado "youtube".

Recursos externos só servem para gerar URLs, não para roteamento.

## Normalização de Caminho (Path Normalization)

O Actix-Web pode normalizar caminhos (URLs):

-   Adicionar uma `/` no final, se necessário.
-   Remover barras duplicadas (`//` vira `/`).

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, middleware};

async fn index() -> impl Responder {
     HttpResponse::Ok().body("Página")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(middleware::NormalizePath::default()) // Habilita a normalização
            .route("/resource/", web::get().to(index))
            .route("/resource", web::get().to(index))

    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
- `wrap(middleware::NormalizePath::default())`: Ativa a normalização.
- Se você acessar `/resource` (sem a barra final), o Actix-Web redirecionará para `/resource/`.

**Importante:** A normalização de caminho redireciona requisições POST para GET, o que pode causar perda de dados.  Use com cuidado!

Você pode configurar a normalização para funcionar apenas com GET:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, middleware};

async fn index() -> impl Responder {
     HttpResponse::Ok().body("Página")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(middleware::NormalizePath::trim()) // Habilita a normalização apenas para o GET
            .route("/resource/", web::get().to(index))
            .route("/resource", web::get().to(index))

    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
- `wrap(middleware::NormalizePath::trim())`: Ativa a normalização somente para GET.

## Prefixo de Aplicação (Application Prefix)

`web::scope()` também pode ser usado como um "prefixo" para toda a aplicação.

```rust
//Este é um exemplo do actix-web porem ele não funciona por que a função .name() não existe mais neste contexto
//use actix_web::{web, App, HttpRequest, HttpResponse, HttpServer, Responder};
//
//async fn show_users() -> impl Responder {
//     HttpResponse::Ok().body("Mostrando usuários")
//}
//#[actix_web::main]
//async fn main() -> std::io::Result<()> {
//    HttpServer::new(|| {
//        App::new()
//            .service(
//                web::scope("/users") //Define o escopo principal
//                    .route("/show", web::get().to(show_users).name("show_users")), //Define o escopo da rota
//            )
//    })
//    .bind(("127.0.0.1", 8080))?
//    .run()
//    .await
//}
```

## Guards Personalizados (Custom Route Guard)

Um *guard* é uma função que recebe uma requisição e retorna `true` ou `false`.  Você pode criar seus próprios guards implementando o trait `Guard`.

```rust
use actix_web::{guard::{Guard, GuardContext}, web, App, HttpResponse, HttpServer, Responder};

struct MyGuard;

impl Guard for MyGuard {
   fn check(&self, ctx: &GuardContext) -> bool {
        ctx.head().headers().contains_key("x-my-header")//Verifica se o header contem a chave "x-my-header"
    }
}

async fn index() -> impl Responder {
     HttpResponse::Ok().body("Index")
}


#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::route().guard(MyGuard).to(index)) //Define o guard customizado
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

-   `struct MyGuard`:  Define um novo guard.
-   `impl Guard for MyGuard`:  Implementa o trait `Guard`.
    -   `check()`:  Verifica se a requisição contém o cabeçalho `x-my-header`.

Você pode combinar guards:

-   `guard::Not(guard)`: Inverte o resultado de um guard.
-   `guard::Any(guard).or(guard)`:  Verdadeiro se *qualquer* guard for verdadeiro.
-   `guard::All(guard).and(guard)`: Verdadeiro se *todos* os guards forem verdadeiros.

## Mudando a Resposta Padrão "Not Found" (Changing the default Not Found response)

Para personalizar a resposta 404, use `App::default_service()`.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, http};

async fn not_found() -> impl Responder {
     HttpResponse::build(http::StatusCode::NOT_FOUND).body("Página não encontrada!") //Define uma resposta 404 personalizada
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .default_service(web::route().to(not_found)) //Define uma resposta personalizada para quando nenhuma rota é encontrada
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
- `default_service(web::route().to(not_found))`: Define `not_found` como o handler padrão para rotas não encontradas.

## Conclusão

O URL Dispatch do Actix-Web é um sistema poderoso e flexível para rotear requisições.  Compreender seus conceitos-chave (recursos, rotas, padrões, guards, escopos) é essencial para construir aplicações web robustas em Rust.  Este guia cobriu os fundamentos, mas há muitos outros recursos avançados a explorar na documentação oficial do Actix-Web.
