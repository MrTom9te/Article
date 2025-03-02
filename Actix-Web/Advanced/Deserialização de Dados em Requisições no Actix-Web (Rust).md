## Introdução

Ao construir APIs web, frequentemente precisamos receber dados do cliente (frontend, outro serviço, etc.). Estes dados podem estar em vários formatos, como JSON, formulários HTML, ou até mesmo streams binários. O Actix-Web oferece ferramentas para lidar com esses diferentes formatos de forma eficiente e segura. Este artigo foca nas opções de desserialização de dados, ou seja, como transformar os dados brutos da requisição em estruturas de dados que seu código Rust possa usar.

## JSON Request

JSON (JavaScript Object Notation) é um formato de dados muito comum em APIs web. O Actix-Web facilita o trabalho com JSON.

### Usando o Extrator `Json`

A maneira mais simples de lidar com JSON é usar o extrator `web::Json`.

1.  **Defina a Estrutura:** Crie uma struct Rust que represente os dados JSON que você espera receber.  Use as anotações `#[derive(Serialize, Deserialize)]` do Serde.

    ```rust
    #[derive(serde::Serialize, serde::Deserialize)]
    struct MyInfo {
        name: String,
        age: u32,
    }
    ```

2.  **Crie o Handler:** Escreva uma função handler que aceite `web::Json<MyInfo>` como parâmetro.

    ```rust
    use actix_web::{web, App, HttpResponse, HttpServer, Responder};

    #[derive(serde::Serialize, serde::Deserialize)]
    struct MyInfo {
        name: String,
        age: u32,
    }

    async fn receive_info(info: web::Json<MyInfo>) -> impl Responder {
        format!("Olá, {}! Você tem {} anos.", info.name, info.age)
    }
    ```
   - O actix-web extrai o corpo da requisição, desserializa ele para o formato Json e passa esse objeto para o handler.

3.  **Registre o Handler:** Use `.route()` e `.to()` para registrar o handler.

    ```rust
    #[actix_web::main]
    async fn main() -> std::io::Result<()> {
        HttpServer::new(|| {
            App::new()
                .route("/info", web::post().to(receive_info))
        })
        .bind(("127.0.0.1", 8080))?
        .run()
        .await
    }
    ```

**Exemplo Completo:**

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

#[derive(serde::Serialize, serde::Deserialize)]
struct MyInfo {
    name: String,
    age: u32,
}

async fn receive_info(info: web::Json<MyInfo>) -> impl Responder {
    format!("Olá, {}! Você tem {} anos.", info.name, info.age)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/info", web::post().to(receive_info))
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
actix-web = "4" # (ou a versão mais recente)
```

**Usando `serde_json::Value`:**

Se você não sabe a estrutura exata do JSON com antecedência, ou se ele for muito complexo, use `serde_json::Value`.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde_json::Value;

async fn receive_anything(data: web::Json<Value>) -> impl Responder {
  //  format!("Recebi: {:?}", data) //Não recomendado para produção
  let meu_valor = data.get("meu_campo").unwrap();
  format!("Meu valor é {}",meu_valor)
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/anything", web::post().to(receive_anything))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```
**Dependências (Cargo.toml - exemplo com `serde_json::Value`):**
```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1"
actix-web = "4"
```

### Carregamento Manual (Manual Payload Loading)

Em alguns casos, você pode querer mais controle sobre o processo de desserialização.  Você pode carregar manualmente o corpo da requisição e, em seguida, desserializá-lo.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Error};
use futures::StreamExt;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct MyObj {
    name: String,
    number: i32,
}

const MAX_SIZE: usize = 262_144; // 256 KB

async fn index_manual(mut payload: web::Payload) -> Result<HttpResponse, Error> {
    // payload é um stream de bytes
    let mut body = web::BytesMut::new();
    while let Some(chunk) = payload.next().await {
        let chunk = chunk?;
        // Limita o tamanho do corpo
        if (body.len() + chunk.len()) > MAX_SIZE {
            return Err(actix_web::error::ErrorBadRequest("Corpo muito grande"));
        }
        body.extend_from_slice(&chunk);
    }

    // Desserializa o corpo
    let obj = serde_json::from_slice::<MyObj>(&body)?;
    Ok(HttpResponse::Ok().json(obj)) // Serializa e retorna como JSON
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().route("/manual", web::post().to(index_manual))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

**Explicação:**

4.  **`web::Payload`:**  Representa o corpo da requisição como um stream de bytes.
5.  **`BytesMut`:**  Um buffer mutável para bytes.
6.  **`while let Some(chunk) = payload.next().await`:**  Itera sobre os pedaços (chunks) do corpo da requisição.
7.  **`serde_json::from_slice::<MyObj>(&body)`:** Desserializa os bytes (armazenados em `body`) para uma instância de `MyObj`.
8. **`HttpResponse::Ok().json(obj)`:** Serializa o objeto `obj` de volta para JSON e retorna como resposta.

**Dependências (Cargo.toml - exemplo manual):**

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1"
futures = "0.3"
actix-web = "4"
```

## Descompressão Automática (Content Encoding)

O Actix-Web descomprime automaticamente payloads compactados (gzip, deflate, brotli, zstd) se o cabeçalho `Content-Encoding` estiver presente.  Você não precisa fazer nada especial para isso. *Importante:* ele não suporta múltiplos codecs (ex: `Content-Encoding: br, gzip`).

## Chunked Transfer Encoding

O Actix-Web também lida automaticamente com *chunked transfer encoding*.  O extrator `web::Payload` já contém o stream de bytes decodificado. Se o payload também estiver compactado, o stream será descomprimido.

## Multipart Body

Para lidar com requisições multipart (geralmente usadas para upload de arquivos), use a crate `actix-multipart`.  Consulte a documentação da crate para detalhes.

## Urlencoded Body (Formulários)

Para dados de formulários HTML (`application/x-www-form-urlencoded`), use o extrator `web::Form`.

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Error, Result};
use serde::Deserialize;

#[derive(Deserialize)]
struct FormData {
    username: String,
    email: String,
}

async fn handle_form(form: web::Form<FormData>) -> Result<HttpResponse, Error> {
  Ok(HttpResponse::Ok().body(format!("Usuário: {}, Email: {}", form.username,form.email)))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {

    HttpServer::new(|| {
        App::new()
            .route("/form", web::post().to(handle_form))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

9.  **`#[derive(Deserialize)]`:**  `FormData` precisa implementar `Deserialize`.
10.  **`web::Form<FormData>`:**  O extrator desserializa os dados do formulário para a struct `FormData`.

**Dependências (Cargo.toml):**

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
actix-web = "4"
```

**Erros Comuns com `web::Form`:**

-   Tipo de conteúdo incorreto (não é `application/x-www-form-urlencoded`).
-   Transfer encoding é `chunked` (não suportado com `web::Form`).
-   Tamanho do conteúdo excede 256KB (limite padrão).
-   Erro no payload.

## Streaming Request

`HttpRequest` é um stream de `Bytes`.  Você pode usá-lo para ler o corpo da requisição de forma incremental.

```rust
use actix_web::{web, App, HttpRequest, HttpResponse, HttpServer, Error, Result};
use futures::StreamExt;

async fn handle_stream(mut req: HttpRequest) -> Result<HttpResponse, Error> {
//async fn handle_stream(mut payload: web::Payload) -> Result<HttpResponse, Error> { //Outra opção
    let mut body = web::BytesMut::new();
    while let Some(chunk) = req.payload().next().await {//Percorre os bytes da requisição
        let chunk = chunk?;
        println!("Chunk: {:?}", chunk); // Exibe cada pedaço
        body.extend_from_slice(&chunk);
    }

    Ok(HttpResponse::Ok().body(body)) // Retorna o corpo completo (apenas para demonstração)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {

    HttpServer::new(|| {
        App::new()
            .route("/stream", web::post().to(handle_stream))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**

11.  **`req.payload().next().await`:**  Obtém o próximo pedaço (chunk) do corpo da requisição.
12.  `body.extend_from_slice(&chunk);`: Adiciona o pedaço ao buffer `BytesMut`.

**Dependências (Cargo.toml - exemplo streaming):**

```toml
[dependencies]
futures = "0.3"
actix-web = "4"
```

## Conclusão

O Actix-Web fornece várias maneiras flexíveis de lidar com dados de requisições, desde a simples desserialização de JSON e formulários até o processamento de streams de bytes. Escolher a abordagem correta depende das necessidades da sua aplicação e do formato dos dados recebidos. Lembre-se sempre de tratar erros e validar os dados recebidos para garantir a segurança e a robustez da sua API.