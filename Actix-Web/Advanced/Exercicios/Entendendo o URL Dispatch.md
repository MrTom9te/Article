# Exercícios: URL Dispatch em Actix-Web

## Nível 1: Fundamentos

### Exercício 1.1: Rota Simples

**Contexto:**

Você está criando um servidor web básico que precisa responder a uma saudação simples na rota principal (`/`).

**Objetivos de Aprendizagem:**

-   Entender a estrutura básica de uma aplicação Actix-Web.
-   Configurar uma rota simples com `App::route()`.
-   Criar um handler assíncrono.

**Requisitos:**

-   Crie uma aplicação Actix-Web que responda a requisições GET na rota `/` com a mensagem "Olá, mundo!".
-   Use a função `HttpResponse::Ok().body()` para construir a resposta.

**Dicas:**

-   Use o exemplo do artigo como ponto de partida.
-   Lembre-se de usar `#[actix_web::main]` para tornar a função `main` assíncrona.

**Código de Exemplo (para referência):**

```rust
use actix_web::{web, App, HttpResponse, HttpServer};

async fn greet() -> HttpResponse {
    HttpResponse::Ok().body("Olá, mundo!")
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

### Exercício 1.2:  Métodos Diferentes

**Contexto:**

Seu servidor precisa responder a diferentes métodos HTTP na mesma rota.

**Objetivos de Aprendizagem:**

-   Usar `web::get()`, `web::post()`, etc. para definir rotas para métodos específicos.
-   Entender a diferença entre os métodos HTTP.

**Requisitos:**

-   Crie uma aplicação que responda:
    -   GET `/mensagem`: "Esta é uma mensagem GET"
    -   POST `/mensagem`: "Você enviou um POST"
    -   Qualquer outro método em `/mensagem`: "Método não suportado" (use `HttpResponse::MethodNotAllowed()`)

**Dicas:**

-   Use `.route()` várias vezes na mesma rota para diferentes métodos.
-   Use `web::method()` como um guard genérico, se necessário.

### Exercício 1.3:  Segmento Variável

**Contexto:**

Seu servidor precisa cumprimentar usuários pelo nome, usando um segmento variável na URL.

**Objetivos de Aprendizagem:**

-   Usar segmentos variáveis na definição de rotas.
-   Extrair o valor de um segmento variável usando `web::Path`.

**Requisitos:**

- Crie uma aplicação que responda a GET `/ola/{nome}` com a mensagem "Olá, {nome}!", onde `{nome}` é o valor fornecido na URL.
- Use `web::Path<String>` para extrair o nome.

**Dicas:**

- Lembre-se de que a struct ou tupla usada com `web::Path` deve corresponder à estrutura do padrão da URL.

**Código de Exemplo (para referência):**
```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

async fn ola(info: web::Path<String>) -> impl Responder {
    HttpResponse::Ok().body(format!("Olá, {}!", info))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/ola/{nome}", web::get().to(ola))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

## Nível 2: Integração de Conceitos

### Exercício 2.1:  Escopos e Parâmetros

**Contexto:**

Você está construindo uma API para um blog simples.  As rotas para postagens devem estar em `/blog/posts`, e você quer uma rota para buscar uma postagem específica por ID.

**Objetivos de Aprendizagem:**

-   Usar `web::scope()` para organizar rotas.
-   Combinar escopos e segmentos variáveis.
-   Extrair parâmetros numéricos usando `web::Path`.

**Requisitos:**

-   Crie uma aplicação com as seguintes rotas:
    -   GET `/blog/posts`:  Retorna "Lista de postagens".
    -   GET `/blog/posts/{id}`: Retorna "Postagem com ID: {id}", onde `{id}` é um número inteiro (`u32`).
-   Use `web::scope()` para agrupar as rotas de `/blog/posts`.
-   Use `web::Path<u32>` para extrair o ID da postagem.

**Dicas:**

-   Você pode usar uma tupla `(u32,)` com `web::Path` para extrair um único valor numérico.

### Exercício 2.2:  Guards e Query Strings

**Contexto:**

Você quer adicionar uma funcionalidade de busca à lista de postagens. A busca deve ser feita através de um parâmetro *query string*.

**Objetivos de Aprendizagem:**

-   Usar *query strings* em URLs.
-   Extrair parâmetros de *query strings* usando `web::Query`.
-   Usar guards para filtrar requisições.

**Requisitos:**

-   Modifique a aplicação do exercício anterior (2.1) para adicionar a seguinte funcionalidade:
    -   GET `/blog/posts?search={termo}`: Retorna "Buscando por: {termo}".
    -   Se o parâmetro `search` não estiver presente, a rota `/blog/posts` deve continuar retornando "Lista de postagens".

**Dicas:**

- Crie uma struct que implemente `Deserialize` para usar com `web::Query`.
-  Você pode usar `Option<String>` dentro da struct para representar um parâmetro opcional.
- Você não precisa de guards personalizados; os guards integrados do Actix-Web (como `web::get()`) são suficientes neste caso. O importante é a extração correta do *query string*.

**Código de Exemplo (para referência - Struct para Query String):**

```rust
#[derive(serde::Deserialize)]
struct SearchParams {
    search: Option<String>,
}
```

### Exercício 2.3:  Resposta Personalizada "Not Found"

**Contexto:**

Seu servidor deve retornar uma mensagem de erro mais amigável quando uma rota não for encontrada.

**Objetivos de Aprendizagem:**
-   Usar `App::default_service()` para personalizar a resposta 404.
-   Criar uma resposta HTTP personalizada com um código de status específico.

**Requisitos:**
- Modifique qualquer um dos exercícios anteriores.
- Quando uma URL não corresponder a nenhuma rota, o servidor deve retornar:
    - Código de status: 404 (Not Found)
    - Corpo da resposta: "Recurso não encontrado. Verifique a URL."

**Dicas:**
- Use `HttpResponse::build()` para criar uma resposta com um código de status específico.

## Nível 3: Projetos Práticos

### Exercício 3.1: Mini API de Tarefas

**Contexto:**

Crie uma API RESTful básica para gerenciar tarefas (similar ao exemplo de estrutura completa no guia de criação de exercícios).

**Objetivos de Aprendizagem:**

-   Aplicar todos os conceitos de URL Dispatch aprendidos.
-   Desenhar uma API com múltiplas rotas e funcionalidades.
-   Simular um cenário real de desenvolvimento.

**Requisitos:**

-   Crie uma API com as seguintes rotas:
    -   POST `/tasks`: Adiciona uma nova tarefa (o corpo da requisição deve ser um JSON com a descrição da tarefa).
    -   GET `/tasks`: Lista todas as tarefas.
    -   GET `/tasks/{id}`: Retorna os detalhes de uma tarefa específica (por ID).
    -   PUT `/tasks/{id}`: Marca uma tarefa como concluída.
    -   DELETE `/tasks/{id}`: Remove uma tarefa.

-   Use estruturas de dados apropriadas (como `Vec` ou `HashMap`) para armazenar as tarefas em memória (não é necessário usar um banco de dados real).
-   Implemente tratamento de erros básico (por exemplo, retornar 404 se uma tarefa não for encontrada).
-   Use *query string* para adicionar uma funcionalidade opcional de *filtragem*, para filtrar as tarefas por *status* (concluída ou pendente).

**Dicas:**

-   Comece definindo as estruturas de dados (`Tarefa`, etc.).
-   Planeje as rotas e seus handlers antes de começar a codificar.
-   Teste cada rota individualmente.
-   Considere usar `Arc<Mutex<...>>` para compartilhar o estado (a lista de tarefas) entre os handlers, já que o Actix-Web usa múltiplas threads.

### Exercício 3.2 : Adicionando External Resources

**Contexto:**

Melhore a API de tarefas do exercício 3.1 adicionando um link para um serviço externo em cada tarefa, utilize external resources.

**Objetivos de Aprendizagem**
- Entender o uso do external resources
- Aplicar o conceito de external resources a um projeto já existente

**Requisitos:**
- Adicione um link para um site como youtube ou google books em cada tarefa, este link deve ser gerado utilizando o external resource
- Ao listar as tarefas o link deve ser exibido

**Dicas:**
- Utilize o conceito de External Resources ensinado no artigo

Este conjunto de exercícios cobre uma ampla gama de funcionalidades do URL Dispatch em Actix-Web, desde o básico até aplicações mais complexas.  Eles são projetados para promover a aprendizagem prática e o desenvolvimento de habilidades em Rust e Actix-Web.