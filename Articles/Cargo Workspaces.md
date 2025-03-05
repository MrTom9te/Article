# Cargo Workspaces: Organizando Projetos Rust em Escala

Os Cargo Workspaces são uma ferramenta poderosa para gerenciar múltiplos pacotes relacionados em projetos Rust. Este artigo explora como os workspaces funcionam, suas vantagens, boas práticas e algumas curiosidades que tornam esta funcionalidade essencial para projetos de médio e grande porte.

## O que são Cargo Workspaces?

Um workspace no Cargo é uma coleção de pacotes Rust (chamados de *members*) que são gerenciados em conjunto, compartilhando configurações comuns e artefatos de compilação. 

Conceitualmente, um workspace pode ser comparado a um "guarda-chuva organizacional" que engloba múltiplos projetos relacionados, permitindo que sejam desenvolvidos em sincronia enquanto mantêm sua modularidade.

## Características Fundamentais

Os workspaces do Cargo apresentam as seguintes características principais:

- **Arquivo `Cargo.lock` compartilhado**: Todos os pacotes usam o mesmo arquivo de lock, garantindo compatibilidade entre as versões de dependências.
- **Diretório `target` compartilhado**: Todos os artefatos de compilação vão para um diretório único, evitando recompilação desnecessária.
- **Comandos unificados**: É possível executar comandos como `cargo check --workspace` que afetam todos os pacotes simultaneamente.
- **Metadata compartilhada**: É possível definir metadados comuns para todos os pacotes através da seção `workspace.package`.

## Como Criar um Workspace

Para criar um workspace, você precisa de um arquivo `Cargo.toml` na raiz do projeto com uma seção `[workspace]`:

```toml
[workspace]
resolver = "2"  # Recomendado usar o resolver mais recente
members = [
    "pacote1",
    "pacote2",
    "pacotes/*"  # Suporta padrões glob
]
```

### Exemplo Prático

Vamos criar um workspace simples com uma biblioteca e um binário:

```bash
# Criar diretório do workspace
mkdir meu_workspace
cd meu_workspace

# Criar o arquivo Cargo.toml do workspace
echo '[workspace]
resolver = "2"
members = ["app", "lib_core"]' > Cargo.toml

# Criar o pacote binário
cargo new app

# Criar o pacote de biblioteca
cargo new lib_core --lib
```

Agora podemos configurar o binário para depender da biblioteca:

```toml
# app/Cargo.toml
[dependencies]
lib_core = { path = "../lib_core" }
```

## Tipos de Workspaces

### 1. Workspace com Pacote Raiz

Quando a seção `[workspace]` é adicionada a um `Cargo.toml` que já possui uma seção `[package]`, este pacote se torna o "pacote raiz" do workspace:

```toml
[workspace]
members = ["componentes/*"]

[package]
name = "meu_app_principal"
version = "0.1.0"
# ...
```

### 2. Workspace Virtual

Um workspace também pode ser criado sem um pacote raiz, apenas com a seção `[workspace]`. Isso é chamado de "manifesto virtual":

```toml
# Cargo.toml na raiz
[workspace]
members = ["lib_a", "lib_b", "app"]
resolver = "2"
```

Este formato é útil quando não há um pacote "principal" ou quando você deseja manter todos os pacotes organizados em diretórios separados com igual importância.

## Compartilhamento de Configurações

Uma funcionalidade poderosa introduzida recentemente (Rust 1.64+) é a capacidade de compartilhar configurações entre os membros do workspace:

### Metadados de Pacote Compartilhados

```toml
# Cargo.toml do workspace
[workspace.package]
version = "1.2.3"
authors = ["Equipe Rust <equipe@exemplo.com>"]
edition = "2021"
license = "MIT OR Apache-2.0"
```

Nos pacotes membros, você pode herdar estas configurações:

```toml
# lib_core/Cargo.toml
[package]
name = "lib_core"
version.workspace = true  # Usa a versão definida no workspace
authors.workspace = true  # Usa os autores definidos no workspace
edition.workspace = true
license.workspace = true
```

### Dependências Compartilhadas

Você também pode definir dependências comuns no workspace:

```toml
# Cargo.toml do workspace
[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
log = "0.4"
```

E usá-las nos pacotes:

```toml
# lib_core/Cargo.toml
[dependencies]
serde.workspace = true
log.workspace = true
```

### Lints Compartilhados (Rust 1.74+)

```toml
# Cargo.toml do workspace
[workspace.lints.rust]
unsafe_code = "forbid"
```

Nos pacotes:

```toml
# lib_core/Cargo.toml
[lints]
workspace = true
```

## Curiosidades Sobre Workspaces

### 1. Resolver de Dependências

O campo `resolver = "2"` é obrigatório em workspaces virtuais (sem pacote raiz), mas opcional em workspaces com pacote raiz. Ele determina como o Cargo resolverá conflitos de versões de dependências. O resolver "2" (introduzido no Rust 1.51) é recomendado para todos os projetos novos.

### 2. Diretório `target` Compartilhado

Quando o Cargo compila um workspace, ele não cria um diretório `target` para cada pacote, mas usa um único diretório na raiz do workspace. Isso tem duas vantagens principais:

- **Economia de espaço em disco**: Os artefatos intermediários não são duplicados.
- **Compilação mais rápida**: O Cargo reutiliza os artefatos entre pacotes, evitando recompilações desnecessárias.

### 3. Package Selection

Ao executar comandos no workspace, você pode selecionar pacotes específicos:

- `cargo build -p pacote1` - Compila apenas o pacote1
- `cargo build --workspace` - Compila todos os pacotes
- `cargo build` (na raiz) - Usa os `default-members` ou todos os pacotes

### 4. "Default Members"

Em um workspace virtual, você pode especificar quais membros devem ser usados por padrão quando nenhum pacote é explicitamente selecionado:

```toml
[workspace]
members = ["pacote1", "pacote2", "pacote3/*"]
default-members = ["pacote1", "pacote3/foo"]
```

### 5. Versões Únicas de Dependências Externas

Um workspace garante que todos os pacotes usem a mesma versão de uma dependência externa. Por exemplo, se vários pacotes dependem de `serde`, todos usarão a mesma versão (desde que eles especifiquem requisitos de versão compatíveis).

### 6. Metadados para Ferramentas Externas

A seção `workspace.metadata` permite armazenar informações para ferramentas externas:

```toml
[workspace.metadata.ferramentax]
config = "path/to/config"
```

## Boas Práticas

### 1. Estrutura de Diretórios Recomendada

Para workspaces de médio a grande porte, uma estrutura comum é:

```
meu_projeto/
├── Cargo.toml (workspace)
├── Cargo.lock
├── .gitignore
├── apps/
│   ├── app1/
│   └── app2/
├── libs/
│   ├── core/
│   ├── utils/
│   └── api/
└── target/
```

Com este formato no Cargo.toml:

```toml
[workspace]
members = [
    "apps/*",
    "libs/*"
]
```

### 2. Versionamento Consistente

É recomendável manter todos os pacotes com a mesma versão em um workspace, facilitando o rastreamento de releases. Isso pode ser feito com `workspace.package.version`.

### 3. Evite Ciclos de Dependência

Embora tecnicamente possíveis em Rust (usando recursos como `[patch]`), ciclos de dependência tornam os projetos difíceis de manter. Prefira uma estrutura direcionada acíclica de dependências.

### 4. Agrupar por Propósito

Agrupe seus pacotes por propósito ou domínio, não apenas por tipo. Por exemplo, se seu projeto tem um componente de autenticação, você pode ter:

```
auth/
├── api/       (biblioteca de API pública)
├── core/      (lógica principal)
└── cli/       (ferramenta de linha de comando)
```

### 5. Use o Feature Flag System

Em vez de criar muitos pacotes pequenos, considere usar o sistema de feature flags do Cargo para partes opcionais da funcionalidade dentro de pacotes maiores.

## Casos de Uso Comuns

### 1. Aplicações com Múltiplas Interfaces

Imagine um serviço com diferentes interfaces:

```
meu_servico/
├── libs/core/      (lógica principal)
├── apps/cli/       (interface de linha de comando)
├── apps/api/       (API web)
└── apps/desktop/   (aplicação desktop)
```

### 2. Bibliotecas com Componentes Opcionais

Ideal para bibliotecas grandes que desejam oferecer componentes opcionais:

```
minha_lib/
├── core/      (funcionalidades principais)
├── extras/    (recursos adicionais)
└── compat/    (camadas de compatibilidade)
```

### 3. Monorepos

Para equipes que preferem manter todo seu código em um único repositório:

```
empresa/
├── produtos/produto1/
├── produtos/produto2/
├── libs/compartilhadas/
└── tools/internas/
```

## Comparação com Outras Linguagens

- **JavaScript/Node.js**: Workspaces são similares aos workspaces do Yarn/npm, mas com integração mais profunda no sistema de build.
- **Golang**: Semelhante a módulos Go em um monorepo, mas com gerenciamento mais centralizado de dependências.
- **C#/.NET**: Análogo às soluções .NET, onde múltiplos projetos são agrupados.

## Considerações de Performance

### Compilação Paralela

Por padrão, o Cargo compila pacotes em paralelo quando possível. Em workspaces complexos, você pode ajustar isso com a variável de ambiente `CARGO_BUILD_JOBS`.

### Impacto na IDE

Em workspaces grandes, IDEs como VS Code com rust-analyzer podem consumir mais recursos. Considere:

- Usar a configuração `rust-analyzer.procMacro.enable` apenas para os pacotes em que você está trabalhando ativamente.
- Configurar `rust-analyzer.cargo.buildScripts.enable` para `false` em projetos muito grandes.

## Quando Usar (e Quando Não Usar) Workspaces

### Use Workspaces Quando:

- Seu projeto tem múltiplos componentes relacionados
- Os componentes são desenvolvidos em conjunto e frequentemente mudam juntos
- Deseja compartilhar configurações e dependências
- A consistência entre componentes é crucial

### Não Use Workspaces Quando:

- Você tem apenas um pequeno pacote
- Os pacotes têm ciclos de release completamente diferentes
- Os pacotes são desenvolvidos por equipes diferentes com pouca coordenação

## Ferramentas Úteis para Workspaces

- **cargo-hakari**: Otimiza a ordem de compilação em workspaces grandes
- **cargo-workspaces**: Facilita a gestão de versões em workspaces
- **cargo-nextest**: Sistema de teste mais rápido, especialmente útil para workspaces grandes

## Conclusão

Os Cargo Workspaces são uma ferramenta poderosa que permite escalar projetos Rust de forma organizada e eficiente. Ao compartilhar configurações, dependências e artefatos de compilação, os workspaces facilitam o desenvolvimento coordenado de múltiplos pacotes relacionados.

À medida que seu projeto cresce, considere migrar de um único pacote para uma estrutura de workspace - isso tornará seu código mais modular, mais fácil de entender e mais eficiente para compilar e testar.

## Referências

- [Documentação oficial do Cargo sobre Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)
- [The Cargo Book - Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)
- [Rust Programming Language - Cargo Workspaces](https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html)