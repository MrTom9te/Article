# Tópicos Avançados e Melhores Práticas em Tera: Dominando a Arte dos Templates

Você já percorreu uma longa jornada no aprendizado do Tera! Agora, vamos consolidar seu conhecimento com alguns tópicos avançados e, mais importante, discutir as *melhores práticas* para criar templates limpos, eficientes e fáceis de manter.

Autoescaping

*Autoescaping* é um mecanismo de segurança que *automaticamente* escapa caracteres especiais em strings para evitar ataques de Cross-Site Scripting (XSS). Quando o autoescaping está ativado, o Tera substitui caracteres como `<`, `>`, `&`, `"`, `'` por suas entidades HTML correspondentes (`&lt;`, `&gt;`, `&amp;`, `&quot;`, `&#x27;`).

Por que o Autoescaping é Importante?

Imagine que você tem um template que exibe comentários de usuários:

```html
<p>{{ comentario }}</p>
```

Se um usuário mal-intencionado inserir um comentário como `<script>alert('XSS!')</script>`, e você *não* usar autoescaping, esse script será executado no navegador de outros usuários que visitarem a página. Isso é um ataque XSS.

Com autoescaping, o Tera transformará o comentário em:

```html
&lt;script&gt;alert(&#x27;XSS!&#x27;)&lt;/script&gt;
```

O que, no navegador, será exibido como texto, e não executado como código.

Configurando o Autoescaping

Por padrão, o Tera *habilita* o autoescaping para arquivos com as extensões `.html`, `.htm` e `.xml`. Você pode controlar esse comportamento usando o método `autoescape_on`:

```rust
use tera::Tera;

fn main() -> Result<(), tera::Error> {
    let mut tera = Tera::new("templates/**/*")?;

    // Escapar apenas arquivos .html.twig:
    tera.autoescape_on(vec![".html.twig"]);

    // Desabilitar completamente o autoescaping (NÃO RECOMENDADO!):
    tera.autoescape_on(vec![]);

    Ok(())

}
```

Desabilitando o Autoescaping (Quando Necessário)

Em situações *muito específicas*, você pode precisar desabilitar o autoescaping. Por exemplo, se você tiver certeza absoluta de que o conteúdo de uma variável é HTML seguro (gerado por você, e não por um usuário), você pode usar o filtro `safe`:

```html
{{ conteudo_html_seguro | safe }}
```

**Importante:** Use `safe` com *extrema cautela*.  Só use se você tiver *certeza absoluta* de que o conteúdo é seguro.  Na dúvida, *não* use `safe`.  É melhor escapar demais do que de menos.

Autoescaping Contextual (Limitações)

O Tera *não* realiza autoescaping contextual. Isso significa que ele não analisa o template para determinar se deve escapar JavaScript, CSS ou HTML de maneira diferente. Ele sempre escapa como HTML.

Se você precisar de um escaping mais sofisticado (por exemplo, escapar corretamente atributos HTML, URLs, etc.), você precisará usar funções ou filtros customizados, ou considerar bibliotecas especializadas em sanitização de HTML.

Extend

Se você estiver usando Tera dentro de uma biblioteca ou framework que já usa o Tera, pode existir a necessidade de herdar configurações, filtros personalizados, etc. Nesses casos, o método `extend` é muito útil.

```rust
let mut tera = Tera::new("templates/**/*")?;
// FRAMEWORK_TERA é a instância de Tera do framework
tera.extend(&FRAMEWORK_TERA)?;
```
Se houver conflitos de nomes de templates, filtros, etc., a sua instância de Tera terá precedência.

Recarregamento de Templates (Reloading)

Durante o desenvolvimento, é muito conveniente que o Tera recarregue automaticamente os templates quando você os modifica, sem precisar reiniciar a aplicação. Você pode fazer isso usando o método `full_reload`:

```rust
use tera::Tera;

fn main() -> Result<(), tera::Error >{
    let mut tera = Tera::new("templates/**/*")?;

    // Em um loop, ou quando você detectar uma mudança nos arquivos:
    tera.full_reload()?;
     Ok(())

}
```

*Importante:* O recarregamento só funciona se você carregar os templates usando um *glob pattern* (`Tera::new("templates/**/*")`).  Ele não funciona se você carregar os templates individualmente.

Templates a partir de Strings

Em algumas situações, você pode querer carregar templates a partir de strings, em vez de arquivos. Por exemplo, se você estiver armazenando templates em um banco de dados, ou se estiver gerando templates dinamicamente.

Você pode fazer isso usando os métodos `add_raw_template` (para um único template) e `add_raw_templates` (para múltiplos templates):

```rust
use tera::Tera;

fn main() -> Result<(), tera::Error>  {
    let mut tera = Tera::default(); // Começa com um Tera vazio.

    tera.add_raw_template("hello.html", "<h1>Olá, {{ name }}!</h1>")?;

    tera.add_raw_templates(vec![
        ("base.html", "{% block content %}{% endblock %}"),
        ("index.html", "{% extends \"base.html\" %}{% block content %}Olá!{% endblock %}"),
    ])?;
     Ok(())
}
```

Renderização One-off

Se você precisar renderizar um template *uma única vez*, sem criar um objeto `Tera`, você pode usar a função `Tera::one_off`:

```rust
use tera::{Context, Tera};

fn main() -> Result<(), tera::Error> {
    let template = "<h1>Olá, {{ name }}!</h1>";
    let mut context = Context::new();
    context.insert("name", "Mundo");

    let rendered = Tera::one_off(template, &context, true)?; // true para autoescaping
    println!("{}", rendered);
    Ok(())
}
```
Tratamento de Erros

Erros podem acontecer em templates: variáveis indefinidas, filtros aplicados a tipos incorretos, sintaxe inválida, etc.  É importante tratar esses erros de forma adequada.

*   **Erros de Parsing:**  O método `Tera::new` retorna um `Result`. Se houver erros de sintaxe nos seus templates, você receberá um erro.
*   **Erros de Renderização:**  O método `tera.render` também retorna um `Result`.  Erros durante a renderização (como variáveis indefinidas) resultarão em um erro.
*  **Função Throw**: A função `throw` pode ser usada dentro do template para gerar erros propositalmente.
*   **Boas Práticas:**
    *   Use `{% if variavel is defined %}` para verificar se uma variável existe antes de usá-la.
    *   Use filtros e testes com cuidado, verificando se os tipos de dados são compatíveis.
    *   Em desenvolvimento, exiba os erros detalhadamente para facilitar a depuração.
    *   Em produção, registre os erros e exiba uma mensagem genérica para o usuário (não exponha detalhes internos).

Melhores Práticas

Aqui estão algumas dicas para escrever templates Tera de alta qualidade:

1.  **Mantenha os Templates Simples:**  Templates devem ser focados na *apresentação*, não na lógica de negócios.  Evite lógica complexa dentro dos templates.  Se você se encontrar escrevendo muitos `if`s aninhados, loops complicados ou expressões longas, provavelmente é hora de mover parte dessa lógica para o seu código Rust.

2.  **Use Nomes Descritivos:**  Use nomes claros e descritivos para variáveis, macros, funções e arquivos de template. Isso torna o código mais fácil de entender.

3.  **Organize seus Templates:**  Use uma estrutura de diretórios consistente (como a discutida no Artigo 5).

4.  **Reutilize Código:**  Use macros, includes e herança de templates para evitar repetição.

5.  **Comente seu Código (quando necessário):**  Use comentários (`{# ... #}`) para explicar partes não óbvias do seu template.  Mas lembre-se: um código limpo e bem organizado geralmente precisa de menos comentários.

6.  **Formate seu Código:**  Use indentação consistente para tornar a estrutura do seu template mais clara. A maioria dos editores de código tem ferramentas para formatar HTML automaticamente.

7.  **Teste seus Templates:**  Embora testar templates possa ser mais desafiador do que testar código Rust, é importante verificar se seus templates estão gerando a saída esperada.  Você pode escrever testes que renderizam seus templates com diferentes contextos e verificam o HTML resultante.

8.  **Evite Lógica de Negócios em Templates:**  Idealmente, seus templates devem receber *todos* os dados de que precisam já pré-processados do seu código Rust.  Evite fazer consultas a bancos de dados, chamadas de API ou cálculos complexos dentro dos templates.

9.  **Use `set` com Moderação:** Embora `set` possa ser útil para criar variáveis auxiliares, o uso excessivo pode tornar o template mais difícil de entender.  Considere se a lógica não seria melhor movida para o código Rust.

10. **Prefira Filtros e Funções Customizadas:** Se você se encontrar repetindo a mesma lógica de manipulação de dados em vários templates, crie um filtro ou função customizada.

Exemplo de Boas Práticas

Em vez de:

```html
{% if user.is_authenticated %}
    {% if user.is_admin %}
        <h1>Olá, Administrador {{ user.first_name | upper }}!</h1>
    {% else %}
        <h1>Olá, {{ user.first_name }}!</h1>
    {% endif %}
{% else %}
    <h1>Olá, Visitante!</h1>
{% endif %}
```

Faça o pré-processamento no Rust:

```rust
// No seu código Rust:

#[derive(Serialize)]
struct Context {
    saudacao: String,
}

fn render_pagina(user: Option<User>) -> Result<String, tera::Error> {
    let mut tera = Tera::new("templates/**/*")?;
    let context = match user {
        Some(u) if u.is_admin => Context {
            saudacao: format!("Olá, Administrador {}!", u.first_name.to_uppercase()),
        },
        Some(u) => Context {
            saudacao: format!("Olá, {}!", u.first_name),
        },
        None => Context {
            saudacao: "Olá, Visitante!".to_string(),
        },
    };
    tera.render("pagina.html", &tera::Context::from_serialize(context)?)
}
```

E no template (`pagina.html`):

```html
<h1>{{ saudacao }}</h1>
```

Muito mais limpo!

Conclusão

Este artigo cobriu tópicos avançados como autoescaping, recarregamento de templates e renderização a partir de strings, além de apresentar as melhores práticas para escrever templates Tera de alta qualidade.  Com este conhecimento, você está pronto para criar aplicações web (ou qualquer outro tipo de projeto que use templates) robustas, seguras e fáceis de manter. Lembre-se sempre: a chave para um bom template é a simplicidade e a clareza.  Mantenha a lógica no código Rust e use o Tera para apresentar seus dados de forma elegante e eficiente.
