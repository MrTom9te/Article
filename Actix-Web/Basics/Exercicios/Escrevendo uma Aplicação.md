 - [x]  **Exercício 1: Olá, Mundo Personalizado** 

*   **Objetivo:** Criar um servidor Actix Web simples que responda com uma saudação personalizada.
*   **Instruções:**
    1.  Crie um novo projeto Cargo (`cargo new exercicio1`).
    2.  Adicione `actix-web` como dependência no `Cargo.toml`.
    3.  Modifique `src/main.rs` para:
        *   Ter uma rota `GET` para `/`.
        *   Essa rota deve retornar uma string que inclua seu nome (ex: "Olá, [Seu Nome]!").
    4.  Execute o projeto (`cargo run`) e verifique o resultado no navegador.

- [x] **Exercício 2: Rota com Parâmetro**

*   **Objetivo:** Receber um parâmetro na URL e usá-lo na resposta.
*   **Instruções:**
    1.  Crie um novo projeto Cargo.
    2.  Adicione `actix-web`.
    3.  Crie uma rota `GET` para `/ola/{nome}`.
        *   Use o *extractor* `web::Path` para extrair o valor de `{nome}` da URL.
        *   Retorne uma string que cumprimente o nome fornecido (ex: "Olá, {nome}!").
    4.  Teste com URLs como `/ola/Maria` e `/ola/Joao`.

**Dica:**  Você precisará usar `web::Path<(String,)>` como o tipo do argumento da sua função handler. Exemplo do handler:
```rust
use actix_web::{web, Responder, HttpResponse};

async fn ola(info: web::Path<(String,)>) -> impl Responder {
    let nome = &info.0;
    HttpResponse::Ok().body(format!("Olá, {nome}!"))
}

```
E o trecho do `App` com a rota:

```rust
.route("/ola/{nome}", web::get().to(ola))
```

- [x] **Exercício 3: Estado Simples - Contador de Visitas**

*   **Objetivo:**  Implementar um contador de visitas usando o estado da aplicação.
*   **Instruções:**
    1.  Crie um novo projeto.
    2.  Adicione `actix-web`.
    3.  Defina uma struct `AppState` com um campo `visitas` (do tipo `i32`).
    4.  Crie uma rota `GET` para `/`.
    5.  No handler:
        *   Acesse o estado usando `web::Data<AppState>`.
        *   *Incremente* o valor de `visitas` (atenção: como não é mutável entre threads, você *não* precisa de `Mutex` neste exercício; cada thread terá seu próprio contador).
        *   Retorne uma string mostrando o número de visitas (ex: "Visitas: {visitas}").

- [x] **Exercício 4: Estado Mutável - Contador Global**

*   **Objetivo:**  Implementar um contador de visitas *global*, compartilhado entre todas as threads.
*   **Instruções:**
    1.  Crie um novo projeto.
    2.  Adicione `actix-web` e `std::sync::Mutex`.
    3.  Defina uma struct `AppState` com um campo `visitas` do tipo `Mutex<i32>`.
    4.  Crie uma rota `GET` para `/`.
    5.  No handler:
        *   Acesse o estado usando `web::Data<AppState>`.
        *   Use `.lock().unwrap()` para obter acesso exclusivo ao contador.
        *   Incremente o contador.
        *   Retorne uma string mostrando o número total de visitas.
    6.  Crie uma rota `GET` para `/reset`
        1. No handler de `/reset`, zere o contador de visitas.
    *   **Teste:**  Abra várias abas do navegador e acesse `/` em cada uma.  Verifique se o contador aumenta corretamente. Acesse `/reset` e veja o contador voltar a 0 em todas as abas abertas.

- [x] **Exercício 5: Escopo e Rotas Múltiplas**

*   **Objetivo:**  Organizar rotas usando `web::scope`.
*   **Instruções:**
    1.  Crie um novo projeto.
    2.  Adicione `actix-web`.
    3.  Crie um escopo `/api`.
    4.  Dentro do escopo `/api`:
        *   Crie uma rota `GET` para `/usuarios` que retorne "Lista de usuários".
        *   Crie uma rota `GET` para `/produtos` que retorne "Lista de produtos".
    5.  Fora do escopo, crie uma rota `GET` para `/` que retorne "Página inicial".
    6.  Teste acessando `/`, `/api/usuarios` e `/api/produtos`.

- [x] **Exercício 6: Guardas - Acesso Restrito**

*   **Objetivo:**  Usar um guarda para restringir o acesso a uma rota.
*   **Instruções:**
    1.  Crie um novo projeto.
    2.  Adicione `actix-web`.
    3.  Crie uma rota `GET` para `/admin` que retorne "Área administrativa".
    4.  Adicione um guarda `guard::Header("X-Admin-Token", "senha-secreta")` à rota `/admin`.
    5.  Teste acessando `/admin`:
        *   Sem o cabeçalho `X-Admin-Token`, você deve receber um erro (provavelmente 404 ou 403).
        *   Com o cabeçalho `X-Admin-Token` e o valor correto, você deve ver a mensagem da área administrativa.

    *   **Dica:** Para testar o envio de cabeçalhos, você pode usar ferramentas como:
        *   **cURL:** `curl -H "X-Admin-Token: senha-secreta" http://127.0.0.1:8080/admin`
        *   **Postman:** Um aplicativo para testar APIs (muito recomendado).
        *   **Extensões do navegador:** Existem extensões que permitem modificar cabeçalhos de requisição.
    
        No código a parte do `App` ficaria:

```rust
.service(
    web::resource("/admin")
        .guard(guard::Header("X-Admin-Token", "senha-secreta"))
        .route(web::get().to(admin_handler))
)
```
