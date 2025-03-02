# Artigo: Servindo Arquivos Estáticos no Actix-Web (Rust)

## Introdução

Servir arquivos estáticos (como HTML, CSS, JavaScript, imagens, etc.) é uma funcionalidade comum em aplicações web. O Actix-Web oferece diferentes maneiras de servir arquivos estáticos, desde arquivos individuais até diretórios completos. Este artigo explora as diferentes abordagens e suas configurações.

## Arquivos Individuais (`NamedFile`)

Para servir arquivos individuais, o Actix-Web fornece a struct `NamedFile`. Esta é uma maneira segura e eficiente de servir arquivos específicos.

### Exemplo Básico

```rust
use actix_web::{get, web, App, Error, HttpServer};
use actix_files::NamedFile;
use std::path::PathBuf;

#[get("/download/{filename}")]
async fn download(path: web::Path<String>) -> Result<NamedFile, Error> {
    let filename = path.into_inner();
    let path = PathBuf::from("static/").join(filename); // Arquivos na pasta 'static'
    Ok(NamedFile::open(path)?)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(download)
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**
- `NamedFile::open(path)?`: Abre o arquivo no caminho especificado.
- `static/`: Um diretório onde os arquivos estáticos são armazenados.
- O handler retorna um `Result<NamedFile, Error>`.

### ⚠️ Alerta de Segurança
Tome muito cuidado ao usar expressões regulares como `[.*]` para corresponder caminhos. Isso pode permitir ataques de "path traversal", onde um atacante pode acessar arquivos fora do diretório pretendido usando `../` na URL.

## Servindo Diretórios (`Files`)

Para servir múltiplos arquivos de um diretório, use a struct `Files`. Esta é uma maneira mais conveniente de servir um diretório inteiro de arquivos estáticos.

### Exemplo Básico

```rust
use actix_web::{App, HttpServer};
use actix_files::Files;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                Files::new("/static", "static/") // Mapeia /static para o diretório static/
                    .show_files_listing() // Opcional: permite listar arquivos
                    .index_file("index.html") // Opcional: define um arquivo índice
            )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Explicação:**
- `Files::new("/static", "static/")`:
  - Primeiro argumento: prefixo da URL
  - Segundo argumento: caminho do diretório no sistema de arquivos
- `.show_files_listing()`: Habilita a listagem de arquivos (desabilitado por padrão)
- `.index_file("index.html")`: Define um arquivo índice para diretórios

### Estrutura de Diretórios Exemplo
```
seu_projeto/
├── src/
│   └── main.rs
└── static/
    ├── index.html
    ├── styles.css
    ├── images/
    │   └── logo.png
    └── js/
        └── app.js
```

## Configurações Avançadas

Tanto `NamedFile` quanto `Files` podem ser configurados com várias opções para controlar como os arquivos são servidos.

### Configurando `NamedFile`

```rust
use actix_web::{get, web, App, Error, HttpServer};
use actix_files::NamedFile;
use std::path::PathBuf;

#[get("/download/{filename}")]
async fn download(path: web::Path<String>) -> Result<NamedFile, Error> {
    let filename = path.into_inner();
    let path = PathBuf::from("static/").join(filename);
    
    Ok(NamedFile::open(path)?
        .use_last_modified(true) // Usa o timestamp de modificação
        .use_etag(true) // Usa ETag para cache
        .set_content_disposition(
            actix_web::http::header::ContentDisposition::attachment()
                .with_filename(filename)
        ) // Força download
    )
}
```

### Configurando `Files`

```rust
use actix_web::{App, HttpServer};
use actix_files::Files;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                Files::new("/static", "static/")
                    .show_files_listing()
                    .index_file("index.html")
                    .use_last_modified(true)
                    .use_etag(true)
                    .prefer_utf8(true) // Prefere codificação UTF-8
            )
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

**Opções de Configuração Importantes:**

1. **`use_last_modified(bool)`**
   - Controla se o cabeçalho `Last-Modified` será incluído
   - Útil para cache do lado do cliente

2. **`use_etag(bool)`**
   - Habilita/desabilita o uso de ETags
   - Ajuda no cache eficiente

3. **`set_content_disposition()`**
   - Define como o arquivo será apresentado no navegador
   - Opções comuns:
     - `attachment()`: força download
     - `inline()`: tenta exibir no navegador

4. **`show_files_listing()`**
   - Permite listar arquivos em diretórios
   - Cuidado com segurança!

5. **`index_file()`**
   - Define o arquivo padrão para diretórios

## Melhores Práticas

6. **Segurança**
   - Sempre valide e sanitize caminhos de arquivo
   - Evite servir arquivos de diretórios sensíveis
   - Use permissões apropriadas nos arquivos

7. **Performance**
   - Habilite cache (ETags e Last-Modified)
   - Considere usar um CDN para arquivos grandes/frequentes
   - Comprima arquivos estáticos quando possível

8. **Organização**
   - Mantenha arquivos estáticos em um diretório dedicado
   - Use uma estrutura de diretórios lógica
   - Documente a estrutura dos arquivos estáticos

## Problemas Comuns e Soluções

9. **Arquivos Não Encontrados**
   - Verifique os caminhos relativos
   - Confirme as permissões dos arquivos
   - Use `cargo run` do diretório correto

10. **Problemas de MIME Type**
   - O Actix-Web geralmente detecta automaticamente
   - Use `set_content_type()` se necessário

11. **Problemas de Cache**
   - Configure apropriadamente ETags e Last-Modified
   - Use cabeçalhos Cache-Control quando necessário

## Conclusão

O Actix-Web oferece maneiras flexíveis e eficientes de servir arquivos estáticos, seja individualmente com `NamedFile` ou em diretórios com `Files`. A chave é escolher a abordagem correta para seu caso de uso e configurar adequadamente as opções de segurança e performance.

Lembre-se sempre de:
- Priorizar a segurança ao lidar com caminhos de arquivo
- Configurar cache apropriadamente
- Organizar seus arquivos estáticos de forma lógica
- Testar diferentes cenários de uso

Para mais informações, consulte a [documentação oficial do Actix-Files](https://docs.rs/actix-files).