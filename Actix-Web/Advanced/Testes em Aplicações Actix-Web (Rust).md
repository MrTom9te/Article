## Introdução

Testes são essenciais para garantir a qualidade de qualquer aplicação. O Actix-Web fornece ferramentas para realizar testes de integração (testando a aplicação como um todo) e testes unitários (testando partes específicas, como extractors e middlewares).

## Testes de Integração (Integration Testing)

Testes de integração verificam se diferentes partes da sua aplicação funcionam juntas corretamente. No Actix-Web, você pode iniciar um servidor de teste real e enviar requisições para ele.

### Usando `test::init_service` e `TestRequest`

1.  **`test::init_service`:**  Esta função cria um `Service` (um objeto que representa sua aplicação) que pode ser usado para testes.  Você passa uma configuração `App` normal para ela.

2.  **`TestRequest`:**  Esta struct permite construir requisições de teste (GET, POST, etc.) e enviá-las para o servidor de teste.

**Exemplo Básico:**

```rust
#[cfg(test)]
mod tests {
    use actix_web::{test, web, App, HttpResponse};

    async fn hello() -> HttpResponse {
        HttpResponse::Ok().body("Hello, world!")
    }

    #[actix_web::test]
    async fn test_hello() {
        let app = test::init_service(App::new().route("/", web::get().to(hello))).await;
        let req = test::TestRequest::get().uri("/").to_request(); //Cria a requisição
        let resp = test::call_service(&app, req).await; //Envia para a aplicação
        assert!(resp.status().is_success()); //Verifica o status da resposta

        // Ler o corpo da resposta como bytes e converter para String
        let bytes = test::read_body(resp).await;
        let body = String::from_utf8(bytes.to_vec()).unwrap();
        assert_eq!(body, "Hello, world!");
    }
}
```

**Explicação:**

-   `#[cfg(test)]`:  Este atributo indica que o módulo `tests` só será compilado quando você executar testes (`cargo test`).
-   `#[actix_web::test]`:  Uma macro que configura o ambiente de teste do Actix-Web.  Substitui `#[tokio::test]`.
-   `test::init_service(...)`:  Cria o `Service` de teste.
-   `TestRequest::get().uri("/").to_request()`: Cria uma requisição GET para a rota `/`.
    -   `TestRequest::get()`, `TestRequest::post()`, etc.:  Métodos para criar requisições de diferentes tipos.
    -   `.uri(...)`:  Define a URI da requisição.
    -   `.to_request()`:  Constrói a `HttpRequest` final.
-   `test::call_service(&app, req).await`: Envia a requisição para a aplicação e retorna a resposta.
-   `resp.status().is_success()`: Verifica se o código de status da resposta indica sucesso (2xx).
-  `test::read_body(resp).await`: Extrai o corpo da resposta

### Configuração Complexa

Se sua aplicação tiver uma configuração mais complexa (estado, middlewares, etc.), o teste será semelhante à configuração normal.

```rust
#[cfg(test)]
mod tests {
    use actix_web::{test, web, App, HttpResponse};
    use std::sync::Mutex;

    // Exemplo de estado da aplicação
    struct AppState {
        counter: Mutex<i32>,
    }

    async fn get_count(data: web::Data<AppState>) -> HttpResponse {
        let mut counter = data.counter.lock().unwrap();
        *counter += 1;
        HttpResponse::Ok().body(format!("Contador: {}", *counter))
    }

    #[actix_web::test]
    async fn test_get_count() {
        let app_state = web::Data::new(AppState {
            counter: Mutex::new(0),
        });

        let app = test::init_service(
            App::new()
                .app_data(app_state.clone()) // Adiciona o estado
                .route("/", web::get().to(get_count)),
        )
        .await;

        let req = test::TestRequest::get().uri("/").to_request();
        let resp = test::call_service(&app, req).await;
        assert!(resp.status().is_success());
        let bytes = test::read_body(resp).await;
        let body = String::from_utf8(bytes.to_vec()).unwrap();

        assert_eq!(body, "Contador: 1");


        //Testa novamente, para verificar se o contador foi incrementado
         let req = test::TestRequest::get().uri("/").to_request();
        let resp = test::call_service(&app, req).await;
        assert!(resp.status().is_success());
        let bytes = test::read_body(resp).await;
        let body = String::from_utf8(bytes.to_vec()).unwrap();

        assert_eq!(body, "Contador: 2");
    }
}
```
- O estado `AppState` é criado e passado para a aplicação usando `.app_data()`.
-  Duas requisições são enviadas para verificar se o contador do estado está funcionando corretamente.

### Testando Respostas de Stream (Stream Response Testing)

Para testar handlers que retornam streams (como Server-Sent Events), você pode usar `into_parts()` para obter o corpo da resposta como um stream e, em seguida, processá-lo.

```rust
#[cfg(test)]
mod tests {
   use actix_web::{web, App, test,rt};
    use futures::StreamExt;

    async fn stream_handler() -> actix_web::HttpResponse {
      let body = futures::stream::once(async { Ok::<_, actix_web::Error>(web::Bytes::from_static(b"teste")) });

      actix_web::HttpResponse::Ok()
          .content_type("text/plain; charset=utf-8")
          .streaming(body)
    }

    #[actix_web::test]
    async fn test_stream() {
      let app = test::init_service(App::new().route("/", web::get().to(stream_handler))).await;

      let req = test::TestRequest::get().uri("/").to_request();
      let resp = test::call_service(&app, req).await;
      assert!(resp.status().is_success());

      // Extrai o corpo como um stream
      let (res,mut body) = resp.into_parts();

      // Coleta os bytes do stream
      let mut bytes = web::BytesMut::new();
      while let Some(item) = body.next().await {
          bytes.extend_from_slice(&item.unwrap());
      }

      assert_eq!(bytes, web::Bytes::from_static(b"teste"));
    }
}
```

**Explicação:**

- `resp.into_parts()`: Divide a resposta em suas partes (cabeçalhos e corpo).
-  O corpo (`body`) é um `Stream` (do crate `futures`).
-  `while let Some(item) = body.next().await`: Itera sobre os itens do stream (os pedaços de bytes).
-  `bytes.extend_from_slice(&item.unwrap())`: Adiciona os bytes ao buffer `BytesMut`.
-   `assert_eq!(bytes, ...)`: Verifica se os bytes recebidos são os esperados.

## Testes Unitários de Extractors (Unit Testing Extractors)

Testes unitários são mais úteis para testar partes isoladas do seu código, como extractors, middlewares e `Responder`s customizados.  Você pode chamar diretamente as funções handler (sem usar as macros de roteamento como `#[get("/")]`) para testar essas partes.

```rust
// src/my_extractor.rs
use actix_web::{FromRequest, HttpRequest, dev, error, web};
use futures::future::{Ready, ok, err};

pub struct MyExtractor(pub String);

impl FromRequest for MyExtractor {
    type Error = actix_web::Error;
    type Future = Ready<Result<Self, Self::Error>>;
    // type Config = (); //Configuração removida, pois não está sendo utilizada

     fn from_request(req: &HttpRequest, _payload: &mut dev::Payload) -> Self::Future {
        //Exemplo simples: extrai um header customizado
        if let Some(value) = req.headers().get("X-My-Header") {
            ok(MyExtractor(value.to_str().unwrap().to_string()))
        } else {
            err(error::ErrorBadRequest("Faltando o header X-My-Header"))
        }
    }
}

// src/main.rs
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
mod my_extractor;
use my_extractor::MyExtractor;

async fn handler_with_extractor(MyExtractor(value): MyExtractor) -> impl Responder {
   HttpResponse::Ok().body(format!("Valor do header: {}", value))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
           .route("/", web::get().to(handler_with_extractor))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::{test, web, App, http};

    #[actix_web::test]
    async fn test_my_extractor() {
        let mut app = test::init_service(App::new().route("/", web::get().to(handler_with_extractor))).await;

        // Caso de sucesso
        let req = test::TestRequest::get()
            .uri("/")
            .insert_header(("X-My-Header", "valor_teste")) //Insere o header na requisição
            .to_request();

        let resp = test::call_service(&app, req).await;
        assert_eq!(resp.status(), http::StatusCode::OK);
        let bytes = test::read_body(resp).await;
        let body_str = String::from_utf8(bytes.to_vec()).unwrap();
        assert_eq!(body_str, "Valor do header: valor_teste");

        // Caso de erro (header faltando)
        let req = test::TestRequest::get().uri("/").to_request();
        let resp = test::call_service(&app, req).await;

        assert_eq!(resp.status(), http::StatusCode::BAD_REQUEST); //Espera um erro 400
    }
}
```

**Explicação:**

-   `MyExtractor`:  Um extractor customizado que lê um cabeçalho HTTP chamado "X-My-Header".
-   `impl FromRequest for MyExtractor`:  Implementa a trait `FromRequest`, que permite que `MyExtractor` seja usado como um extractor.
    -   `from_request(...)`:  A lógica para extrair o valor do cabeçalho.
- `handler_with_extractor`: Um handler que usa `MyExtractor`.
-   `test_my_extractor()`:
    -   Cria uma requisição de teste *com* o cabeçalho e verifica se o handler retorna o valor correto.
    -   Cria uma requisição de teste *sem* o cabeçalho e verifica se o handler retorna um erro 400 (Bad Request).

## Conclusão

O Actix-Web oferece um conjunto completo de ferramentas para testar suas aplicações, desde testes de integração que simulam o comportamento completo do servidor até testes unitários que focam em componentes específicos.  Escrever testes é fundamental para garantir que sua aplicação funcione como esperado e para facilitar a refatoração e a manutenção do código.
