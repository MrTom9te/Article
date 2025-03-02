**1. Servindo Arquivos MD Estáticos Diretamente (Simples):**

   *   **Vantagem:** Extremamente simples de configurar.
   *   **Desvantagem:** Não renderiza o Markdown como HTML. O navegador exibirá o texto bruto do arquivo MD, incluindo a formatação Markdown.
   *   **Como fazer:** Use o recurso `actix-files` para servir diretórios estáticos.

     ```rust
     use actix_files as fs;
     use actix_web::{App, HttpServer};

     #[actix_web::main]
     async fn main() -> std::io::Result<()> {
         HttpServer::new(|| {
             App::new()
                 // Sirva arquivos estáticos da pasta "markdown_files"
                 // no caminho /articles
                 .service(fs::Files::new("/articles", "markdown_files").show_files_listing())
         })
         .bind(("127.0.0.1", 8080))?
         .run()
         .await
     }
     ```

     *   Crie uma pasta chamada `markdown_files` na raiz do seu projeto e coloque seus arquivos `.md` lá.
     *   Ao acessar `http://127.0.0.1:8080/articles/seu_arquivo.md`, o navegador exibirá o conteúdo do arquivo `seu_arquivo.md`.  O `.show_files_listing()` *opcionalmente* permite que você veja uma listagem de arquivos se acessar `/articles/`.

**2. Renderizando Markdown para HTML (Recomendado):**

   *   **Vantagem:**  Renderiza o Markdown corretamente como HTML, permitindo que os usuários vejam o conteúdo formatado como uma página da web.
   *   **Desvantagem:** Requer um pouco mais de código e uma biblioteca para processar o Markdown.
   *   **Como fazer:**

     1.  **Escolha uma biblioteca de Markdown:**  `pulldown-cmark` e `comrak` são duas opções populares e bem mantidas. Adicione a biblioteca escolhida às suas dependências no `Cargo.toml`:

         ```toml
         [dependencies]
         actix-web = "4"  # (ou a versão que você está usando)
         actix-files = "0.6" # (ou a versão que você está usando)
         pulldown-cmark = "0.9" # ou a versão que você está usando
         # OU
         # comrak = "0.18"    # ou a versão que você está usando

         # Se você quiser usar templates (próximo passo), adicione também:
         tera = "1"  # (ou outra engine de templates, como handlebars ou askama)
         serde = { version = "1.0", features = ["derive"] } // Para serializar dados para o template
         ```
     2.  **Crie uma função handler para ler, processar e renderizar o Markdown:**

         ```rust
         use actix_web::{get, web, App, HttpResponse, HttpServer, Responder,  error};
         use pulldown_cmark::{html, Parser}; // Ou use comrak
         use std::fs;
         use std::path::PathBuf;
         use actix_files::NamedFile;
         use thiserror::Error;

         #[derive(Error, Debug)]
         enum MyError {
             #[error("Failed to read file: {0}")]
             IoError(#[from] std::io::Error),
             #[error("File not found")]
             FileNotFound,
         }
         impl error::ResponseError for MyError {}

         // Handler para um artigo específico
         #[get("/articles/{article_name}.html")] // .html é importante para o navegador interpretar como HTML
         async fn article_handler(article_name: web::Path<String>) -> Result<HttpResponse, MyError> {
             let file_path = format!("markdown_files/{}.md", article_name);
             let file_path = PathBuf::from(file_path);

             if !file_path.exists() {
                return Err(MyError::FileNotFound);
             }

             let markdown_input = fs::read_to_string(file_path)?;

             // --- Usando pulldown-cmark ---
             let parser = Parser::new(&markdown_input);
             let mut html_output = String::new();
             html::push_html(&mut html_output, parser);

             // --- Usando comrak (alternativa) ---
             // let arena = comrak::Arena::new();
             // let root = comrak::parse_document(
             //    &arena,
             //    &markdown_input,
             //    &comrak::ComrakOptions::default(),
             // );
             // let mut html_output = Vec::new();
             // comrak::format_html(root, &comrak::ComrakOptions::default(), &mut html_output).unwrap();
             // let html_output = String::from_utf8(html_output).unwrap();


             Ok(HttpResponse::Ok().content_type("text/html; charset=utf-8").body(html_output))
         }

         // Index simples, mostrando links para alguns artigos (exemplo)
         #[get("/")]
         async fn index() -> impl Responder {
            // Poderia ser gerado dinamicamente, lendo os arquivos .md
            let links = r#"
                <!DOCTYPE html>
                <html>
                <head><title>Meu Blog</title></head>
                <body>
                    <h1>Bem-vindo ao meu blog!</h1>
                    <ul>
                        <li><a href="/articles/artigo1.html">Artigo 1</a></li>
                        <li><a href="/articles/artigo2.html">Artigo 2</a></li>
                    </ul>
                </body>
                </html>
            "#;
            HttpResponse::Ok().content_type("text/html; charset=utf-8").body(links)
         }

         // Handler para servir arquivos estáticos (CSS, imagens, etc. - opcional)
         async fn static_files(req: actix_web::HttpRequest) -> Result<NamedFile, actix_web::Error> {
             let path: PathBuf = req.match_info().query("filename").parse().unwrap();
             let file = NamedFile::open(path)?;
             Ok(file)
         }



         #[actix_web::main]
         async fn main() -> std::io::Result<()> {
             HttpServer::new(|| {
                 App::new()
                    .service(index)  // Página inicial
                    .service(article_handler) // Handler para os artigos
                    //Configuração para arquivos estáticos (CSS, etc.) - Opcional
                    .route("/{filename:.*}", web::get().to(static_files))
             })
             .bind(("127.0.0.1", 8080))?
             .run()
             .await
         }

         ```

**3. Usando um Template Engine (Altamente Recomendado para Blogs):**

   *   **Vantagem:** Permite separar o conteúdo (Markdown) da apresentação (HTML).  Você cria um template HTML que define a estrutura geral da sua página (cabeçalho, rodapé, menu, etc.), e o conteúdo Markdown é inserido em um local específico do template. Isso torna a manutenção do seu blog muito mais fácil.
   *   **Desvantagem:**  Requer um pouco mais de configuração.
   *   **Como fazer:**

     1.  **Escolha um template engine:**  `tera`, `handlebars`, e `askama` são opções populares.  O exemplo abaixo usa `tera`.
     2.  **Crie seus templates:**  Crie uma pasta `templates` na raiz do seu projeto.  Dentro dela, crie pelo menos dois arquivos:
         *   `base.html` (ou um nome similar):  Este é o template base, que define a estrutura geral do seu site.
         *   `article.html` (ou um nome similar):  Este template é específico para exibir um artigo.

         **`templates/base.html`:**

         ```html
         <!DOCTYPE html>
         <html>
         <head>
             <title>{{ title }}</title>
             <link rel="stylesheet" href="/static/style.css"> </link>  <!-- Exemplo de CSS -->

         </head>
         <body>
             <header>
                 <h1>Meu Blog</h1>
                 <nav>
                     <!-- Menu de navegação (você pode gerar isso dinamicamente) -->
                 </nav>
             </header>
             <main>
                 {% block content %}{% endblock %}
             </main>
             <footer>
                 <!-- Rodapé -->
             </footer>
         </body>
         </html>
         ```

         **`templates/article.html`:**

         ```html
         {% extends "base.html" %}

         {% block content %}
             {{ content | safe }}
         {% endblock %}
         ```
     3. **Modifique o handler para usar os templates:**
         *   Crie uma instância do `Tera`.
         *   Passe os dados (o HTML gerado a partir do Markdown) para o template.
         *  Renderize o template.

        ```rust
        // ... (imports anteriores)
        use tera::{Tera, Context};
        use serde::Serialize;


        #[derive(Serialize)]
        struct ArticleData {
            title: String, // Adicione um título, se quiser
            content: String,
        }


        #[get("/articles/{article_name}.html")]
        async fn article_handler(
            article_name: web::Path<String>,
            tera: web::Data<Tera>, // Adicione Tera ao estado do aplicativo
        ) -> Result<HttpResponse, MyError> {

            let file_path = format!("markdown_files/{}.md", article_name);
            let file_path = PathBuf::from(file_path);

            if !file_path.exists() {
                return Err(MyError::FileNotFound);
            }

            let markdown_input = fs::read_to_string(file_path)?;
            let parser = Parser::new(&markdown_input);
            let mut html_output = String::new();
            html::push_html(&mut html_output, parser);

            //Cria um ArticleData para passar ao template
            let data = ArticleData {
                title: article_name.to_string(), // Usando o nome do arquivo como título (exemplo)
                content: html_output,
            };

            let mut context = Context::new();
            context.insert("title", &data.title); // Passa o título para base.html
            context.insert("content", &data.content); //Passa o conteúdo para article.html

            let rendered = tera.render("article.html", &context)
                .map_err(|e| MyError::IoError(std::io::Error::new(std::io::ErrorKind::Other, e)))?; // Handle Tera errors

            Ok(HttpResponse::Ok().content_type("text/html; charset=utf-8").body(rendered))
        }


        #[actix_web::main]
        async fn main() -> std::io::Result<()> {
            HttpServer::new(|| {
                // Inicializa o Tera e carrega os templates
                let tera = Tera::new("templates/**/*").expect("Failed to load templates");

                App::new()
                    .app_data(web::Data::new(tera)) // Adiciona Tera ao estado do aplicativo
                    .service(index)
                    .service(article_handler)
                    // ... (rota para arquivos estáticos, se necessário)

            })
            .bind(("127.0.0.1", 8080))?
            .run()
            .await
        }

        ```

**Explicações e Melhorias Importantes:**

*   **`#[get("/articles/{article_name}.html")]`:**  Esta anotação define um *path parameter* (`article_name`).  Actix Web extrairá o nome do artigo da URL e o passará para a função `article_handler`.  A extensão `.html` é importante para que o navegador interprete a resposta como HTML.
*   **`fs::read_to_string`:** Lê o conteúdo do arquivo Markdown como uma string.
*   **`pulldown-cmark` (ou `comrak`)**: Converte o Markdown em HTML.
*   **`HttpResponse::Ok().content_type("text/html; charset=utf-8").body(...)`:**  Constrói a resposta HTTP.  `content_type` define o tipo de conteúdo (HTML) e o conjunto de caracteres (UTF-8, para suportar caracteres especiais).
*   **Tratamento de Erros:** O código agora inclui um enum `MyError` e usa `thiserror` para um tratamento de erros mais robusto.  Isso é *essencial* para aplicativos web reais. Em caso de erro (arquivo não encontrado, erro de leitura), uma resposta de erro apropriada é retornada.
*   **`web::Data<Tera>`:** O template engine `Tera` é armazenado no estado da aplicação (usando `web::Data`). Isso permite que todos os handlers acessem a instância do `Tera` para renderizar templates.
* **Arquivos Estáticos:** Se você tiver arquivos CSS, JavaScript ou imagens, você precisará configurá-los para serem servidos.  O exemplo inclui um handler `static_files`, mas você precisará ajustá-lo para a sua estrutura de diretórios.  Normalmente, você teria uma pasta `static` (ou `public`) e configuraria o Actix para servir arquivos dessa pasta.  Por exemplo:

   ```rust
    // ... dentro de HttpServer::new
    .service(fs::Files::new("/static", "static").show_files_listing())
   ```
   E então coloque seus arquivos CSS em `static/style.css` (por exemplo).

* **Segurança:**
    * **Validação de Entrada:**  Embora o `pulldown-cmark` e o `comrak` sejam geralmente seguros, é uma boa prática sanitizar a entrada do usuário (o conteúdo Markdown) antes de processá-la, especialmente se você permitir que os usuários enviem conteúdo.  Isso ajuda a prevenir ataques de Cross-Site Scripting (XSS).  Você pode usar uma biblioteca como `ammonia` para limpar o HTML gerado.
    * **Paths Absolutos:** Evite usar paths absolutos (como `/home/user/myblog/markdown_files`). Use paths relativos ao diretório do executável ou use `std::env::current_dir()` para construir paths absolutos de forma mais segura.  O exemplo usa paths relativos.
    * **`show_files_listing()`:** Se você usar `show_files_listing()`, tenha cuidado para não expor arquivos sensíveis.

* **Index Dinâmico:** Em um blog real, você provavelmente geraria a página inicial (index) dinamicamente, listando os artigos disponíveis (lendo os nomes dos arquivos `.md` no diretório).

* **Cache:** Para melhorar o desempenho, você pode implementar um sistema de cache. Você poderia armazenar o HTML renderizado em memória (usando um `HashMap` ou uma biblioteca de cache mais sofisticada como `cached`) ou usar um cache HTTP (configurando cabeçalhos `Cache-Control` apropriados).

Este exemplo completo, com templates e tratamento de erros, fornece uma base sólida para construir um blog com Actix Web e Markdown.  Adapte-o às suas necessidades e adicione mais funcionalidades (como paginação, tags, categorias, comentários, etc.) conforme necessário.
