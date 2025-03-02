#  Extração de Informações da Requisição com Tipagem Segura

O Actix Web fornece um mecanismo chamado *extractors* (extratores) para acessar informações da requisição de maneira segura em relação aos tipos. Isso significa que você obtém os dados da requisição já no tipo correto (ex: `String`, `u32`, uma struct personalizada), sem precisar fazer conversões manuais e verificações de tipo.

**Conceitos-Chave:**

*   **`impl FromRequest`:**  Os extratores são tipos que implementam a trait `FromRequest`.  Isso significa que eles sabem como se "construir" a partir de uma requisição HTTP.
*   **Handler Arguments (Argumentos do Handler):**  Você acessa os extratores simplesmente declarando-os como argumentos na sua função handler. O Actix Web automaticamente "injeta" os valores corretos.
*   **Limite de Extratores:**  Um handler pode ter, no máximo, 12 extratores.
*   **Ordem dos Argumentos:**  Na maioria dos casos, a ordem dos argumentos do handler *não* importa. O Actix Web é inteligente o suficiente para descobrir qual extrator corresponde a qual parte da requisição.
*   **Extração do Corpo (Body):** Existe uma exceção importante em relação à ordem. Se um extrator *lê o corpo da requisição* (os dados enviados pelo cliente, como em um formulário ou JSON), então apenas o *primeiro* extrator que tentar ler o corpo terá sucesso. Os demais falharão, porque o corpo da requisição só pode ser lido uma vez. Se precisar fazer algo como ler em diversos formatos deve usar `Either`.

**Exemplo Básico (Quebrado no Artigo, mas Corrigido):**

O exemplo original do artigo está incompleto. Aqui está uma versão corrigida e comentada:

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::Deserialize;

// 1. Define structs para os dados que você quer extrair
#[derive(Deserialize)]
struct Info {
    username: String,
}

#[derive(Deserialize)]
struct MyData {
    data: String
}

// 2. Handler com extratores
async fn index(path: web::Path<Info>, data: web::Json<MyData>) -> impl Responder {
    let username = &path.username;
    let infoData = &data.data;
    HttpResponse::Ok().body(format!("Olá, {username}! Dados: {infoData}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/{username}", web::post().to(index)) // Configura a rota
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
Para testar:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"data": "Meus Dados"}' http://127.0.0.1:8080/AlgumNome
```
*   **`#[derive(Deserialize)]`:**  Usa a biblioteca `serde` para definir que as structs `Info` e `MyData` podem ser *deserializadas* (convertidas) a partir de dados externos (como JSON).  Você precisa ter o `serde` no seu `Cargo.toml` com a feature `derive` habilitada: `serde = { version = "1.0", features = ["derive"] }`.
*   **`web::Path<Info>`:** Extrai informações do *caminho* da URL (a parte depois do domínio e da porta). Neste caso, extrai o valor de `{username}`.
*   **`web::Json<MyData>`:** Extrai o corpo da requisição como JSON e o converte para a struct `MyData`.
*   **`/{username}` (na rota):** Define um segmento dinâmico na URL. O valor desse segmento será extraído para o `web::Path`.
*   **`web::post()`:** Configura a rota para responder apenas a requisições `POST`.

**Path (Caminho):**

O extrator `Path` é usado para extrair informações de "segmentos dinâmicos" na URL.  Segmentos dinâmicos são partes da URL delimitadas por chaves (`{}`).

**Exemplo (Tupla):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};

// Handler que extrai dois segmentos da URL
async fn user_info(info: web::Path<(u32, String)>) -> impl Responder {
    let (user_id, friend) = info.into_inner(); // Extrai os valores da tupla
    HttpResponse::Ok().body(format!("ID do Usuário: {user_id}, Amigo: {friend}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/users/{user_id}/{friend}", web::get().to(user_info))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

*   **`web::Path<(u32, String)>`:** Extrai dois segmentos: um como `u32` (inteiro sem sinal de 32 bits) e outro como `String`. A ordem importa.
*   **`info.into_inner()`:** Converte o `web::Path` na tupla subjacente.
*   **`/users/{user_id}/{friend}`:** Define os segmentos dinâmicos na rota.
* Para testar, acesse: `http://127.0.0.1:8080/users/123/Maria`

**Exemplo (Struct com `serde`):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::Deserialize;

#[derive(Deserialize)]
struct UserInfo {
    user_id: u32,
    friend: String,
}

// Handler que extrai os segmentos para uma struct
async fn user_info(info: web::Path<UserInfo>) -> impl Responder {
    let user_id = info.user_id;
    let friend = &info.friend;
    HttpResponse::Ok().body(format!("ID do Usuário: {user_id}, Amigo: {friend}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/users/{user_id}/{friend}", web::get().to(user_info))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
* A url de acesso é a mesma: `http://127.0.0.1:8080/users/123/Maria`

*   **`#[derive(Deserialize)]`:** Permite que `UserInfo` seja deserializada.
*   **`web::Path<UserInfo>`:** Extrai os segmentos e os mapeia para os campos da struct (os nomes precisam ser iguais).

**Exemplo (Parâmetros Não Type-Safe):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpRequest, HttpResponse};

async fn user_info(req: HttpRequest) -> impl Responder {
    let user_id: u32 = req.match_info().get("user_id").unwrap().parse().unwrap();
    let friend: String = req.match_info().get("friend").unwrap().to_string();
     HttpResponse::Ok().body(format!("ID do Usuário: {user_id}, Amigo: {friend}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/users/{user_id}/{friend}", web::get().to(user_info))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```
* A url de acesso é a mesma: `http://127.0.0.1:8080/users/123/Maria`

*   **`req.match_info().get("user_id")`:**  Obtém o valor do parâmetro `user_id` da URL.  Isso retorna um `Option<&str>`, então você precisa usar `.unwrap()` para obter o valor (e o programa entrará em pânico se o parâmetro não existir).
*   **`.parse().unwrap()`:**  Converte a string para `u32`.  Novamente, usa `unwrap()` (e pode causar pânico).
*   **`.to_string()`:**  Copia a string.
*   **Menos Seguro:** Essa abordagem é menos segura porque você precisa fazer conversões manuais e lidar com possíveis erros (como parâmetros ausentes ou tipos inválidos).

**Query (Consulta):**

O extrator `Query<T>` é usado para extrair parâmetros de *query* da URL (a parte depois do `?`).  Ele usa a biblioteca `serde_urlencoded` internamente.

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::Deserialize;

#[derive(Deserialize)]
struct SearchParams {
    query: String,
    page: Option<u32>, // O parâmetro 'page' é opcional
}

async fn search(params: web::Query<SearchParams>) -> impl Responder {
    let query = &params.query;
    let page = params.page.unwrap_or(1); // Usa 1 como valor padrão se 'page' não for fornecido
    HttpResponse::Ok().body(format!("Pesquisando por: {query}, Página: {page}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/search", web::get().to(search))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

*   **`#[derive(Deserialize)]`:**  `SearchParams` pode ser deserializada.
*   **`web::Query<SearchParams>`:** Extrai os parâmetros de query.
*   **`Option<u32>`:**  O parâmetro `page` é opcional. Se não for fornecido na URL, `params.page` será `None`.
*   **`unwrap_or(1)`:**  Fornece um valor padrão (1) se `page` for `None`.
*   **Teste:**
    *   `http://127.0.0.1:8080/search?query=livros`:  "Pesquisando por: livros, Página: 1"
    *   `http://127.0.0.1:8080/search?query=filmes&page=3`: "Pesquisando por: filmes, Página: 3"

**JSON:**

O extrator `Json<T>` é usado para deserializar o *corpo* da requisição (o conteúdo enviado pelo cliente) como JSON.  O tipo `T` precisa implementar `serde::Deserialize`.

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::Deserialize;

#[derive(Deserialize)]
struct Product {
    name: String,
    price: f64,
}

async fn create_product(product: web::Json<Product>) -> impl Responder {
    let name = &product.name;
    let price = product.price;
    HttpResponse::Ok().body(format!("Produto Criado: {name}, Preço: {price}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/products", web::post().to(create_product)) // Geralmente, POST para criar
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```
Para testar com `curl`:

```bash
curl -X POST -H "Content-Type: application/json" -d '{"name": "Teclado Mecânico", "price": 299.99}' http://127.0.0.1:8080/products
```
*   **`#[derive(Deserialize)]`:**  `Product` pode ser deserializada a partir de JSON.
*   **`web::Json<Product>`:** Extrai o corpo da requisição como JSON e o converte para `Product`.
*   **`web::post()`:**  A rota responde a requisições `POST`.

**Configuração do JSON (Exemplo):**

Você pode configurar o extrator `Json` (e outros) usando `app_data`. Isso permite, por exemplo, limitar o tamanho máximo do payload JSON e fornecer uma função para lidar com erros de forma personalizada.

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse, error};
use serde::Deserialize;

#[derive(Deserialize)]
struct Product {
    name: String,
    price: f64,
}
// função de tratamento de erro
async fn handle_json_error(err: error::JsonPayloadError) -> error::Error {
       let error_message = format!("Erro ao processar JSON: {}", err);
        error::ErrorBadRequest(error_message)

}
async fn create_product(product: web::Json<Product>) -> impl Responder {
    let name = &product.name;
    let price = product.price;
     HttpResponse::Ok().body(format!("Produto Criado: {name}, Preço: {price}"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            // Configura o extrator Json: limite de 4KB e handler de erro
            .app_data(web::JsonConfig::default().limit(4096).error_handler(handle_json_error))
            .route("/products", web::post().to(create_product))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```

*   **`.app_data(web::JsonConfig::default().limit(4096))`:**  Configura o extrator `Json`.  `limit(4096)` define o tamanho máximo do payload como 4KB (4096 bytes).
*   `.error_handler(handle_json_error)`: Define uma função que trata de erros, para retornar mensagens mais amigáveis.

**URL-Encoded Forms (Formulários Codificados na URL):**

Semelhante ao `Json<T>`, você pode extrair dados de um formulário codificado na URL (o formato padrão de envio de formulários HTML) para uma struct. A struct também precisa implementar `serde::Deserialize`.

**Exemplo:**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::Deserialize;

#[derive(Deserialize)]
struct ContactForm {
    name: String,
    email: String,
    message: String,
}

async fn submit_contact(form: web::Form<ContactForm>) -> impl Responder {
    let name = &form.name;
    let email = &form.email;
    let message = &form.message;
        HttpResponse::Ok().body(format!("Nome: {name}, Email: {email}, Mensagem: {message}"))

}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/contact", web::post().to(submit_contact))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}

```
Para testar com `curl`, lembre-se que os dados precisam ser codificados para a URL:

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "name=Fulano&email=fulano%40example.com&message=Ol%C3%A1%2C%20tudo%20bem%3F" http://127.0.0.1:8080/contact
```

*   **`#[derive(Deserialize)]`:**  `ContactForm` pode ser deserializada.
*   **`web::Form<ContactForm>`:** Extrai os dados do formulário.
*   **`application/x-www-form-urlencoded`:**  É o `Content-Type` correto para formulários codificados na URL.

**Outros Extratores:**

O artigo lista outros extratores importantes:

*   **`Data`:** Acessa o estado da aplicação (já vimos exemplos).
*   **`HttpRequest`:**  Dá acesso ao objeto `HttpRequest` completo, permitindo inspecionar cabeçalhos, método, etc.
*   **`String`:** Converte o corpo da requisição para uma `String`.
*   **`Bytes`:** Converte o corpo da requisição para um vetor de bytes (`Bytes`).
*   **`Payload`:**  Extrator de baixo nível para o payload (fluxo de bytes) da requisição. Usado principalmente para construir outros extratores.

**Application State Extractor (Extrator de Estado da Aplicação):**

Já vimos como usar `web::Data` para acessar o estado da aplicação. O artigo reforça que o estado é acessado como uma referência *read-only* (somente leitura). Se você precisar modificar o estado, precisa usar tipos que permitam acesso mutável seguro entre threads (como `Arc<Mutex<T>>`).

**Exemplo (Estado Read-Only - Incorreto para Contagem Global):**

```rust
// ... (código para definir AppState e o handler)

async fn my_handler(data: web::Data<AppState>) -> impl Responder {
    // ERRADO: Isso conta apenas as requisições por thread, não o total global
    // data.count += 1; // Não compila! data é read-only
   let msg = format!("Contador (local): {}",data.count);
    HttpResponse::ok().body(msg)
}
```

**Exemplo (Estado Mutável - Correto para Contagem Global):**

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use std::sync::{Arc, Mutex};

// Estado com Mutex para acesso seguro
struct AppState {
    count: Mutex<i32>,
}

async fn my_handler(data: web::Data<AppState>) -> impl Responder {
    let mut count = data.count.lock().unwrap(); // Obtém acesso exclusivo
    *count += 1; // Incrementa o contador
    let msg = format!("Contador (global): {}",count);
     HttpResponse::Ok().body(msg)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    // Cria o estado FORA da closure
    let app_state = web::Data::new(AppState {
        count: Mutex::new(0),
    });

    HttpServer::new(move || { // 'move' captura app_state
        App::new()
            .app_data(app_state.clone()) // Clona para cada thread
            .route("/", web::get().to(my_handler))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```
* O acesso é feito pelo `Mutex`, e a instancia do app_state é criada fora do escopo.

**Considerações sobre Blocking (Bloqueio):**

O artigo termina com um aviso importante sobre o uso de primitivas de sincronização bloqueantes (como `Mutex` e `RwLock`) dentro de handlers.  Como o Actix Web é assíncrono, bloquear a thread dentro de um handler é *muito ruim* para o desempenho. O artigo recomenda evitar `Mutex` e `RwLock` sempre que possível, e usar as versões assíncronas fornecidas pelo Tokio se você realmente precisar deles.

Este artigo detalha os *extractors*, que são uma parte fundamental do Actix Web para lidar com dados de requisição de forma eficiente e segura. Se tiver alguma dúvida, me diga!

# [[Extração de Informações da Requisição com Tipagem Segura]]
