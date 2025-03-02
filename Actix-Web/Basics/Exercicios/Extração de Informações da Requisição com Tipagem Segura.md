
---

- [x] **1. Extração de Path (Tupla)**

    **Título:** Usuário e Amigo na URL

    **Exercício:** Crie um servidor Actix Web que extraia dois valores da URL: um ID de usuário (número inteiro) e um nome de amigo (string).

    **Requisitos:**

    *   Use uma tupla para extrair os valores do path.
    *   A rota deve ser `/users/{user_id}/{friend}`.
    *   O handler deve retornar uma mensagem formatada com os valores extraídos (ex: "ID: 123, Amigo: Ana").

    **Resposta Esperada:** Ao acessar, por exemplo, `/users/123/Ana`, você deve ver "ID: 123, Amigo: Ana".

    **Exemplos:**

    *   Use `web::Path<(u32, String)>`.
    *   Use `.into_inner()` para obter a tupla.

---

- [x] **2. Extração de Path (Struct)**

    **Título:** Informações do Produto na URL

    **Exercício:** Crie um servidor que extraia informações de um produto de segmentos da URL.

    **Requisitos:**

    *   Use uma struct (com `#[derive(Deserialize)]`) para representar as informações do produto: `id` (u32) e `nome` (String).
    *   A rota deve ser `/produtos/{id}/{nome}`.
    *   O handler deve retornar uma mensagem formatada com os dados do produto.

    **Resposta Esperada:** Ao acessar, por exemplo, `/produtos/456/Teclado`, você deve ver algo como "Produto: Teclado, ID: 456".

    **Exemplos:**

    *   Lembre-se de adicionar `serde = { version = "1.0", features = ["derive"] }` ao seu `Cargo.toml`.
    *   Os nomes dos campos da struct devem corresponder aos nomes dos segmentos na URL.

---

- [x] **3. Parâmetros de Query**

    **Título:** Pesquisa com Parâmetros Opcionais

    **Exercício:** Crie um servidor que lide com uma pesquisa, onde o termo de pesquisa é obrigatório e a página é opcional.

    **Requisitos:**

    *   Use uma struct (com `#[derive(Deserialize)]`) para representar os parâmetros: `q` (String, obrigatório) e `pagina` (u32, opcional).
    *   A rota deve ser `/pesquisa`.
    *   O handler deve retornar uma mensagem formatada, usando "1" como a página padrão se ela não for fornecida.

    **Resposta Esperada:**

    *   `/pesquisa?q=livros`: "Pesquisando por: livros, Página: 1"
    *   `/pesquisa?q=filmes&pagina=3`: "Pesquisando por: filmes, Página: 3"

    **Exemplos:**

    *   Use `web::Query<SuaStruct>`.
    *   Use `Option<u32>` para o campo opcional.
    *   Use `.unwrap_or(valor_padrao)` para lidar com o caso opcional.

---

- [x] **4. Extração de JSON**

    **Título:** Criação de Usuário com JSON

    **Exercício:** Crie um servidor que receba dados de um usuário (nome e idade) em formato JSON e retorne uma confirmação.

    **Requisitos:**

    *   Use uma struct (com `#[derive(Deserialize)]`) para representar o usuário: `nome` (String) e `idade` (u32).
    *   A rota deve ser `/usuarios` e deve aceitar requisições `POST`.
    *   O handler deve retornar uma mensagem formatada confirmando o recebimento dos dados.

    **Resposta Esperada:** Ao enviar um JSON como `{"nome": "Maria", "idade": 30}` para `/usuarios` (usando `curl` ou outra ferramenta), você deve ver algo como "Usuário criado: Maria, Idade: 30".

    **Exemplos:**

    *   Use `web::Json<SuaStruct>`.
    *   Use `-X POST` e `-H "Content-Type: application/json"` com `curl`.
    * Lembre-se que, por questões de segurança, criar/alterar dados em uma API quase sempre é feito com POST.

---

- [x] **5. Limitando o Tamanho do JSON**

    **Título:** Protegendo Contra Payloads Grandes

    **Exercício:** Modifique o exercício anterior para limitar o tamanho do payload JSON a 2KB.

    **Requisitos:**

    *   Use `web::JsonConfig` para configurar o limite.
    *   Se o payload exceder o limite, o servidor deve retornar um erro 400 Bad Request.

    **Resposta Esperada:**

    *   Payloads JSON menores que 2KB devem funcionar normalmente.
    *   Payloads maiores que 2KB devem resultar em um erro 400.

    **Exemplos:**

    *   Use `.app_data(web::JsonConfig::default().limit(2048))`.

---

- [x] **6. Formulário URL-Encoded**

    **Título:** Processando Dados de um Formulário

    **Exercício:** Crie um servidor que processe dados de um formulário de contato simples (nome e email).

    **Requisitos:**

    *   Use uma struct (com `#[derive(Deserialize)]`) para representar os dados do formulário: `nome` (String) e `email` (String).
    *   A rota deve ser `/contato` e deve aceitar requisições `POST`.
    *   Use `web::Form` para extrair os dados.
    *   Retorne uma mensagem formatada com os dados recebidos.

    **Resposta Esperada:** Ao enviar dados codificados como URL (usando `curl` ou um formulário HTML) para `/contato`, você deve ver uma mensagem com os dados.

    **Exemplos:**

    *   Use `web::Form<SuaStruct>`.
    *   Use `-X POST` e `-H "Content-Type: application/x-www-form-urlencoded"` com `curl`.
    *   Lembre-se de codificar os dados para a URL (ex: espaços viram `+`, `@` vira `%40`).

---

- [x] **7. Acesso ao HttpRequest**

    **Título:** Inspecionando Cabeçalhos

    **Exercício:** Crie um servidor que retorne o valor de um cabeçalho HTTP específico, enviado pelo cliente.

    **Requisitos:**
    *   Use o extrator `HttpRequest`.
    *   A rota pode ser `/`.
    *   O handler deve tentar obter o valor do cabeçalho `X-Meu-Cabecalho`.
    *   Se o cabeçalho estiver presente, retorne seu valor. Caso contrário, retorne "Cabeçalho ausente".

    **Resposta Esperada:**
    *   Se você enviar a requisição com `-H "X-Meu-Cabecalho: ValorQualquer"`, o servidor deve retornar "ValorQualquer".
    *   Se você não enviar o cabeçalho, o servidor deve retornar "Cabeçalho ausente".

    **Exemplos:**
        *   `req.headers().get("X-Meu-Cabecalho")` retorna um `Option<&HeaderValue>`.
        *   Você pode usar `.and_then(|hv| hv.to_str().ok())` para converter o `HeaderValue` para `Option<&str>`.

---

- [x] **8. Estado da Aplicação (Mutável)**

    **Título:** Contador de Acessos Global

    **Exercício:** Crie um servidor que mantenha um contador global de acessos à rota `/`.

    **Requisitos:**

    *   Use uma struct `AppState` com um campo `contador` (do tipo `Mutex<i32>`).
    *   Crie o estado *fora* da closure do `HttpServer::new`.
    *   Use `web::Data` para acessar o estado no handler.
    *   Incremente o contador a cada acesso.
    *   Retorne o valor atual do contador.

    **Resposta Esperada:** Cada vez que você acessar `/`, o contador deve aumentar em 1, e esse valor deve ser compartilhado entre todas as threads (workers).

    **Exemplos:**

    *   Lembre-se de usar `Arc` e `Mutex`: `web::Data::new(AppState { contador: Mutex::new(0) })`.
    *   Use `data.contador.lock().unwrap()` para obter acesso exclusivo ao contador.
    *   Use `*contador += 1` para incrementar.
    *   Use o `move` ao criar a closure.

---

- [ ] **9. Combinando Extratores**

    **Título:** Path e Query Juntos

    **Exercício:** Crie um servidor que combine a extração de informações do path e da query string.

    **Requisitos:**
     *   Rota: `/usuario/{id}/info?detalhes=true` ou `/usuario/{id}/info`
        *   Path:
            *   `id`: um `u32` que representa o ID do usuário.
        *   Query:
            *  `detalhes`: um booleano *opcional*.
    * Use structs separadas com derive para o path e para a query
    *   O handler deve retornar uma mensagem formatada, indicando o ID do usuário e se os detalhes foram solicitados ou não.

    **Resposta Esperada:**
      * `/usuario/123/info?detalhes=true`: "Usuário 123: Detalhes solicitados"
       * `/usuario/456/info`: "Usuário 456: Detalhes não solicitados"

    **Exemplos:**
     *  Você irá precisar usar dois derives, um com a struct para o path e outra para a query.
     * Use `Option<bool>` para o parâmetro `detalhes` na struct da query.
     * Use `.unwrap_or(false)` para definir `false` como padrão caso não informe o parametro.

---
**Links para Aprofundamento:**

*   **Serde:** [https://serde.rs/](https://serde.rs/) - A biblioteca Serde é fundamental para a serialização e deserialização de dados em Rust. É usada por muitos extratores do Actix Web.
*   **`std::sync::Mutex`:** [https://doc.rust-lang.org/std/sync/struct.Mutex.html](https://doc.rust-lang.org/std/sync/struct.Mutex.html) - Documentação oficial do `Mutex`.
* **`tokio::sync::Mutex`**: [https://docs.rs/tokio/latest/tokio/sync/struct.Mutex.html](https://docs.rs/tokio/latest/tokio/sync/struct.Mutex.html) - A versão assíncrona do Mutex, para uso dentro de código assíncrono.
* **`Option` type**: [https://doc.rust-lang.org/std/option/](https://doc.rust-lang.org/std/option/) - O tipo Option é essencial para lidar com valores que podem ou não estar presentes.
* **`Result` type:** [https://doc.rust-lang.org/std/result/](https://doc.rust-lang.org/std/result/) - O tipo Result é essencial para lidar com operações que podem falhar.

Esses exercícios cobrem uma boa variedade de casos de uso para extratores, desde os mais simples até os mais avançados, incluindo estado mutável e combinação de diferentes tipos de extração. Eles também introduzem alguns conceitos importantes do Rust, como `Option`, `Result`, `Mutex` e o uso de bibliotecas externas como `serde`.