# O Servidor HTTP

Este artigo explica o componente central do Actix Web: o `HttpServer`. Ele é responsável por "servir" (lidar com) requisições HTTP. Ou seja, receber as requisições que chegam, processá-las e enviar as respostas.

**Conceitos-Chave:**

*   **Application Factory (Fábrica de Aplicação):**  O `HttpServer` não recebe diretamente a sua aplicação (`App`). Em vez disso, ele recebe uma *fábrica de aplicação*. Uma fábrica é uma função (geralmente uma closure) que *cria* uma instância da sua aplicação. Isso é importante por causa do multi-threading (que veremos em detalhes mais adiante).  A fábrica precisa ter as traits `Send` + `Sync` (explicarei isso também).

*   **Binding (Vinculação):** Antes de começar a receber requisições, o servidor precisa ser "vinculado" (bound) a um endereço de rede e porta.  É como dizer ao servidor: "Escute requisições que chegarem neste endereço IP e nesta porta".  Isso é feito com o método `HttpServer::bind()`.  Se a porta já estiver em uso por outro programa, o `bind` falhará.

*   **`Server` Instance (Instância do Servidor):**  Depois de vincular com sucesso, você chama `HttpServer::run()` para obter uma instância de `Server`.  Essa instância representa o servidor em execução. Para que o servidor realmente comece a funcionar, você precisa usar `.await` (se estiver dentro de uma função assíncrona) ou `spawn` (para executá-lo em uma thread separada). O servidor continuará rodando até receber um sinal de desligamento (como Ctrl+C).

**Exemplo Básico (Completo):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Olá, mundo!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| { // 1. Fábrica de aplicação
        App::new()
            .route("/", web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))? // 2. Vincula ao endereço e porta
    .run() // 3. Obtém a instância do servidor
    .await // 4. Inicia o servidor (e espera)
}
```

1.  **`HttpServer::new(|| { ... })`:**  Cria o servidor HTTP, recebendo a fábrica de aplicação (a closure).
2.  **`.bind(("127.0.0.1", 8080))?`:**  Vincula o servidor ao endereço `127.0.0.1` (localhost) e à porta `8080`. O `?` lida com erros (se a porta estiver ocupada).
3.  **`.run()`:**  Retorna a instância `Server`.
4.  **`.await`:**  Inicia o servidor e *aguarda* até que ele seja finalizado.

**Multi-Threading (Múltiplas Threads):**

Um dos pontos fortes do Actix Web é sua capacidade de lidar com muitas requisições simultaneamente usando múltiplas threads.  O `HttpServer` faz isso automaticamente:

*   **Workers (Trabalhadores):**  O `HttpServer` inicia um número de "workers" (threads de trabalho).  Por padrão, esse número é igual ao número de núcleos *físicos* da sua CPU. Você pode controlar esse número com o método `HttpServer::workers()`.

*   **Application Instances (Instâncias da Aplicação):**  Cada worker recebe sua *própria* instância da aplicação.  É por isso que usamos uma *fábrica* de aplicação – o `HttpServer` chama a fábrica *várias vezes*, uma vez para cada worker.

*   **State (Estado) e Concorrência:**
    *   O estado da aplicação *não* é compartilhado automaticamente entre as threads. Cada worker tem sua própria cópia. Isso significa que você pode modificar o estado dentro de um handler sem se preocupar com problemas de concorrência (race conditions), desde que o estado não precise ser compartilhado.
    *   As *fábricas* de aplicação, por outro lado, *precisam* ser `Send` + `Sync`. Isso significa que elas podem ser enviadas entre threads (`Send`) e acessadas de forma segura por múltiplas threads simultaneamente (`Sync`).  Closures geralmente são `Send` + `Sync` automaticamente, a menos que capturem variáveis que não sejam.
    *   Se você *precisa* de estado compartilhado, use tipos como `Arc` (Atomic Reference Counted) e mecanismos de sincronização como `Mutex` ou `RwLock` (Read-Write Lock). Mas o artigo alerta que o uso excessivo de locks pode prejudicar o desempenho.  É melhor evitar locking sempre que possível.

**Exemplo (Controlando o Número de Workers):**

```rust
// ... (resto do código)

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
    })
    .workers(4) // Define 4 workers (threads)
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Handlers Síncronos vs. Assíncronos:**

O artigo enfatiza um ponto *crucial*:

*   **Handlers Síncronos (Bloqueantes):** Se um handler (função que lida com uma requisição) fizer operações que bloqueiam a thread (ex: `std::thread::sleep`, operações de I/O síncronas), o worker *inteiro* ficará bloqueado. Isso significa que ele não poderá processar outras requisições enquanto estiver esperando. Isso é *muito ruim* para o desempenho.

*   **Handlers Assíncronos (Não Bloqueantes):**  Handlers assíncronos (definidos com `async fn`) usam *futures*. Eles podem "pausar" sua execução enquanto esperam por operações de I/O (como ler de um arquivo ou fazer uma requisição de rede), permitindo que o worker processe outras requisições nesse meio tempo.  Isso é *essencial* para manter a alta performance do servidor.

**Exemplo (Handler Síncrono - RUIM):**

```rust
use actix_web::{Responder, HttpResponse};
use std::time::Duration;

fn bad_handler() -> impl Responder {
    std::thread::sleep(Duration::from_secs(5)); // Bloqueia a thread por 5 segundos!
    HttpResponse::Ok().body("Resposta (depois de muita espera)")
}
```

**Exemplo (Handler Assíncrono - BOM):**

```rust
use actix_web::{Responder, HttpResponse};
use tokio::time::sleep; // Use a versão assíncrona do sleep
use std::time::Duration;

async fn good_handler() -> impl Responder {
    sleep(Duration::from_secs(5)).await; // Espera 5 segundos, mas NÃO bloqueia a thread
    HttpResponse::Ok().body("Resposta (sem bloquear)")
}
```

**Extractors (Extratores):**

O artigo também menciona que os *extractors* (tipos que extraem dados da requisição, como `web::Path`, `web::Json`, `web::Data`, etc.) também podem ser síncronos ou assíncronos. Se um extractor bloquear a thread, ele também causará problemas de desempenho. Portanto, é importante usar extractors assíncronos quando necessário.

**TLS / HTTPS:**

O Actix Web suporta conexões seguras (HTTPS) usando TLS. Ele oferece suporte integrado a duas bibliotecas TLS:

*   **`rustls`:** Uma biblioteca TLS moderna escrita em Rust.
*   **`openssl`:** Uma biblioteca TLS amplamente usada (escrita em C).

Para usar TLS, você precisa:

1.  Adicionar a dependência apropriada no seu `Cargo.toml` (ex: `actix-web = { version = "4", features = ["openssl"] }`).
2.  Criar um certificado e uma chave privada (o artigo mostra como fazer isso com o comando `openssl`).
3.  Configurar o `HttpServer` para usar TLS.

**Exemplo (Usando OpenSSL):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use openssl::ssl::{SslAcceptor, SslFiletype, SslMethod};

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Olá, mundo (com HTTPS)!")
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 1. Configura o OpenSSL
    let mut builder = SslAcceptor::mozilla_intermediate(SslMethod::tls())?;
    builder.set_private_key_file("key.pem", SslFiletype::PEM)?; // Caminho para a chave
    builder.set_certificate_chain_file("cert.pem")?; // Caminho para o certificado

    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
    })
    .bind_openssl("127.0.0.1:8443", builder)? // Usa bind_openssl em vez de bind
    .run()
    .await
}
```
*   **`bind_openssl`:**  Em vez de `bind`, você usa `bind_openssl` para configurar o servidor para usar TLS.
* A url de acesso será: `https://127.0.0.1:8443/`
**Keep-Alive:**

Keep-alive é um mecanismo que permite que uma conexão TCP entre o cliente (ex: navegador) e o servidor permaneça aberta por um tempo, mesmo depois que uma requisição é respondida. Isso evita a necessidade de abrir uma nova conexão para cada requisição, melhorando o desempenho.

O Actix Web suporta keep-alive e permite configurá-lo:

*   **`Duration::from_secs(75)` ou `KeepAlive::Timeout(75)`:**  Mantém a conexão aberta por 75 segundos.
*   **`KeepAlive::Os`:**  Usa as configurações de keep-alive do sistema operacional.
*   **`None` ou `KeepAlive::Disabled`:**  Desativa o keep-alive.

Por padrão:

*   **HTTP/1.0:** Keep-alive é desativado.
*   **HTTP/1.1 e HTTP/2.0:** Keep-alive é ativado.

**Exemplo (Configurando Keep-Alive):**

```rust
// ... (resto do código)

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
    })
    .keep_alive(actix_web::http::KeepAlive::Timeout(60)) // Keep-alive de 60 segundos
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Graceful Shutdown (Desligamento Gracioso):**

O `HttpServer` suporta "desligamento gracioso". Isso significa que, quando você pede para o servidor parar (ex: pressionando Ctrl+C), ele não interrompe as requisições em andamento imediatamente. Em vez disso, ele dá aos workers um tempo para terminar de processar as requisições atuais antes de fechar.

*   **Timeout:**  O tempo padrão para o desligamento gracioso é de 30 segundos. Você pode configurar isso com `HttpServer::shutdown_timeout()`.
*   **Sinais:**  O `HttpServer` lida com sinais do sistema operacional (como `SIGINT`, `SIGTERM`, `SIGQUIT` em sistemas Unix). Esses sinais são usados para iniciar o desligamento.
*   **`HttpServer::disable_signals()`:**  Você pode desativar o tratamento de sinais se precisar.

**Exemplo (Configurando o Timeout de Desligamento):**

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))?
    .shutdown_timeout(10) // Define o timeout de desligamento para 10 segundos
    .run()
    .await
}
```

Este artigo cobre os aspectos mais importantes do `HttpServer` do Actix Web. É um componente fundamental, então entender bem como ele funciona é essencial para construir aplicações web robustas e eficientes com Actix. Se você tiver alguma dúvida ou quiser explorar algum tópico em mais detalhes, é só perguntar!

# [[Actix-Web/Basics/Exercicios/O Servidor HTTP]]
