
---

- [ ] **1. Propagando `io::Error`**

    **Título:** Simulando um Erro de Arquivo

    **Contexto:** Você está trabalhando com arquivos e precisa lidar com a possibilidade de o arquivo não ser encontrado.

    **Objetivos de Aprendizagem:**

    *   Propagar erros usando o operador `?`.
    *   Entender como o Actix Web lida com `io::Error`.

    **Requisitos:**

    *   Crie um handler que tente abrir um arquivo chamado `arquivo_inexistente.txt`.
    *   Use `NamedFile::open()` e o operador `?` para propagar o erro.
    *   O handler deve responder à rota `/arquivo`.
    * Não é necessário criar o arquivo. O objetivo é simular o erro.

    **Resposta Esperada:** Ao acessar `/arquivo`, você deve receber um erro 500 Internal Server Error, porque `io::Error` tem uma implementação padrão de `ResponseError`.

    **Exemplos:**

    *   Use `actix_files::NamedFile`.
    *   O tipo de retorno do handler deve ser `io::Result<NamedFile>`.

    **Recursos:**

    *   [Handling Errors in Actix Web](https://actix.rs/docs/errors/)
    *   [`std::io::Error`](https://doc.rust-lang.org/std/io/struct.Error.html)

---

- [ ] **2. Erro Personalizado (Simples)**

    **Título:** Definindo um Tipo de Erro Customizado

    **Contexto:** Você quer ter mais controle sobre as mensagens de erro retornadas pela sua API.

    **Objetivos de Aprendizagem:**

    *   Criar um tipo de erro personalizado (enum).
    *   Implementar `Display` para o tipo de erro.
    *   Implementar `ResponseError` para o tipo de erro.

    **Requisitos:**

    *   Crie um enum `MeuErro` com uma variante `RecursoNaoEncontrado`.
    *   Implemente `Display` para `MeuErro`, formatando a mensagem de erro.
    *   Implemente `ResponseError` para `MeuErro`, retornando um status 404 Not Found para `RecursoNaoEncontrado`.
    *   Crie um handler que sempre retorne `Err(MeuErro::RecursoNaoEncontrado)`.
    *   O handler deve responder à rota `/erro`.

    **Resposta Esperada:** Ao acessar `/erro`, você deve receber um erro 404 Not Found com a mensagem de erro que você definiu.

    **Exemplos:**

    *   Use `derive_more` para facilitar as implementações.
    *   O tipo de retorno do handler deve ser `Result<impl Responder, MeuErro>`.

    **Recursos:**
    * [`derive_more` crate](https://crates.io/crates/derive_more)

---

- [ ] **3. Erro Personalizado (Completo)**

    **Título:** Modelando Diferentes Tipos de Erro

    **Contexto:** Sua aplicação tem diferentes tipos de erro que precisam ser tratados de forma distinta.

    **Objetivos de Aprendizagem:**

    *   Criar um enum de erro com múltiplas variantes.
    *   Usar `#[display(fmt = "...")]` para formatar mensagens de erro.
    *   Mapear diferentes tipos de erro para diferentes códigos de status HTTP.

    **Requisitos:**

    *   Crie um enum `ErroApp` com as seguintes variantes:
        *   `UsuarioInvalido`: Com uma mensagem de erro descritiva. Status 400.
        *   `RecursoNaoEncontrado`: Com uma mensagem de erro descritiva. Status 404.
        *   `ErroInterno`:  Com uma mensagem genérica ("Erro interno do servidor"). Status 500.
    *   Implemente `Display` e `ResponseError` para `ErroApp`.
    *   Crie um handler que receba um parâmetro de query chamado `tipo_erro`.
    *   Com base no valor de `tipo_erro`, retorne um dos erros definidos em `ErroApp`.
    *   O handler deve responder à rota `/erro_variado`.

    **Resposta Esperada:**

    *   `/erro_variado?tipo_erro=usuario`: 400 Bad Request com a mensagem de erro de usuário inválido.
    *   `/erro_variado?tipo_erro=recurso`: 404 Not Found com a mensagem de recurso não encontrado.
    *   `/erro_variado?tipo_erro=interno`: 500 Internal Server Error com a mensagem genérica.
    *   `/erro_variado`: Algum erro consistente (400 ou 500, dependendo da sua escolha para o caso padrão).

    **Exemplos:**

    *   Use `web::Query` e `Option<String>` para obter o parâmetro.
    *   Use `match` para determinar qual erro retornar.

---

- [ ] **4. Usando `map_err`**

    **Título:** Convertendo Erros com `map_err`

    **Contexto:** Você tem uma função que retorna um tipo de erro e quer convertê-lo para um erro HTTP antes de retorná-lo do handler.

    **Objetivos de Aprendizagem:**

    *   Usar `map_err` para transformar erros.
    *   Entender a diferença entre erros internos e erros de aplicação.

    **Requisitos:**

    *   Crie uma função `simular_falha` que sempre retorne `Err("Falha simulada")`.
    *   Crie um handler que chame `simular_falha`.
    *   Use `map_err` para converter o erro retornado por `simular_falha` em um `ErrorBadRequest`.
    *   O handler deve responder à rota `/falha`.

    **Resposta Esperada:** Ao acessar `/falha`, você deve receber um erro 400 Bad Request com a mensagem "Falha simulada".

    **Exemplos:**

    *   Use `error::ErrorBadRequest`.
    * O retorno deve ser:  `Result<impl Responder, actix_web::Error>`

---

- [ ] **5. Erros de Usuário vs. Erros Internos**

    **Título:** Separando Responsabilidades

    **Contexto:** Você quer organizar seus erros em categorias distintas para facilitar o tratamento e a apresentação.

    **Objetivos de Aprendizagem:**

    *   Definir enums separados para erros de usuário e erros internos.
    *   Implementar `ResponseError` para ambos os tipos de erro.
    *   Escolher mensagens de erro apropriadas para cada categoria.

    **Requisitos:**

    *   Crie um enum `ErroUsuario` com variantes para erros comuns de entrada do usuário (ex: `CampoObrigatorioAusente`, `FormatoInvalido`).
    *   Crie um enum `ErroInterno` com variantes para erros internos (ex: `FalhaNoBancoDeDados`, `ErroDeConfiguracao`).
    *   Implemente `ResponseError` para ambos os enums.
        *   `ErroUsuario`: Retorne códigos de status 4xx (ex: 400, 422) e mensagens descritivas.
        *   `ErroInterno`: Retorne código de status 500 e uma mensagem genérica ("Erro interno do servidor").
    *   Crie um handler que, dependendo de alguma condição (ex: um parâmetro de query), retorne ou um `ErroUsuario` ou um `ErroInterno`.
    *   O handler deve responder à rota `/erro_categorizado`.

    **Resposta Esperada:**

    *   Acessando a rota com condições que levem a um `ErroUsuario`, você deve receber um erro 4xx com uma mensagem descritiva.
    *   Acessando a rota com condições que levem a um `ErroInterno`, você deve receber um erro 500 com a mensagem genérica.

    **Exemplos:**
     *   Use `match` ou `if/else` para decidir qual tipo de erro retornar.
     * Crie as Struct com derive: `#[derive(Debug, Display, Error)]`

---

- [ ] **6. Combinando Diferentes Tipos de Erro**

    **Título:** Unificando o Tratamento de Erros

    **Contexto:** Você tem um handler que pode retornar tanto erros de usuário quanto erros internos, e quer tratá-los de forma unificada.

    **Objetivos de Aprendizagem:**

    *   Criar um enum que englobe diferentes tipos de erro (de usuário e internos).
    *   Usar esse enum unificado como o tipo de erro do `Result` retornado pelo handler.

    **Requisitos:**

    *   Reutilize os enums `ErroUsuario` e `ErroInterno` do exercício anterior.
    *   Crie um novo enum `ErroApp` que tenha duas variantes:
        *   `Usuario(ErroUsuario)`
        *   `Interno(ErroInterno)`
    *   Implemente `ResponseError` para `ErroApp`, delegando a implementação para os enums internos (`ErroUsuario` e `ErroInterno`).
        *   Dica: Use `match` dentro de `status_code` e `error_response` para chamar os métodos correspondentes dos enums internos.
    *   Crie um handler que possa retornar tanto `ErroUsuario` quanto `ErroInterno`, mas use `ErroApp` como o tipo de erro do `Result`.
    * O handler deve responder a rota `/erros`.

    **Resposta Esperada:**

    *   O comportamento deve ser semelhante ao do exercício anterior, mas agora você está usando um único tipo de erro (`ErroApp`) para representar todos os erros possíveis.

    **Exemplos:**
    *  Exemplo de como implementar `ResponseError` para o `ErroApp`:

    ```rust
    impl ResponseError for ErroApp {
        fn status_code(&self) -> StatusCode {
            match self {
                ErroApp::Usuario(e) => e.status_code(), // Delega para ErroUsuario
                ErroApp::Interno(e) => e.status_code(), // Delega para ErroInterno
            }
        }
        fn error_response(&self) -> HttpResponse {
            match self {
                ErroApp::Usuario(e) => e.error_response(), // Delega para ErroUsuario
                ErroApp::Interno(e) => e.error_response(), // Delega para ErroInterno
            }
        }
    }
    ```

---
**Recursos Adicionais (para todos os exercícios):**
 * **derive_more crate:** Ja foi mensionado 
*   **Error Handling in Actix Web:** [https://actix.rs/docs/errors/](https://actix.rs/docs/errors/)
*   **The Rust Programming Language - Error Handling:** [https://doc.rust-lang.org/book/ch09-00-error-handling.html](https://doc.rust-lang.org/book/ch09-00-error-handling.html)

Esses exercícios cobrem os principais conceitos de tratamento de erros no Actix Web, desde a propagação de erros simples até a criação de tipos de erro personalizados e a organização de erros em categorias. Eles também incentivam a pensar sobre a diferença entre erros que são destinados ao usuário final e erros que representam problemas internos da aplicação.