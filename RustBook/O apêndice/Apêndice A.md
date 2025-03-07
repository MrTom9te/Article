# Palavras-chave (Keywords)

A lista a seguir contém palavras-chave reservadas para uso atual ou futuro pela linguagem Rust. Como tal, elas não podem ser usadas como identificadores (exceto como identificadores brutos, como discutiremos na seção "Identificadores Brutos"). Identificadores são nomes de funções, variáveis, parâmetros, campos de struct, módulos, crates, constantes, macros, valores estáticos, atributos, tipos, traits ou tempos de vida.

## Palavras-chave Atualmente em Uso

A seguir está uma lista de palavras-chave atualmente em uso, com sua funcionalidade descrita.

*   `as` - realizar coerção (cast) primitiva, desambiguar o trait específico que contém um item ou renomear itens em declarações `use`
*   `async` - retornar um `Future` em vez de bloquear a thread atual
*   `await` - suspender a execução até que o resultado de um `Future` esteja pronto
*   `break` - sair de um loop imediatamente
*   `const` - definir itens constantes ou ponteiros brutos constantes
*   `continue` - continuar para a próxima iteração do loop
*   `crate` - em um caminho de módulo, refere-se à raiz do crate
*   `dyn` - despacho dinâmico para um objeto trait
*   `else` - alternativa para construções de fluxo de controle `if` e `if let`
*   `enum` - definir uma enumeração
*   `extern` - vincular uma função ou variável externa
*   `false` - literal booleano falso
*   `fn` - definir uma função ou o tipo ponteiro de função
*   `for` - iterar sobre itens de um iterador, implementar um trait ou especificar um tempo de vida de ordem superior
*   `if` - ramificar com base no resultado de uma expressão condicional
*   `impl` - implementar funcionalidade inerente ou de trait
*   `in` - parte da sintaxe do loop `for`
*   `let` - vincular uma variável
*   `loop` - loop incondicional
*   `match` - comparar um valor com padrões
*   `mod` - definir um módulo
*   `move` - fazer uma closure assumir a propriedade de todas as suas capturas
*   `mut` - denotar mutabilidade em referências, ponteiros brutos ou vinculações de padrão
*   `pub` - denotar visibilidade pública em campos de struct, blocos `impl` ou módulos
*   `ref` - vincular por referência
*   `return` - retornar de uma função
*   `Self` - um alias de tipo para o tipo que estamos definindo ou implementando
*   `self` - sujeito do método ou módulo atual
*   `static` - variável global ou tempo de vida que dura toda a execução do programa
*   `struct` - definir uma estrutura
*   `super` - módulo pai do módulo atual
*   `trait` - definir um trait
*   `true` - literal booleano verdadeiro
*   `type` - definir um alias de tipo ou tipo associado
*   `union` - definir uma união; é apenas uma palavra-chave quando usada em uma declaração de união
*   `unsafe` - denotar código, funções, traits ou implementações inseguras
*   `use` - trazer símbolos para o escopo
*   `where` - denotar cláusulas que restringem um tipo
*   `while` - loop condicionalmente com base no resultado de uma expressão

## Palavras-chave Reservadas para Uso Futuro

As seguintes palavras-chave ainda não têm nenhuma funcionalidade, mas são reservadas pelo Rust para possível uso futuro.

*   `abstract`
*   `become`
*   `box`
*   `do`
*   `final`
*   `gen`
*    `macro`
*   `override`
*   `priv`
*   `try`
*   `typeof`
*   `unsized`
*   `virtual`
*   `yield`

## Identificadores Brutos (Raw Identifiers)

*Identificadores brutos* são a sintaxe que permite usar palavras-chave onde normalmente não seriam permitidas. Você usa um identificador bruto prefixando uma palavra-chave com `r#`.

Por exemplo, `match` é uma palavra-chave. Se você tentar compilar a seguinte função que usa `match` como seu nome:

*Arquivo: src/main.rs*

```rust
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

você receberá este erro:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword

```

O erro mostra que você não pode usar a palavra-chave `match` como identificador da função. Para usar `match` como um nome de função, você precisa usar a sintaxe de identificador bruto, assim:

*Arquivo: src/main.rs*

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Este código será compilado sem erros. Observe o prefixo `r#` no nome da função em sua definição, bem como onde a função é chamada em `main`.

Identificadores brutos permitem que você use qualquer palavra que escolher como identificador, mesmo que essa palavra seja uma palavra-chave reservada. Isso nos dá mais liberdade para escolher nomes de identificadores, além de nos permitir integrar com programas escritos em uma linguagem onde essas palavras não são palavras-chave. Além disso, identificadores brutos permitem que você use bibliotecas escritas em uma edição Rust diferente da que seu crate usa. Por exemplo, `try` não é uma palavra-chave na edição 2015, mas é nas edições 2018, 2021 e 2024. Se você depende de uma biblioteca escrita usando a edição 2015 e tem uma função `try`, você precisará usar a sintaxe de identificador bruto, `r#try` neste caso, para chamar essa função do seu código da edição 2018. Consulte o Apêndice E para obter mais informações sobre edições.