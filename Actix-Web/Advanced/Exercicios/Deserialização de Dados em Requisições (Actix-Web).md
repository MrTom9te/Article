## Nível 1: Fundamentos

### Exercício 1.1: JSON Simples

**Contexto:**

Você está construindo uma API que recebe informações de um usuário (nome e idade) em formato JSON.

**Objetivos de Aprendizagem:**

-   Usar o extrator `web::Json` para desserializar dados JSON.
-   Definir uma struct Rust para representar os dados JSON.
-   Criar um handler que recebe e processa os dados JSON.

**Requisitos:**

-   Crie uma aplicação Actix-Web que responda a requisições POST em `/user` com o seguinte formato JSON:

    ```json
    {
        "name": "Ana",
        "age": 30
    }
    ```

-   O handler deve extrair o nome e a idade e retornar uma resposta formatada como: "Olá, {nome}! Você tem {idade} anos.".

- [ ] Tarefa Completa

**Dicas:**
- Use as anotações `#[derive(Serialize, Deserialize)]`
- Utilize o actix para fazer os testes

### Exercício 1.2:  JSON com Campo Opcional

**Contexto:**

A API do exercício anterior agora precisa aceitar um campo opcional "email".

**Objetivos de Aprendizagem:**

-   Lidar com campos opcionais em dados JSON.
-   Usar `Option<T>` para representar campos opcionais na struct.

**Requisitos:**

-   Modifique a aplicação do exercício 1.1 para que o JSON de entrada possa ter um campo "email" opcional:

    ```json
    {
        "name": "Ana",
        "age": 30,
        "email": "ana@example.com"
    }
    ```

    ou

    ```json
    {
        "name": "Ana",
        "age": 30
    }
    ```

-   O handler deve retornar:
    -   "Olá, {nome}! Você tem {idade} anos. Email: {email}" se o email estiver presente.
    -   "Olá, {nome}! Você tem {idade} anos." se o email não estiver presente.

- [ ] Tarefa Completa

**Dicas:**

-   Use `Option<String>` para o campo `email` na struct.
-   Use `match` ou `if let` para verificar se o `Option` é `Some` ou `None`.

### Exercício 1.3: Formulário HTML

**Contexto:**

Você está construindo um formulário de contato simples em HTML que envia dados para o seu servidor Actix-Web.

**Objetivos de Aprendizagem:**

-   Usar o extrator `web::Form` para desserializar dados de formulário.
-   Entender a diferença entre dados JSON e dados de formulário.

**Requisitos:**

-   Crie uma aplicação Actix-Web que responda a requisições POST em `/contact` com dados de formulário (application/x-www-form-urlencoded):
    -   Campos: `name`, `email`, `message`
-   O handler deve retornar uma resposta formatada como: "Mensagem de {name} ({email}): {message}".

- [ ] Tarefa Completa

**Dicas:**

-   Crie uma struct que represente os dados do formulário e use `#[derive(Deserialize)]`.
-   Você pode testar usando ferramentas como `curl` ou Postman, ou criando um arquivo HTML simples com um formulário.

Exemplo de como testar com curl:

```bash
curl -X POST -d "name=Fulano&email=fulano@example.com&message=Olá!" http://localhost:8080/contact
```

## Nível 2: Integração de Conceitos

### Exercício 2.1:  Validando JSON com `serde`

**Contexto:**

Você quer garantir que a idade do usuário na API do exercício 1.1 seja um número positivo.

**Objetivos de Aprendizagem:**

-   Usar atributos do `serde` para validação básica.
-   Lidar com erros de desserialização.

**Requisitos:**

-   Modifique a aplicação do exercício 1.1.
-   Use atributos do `serde` para garantir que a idade seja maior que zero.  (Dica: pesquise por `#[serde(validate = ...)]` ou uma abordagem similar, como criar uma função customizada e chamá-la com `#[serde(deserialize_with = ...)]`).
-   Se a idade for inválida, retorne um erro 400 (Bad Request) com uma mensagem de erro apropriada.

- [ ] Tarefa Completa

**Dicas:**
-   Você pode retornar um `Result` do handler, onde o `Err` é um tipo de erro do Actix-Web (como `actix_web::Error`).
-   Use `HttpResponse::BadRequest().body(...)` para retornar o erro 400.

### Exercício 2.2:  Lendo o Corpo Manualmente e Limitando o Tamanho

**Contexto:**

Você precisa receber um arquivo de texto pequeno via POST, mas quer limitar o tamanho do arquivo para evitar abusos.

**Objetivos de Aprendizagem:**

-   Ler o corpo da requisição manualmente usando `web::Payload`.
-   Limitar o tamanho máximo do corpo da requisição.
-   Lidar com erros de forma apropriada.

**Requisitos:**

-   Crie uma aplicação que responda a POST em `/upload`.
-   O corpo da requisição deve ser um texto simples (não JSON).
-   O tamanho máximo do corpo deve ser 1KB (1024 bytes).
-   Se o tamanho exceder o limite, retorne um erro 400 (Bad Request).
-   Se o tamanho estiver dentro do limite, retorne o texto recebido.

- [ ] Tarefa Completa
**Dicas:**

- Use o exemplo de carregamento manual do artigo como base.
- Use `BytesMut` para armazenar os bytes recebidos.
- Verifique o tamanho do `BytesMut` a cada iteração do loop.

### Exercício 2.3:  Combinando JSON e Formulário

**Contexto:**

Você tem um endpoint que pode receber dados tanto em formato JSON quanto em formato de formulário.

**Objetivos de Aprendizagem:**

-   Lidar com múltiplos tipos de conteúdo em um único handler.
-   Usar um enum para representar os diferentes tipos de dados possíveis.

**Requisitos:**

-   Crie uma aplicação que responda a POST em `/data`.
-   O endpoint deve aceitar tanto `application/json` quanto `application/x-www-form-urlencoded`.
-   Os dados, em ambos os formatos, devem ter os campos: `name` (string) e `value` (inteiro).
-   Crie um enum `Data` com duas variantes: `JsonData` e `FormData`, cada uma contendo os campos `name` e `value`.
-   O handler deve usar um `match` para determinar o tipo de conteúdo e extrair os dados para o enum `Data`.
-   Retorne uma resposta formatada como: "Recebido: {tipo}, Nome: {name}, Valor: {value}".

- [ ] Tarefa Completa

**Dicas:**

-   Você precisará usar um `if let` ou `match` com `req.content_type()` para verificar o tipo de conteúdo.
-   Dentro de cada `if` ou `match` case, use o extrator apropriado (`web::Json` ou `web::Form`).
-  Use um novo tipo, que pode ser um enum, que implemente a trait `FromRequest`. Esse novo tipo deve conter os extratores `Json` e `Form`.

## Nível 3: Projetos Práticos

### Exercício 3.1: API de Upload de Arquivos (Multipart)

**Contexto:**

Crie uma API que permita o upload de arquivos.

**Objetivos de Aprendizagem:**

-   Usar a crate `actix-multipart` para lidar com requisições multipart.
-   Salvar os arquivos enviados em disco.
-   Lidar com erros (arquivo muito grande, tipo de arquivo inválido, etc.).

**Requisitos:**

-   Crie uma aplicação que responda a POST em `/upload`.
-   O endpoint deve aceitar um campo de formulário chamado "file" que contém o arquivo.
-   Limite o tamanho máximo do arquivo (por exemplo, 10MB).
-   Salve o arquivo em disco com um nome único (você pode usar `uuid::Uuid::new_v4().to_string()`).
-   Retorne o nome do arquivo salvo.
-   Implemente tratamento de erros:
    -   Se o campo "file" não estiver presente, retorne 400.
    -   Se o arquivo for muito grande, retorne 413 (Payload Too Large).
    -   Se ocorrer um erro ao salvar o arquivo, retorne 500 (Internal Server Error).

- [ ] Tarefa Completa

**Dicas:**

-   Consulte a documentação da crate `actix-multipart` para exemplos de uso.
-   Use `tokio::fs::File` para operações de arquivo assíncronas.
-   Use `futures::StreamExt` e `futures::TryStreamExt` para processar o stream multipart.

### Exercício 3.2: API de Download de Arquivos

**Contexto:**
Crie uma api que permita fazer download de arquivos

**Objetivos de Aprendizagem:**

- Realizar download de arquivos salvos

**Requisitos:**
- Crie uma aplicação que responda a GET em /download/{nome_do_arquivo}
- Retorne o arquivo solicitado, caso ele exista
- Caso o arquivo não exista retorne um erro 404

**Dicas:**
- Consulte a documentação do actix-files

- [ ] Tarefa Completa

Estes exercícios cobrem uma boa variedade de cenários de desserialização de dados em Actix-Web, desde os mais simples até os mais complexos, incluindo validação, tratamento de erros e upload de arquivos. Eles são projetados para serem desafiadores e promoverem o aprendizado prático.