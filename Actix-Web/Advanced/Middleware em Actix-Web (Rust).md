# Artigo: Middleware em Actix-Web (Rust)

## Introdução

Middleware em Actix-Web permite adicionar comportamentos extras ao processamento de requisições e respostas.  Pense em middleware como "camadas" que envolvem sua aplicação, interceptando requisições e respostas para realizar tarefas como:

-   Pré-processar a requisição (ex: autenticação, logging).
-   Pós-processar a resposta (ex: compressão, adicionar cabeçalhos).
-   Modificar o estado da aplicação.
-   Acessar serviços externos (ex: bancos de dados, caches).

Middleware é registrado para cada `App`, `Scope` ou `Resource` e executado na ordem *inversa* do registro (o último registrado é executado primeiro).

## Criando Middleware

Em geral, um middleware é um tipo que implementa as traits `Service` e `Transform`.  Cada método nessas traits tem uma implementação padrão, e você pode sobrescrever os métodos para customizar o comportamento. Os métodos podem retornar um resultado imediatamente ou uma `Future`.

**Exemplo Básico:**

```rust
//Este é apenas um exemplo do actix-web e não vai funcionar
use actix_web::{
    dev::{Service, ServiceRequest, ServiceResponse, Transform},
    Error,
};
use futures::future::{ok, Ready};

// Há duas etapas na definição do middleware.  "Transform" e "Service".
// A etapa 'Transform' retorna um 'Service' e realiza algumas configurações iniciais.
// A etapa 'Service' é genérica sobre o tipo de requisição e o tipo de resposta do próximo serviço.
pub struct SayHi;

// Middleware factory é `Transform` trait de actix-service crate
// `S` - tipo do próximo serviço
// `B` - tipo do corpo da resposta.
impl<S, B> Transform<S, ServiceRequest> for SayHi
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type InitError = ();
    type Transform = SayHiMiddleware<S>;
    type Future = Ready<Result<Self::Transform, Self::InitError>>;

    fn new_transform(&self, service: S) -> Self::Future {
        ok(SayHiMiddleware { service })
    }
}

pub struct SayHiMiddleware<S> {
    service: S,
}

impl<S, B> Service<ServiceRequest> for SayHiMiddleware<S>
where
    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Future = S::Future;

    //No actix-web é assim:
    //fn call(&self, req: ServiceRequest) -> Self::Future {
     //   println!("Oi do middleware!");
     //   self.service.call(req)
    //}
    //No actix-web-lab é assim:
    fn call(&mut self, req: ServiceRequest) -> Self::Future {
        println!("Oi do middleware!");
        self.service.call(req)
    }
}
```

**Explicação:**

-   `SayHi`:  Uma struct vazia que serve como "factory" para o middleware.
-   `impl<S, B> Transform<S, ServiceRequest> for SayHi`:  Implementa a trait `Transform`, que é responsável por criar a instância do middleware (a "service").
    -   `new_transform`:  Este método é chamado para criar o middleware.  Ele recebe o próximo serviço na cadeia (`service: S`) e retorna uma `Future` que resolve para o middleware.
-   `SayHiMiddleware<S>`:  A struct que representa o middleware em si.  Ela armazena o próximo serviço (`service: S`).
-   `impl<S, B> Service<ServiceRequest> for SayHiMiddleware<S>`:  Implementa a trait `Service`, que é responsável por processar a requisição.
    -   `call`:  Este método é chamado para cada requisição.  Ele pode modificar a requisição, a resposta, ou ambos.  No exemplo simples, ele apenas imprime uma mensagem no console.

### Usando `wrap_fn` (Forma Mais Simples)

Para casos de uso simples, você pode usar `wrap_fn` para criar middleware a partir de uma função ou closure:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, middleware, dev::ServiceRequest};

async fn index() -> HttpResponse {
    HttpResponse::Ok().body("Hello, world!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap_fn(|req: ServiceRequest, srv| { // Utiliza o wrap_fn para definir o middleware
                println!("Antes da requisição: {}", req.path());
                let fut = srv.call(req);
                async {
                    let res = fut.await?;
                    println!("Depois da requisição");
                    Ok(res)
                }
            })
            .route("/", web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `wrap_fn(|req, srv| ...)`:  Recebe uma closure que recebe a `ServiceRequest` (`req`) e o próximo serviço (`srv`).
-   `srv.call(req)`:  Chama o próximo serviço (o handler ou outro middleware).  É *muito importante* chamar `srv.call(req)` em algum ponto, caso contrário, a requisição não será processada.
- A closure *deve* retornar um futuro (por isso o bloco `async {}` interno), mesmo que o código dentro dela seja síncrono.

### Usando `from_fn` e `wrap`

Você pode usar a combinação de `from_fn` e `wrap`, para criar middlewares a partir de funções.
```rust
//Este é apenas um exemplo do actix-web, mas não funciona por que o from_fn foi depreciado
//use actix_web::{
//    dev::{forward_ready, Service, ServiceRequest, ServiceResponse, Transform},
//    error, web, App, Error, HttpResponse,
//};
//
//use futures_util::future::LocalBoxFuture;
//use std::future::{ready, Ready};
//
//// Este middleware checa se o header "Meu-Header" está presente
//pub struct ChecaHeader;
//
//// Implementa a trait Transform para o middleware ChecaHeader
//impl<S, B> Transform<S, ServiceRequest> for ChecaHeader
//where
//    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
//    S::Future: 'static,
//    B: 'static,
//{
//    type Response = ServiceResponse<B>;
//    type Error = Error;
//    type InitError = ();
//    type Transform = ChecaHeaderMiddleware<S>;
//    type Future = Ready<Result<Self::Transform, Self::InitError>>;
//
//    fn new_transform(&self, service: S) -> Self::Future {
//        ready(Ok(ChecaHeaderMiddleware { service }))
//    }
//}
//
//// Middleware ChecaHeaderMiddleware que implementa a lógica de verificação
//pub struct ChecaHeaderMiddleware<S> {
//    service: S,
//}
//
//// Implementa a trait Service para o middleware ChecaHeaderMiddleware
//impl<S, B> Service<ServiceRequest> for ChecaHeaderMiddleware<S>
//where
//    S: Service<ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
//    S::Future: 'static,
//    B: 'static,
//{
//    type Response = ServiceResponse<B>;
//    type Error = Error;
//    type Future = S::Future;
//    // Indica se o serviço está pronto para processar a próxima requisição
//    forward_ready!(service);
//
//    fn call(&mut self, req: ServiceRequest) -> Self::Future {
//        // Verifica se o header "Meu-Header" está presente
//        if req.headers().contains_key("Meu-Header") {
//           self.service.call(req) // Se presente, chama o próximo serviço na cadeia
//        } else {
//            // Se ausente, retorna uma resposta de erro 400 Bad Request
//            let (http_req, _payload) = req.into_parts();
//            let response =  HttpResponse::BadRequest().finish().map_into_right_body();
//            let resp = ServiceResponse::new(http_req,response);
//            return Box::pin(async {Err(error::ErrorBadRequest("Meu-Header faltando"))});
//
//        }
//    }
//}

```

**Ordem de Execução:**

Lembre-se: middleware é executado na ordem *inversa* do registro.  Se você usar `wrap()` ou `wrap_fn()` várias vezes, o *último* middleware registrado será executado *primeiro*.

## Middleware Padrão (Built-in Middleware)

O Actix-Web fornece vários middlewares prontos para uso:

-   **`Logger`:**  Faz logging das requisições (ver detalhes abaixo).
-   **`Compress`:**  Comprime as respostas (já mencionado no artigo anterior).
-   **`DefaultHeaders`:**  Adiciona cabeçalhos padrão às respostas.
-   **`Session` (actix-session):**  Gerencia sessões de usuário (ver detalhes abaixo).
-   **`ErrorHandlers`:**  Permite definir handlers customizados para códigos de erro HTTP (ver detalhes abaixo).

## Logging (`Logger`)

O middleware `Logger` usa a crate `log` padrão do Rust para registrar informações sobre as requisições.

**Uso:**

```rust
use actix_web::{web, App, HttpResponse, HttpServer, middleware::Logger};
use env_logger::Env;


async fn index() -> HttpResponse {
    HttpResponse::Ok().body("Hello, world!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
     env_logger::init_from_env(Env::default().default_filter_or("info")); // Inicializa o logger

    HttpServer::new(|| {
        App::new()
            .wrap(Logger::default()) // Habilita o logging com o formato padrão
            // ou .wrap(Logger::new("%a %{User-Agent}i")) para um formato customizado
            .route("/", web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `env_logger::init_from_env(...)`:  Inicializa o sistema de logging (você precisa da crate `env_logger`).
-   `wrap(Logger::default())`:  Registra o middleware `Logger` com o formato de log padrão.
-   `Logger::new(...)`:  Permite especificar um formato de log customizado.

**Formato Padrão:**

```
%a %t "%r" %s %b "%{Referer}i" "%{User-Agent}i" %T
```

**Exemplo de Saída:**

```
INFO:actix_web::middleware::logger: 127.0.0.1:59934 [02/Dec/2017:00:21:43 -0800] "GET / HTTP/1.1" 302 0 "-" "curl/7.54.0" 0.000397
```

**Variáveis do Formato:**

| Variável      | Descrição                                                                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `%%`          | O sinal de porcentagem (%)                                                                                                                      |
| `%a`          | Endereço IP remoto (ou IP do proxy, se usando um proxy reverso)                                                                                 |
| `%t`          | Hora em que a requisição começou a ser processada                                                                                               |
| `%P`          | ID do processo filho que atendeu a requisição                                                                                                 |
| `%r`          | Primeira linha da requisição (ex: `GET / HTTP/1.1`)                                                                                             |
| `%s`          | Código de status da resposta                                                                                                                   |
| `%b`          | Tamanho da resposta em bytes, incluindo cabeçalhos                                                                                              |
| `%T`          | Tempo para servir a requisição, em segundos (com fração)                                                                                        |
| `%D`          | Tempo para servir a requisição, em milissegundos                                                                                                |
| `%{FOO}i`     | Valor do cabeçalho de requisição `FOO`                                                                                                        |
| `%{FOO}o`     | Valor do cabeçalho de resposta `FOO`                                                                                                         |
| `%{FOO}e`     | Valor da variável de ambiente `FOO`                                                                                                             |

## Cabeçalhos Padrão (`DefaultHeaders`)

O middleware `DefaultHeaders` adiciona cabeçalhos padrão às respostas.  Ele *não* sobrescreve cabeçalhos que já foram definidos.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, middleware::DefaultHeaders};

async fn index() -> HttpResponse {
  HttpResponse::Ok().body("Hello")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
          .wrap(DefaultHeaders::new().add(("X-Version", "0.1"))) // Adiciona o cabeçalho X-Version
          .route("/",web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

## Sessões de Usuário (`actix-session`)

O Actix-Web fornece gerenciamento de sessões através da crate `actix-session`.  Ele suporta diferentes backends para armazenar os dados da sessão (por padrão, cookies).

**Importante:** Para usar `actix-session`, você precisa adicionar a dependência no seu `Cargo.toml`:

```toml
[dependencies]
actix-session = "0.8" # (ou a versão mais recente)
actix-web = "4"
```

**Exemplo (Usando Cookies):**

```rust
use actix_web::{web, App, HttpRequest, HttpResponse, HttpServer, Responder, get};
use actix_session::{Session, storage::CookieSessionStore, config::PersistentSession, SessionMiddleware};
use time::Duration;


// Handler para definir um valor na sessão
#[get("/set")]
async fn set_session_value(session: Session) -> impl Responder {
    session.insert("my_key", "my_value").unwrap();
    HttpResponse::Ok().body("Valor definido na sessão")
}

// Handler para obter um valor da sessão
#[get("/get")]
async fn get_session_value(session: Session) -> impl Responder {
    if let Ok(Some(value)) = session.get::<String>("my_key") {
        HttpResponse::Ok().body(format!("Valor da sessão: {}", value))
    } else {
        HttpResponse::Ok().body("Valor não encontrado na sessão")
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let secret_key = actix_web::cookie::Key::generate(); // Gere uma chave secreta aleatória

    HttpServer::new(move || {
        App::new()
            .wrap(
                SessionMiddleware::builder(CookieSessionStore::default(), secret_key.clone())
                    .cookie_name("my-app-cookie".to_string())
                    .cookie_secure(false) // Em produção, use true para HTTPS
                    // Configura a sessão para ser persistente (opcional)
                    .session_lifecycle(
                        PersistentSession::default().session_ttl(Duration::minutes(5)),
                    )
                    .build(),
            )
            .service(set_session_value) // Rota para definir valor
            .service(get_session_value) // Rota para obter valor
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `actix_web::cookie::Key::generate()`:  Gera uma chave secreta aleatória para assinar os cookies de sessão.  *Em produção, gere uma chave forte e armazene-a de forma segura!*
-   `SessionMiddleware::builder(...)`:  Cria o middleware de sessão.
    -   `CookieSessionStore::default()`:  Usa o backend de cookies para armazenar a sessão.
    -   `secret_key.clone()`:  Passa a chave secreta.
    -   `.cookie_name(...)`: Define o nome do cookie.
    -   `.cookie_secure(false)`:  *Em produção*, defina como `true` para que o cookie só seja enviado em conexões HTTPS.
    -   `session_lifecycle(...)`: Define o tempo de vida da sessão (opcional).
-   `Session`:  O extractor `Session` é usado para acessar os dados da sessão dentro dos handlers.
    -   `session.insert(...)`:  Define um valor na sessão.
    -   `session.get(...)`:  Obtém um valor da sessão.

**Tipos de Cookies de Sessão:**

-   **Signed:**  O cliente pode ver o conteúdo do cookie, mas não pode modificá-lo (a menos que tenha a chave secreta).
-   **Private:**  O cliente não pode ver nem modificar o conteúdo do cookie.

**Limite de Tamanho (CookieSession):**

Cookies têm um limite de tamanho (geralmente 4KB).  Se você tentar armazenar mais dados do que isso na sessão, o Actix-Web gerará um erro interno do servidor.  Para armazenar mais dados, use um backend diferente (como Redis, banco de dados, etc.).

## Error Handlers (`ErrorHandlers`)

O middleware `ErrorHandlers` permite definir handlers customizados para códigos de erro HTTP (como 404 Not Found, 500 Internal Server Error, etc.).

```rust
use actix_web::{web, App, HttpResponse, HttpServer, http::StatusCode, middleware::ErrorHandlers};

async fn index() -> HttpResponse {
    HttpResponse::Ok().body("Página Inicial")
}

// Handler customizado para 404 Not Found
async fn not_found() -> HttpResponse {
   HttpResponse::build(StatusCode::NOT_FOUND)
        .content_type("text/plain; charset=utf-8")
        .body("Página não encontrada!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(
              ErrorHandlers::new()
                  .handler(StatusCode::NOT_FOUND, |res| {
                    // `res` é um ServiceResponse, permitindo acesso à requisição e resposta originais
                   let req = res.request();
                    println!("Erro 404 para: {}", req.path()); // Exemplo de uso da requisição original

                    let (http_req, _payload) = res.into_parts();
                    //let new_resp = not_found().into_body(); //Reaproveita a função not_found()
                    let response = HttpResponse::build(StatusCode::NOT_FOUND) // ou HttpResponse::NotFound()
                        .content_type("text/plain; charset=utf-8")
                        .body("Página não encontrada! (via middleware)")
                        .map_into_right_body();
                    let new_resp = ServiceResponse::new(http_req, response);
                    Ok(new_resp)
                }),
            )
            .route("/", web::get().to(index))
            //.default_service(web::route().to(not_found)) //Poderia usar um default service, para retornar a resposta 404 customizada.
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

-   `ErrorHandlers::new()`:  Cria o middleware `ErrorHandlers`.
-   `.handler(StatusCode::NOT_FOUND, ...)`:  Registra um handler para o código de status 404.
    - A closure recebe um `ServiceResponse`, permitindo acesso ao request original e modificação/criação da resposta
- Dentro da closure, você pode:
  - Acessar a requisição original (`res.request()`).
  - Construir uma nova resposta (usando `HttpResponse::build()` ou métodos como `HttpResponse::NotFound()`).
  - Retornar um `Result<ServiceResponse, Error>`.

## Conclusão

Middleware é um conceito poderoso no Actix-Web, permitindo que você adicione funcionalidades comuns a todas as suas rotas de forma organizada e reutilizável.  Este artigo cobriu os fundamentos da criação de middleware, o uso de `wrap_fn`, e vários middlewares padrão, incluindo logging, compressão, cabeçalhos padrão, sessões e tratamento de erros.
