# Artigo: Construindo Respostas HTTP no Actix-Web (Rust)

## Introdução

Em um framework web como o Actix-Web, construir respostas HTTP de forma correta e eficiente é crucial. Este artigo aborda como criar respostas no Actix-Web, incluindo a serialização de dados para JSON e o uso de compressão de conteúdo.

## Padrão Builder

O Actix-Web usa um padrão *builder* para construir respostas. Isso significa que você encadeia chamadas de métodos para configurar a resposta, e no final, chama um método para "finalizar" a construção.

```rust
use actix_web::{HttpResponse, http::StatusCode};

let response = HttpResponse::Ok() // Inicia a construção (status 200 OK)
    .content_type("text/plain; charset=utf-8") // Define o tipo de conteúdo
    .body("Olá, mundo!"); // Define o corpo da resposta

```

**Explicação:**

-   `HttpResponse::Ok()`:  Cria um `HttpResponseBuilder` com o código de status 200 (OK).  Existem outros métodos como `HttpResponse::Created()`, `HttpResponse::NotFound()`, etc., para diferentes códigos de status.
-   `.content_type(...)`:  Define o cabeçalho `Content-Type`.
-   `.body(...)`:  Define o corpo da resposta.

**Métodos Importantes do `HttpResponseBuilder`:**

-   `status(code: StatusCode)`: Define o código de status (se não usar um dos métodos como `HttpResponse::Ok()`).
-   `content_type(content_type: &str)`: Define o cabeçalho `Content-Type`.
-   `insert_header(...)`: Adiciona um cabeçalho customizado.
-   `body(body: impl Into<Body>)`: Define o corpo da resposta.  `Body` é um enum que pode ser uma string, bytes, etc.
-   `finish()`: Finaliza a construção e retorna um `HttpResponse`.  Útil quando você não precisa de um corpo.
-   `json(data: impl Serialize)`: Serializa `data` para JSON e define o `Content-Type` para `application/json`.

**Importante:**  Os métodos `.body()`, `.finish()`, e `.json()` finalizam a construção.  Se você chamá-los mais de uma vez no mesmo builder, o programa entrará em pânico (panic).

## Respostas JSON (JSON Response)

Para retornar dados JSON, a maneira mais clara é usar o tipo `web::Json`.

1.  **Defina a Estrutura:**  Crie uma struct Rust que represente os dados que você quer serializar para JSON. Use `#[derive(Serialize)]`.

    ```rust
    #[derive(serde::Serialize)]
    struct MyResponse {
        message: String,
        value: i32,
    }
    ```

2.  **Retorne `Json<T>`:**  No seu handler, retorne um valor do tipo `web::Json<MyResponse>`.

    ```rust
    use actix_web::{web, App, HttpResponse, HttpServer, Responder};

    #[derive(serde::Serialize)]
    struct MyResponse {
        message: String,
        value: i32,
    }

    async fn get_data() -> impl Responder {
        let data = MyResponse {
            message: "Dados aqui!".to_string(),
            value: 42,
        };
        web::Json(data)
    }
    ```

3. **Configure a rota normalmente.**

    ```rust
    #[actix_web::main]
    async fn main() -> std::io::Result<()> {
       HttpServer::new(|| {
           App::new()
              .route("/data", web::get().to(get_data))
       })
       .bind(("127.0.0.1", 8080))?
       .run()
       .await
    }
    ```

**Exemplo Completo:**

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

#[derive(serde::Serialize)]
struct MyResponse {
    message: String,
    value: i32,
}

async fn get_data() -> impl Responder {
    let data = MyResponse {
        message: "Dados aqui!".to_string(),
        value: 42,
    };
    web::Json(data)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
   HttpServer::new(|| {
       App::new()
          .route("/data", web::get().to(get_data))
   })
   .bind(("127.0.0.1", 8080))?
   .run()
   .await
}
```

**Dependências (Cargo.toml):**

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
actix-web = "4"  # (ou a versão mais recente)
```
Usar `web::Json` desta forma deixa claro que a função retorna JSON, em vez de outro tipo de resposta.

**Alternativa (Menos Clara):**

Você *poderia* usar `.json()` no `HttpResponseBuilder`, mas é menos explícito:

```rust
// Menos recomendado
async fn get_data_alt() -> HttpResponse {
  let data = MyResponse {
    message: "Dados aqui!".to_string(),
    value: 42,
  };
  HttpResponse::Ok().json(data) // Funciona, mas é menos claro
}
```

## Compressão de Conteúdo (Content Encoding)

O Actix-Web pode comprimir automaticamente o corpo da resposta para economizar largura de banda.  Isso é feito através do middleware `Compress`.

-   **Codecs Suportados:** Brotli, Gzip, Deflate, Identity (sem compressão).
-   **Negociação Automática:** Por padrão (`ContentEncoding::Auto`), o Actix-Web negocia a compressão com o cliente, com base no cabeçalho `Accept-Encoding` da requisição.
- Para usar o middleware de compressão, você precisa usar o método wrap na sua app:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, middleware};

#[derive(serde::Serialize)]
struct MyResponse {
    message: String,
}

async fn get_compressed_data() -> impl Responder {
   let data = MyResponse {
       message: "Esta resposta será comprimida (se o cliente suportar)!".to_string(),
    };
   web::Json(data)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
      App::new()
          .wrap(middleware::Compress::default()) // Habilita a compressão
          .route("/compressed", web::get().to(get_compressed_data))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Desabilitando a Compressão:**

Para desabilitar explicitamente a compressão para um handler específico, defina `ContentEncoding::Identity`:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, middleware};
use actix_web::http::header::ContentEncoding;

#[derive(serde::Serialize)]
struct MyResponse {
    message: String,
}

async fn no_compression() -> impl Responder {
  let data = MyResponse {
      message: "Esta resposta NÃO será comprimida.".to_string(),
    };

  web::Json(data)
      .with_header(ContentEncoding::Identity)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
  HttpServer::new(|| {
      App::new()
          .wrap(middleware::Compress::default())
          .route("/nocompress", web::get().to(no_compression))
  })
  .bind(("127.0.0.1", 8080))?
  .run()
  .await
}
```

**Corpos Já Comprimidos:**

Se você estiver servindo conteúdo que já está comprimido (por exemplo, arquivos estáticos pré-comprimidos), defina manualmente o cabeçalho `Content-Encoding` para evitar compressão dupla:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, middleware};
use actix_web::http::header::ContentEncoding;

async fn precompressed() -> HttpResponse {
    HttpResponse::Ok()
      .insert_header(ContentEncoding::Gzip) // Informa que já está comprimido com gzip
      .body(include_bytes!("../static/meu_arquivo.gz")) // Inclui o arquivo binário
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
  HttpServer::new(|| {
      App::new()
        //  .wrap(middleware::Compress::default()) // Remova o middleware de compressão neste caso!
          .route("/precompressed", web::get().to(precompressed))
  })
  .bind(("127.0.0.1", 8080))?
  .run()
  .await
}
```

## Conclusão

O Actix-Web oferece um sistema flexível e eficiente para construir respostas HTTP.  O padrão builder, o tipo `Json`, e o middleware `Compress` tornam fácil criar APIs que retornam dados corretamente formatados e otimizados para a rede.  Lembre-se de usar `#[derive(Serialize)]` para seus tipos de dados JSON e de habilitar o middleware `Compress` para melhorar o desempenho.