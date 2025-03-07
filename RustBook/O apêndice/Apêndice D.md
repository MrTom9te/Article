# Ferramentas Úteis de Desenvolvimento

Neste apêndice, falaremos sobre algumas ferramentas úteis de desenvolvimento que o projeto Rust fornece. Veremos formatação automática, maneiras rápidas de aplicar correções de avisos, um linter e integração com IDEs.

### Formatação Automática com `rustfmt`

A ferramenta `rustfmt` reformata seu código de acordo com o estilo de código da comunidade. Muitos projetos colaborativos usam `rustfmt` para evitar discussões sobre qual estilo usar ao escrever Rust: todos formatam seu código usando a ferramenta.

Para instalar o `rustfmt`, digite o seguinte:

```
$ rustup component add rustfmt
```

Este comando fornece `rustfmt` e `cargo-fmt`, semelhante a como o Rust fornece `rustc` e `cargo`. Para formatar qualquer projeto Cargo, digite o seguinte:

```
$ cargo fmt
```

Executar este comando reformata todo o código Rust no crate atual. Isso deve alterar apenas o estilo do código, não a semântica do código. Para obter mais informações sobre `rustfmt`, consulte sua documentação.

### Corrija Seu Código com `rustfix`

A ferramenta `rustfix` está incluída nas instalações do Rust e pode corrigir automaticamente avisos do compilador que têm uma maneira clara de corrigir o problema que provavelmente é o que você deseja. É provável que você já tenha visto avisos do compilador antes. Por exemplo, considere este código:

*Arquivo: src/main.rs*

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

Aqui, estamos chamando a função `do_something` 100 vezes, mas nunca usamos a variável `i` no corpo do loop `for`. O Rust nos avisa sobre isso:

```
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

O aviso sugere que usemos `_i` como nome: o sublinhado indica que pretendemos que esta variável não seja usada. Podemos aplicar automaticamente essa sugestão usando a ferramenta `rustfix` executando o comando `cargo fix`:

```
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Quando olharmos para *src/main.rs* novamente, veremos que `cargo fix` alterou o código:

*Arquivo: src/main.rs*

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

A variável do loop `for` agora se chama `_i`, e o aviso não aparece mais.

Você também pode usar o comando `cargo fix` para fazer a transição do seu código entre diferentes edições do Rust. As edições são abordadas no Apêndice E.

### Mais Lints com Clippy

A ferramenta Clippy é uma coleção de lints para analisar seu código para que você possa identificar erros comuns e melhorar seu código Rust.

Para instalar o Clippy, digite o seguinte:

```
$ rustup component add clippy
```

Para executar os lints do Clippy em qualquer projeto Cargo, digite o seguinte:

```
$ cargo clippy
```

Por exemplo, digamos que você escreva um programa que usa uma aproximação de uma constante matemática, como pi, como este programa faz:

*Arquivo: src/main.rs*

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Executar `cargo clippy` neste projeto resulta neste erro:

```
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Este erro informa que o Rust já tem uma constante `PI` mais precisa definida e que seu programa seria mais correto se você usasse a constante. Você então alteraria seu código para usar a constante `PI`. O código a seguir não resulta em nenhum erro ou aviso do Clippy:

*Arquivo: src/main.rs*

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

Para obter mais informações sobre o Clippy, consulte sua documentação.

### Integração de IDE Usando `rust-analyzer`

Para ajudar na integração de IDE, a comunidade Rust recomenda o uso de `rust-analyzer`. Esta ferramenta é um conjunto de utilitários centrados no compilador que falam o Protocolo de Servidor de Linguagem (Language Server Protocol), que é uma especificação para IDEs e linguagens de programação se comunicarem entre si. Diferentes clientes podem usar `rust-analyzer`, como o plug-in Rust analyzer para Visual Studio Code.

Visite a página inicial do projeto `rust-analyzer` para obter instruções de instalação e, em seguida, instale o suporte ao servidor de linguagem em sua IDE específica. Sua IDE ganhará habilidades como autocompletar, ir para definição e erros embutidos.
