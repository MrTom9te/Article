# Organização e Herança de Templates: Estruturando seus Projetos Tera

À medida que seus projetos crescem, a organização dos seus templates se torna crucial. Uma boa estrutura facilita a manutenção, a reutilização de código e a colaboração em equipe. Além disso, a *herança de templates* é um recurso poderoso que permite criar layouts base e estendê-los em páginas específicas, evitando repetição desnecessária.

Herança

A herança de templates permite que você defina um "esqueleto" comum para suas páginas (o *template base*) e, em seguida, crie *templates filhos* que herdam essa estrutura e modificam apenas as partes que precisam ser diferentes.

Template Base

O template base contém a estrutura geral do seu site, incluindo elementos como:

*   Declaração `<!DOCTYPE html>`
*   Tags `<head>` com metadados, links para CSS, etc.
*   Estrutura básica do `<body>`, como cabeçalho, rodapé, menu de navegação.
*   *Blocos* (`{% block %}`) que definem áreas que os templates filhos podem sobrescrever.

Exemplo (`base.html`):

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>{% block titulo %}Meu Site{% endblock titulo %}</title>
    <link rel="stylesheet" href="/estilo.css">
    {% block head %}{% endblock head %}
</head>
<body>
    <header>
        <h1>Meu Site</h1>
        <nav>
            {# ... menu de navegação ... #}
        </nav>
    </header>

    <main>
        {% block conteudo %}{% endblock conteudo %}
    </main>

    <footer>
        {% block rodape %}&copy; 2023 Meu Site{% endblock rodape %}
    </footer>
</body>
</html>
```

Neste exemplo:

*   `{% block titulo %}Meu Site{% endblock titulo %}`: Define um bloco chamado "titulo". Se um template filho não sobrescrever esse bloco, o título será "Meu Site".
*   `{% block head %}{% endblock head %}`: Um bloco vazio chamado "head". Templates filhos podem adicionar conteúdo aqui (como links para CSS ou JavaScript específicos da página).
*   `{% block conteudo %}{% endblock conteudo %}`: O bloco principal, onde o conteúdo específico de cada página será inserido.
*   `{% block rodape %}&copy; 2023 Meu Site{% endblock rodape %}`: Um bloco para o rodapé, com um conteúdo padrão.

Template Filho

Um template filho usa a tag `{% extends %}` para indicar qual template base ele herda. Em seguida, ele sobrescreve os blocos do template base que precisam ser modificados.

Exemplo (`home.html`):

```html
{% extends "base.html" %}

{% block titulo %}Página Inicial{% endblock titulo %}

{% block head %}
    <link rel="stylesheet" href="/home.css">
{% endblock head %}

{% block conteudo %}
    <h1>Bem-vindo à Página Inicial!</h1>
    <p>Este é o conteúdo da página inicial.</p>
{% endblock conteudo %}
```

Neste exemplo:

*   `{% extends "base.html" %}`: Indica que este template herda de `base.html`.  *Esta deve ser a primeira tag do arquivo*.
*   `{% block titulo %}Página Inicial{% endblock titulo %}`: Sobrescreve o bloco "titulo" do template base.
*   `{% block head %}<link rel="stylesheet" href="/home.css">{% endblock head %}`: Adiciona um link para um arquivo CSS específico da página inicial.
*   `{% block conteudo %}...{% endblock conteudo %}`: Define o conteúdo principal da página.

`super()`

Dentro de um bloco em um template filho, você pode usar a variável `{{ super() }}` para renderizar o conteúdo do bloco correspondente no template pai. Isso é útil quando você quer *adicionar* conteúdo a um bloco, em vez de substituí-lo completamente.

Exemplo (`sobre.html`):

```html
{% extends "base.html" %}

{% block rodape %}
    {{ super() }}
    <p>Informações adicionais do rodapé da página "Sobre".</p>
{% endblock rodape %}
```

Isso renderizará o conteúdo padrão do bloco "rodape" do template base (`&copy; 2023 Meu Site`) seguido do texto adicional.

Nested Blocks

Você pode aninhar blocos dentro de outros blocos, criando hierarquias de herança mais complexas.

```html
// grandparent
{% block hey %}hello{% endblock hey %}

// parent
{% extends "grandparent" %}
{% block hey %}hi and grandma says {{ super() }} {% block ending %}sincerely{% endblock ending %}{% endblock hey %}

// child
{% extends "parent" %}
{% block hey %}dad says {{ super() }}{% endblock hey %}
{% block ending %}{{ super() }} with love{% endblock ending %}
```

Organização de Projetos

Uma estrutura de diretórios bem organizada torna seu projeto mais fácil de entender e manter. Aqui estão algumas sugestões:

*   **Pasta `templates`:**  Coloque todos os seus templates dentro de uma pasta chamada `templates` na raiz do seu projeto (no mesmo nível do `src`).
*   **Subpastas por Funcionalidade:**  Se você tiver muitas páginas, organize-as em subpastas dentro de `templates`. Por exemplo:
    ```
    templates/
        base.html
        home.html
        sobre.html
        produtos/
            index.html
            detalhe.html
        blog/
            index.html
            artigo.html
    ```
*   **Nomenclatura Descritiva:**  Use nomes de arquivos descritivos que indiquem o conteúdo ou a finalidade do template.
*   **Componentes Reutilizáveis:**  Se você tiver blocos de código que se repetem em vários templates, considere criar macros ou usar `include` para evitar duplicação.
*   **Templates Parciais:** Crie templates parciais (ex: `_header.html`, `_footer.html`) para partes do layout que são usadas em vários lugares. Use um underscore (`_`) no início do nome para indicar que esses arquivos não são páginas completas.

Exemplo de Estrutura Completa

```
meu_projeto/
├── Cargo.toml
├── src/
│   └── main.rs
└── templates/
    ├── base.html          <- Template base
    ├── _header.html      <- Template parcial (cabeçalho)
    ├── _footer.html      <- Template parcial (rodapé)
    ├── home.html          <- Página inicial
    ├── about.html         <- Página "Sobre"
    ├── products/          <- Subpasta para produtos
    │   ├── index.html     <- Listagem de produtos
    │   └── detail.html    <- Detalhe do produto
    └── blog/              <- Subpasta para o blog
        ├── index.html     <- Listagem de posts
        └── post.html      <- Post individual

```

Conclusão

A organização e a herança de templates são ferramentas essenciais para criar projetos Tera escaláveis e de fácil manutenção. Com uma estrutura clara e o uso inteligente da herança, você pode evitar repetição de código, manter seus templates organizados e colaborar de forma mais eficiente com outros desenvolvedores (ou designers!). Lembre-se sempre: Templates devem ser limpos e focados na apresentação. A lógica de negócios deve ficar, sempre que possível, no seu código Rust.
