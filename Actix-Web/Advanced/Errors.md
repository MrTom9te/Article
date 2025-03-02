O Actix Web utiliza seu próprio tipo `actix_web::error::Error` e a trait `actix_web::error::ResponseError` para lidar com erros que ocorrem dentro dos *handlers* (manipuladores de requisição).

**Conceitos Fundamentais:**

*   **`actix_web::error::Error`:**  Este é o tipo de erro "padrão" do Actix Web.  Ele é usado internamente e também serve como um "tipo guarda-chuva" para outros tipos de erro.
*   **`actix_web::error::ResponseError`:**  Esta trait é a *chave* para o tratamento de erros no Actix Web.  Qualquer tipo de erro que implemente `ResponseError` pode ser automaticamente convertido em uma resposta HTTP apropriada.
*   **`std::error::Error`:** A trait geral de erro do Rust. Muitos tipos de erro (inclusive o `actix_web::error::Error`) implementam essa trait.
*   **`Result<T, E>`:**  Handlers que podem falhar devem retornar um `Result`.  Se o `Result` for um `Err`, e o tipo de erro `E` implementar `ResponseError`, o Actix Web automaticamente transforma esse erro em uma resposta HTTP.

**`ResponseError` Trait:**

A trait `ResponseError` define dois métodos principais:

```rust
pub trait ResponseError {
    fn error_response(&self) -> HttpResponse<BoxBody>; // Cria a resposta HTTP
    fn status_code(&self) -> StatusCode; // Retorna o código de status HTTP
}
```

*   **`error_response()`:**  Este método é responsável por *criar* a resposta HTTP que será enviada ao cliente quando um erro ocorrer.  Por padrão, ele retorna um erro 500 Internal Server Error.
*   **`status_code()`:** Este método retorna o código de status HTTP associado ao erro (ex: 400 Bad Request, 404 Not Found, 500 Internal Server Error).  Se você não sobrescrever este método, o padrão é 500.

**`Responder` e `Result`:**

A trait `Responder` (que vimos no artigo anterior) tem uma implementação especial para `Result`:

```rust
impl<T: Responder, E: Into<Error>> Responder for Result<T, E>
```

Isso significa que, se um handler retornar um `Result<T, E>`, onde `T` implementa `Responder` e `E` pode ser convertido em `actix_web::error::Error`, o Actix Web automaticamente:

1.  Se o `Result` for `Ok(t)`, usa o valor `t` para gerar a resposta (como vimos antes).
2.  Se o `Result` for `Err(e)`, converte o erro `e` em `actix_web::error::Error` e, em seguida, usa a implementação de `ResponseError` do erro original para gerar a resposta HTTP de erro.

**Implementações Padrão de `ResponseError`:**

O Actix Web fornece implementações prontas de `ResponseError` para alguns tipos de erro comuns do Rust, como `std::io::Error`.  Isso significa que você pode, muitas vezes, simplesmente propagar erros (usando o operador `?`) sem precisar criar tipos de erro personalizados.

**Exemplo (Propagando `io::Error`):**

```rust
use std::io;
use actix_files::NamedFile;
use actix_web::HttpRequest;

fn index(_req: HttpRequest) -> io::Result<NamedFile> {
    Ok(NamedFile::open("static/index.html")?) // Propaga o erro se o arquivo não existir
}
```

*   `io::Result<NamedFile>`:  O handler retorna um `Result` que pode conter um `io::Error`.
*   `NamedFile::open("static/index.html")?`:  Tenta abrir o arquivo.  Se ocorrer um erro (ex: arquivo não encontrado), o operador `?` retorna o erro *imediatamente* da função `index`.
*   **Conversão Automática:** Como `io::Error` implementa `ResponseError`, o Actix Web automaticamente converte esse erro em um erro 500 Internal Server Error.

**Exemplo de Resposta de Erro Customizada:**

O artigo mostra como criar um tipo de erro personalizado e implementar `ResponseError` para ele, permitindo que você controle totalmente a resposta HTTP de erro.

```rust
use actix_web::{
    error::ResponseError,
    http::{header::ContentType, StatusCode},
    HttpResponse,
};
use derive_more::Display;

// 1. Define o tipo de erro (usando derive_more para gerar implementações)
#[derive(Debug, Display)]
enum MyError {
    #[display(fmt = "Recurso não encontrado")]
    NotFound,
    #[display(fmt = "Erro interno: {}", _0)]
    InternalError(String),
}

// 2. Implementa ResponseError
impl ResponseError for MyError {
    fn status_code(&self) -> StatusCode {
        match *self {
            MyError::NotFound => StatusCode::NOT_FOUND, // 404
            MyError::InternalError(_) => StatusCode::INTERNAL_SERVER_ERROR, // 500
        }
    }

    fn error_response(&self) -> HttpResponse {
        HttpResponse::build(self.status_code())
            .content_type(ContentType::plaintext())
            .body(self.to_string()) // Usa o Display para gerar a mensagem de erro
    }
}

// 3. Handler que pode retornar o erro personalizado
use actix_web::{get, Responder, Result};
#[get("/")]
async fn index() -> Result<String, MyError> {
    // Simula um erro
    Err(MyError::NotFound)
    //Err(MyError::InternalError("Algo deu errado".to_string()))

    // Ou, em caso de sucesso:
    // Ok("Tudo certo!".to_string())
}
```

*   **`#[derive(Debug, Display)]`:**  Usa a biblioteca `derive_more` para gerar automaticamente implementações de `Debug` (útil para logging) e `Display` (para formatar o erro como uma string).  Você precisa adicionar `derive_more = "0.99"` (ou versão compatível) ao seu `Cargo.toml`.
*   **`enum MyError`:** Define um *enum* para representar os diferentes tipos de erro que podem ocorrer na sua aplicação.
*   **`#[display(fmt = ...)]`:**  Define como cada variante do enum deve ser formatada como uma string (usando `derive_more`).
*   **`impl ResponseError for MyError`:**  Implementa a trait `ResponseError`.
    *   **`status_code()`:**  Retorna o código de status HTTP apropriado para cada tipo de erro.
    *   **`error_response()`:**  Constrói a resposta HTTP.  Usa `self.to_string()` para obter a mensagem de erro formatada (graças ao `Display`).
*   **`Result<String, MyError>`:** O handler retorna um `Result` que pode conter uma `String` (em caso de sucesso) ou um `MyError` (em caso de erro).
*   **`Err(MyError::NotFound)`:**  Retorna um erro do tipo `MyError::NotFound`.

**Error Helpers (Funções Auxiliares para Erros):**

O Actix Web fornece funções auxiliares para converter outros tipos de erro em erros HTTP específicos. Isso é útil quando você quer, por exemplo, transformar um erro interno em um erro 400 Bad Request.

**Exemplo (Usando `map_err`):**

```rust
use actix_web::{error, web, HttpResponse, Responder, Result};
use serde::Deserialize;

#[derive(Deserialize)]
struct Info {
    username: String,
}

// Função que simula uma operação que pode falhar (erro interno)
fn do_something(_username: &str) -> Result<(), String> {
    // ... (lógica que pode falhar)
    Err("Erro interno simulado".to_string())
}
async fn index(info: web::Form<Info>) -> Result<impl Responder, actix_web::Error>
{
    do_something(&info.username)
    .map_err(|e| error::ErrorBadRequest(e))?; // Converte String para 400 Bad Request

    Ok(HttpResponse::Ok().body("Sucesso!"))

}
```

*   **`do_something()`:**  Simula uma função que pode retornar um erro (neste caso, uma `String`).
*   **`.map_err(|e| error::ErrorBadRequest(e))`:**  Usa o método `.map_err` do `Result`.  Se `do_something` retornar um `Err`, essa função é chamada para transformar o erro original (a `String`) em um erro `ErrorBadRequest` (que o Actix Web converterá em uma resposta 400).
*   **`?` (após `map_err`):**  Propaga o erro (agora um `ErrorBadRequest`) se ele ocorrer.
* **Nota:** O retorno tem que ser `Result<impl Responder, actix_web::Error>`

**Error Logging (Log de Erros):**

O Actix Web registra automaticamente todos os erros no nível `WARN`. Se o nível de log da aplicação for definido como `DEBUG` e a variável de ambiente `RUST_BACKTRACE` estiver habilitada, o *backtrace* (pilha de chamadas) do erro também será registrado.

*   **`RUST_BACKTRACE=1`:**  Habilita o backtrace.
*   **`RUST_LOG=actix_web=debug`:**  Define o nível de log para `debug` para o módulo `actix_web`.

**Exemplo (Configurando o Log):**

```bash
RUST_BACKTRACE=1 RUST_LOG=actix_web=debug cargo run
```

**Recommended practices in error handling (Práticas Recomendadas para Tratamento de Erros)**

O artigo sugere dividir os erros em duas categorias:

1.  **Erros Destinados ao Usuário:**  São erros que fornecem informações úteis para o usuário final (ex: "Nome de usuário inválido", "Senha incorreta").  As mensagens desses erros devem ser claras e amigáveis.
2.  **Erros Internos:**  São erros que representam problemas internos da aplicação (ex: falha na conexão com o banco de dados, erro de programação).  As mensagens desses erros *não* devem ser expostas diretamente ao usuário, pois podem conter informações sensíveis ou irrelevantes.

**Exemplo (Separando Erros de Usuário e Internos):**

```rust
use actix_web::{
    error::ResponseError,
    http::{header::ContentType, StatusCode},
    HttpResponse,
};
use derive_more::{Display, Error};

// Erros de usuário (com mensagens amigáveis)
#[derive(Debug, Display, Error)]
enum UserError {
    #[display(fmt = "Nome de usuário inválido")]
    InvalidUsername,
    #[display(fmt = "Senha incorreta")]
    IncorrectPassword,
    #[display(fmt = "O campo {} é obrigatório.", _0)]
    MissingField(String),
}

// Erros internos (com mensagens genéricas)
#[derive(Debug, Display, Error)]
enum InternalError {
    #[display(fmt = "Erro interno do servidor")]
    DatabaseError,
    #[display(fmt = "Erro interno do servidor")]
    TemplateError,
}
// Implemente ResponseError para ambos os tipos
impl ResponseError for UserError {
     fn status_code(&self) -> StatusCode {
        match *self {
            UserError::InvalidUsername => StatusCode::BAD_REQUEST,
            UserError::IncorrectPassword => StatusCode::UNAUTHORIZED,
            UserError::MissingField(_)  => StatusCode::BAD_REQUEST,
        }
    }
    fn error_response(&self) -> HttpResponse {
          HttpResponse::build(self.status_code())
            .content_type(ContentType::plaintext())
            .body(self.to_string())
    }
}
impl ResponseError for InternalError {
     fn status_code(&self) -> StatusCode {
        match *self {
           InternalError::DatabaseError => StatusCode::INTERNAL_SERVER_ERROR,
           InternalError::TemplateError  => StatusCode::INTERNAL_SERVER_ERROR,
        }
    }
    fn error_response(&self) -> HttpResponse {
           HttpResponse::build(self.status_code())
            .content_type(ContentType::plaintext())
            .body(self.to_string())
    }
}
// Exemplo de handler que pode retornar diferentes tipos de erro
use actix_web::{get, web, Responder, Result};

#[get("/login")]
async fn login(info: web::Query<String>) -> Result<String, UserError> {
   // Simulando validações
    if info.as_ref().unwrap_or(&String::new()).is_empty()
     {
       return Err(UserError::MissingField("username".to_string()));
    }

    if info.as_ref().unwrap_or(&String::new()) == "admin" {
       return Err(UserError::InvalidUsername);
    }

    if info.as_ref().unwrap_or(&String::new()) == "senha"{
        return Err(UserError::IncorrectPassword)
    }
        Ok(format!("Login bem-sucedido!"))
}

#[get("/db")] //Simula acesso a DB
async fn acessoDB() -> Result<String, InternalError> {
   //simula erro
   Err(InternalError::DatabaseError)

}
```

*   **`UserError`:**  Um enum para erros que podem ser mostrados ao usuário.
*   **`InternalError`:** Um enum para erros internos.  A mensagem "Erro interno do servidor" é genérica e não revela detalhes sensíveis.
*   **Handlers:**  Os handlers podem retornar `Result<..., UserError>` ou `Result<..., InternalError>`, dependendo do tipo de erro que pode ocorrer.  Se um handler puder retornar *ambos* os tipos de erro, você pode usar `Either` ou criar um enum que englobe ambos.

Este artigo aborda um tópico crucial no desenvolvimento de aplicações web: o tratamento de erros.  O Actix Web oferece um sistema flexível e poderoso para lidar com erros de forma organizada, segura e eficiente.
agora os exercicios 