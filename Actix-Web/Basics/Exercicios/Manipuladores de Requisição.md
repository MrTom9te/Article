- [ ] **1. Retorno Básico**

    **Título:** Respondendo com Texto Simples

    **Contexto:** Você está criando um servidor web básico que precisa retornar mensagens de texto simples para os usuários.

    **Objetivos de Aprendizagem:**

    *   Entender a estrutura básica de um handler.
    *   Retornar uma string estática como resposta.

    **Requisitos:**

    *   Crie um handler que responda à rota raiz (`/`).
    *   O handler deve retornar a string `"Olá, mundo!"`.

    **Resposta Esperada:** Ao acessar `http://127.0.0.1:8080/`, você deve ver "Olá, mundo!".

    **Exemplos:**

    *   Você pode retornar um `&'static str` diretamente.
    *   Lembre-se da estrutura básica do `HttpServer` e `App`.

    **Recursos:**

    *   [String literals in Rust](https://doc.rust-lang.org/book/ch03-02-data-types.html#the-string-type)

---

- [ ] **2. Retorno Dinâmico**

    **Título:** Construindo uma Resposta Dinâmica

    **Contexto:** Seu servidor precisa gerar respostas personalizadas com base em alguma lógica.

    **Objetivos de Aprendizagem:**

    *   Retornar uma string dinâmica (`String`).
    *   Usar formatação de strings.

    **Requisitos:**

    *   Crie um handler que responda à rota `/ola`.
    *   O handler deve retornar uma string formatada como "Olá, [seu nome aqui]!". Substitua `[seu nome aqui]` pelo seu nome real.

    **Resposta Esperada:** Ao acessar `/ola`, você deve ver "Olá, [seu nome]!".

    **Exemplos:**

    *   Use `String::from()` ou `.to_string()` para criar uma string dinâmica.
    *   Use `format!()` para criar a string formatada.

    **Recursos:**

    *   [`String` type](https://doc.rust-lang.org/std/string/struct.String.html)
    *   [`format!` macro](https://doc.rust-lang.org/std/macro.format.html)

---

- [ ] **3. Retorno com `impl Responder`**

    **Título:** Usando `impl Responder`

    **Contexto:** Você está experimentando diferentes formas de retornar respostas e quer usar `impl Responder` para maior flexibilidade.

    **Objetivos de Aprendizagem:**

    *   Entender o uso de `impl Responder`.
    *   Retornar diferentes tipos que implementam `Responder`.

    **Requisitos:**
     * Crie um handler que responda a rota `/`.
    *   Use `impl Responder` como tipo de retorno.
    *   Dentro do handler, retorne `HttpResponse::Ok().body("Resposta com impl Responder")`.

    **Resposta Esperada:** Ao acessar `/`, você deve ver "Resposta com impl Responder".

    **Exemplos:**

    *   Você pode retornar `HttpResponse`, `String`, `&'static str`, entre outros, com `impl Responder`.

    **Recursos:**
    *   [`HttpResponse` struct](https://docs.rs/actix-web/latest/actix_web/struct.HttpResponse.html)

---

- [ ] **4. Resposta Customizada (JSON)**

    **Título:** Criando um Tipo de Resposta JSON

    **Contexto:** Seu servidor precisa retornar dados estruturados no formato JSON.

    **Objetivos de Aprendizagem:**

    *   Criar um tipo de resposta personalizado.
    *   Implementar a trait `Responder`.
    *   Usar `serde` para serialização JSON.

    **Requisitos:**

    *   Crie uma struct `JsonResponse` com um campo `message` (String).
    *   Implemente `Responder` para `JsonResponse`, serializando-o para JSON.
    *   Crie um handler que retorne um `JsonResponse`.
    *   A rota pode ser `/json`.

    **Resposta Esperada:** Ao acessar `/json`, você deve ver uma resposta JSON válida, como `{"message": "Alguma mensagem"}`.

    **Exemplos:**

    *   Use `#[derive(Serialize)]`.
    *   Use `serde_json::to_string()`.
    *   Defina o `Content-Type` como `application/json`.
    *   Lembre-se de adicionar `serde` (com a feature `derive`) e `serde_json` ao seu `Cargo.toml`.

    **Recursos:**

    *   [`serde` crate](https://serde.rs/)
    *   [`serde_json` crate](https://docs.rs/serde_json/latest/serde_json/)
    *   [Implementing `Responder`](https://actix.rs/docs/response/#custom-response)

---

- [ ] **5. Streaming de Resposta**

    **Título:** Enviando Dados em Tempo Real

    **Contexto:** Você precisa enviar uma grande quantidade de dados ou dados que são gerados dinamicamente, sem carregar tudo na memória.

    **Objetivos de Aprendizagem:**

    *   Entender o conceito de streaming.
    *   Criar um `Stream` de bytes.
    *   Usar `HttpResponse::streaming()`.

    **Requisitos:**

    *   Crie um handler que responda à rota `/stream`.
    *   O handler deve retornar uma resposta com streaming.
    *   Crie um `Stream` que emita pelo menos três partes de texto (ex: "Parte 1", "Parte 2", "Parte 3").
    *   Defina o `Content-Type` como `text/plain`.

    **Resposta Esperada:** Ao acessar `/stream`, você deve ver as partes do texto sendo exibidas gradualmente no navegador (ou na ferramenta que você estiver usando para testar).

    **Exemplos:**

    *   Use `futures_util::stream::iter`.
    *   Use `Bytes::from_static()` para criar `Bytes` a partir de strings estáticas.
    *   Cada item do stream deve ser um `Result<Bytes, Error>`.
    * Adicione `futures-util = "0.3"` em seu `Cargo.toml`.

    **Recursos:**

    *   [`futures` crate](https://crates.io/crates/futures)
    *   [Streaming responses in Actix Web](https://actix.rs/docs/response/#streaming-response-body)

---

- [ ] **6. Retornos Diferentes (Either)**

    **Título:** Tratando Sucesso e Erro com `Either`

    **Contexto:** Seu handler precisa retornar tipos diferentes dependendo do resultado de uma operação.

    **Objetivos de Aprendizagem:**

    *   Usar o tipo `Either`.
    *   Retornar diferentes tipos que implementam `Responder`.
    *   Lidar com casos de sucesso e erro.

    **Requisitos:**
    * Rota `/saudacao`
    *   Crie um handler que receba um parâmetro de query opcional chamado `nome`.
    *   Se `nome` for fornecido, retorne uma saudação personalizada ("Olá, [nome]!").
    *   Se `nome` não for fornecido, retorne um erro 400 Bad Request com uma mensagem de erro.

    **Resposta Esperada:**

    *   `/saudacao?nome=Fulano`: "Olá, Fulano!"
    *   `/saudacao`: Uma resposta 400 Bad Request com uma mensagem de erro.

    **Exemplos:**

    *   Use `Either<String, HttpResponse>`.
    *   Use `Either::Left` para o caso de sucesso e `Either::Right` para o caso de erro.
    *   Use `web::Query` para extrair o parâmetro.
    *   Use `Option` e `unwrap_or` (ou similar) para lidar com o parâmetro opcional.

    **Recursos:**
     * [`Either` type](https://docs.rs/actix-web/latest/actix_web/enum.Either.html)

---
- [ ] **7. Retornando Result**
    **Título:**  Handler com Potencial de Erro
     **Contexto:** Seu handler acessa um recurso que pode falhar (ex: leitura de arquivo, chamada de API externa).
     **Objetivos de Aprendizagem:**
    * Retornar um `Result` de um handler.
    * Propagar erros corretamente.
    * Diferenciar erros de aplicação de erros internos do servidor.
    **Requisitos:**
        * Crie uma função `simular_operacao_arriscada` que:
            * Recebe um booleano.
            * Se o booleano for `true`, retorna `Ok("Sucesso!")`.
            * Se o booleano for `false`, retorna um `Err` com uma mensagem de erro personalizada (ex: `Err(MeuErro::FalhaNaOperacao)`.
        * Crie um tipo de erro customizado `MeuErro` (pode ser um enum simples).
        * Implemente `ResponseError` para `MeuErro`, retornando um `HttpResponse` apropriado (ex: 400 Bad Request para `FalhaNaOperacao`).
        * Crie um handler que chame `simular_operacao_arriscada` e retorne o resultado (usando `?` para propagar o erro, se houver).
        * O handler deve responder à rota `/risco`.
    **Resposta Esperada:**
        * `/risco?sucesso=true`: "Sucesso!" (com status 200 OK)
        * `/risco?sucesso=false`: Uma resposta de erro (ex: 400 Bad Request) com a mensagem de erro.
        * `/risco`: Deve apresentar erro, pode ser um erro interno 500.
    **Exemplos:**
        * Use `Result<impl Responder, MeuErro>` como o tipo de retorno do handler.
        * Implemente `std::fmt::Display` e `actix_web::error::ResponseError` para `MeuErro`.
        *  Use `web::Query` e um `Option<bool>` para receber o parâmetro booleano da *query string*.
        * Para converter o `Option<bool>` em um bool de forma segura você pode usar `.unwrap_or(false)`

    **Recursos:**
      * [`Result` type](https://doc.rust-lang.org/std/result/enum.Result.html)
      * [Error Handling in Actix Web](https://actix.rs/docs/errors/)
      * [`ResponseError` trait](https://docs.rs/actix-web/latest/actix_web/error/trait.ResponseError.html)
     * Exemplo da implementação de `ResponseError`

```rust
#[derive(Debug)] //Permite usar {:?} para debugar
enum MeuErro {
    FalhaNaOperacao,
}
//Implementa Display, necessário para ResponseError
impl std::fmt::Display for MeuErro {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            MeuErro::FalhaNaOperacao => write!(f, "A operação falhou!"),
        }
    }
}

//Implementa ResponseError
impl actix_web::error::ResponseError for MeuErro {
     fn error_response(&self) -> HttpResponse {
        match self {
           MeuErro::FalhaNaOperacao => {
                HttpResponse::BadRequest().body("Erro: A operação falhou!")
            }
        }
    }
}
```
---
Esses exercícios cobrem os principais aspectos dos *request handlers* no Actix Web, incluindo diferentes formas de retornar respostas, lidar com erros, usar streaming e criar tipos de resposta personalizados. Eles também reforçam o uso de extratores e a integração com outras bibliotecas importantes do ecossistema Rust, como `serde` e `futures`. A adição do exercicio 7 aborda um topico fundamental, o tratamento de erros.