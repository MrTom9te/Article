
**Introdução:**

O texto começa dizendo que o Actix Web fornece várias "primitivas" (ferramentas básicas) para construir servidores e aplicações web em Rust.  Ele menciona recursos como:

*   **Routing (Roteamento):**  Definir quais URLs correspondem a quais funções (handlers) no seu código.
*   **Middleware:**  Funções que executam antes ou depois do processamento de uma requisição. Útil para autenticação, logging, compressão de dados, etc.
*   **Pre-processing of requests (Pré-processamento de requisições):**  Modificar a requisição antes que ela chegue ao seu handler principal.
*   **Post-processing of responses (Pós-processamento de respostas):**  Modificar a resposta antes de enviá-la ao cliente.

O ponto central é que tudo no Actix Web gira em torno da instância `App`.  É o "coração" da sua aplicação. Você usa `App` para:

1.  **Registrar rotas:**  Conectar URLs a funções específicas.
2.  **Registrar middleware:**  Adicionar funcionalidades extras ao processamento de requisições/respostas.
3.  **Armazenar estado:**  Dados que são compartilhados entre todas as funções (handlers) da sua aplicação.

**Application Scope (Escopo da Aplicação):**

O "escopo" de uma aplicação (`scope`) funciona como um *namespace* (espaço de nomes) para as rotas.  É como uma pasta que contém um conjunto de rotas relacionadas. Todas as rotas dentro de um escopo compartilham o mesmo prefixo no URL.

*   **Exemplo:** Se você tem um escopo `/app`, todas as rotas dentro dele começarão com `/app`.  Por exemplo, `/app/usuarios`, `/app/produtos`, etc.
*   **Importante:** O Actix Web garante que o prefixo sempre comece com uma barra (`/`).  Se você esquecer, ele adiciona automaticamente.
*   **Correspondência de rotas:**  Um escopo `/app` corresponderá a requisições para `/app`, `/app/` e `/app/qualquercoisa`.  Mas *não* corresponderá a `/application`.

O código de exemplo, infelizmente quebrado mostra essa parte, mas a ideia é clara. Abaixo vou montar um código comentado para mostrar como isso se encaixa, baseado no que o texto forneceu:

```rust
use actix_web::{web, App, HttpServer, Responder};

async fn index() -> impl Responder {
    "Página Inicial"
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::scope("/app") // Define o escopo da aplicação
                    .route("/index.html", web::get().to(index)) // Rota dentro do escopo
            )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Nesse exemplo:

*   `web::scope("/app")`: Cria um escopo chamado `/app`.
*   `.route("/index.html", web::get().to(index))`:  Dentro do escopo, define uma rota que responde a requisições `GET` para `/app/index.html`. A função `index` será chamada.
* A url final para acessar essa rota é `http://127.0.0.1:8080/app/index.html`

**State (Estado):**

A aplicação Actix pode ter um "estado" – dados que são compartilhados entre todas as rotas e middlewares dentro do mesmo escopo. Isso é útil para coisas como:

*   Configurações da aplicação
*   Conexões com banco de dados
*   Contadores
*   Qualquer informação que precise ser acessada por diferentes partes da sua aplicação

Você acessa o estado usando o *extractor* `web::Data<T>`, onde `T` é o tipo do dado que você quer acessar.

**Exemplo (Estado Simples):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};

// 1. Define uma struct para o estado da aplicação
struct AppState {
    app_name: String,
}

// 2. Handler que usa o estado
async fn greet(data: web::Data<AppState>) -> impl Responder {
    let app_name = &data.app_name; // Acessa o nome da aplicação
    HttpResponse::Ok().body(format!("Olá, bem-vindo ao {app_name}!"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 3. Cria o estado e o passa para a aplicação
    let app_state = web::Data::new(AppState {
        app_name: String::from("Meu Super App"),
    });

    HttpServer::new(move || { // 'move' é importante aqui!
        App::new()
            .app_data(app_state.clone()) // 4. Clona o estado para cada thread
            .route("/", web::get().to(greet))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

*   **`AppState`:**  Uma struct que contém o nome da aplicação.
*   **`greet(data: web::Data<AppState>)`:** O handler recebe o estado como um argumento `web::Data`.
*   **`let app_name = &data.app_name;`:**  Acessa o campo `app_name` do estado.
*   **`web::Data::new(...)`:** Cria uma instância do `web::Data` para encapsular o estado.
*   **`move || { ... }`:**  A closure precisa "capturar" (move) a variável `app_state`. Isso é necessário porque cada thread do servidor terá sua própria cópia do estado.
*    `.app_data(app_state.clone())`:  Registra o estado na aplicação.  O `.clone()` é importante porque cada thread do Actix Web precisa de sua própria cópia do estado.
* A url para acessar é `http://127.0.0.1:8080/`

**Shared Mutable State (Estado Mutável Compartilhado):**

Se você precisa que o estado seja *modificado* por diferentes partes da sua aplicação (e que as mudanças sejam vistas por todas as threads), você precisa usar tipos que permitam acesso simultâneo seguro (thread-safe). O Actix Web recomenda usar tipos que implementem as traits `Send` e `Sync`.

O artigo explica que `web::Data` usa internamente `Arc` (Atomic Reference Counting), um tipo de ponteiro inteligente que permite compartilhamento seguro entre threads. E para evitar a criação de `Arc` duplicados a `Data` deve ser criada antes do registro com `App::app_data()`.

**Exemplo (Estado Mutável Compartilhado):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use std::sync::Mutex;

// 1. Define o estado com um Mutex para acesso seguro
struct CounterState {
    counter: Mutex<i32>, // Contador dentro de um Mutex
}

// 2. Handlers que incrementam e mostram o contador
async fn increment(data: web::Data<CounterState>) -> impl Responder {
    let mut counter = data.counter.lock().unwrap(); // Trava o Mutex para acesso exclusivo
    *counter += 1; // Incrementa o contador
    HttpResponse::Ok().body(format!("Contador incrementado: {counter}"))
}

async fn show_count(data: web::Data<CounterState>) -> impl Responder {
    let counter = data.counter.lock().unwrap(); // Trava o Mutex
    HttpResponse::Ok().body(format!("Contador atual: {counter}"))
}
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // 3. Cria o estado fora da closure
    let counter_state = web::Data::new(CounterState {
        counter: Mutex::new(0), // Inicializa o contador com 0
    });

    HttpServer::new(move || {
        App::new()
            .app_data(counter_state.clone()) // Clona para cada thread
            .route("/increment", web::get().to(increment))
            .route("/show", web::get().to(show_count))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
Explicação:

*   **`CounterState`:**  Contém um `Mutex<i32>`.  O `Mutex` (Mutual Exclusion) garante que apenas uma thread por vez possa modificar o contador.
*   **`data.counter.lock().unwrap()`:**
    *   `lock()`:  Tenta obter acesso exclusivo ao contador.  Se outra thread estiver usando o contador, a thread atual vai esperar.
    *   `unwrap()`:  "Desembrulha" o resultado do `lock()`.  Se o `lock()` falhar (o que é raro), o programa vai entrar em pânico (panic).
*   **`*counter += 1;`:**  Incrementa o valor do contador.  O `*` é necessário para acessar o valor *dentro* do `Mutex`.
*   **`counter_state` criado fora:** O estado `counter_state` é criado fora da closure do `HttpServer::new`.  Isso é crucial. Se fosse criado dentro, cada thread teria seu próprio contador, e as mudanças não seriam compartilhadas.

**Using an Application Scope to Compose Applications (Usando Escopo da Aplicação para Compor Aplicações):**

O método `web::scope()` permite definir um prefixo para um grupo de rotas. Isso é útil para organizar sua aplicação em módulos e para montar diferentes partes da aplicação em caminhos diferentes do URL.

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};

async fn show_users() -> impl Responder {
    HttpResponse::Ok().body("Mostrando usuários...")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::scope("/users") // Define o escopo /users
                    .route("/show", web::get().to(show_users)) // Rota dentro do escopo
            )
            // Outras rotas podem ser adicionadas aqui, fora do escopo
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

Neste exemplo, a rota `show_users` só será acessível através do URL `/users/show`. O `web::scope` adiciona o prefixo `/users` à rota.
* A url de acesso será: `http://127.0.0.1:8080/users/show`

**Application guards and virtual hosting (Guardas de Aplicação e Hospedagem Virtual):**

"Guards" (guardas) são funções que verificam se uma requisição deve ser processada ou não.  Eles agem como filtros.  Um guarda recebe uma referência à requisição e retorna `true` (permitir) ou `false` (bloquear).

O Actix Web fornece vários guardas prontos, como o `Host` (guarda de host), que permite filtrar requisições com base no cabeçalho `Host` (o domínio do site).

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, guard};

async fn my_handler() -> impl Responder {
    HttpResponse::Ok().body("Acesso permitido!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::resource("/")
                    .guard(guard::Host("www.meusite.com")) // Guarda de Host
                    .route(web::get().to(my_handler))
            )
            // Outras rotas, sem guarda...
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Nesse exemplo, a rota `/` só será processada se o cabeçalho `Host` da requisição for `"www.meusite.com"`.  Se o usuário acessar de outro domínio (ou diretamente pelo IP), a requisição não será processada por essa rota (e provavelmente resultará em um erro 404, a menos que você defina outras rotas).

**Configure (Configurar):**

Tanto `App` quanto `web::Scope` têm um método `configure`.  Ele permite que você organize a configuração da sua aplicação em diferentes módulos ou bibliotecas, tornando o código mais limpo e reutilizável.

**Exemplo:**

```rust
// src/my_routes.rs (outro arquivo)
use actix_web::{web, HttpResponse, Responder};

async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Olá de outro módulo!")
}

pub fn config(cfg: &mut web::ServiceConfig) {
    cfg.service(web::resource("/hello").route(web::get().to(hello)));
}
// src/main.rs
use actix_web::{web, App, HttpServer};

mod my_routes; // Importa o módulo

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .configure(my_routes::config) // Configura a aplicação usando a função do outro módulo

    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

*   **`my_routes.rs`:**  Define um módulo separado com uma função `config`.  Essa função recebe uma referência mutável a `web::ServiceConfig` e adiciona rotas a ela.
*   **`main.rs`:**  Importa o módulo `my_routes` e chama a função `config` dentro do método `configure` da aplicação. Isso "injeta" as rotas definidas em `my_routes.rs` na aplicação principal.
* A url de acesso será: `http://127.0.0.1:8080/hello`

**Resumo dos Pontos Principais:**

*   **`App`:**  A classe central do Actix Web.  Você a usa para registrar rotas, middleware e estado.
*   **`web::scope`:**  Define um prefixo para um grupo de rotas.
*   **Estado:**  Dados compartilhados entre handlers.  Use `web::Data<T>` para acessar o estado.
*   **Estado Mutável:**  Use `Mutex` (ou outros tipos thread-safe) para estado que precisa ser modificado.  Crie o estado *fora* da closure do `HttpServer::new`.
*   **Guards:**  Funções que filtram requisições.
*   **`configure`:**  Organize a configuração em módulos.

# [[Actix-Web/Basics/Exercicios/Escrevendo uma Aplicação]]
