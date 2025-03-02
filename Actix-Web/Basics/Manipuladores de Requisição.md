Um *request handler* (manipulador de requisição) é a função que efetivamente processa uma requisição HTTP no Actix Web. É a função que você escreve para definir o que acontece quando uma determinada URL é acessada.

**Características Principais:**

*   **Função Assíncrona:**  Handlers são funções assíncronas (`async fn`). Isso é fundamental para a alta performance do Actix Web, permitindo que o servidor lide com muitas requisições simultaneamente sem bloquear.
*   **Parâmetros (Extratores):**  Handlers podem receber zero ou mais parâmetros. Esses parâmetros são *extratores* (tipos que implementam `FromRequest`), que extraem informações da requisição (como vimos no artigo anterior).
*   **Retorno (Responder):**  Handlers retornam um tipo que pode ser convertido em uma resposta HTTP (`HttpResponse`).  Isso significa que o tipo de retorno deve implementar a trait `Responder`.
*   **Duas Etapas:** O processamento do handler ocorre em duas etapas:
    1.  **Chamada do Handler:**  A função handler é chamada com os extratores como argumentos.
    2.  **`respond_to()`:**  O método `respond_to()` é chamado no valor retornado pelo handler. Esse método converte o valor em um `HttpResponse` (ou em um `Error`, se algo der errado).

**`Responder` Trait:**

O Actix Web fornece implementações padrão de `Responder` para vários tipos comuns, como:

*   `&'static str`:  Uma string estática.
*   `String`:  Uma string dinâmica.
*   `HttpResponse`:  Você pode retornar diretamente um `HttpResponse` se precisar de controle total sobre a resposta.
*   `Result<T, E>`: Onde `T` implementa `Responder` e `E` implementa `Into<Error>`. Permite retornar um resultado que pode ser um sucesso ou um erro.

**Exemplos de Handlers Válidos (do Artigo, com Pequenas Melhorias):**

```rust
use actix_web::{HttpRequest, HttpResponse, Responder};

// Exemplo 1: Retorna &'static str
async fn index1(_req: HttpRequest) -> &'static str {
    "Hello world!"
}

// Exemplo 2: Retorna String
async fn index2(_req: HttpRequest) -> String {
    "Hello world!".to_owned()
}

// Exemplo 3: Retorna impl Responder (bom para tipos complexos)
async fn index3(_req: HttpRequest) -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
    // Ou, usando web::Bytes:
    // web::Bytes::from_static(b"Hello world!")
}

// Exemplo 4 (Avançado): Retorna um Future (não recomendado na maioria dos casos)
// use std::future::Future;
// use actix_web::Error;
// async fn index4(req: HttpRequest) -> Box<dyn Future<Output = Result<HttpResponse, Error>>> {
//     // ... (lógica complexa assíncrona)
//     Box::new(async { Ok(HttpResponse::Ok().body("Hello from a future!")) })
// }

```

*   **`&'static str`:**  A forma mais simples. Bom para respostas estáticas.
*   **`String`:**  Para respostas dinâmicas (que você constrói em tempo de execução).
*   **`impl Responder`:**  Uma forma mais genérica de dizer "esta função retorna algo que pode ser convertido em uma resposta".  É útil quando o tipo de retorno é complexo ou quando você quer evitar especificar o tipo exato.
*   **`Box<dyn Future<...>>`:**  Essa forma (exemplo 4) é *altamente desencorajada* na maioria dos casos. Você quase sempre deve usar `async fn` diretamente, que já retorna um `Future` de forma implícita.  Usar `Box<dyn Future>` adiciona complexidade desnecessária e pode prejudicar o desempenho.

**Response with custom type (Resposta com Tipo Personalizado):**

Você pode criar seus próprios tipos de resposta customizados. Para isso, você precisa implementar a trait `Responder` para o seu tipo.

**Exemplo (Serialização para JSON):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, Error, http::header::ContentType};
use serde::Serialize;

// 1. Define a struct para o seu tipo personalizado
#[derive(Serialize)]
struct MyResponse {
    message: String,
}

// 2. Implementa Responder para o seu tipo
impl Responder for MyResponse {
    type Body = String; // O tipo é String

    fn respond_to(self, _req: &actix_web::HttpRequest) -> HttpResponse<Self::Body> {
        // 3. Serializa o objeto para JSON
        let body = serde_json::to_string(&self).unwrap();

        // 4. Cria a resposta HTTP
        HttpResponse::Ok()
            .content_type(ContentType::json()) // Define o Content-Type como application/json
            .body(body)
    }
}

// 5. Handler que usa o tipo personalizado
async fn greet() -> impl Responder {
    MyResponse {
        message: "Olá, mundo!".to_owned(),
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(greet))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

*   **`#[derive(Serialize)]`:**  Usa `serde` para permitir que `MyResponse` seja *serializada* (convertida) para JSON.
*   **`impl Responder for MyResponse`:**  Implementa a trait `Responder`.
*   **`type Body = String;`:** Define que essa implementação tem corpo do tipo `String`.
*   **`respond_to()`:**
    *   `serde_json::to_string(&self).unwrap()`:  Serializa o objeto `MyResponse` para uma string JSON.  O `.unwrap()` causará pânico se a serialização falhar (o que é raro).
    *   `HttpResponse::Ok()...`:  Cria uma resposta HTTP 200 OK.
    *   `.content_type(ContentType::json())`:  Define o cabeçalho `Content-Type` para `application/json`. Isso informa ao cliente (ex: navegador) que a resposta é JSON.
    *   `.body(body)`:  Define o corpo da resposta como a string JSON.

**Streaming response body (Corpo de Resposta em Streaming):**

Você pode enviar respostas com *streaming*, onde o corpo da resposta é gerado e enviado de forma assíncrona, em pedaços. Isso é útil para:

*   Respostas muito grandes que não cabem na memória.
*   Respostas geradas dinamicamente (ex: dados de um banco de dados, eventos em tempo real).

Para fazer streaming, o tipo do corpo da resposta deve implementar a trait `Stream` do Rust (na verdade, a trait `Stream` do crate `futures`).

**Exemplo (Streaming Simples):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, Error};
use futures_util::stream::{iter, Stream};
use futures_util::StreamExt;
use bytes::Bytes;

// Função que cria um Stream de Bytes
fn my_stream() -> impl Stream<Item = Result<Bytes, Error>> {
    // Cria um stream a partir de um vetor de strings
    iter(vec![
        Ok(Bytes::from_static(b"Parte 1\n")),
        Ok(Bytes::from_static(b"Parte 2\n")),
        Ok(Bytes::from_static(b"Parte 3\n")),
    ])
}

// Handler que usa o stream
async fn stream_handler() -> impl Responder {
    HttpResponse::Ok()
        .content_type("text/plain") // Define o Content-Type
        .streaming(my_stream()) // Usa o método .streaming()
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
        .route("/", web::get().to(stream_handler))
    })
      .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

*   **`futures_util::stream::iter`:** Cria um `Stream` a partir de um iterador (neste caso, um vetor).
*   **`Ok(Bytes::from_static(...))`:**  Cada item do stream deve ser um `Result<Bytes, Error>`. `Bytes` é um tipo do Actix Web para representar sequências de bytes.
*   **`HttpResponse::Ok().streaming(my_stream())`:**  Usa o método `.streaming()` para criar uma resposta com streaming.
*   **Importante:** Você precisa adicionar `futures-util = "0.3"` (ou uma versão compatível) ao seu `Cargo.toml`.

**Different return types (Either) (Tipos de Retorno Diferentes):**

Às vezes, você precisa que um handler retorne tipos *diferentes* dependendo de alguma condição (ex: sucesso ou erro, um tipo ou outro).  O tipo `Either` do Actix Web permite fazer isso.

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, Either};

// Handler que pode retornar String ou HttpResponse
async fn my_handler(param: web::Query<String>) -> Either<String, HttpResponse> {
    if let Some(name) = param.into_inner().as_ref() {
        // Se o parâmetro 'name' for fornecido, retorna uma String
        Either::Left(format!("Olá, {}!", name))
    } else {
        // Caso contrário, retorna um erro 400 Bad Request
        Either::Right(HttpResponse::BadRequest().body("Parâmetro 'name' ausente!"))
    }
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
     HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(my_handler))
     })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

*   **`Either<String, HttpResponse>`:**  O handler pode retornar ou uma `String` (lado esquerdo - `Left`) ou um `HttpResponse` (lado direito - `Right`).
*   **`Either::Left(...)`:**  Cria um valor `Either` com o lado esquerdo preenchido.
*   **`Either::Right(...)`:**  Cria um valor `Either` com o lado direito preenchido.
*   **Importante:** Ambos os tipos (neste caso, `String` e `HttpResponse`) precisam implementar `Responder`.

# [[Actix-Web/Basics/Exercicios/Manipuladores de Requisição|Exercicios]]
