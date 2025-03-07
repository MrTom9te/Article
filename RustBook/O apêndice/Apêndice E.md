# Edições

No Capítulo 1, você viu que `cargo new` adiciona um pouco de metadados ao seu arquivo *Cargo.toml* sobre uma edição. Este apêndice fala sobre o que isso significa!

A linguagem e o compilador Rust têm um ciclo de lançamento de seis semanas, o que significa que os usuários obtêm um fluxo constante de novos recursos. Outras linguagens de programação lançam alterações maiores com menos frequência; o Rust lança atualizações menores com mais frequência. Depois de um tempo, todas essas pequenas mudanças se somam. Mas de lançamento em lançamento, pode ser difícil olhar para trás e dizer: "Uau, entre Rust 1.10 e Rust 1.31, Rust mudou muito!"

A cada dois ou três anos, a equipe Rust produz uma nova *edição* do Rust. Cada edição reúne os recursos que chegaram em um pacote claro com documentação e ferramentas totalmente atualizadas. Novas edições são lançadas como parte do processo usual de lançamento de seis semanas.

As edições servem a diferentes propósitos para diferentes pessoas:

*   Para usuários ativos do Rust, uma nova edição reúne mudanças incrementais em um pacote fácil de entender.
*   Para não usuários, uma nova edição sinaliza que alguns avanços importantes chegaram, o que pode fazer com que valha a pena dar outra olhada no Rust.
*   Para aqueles que desenvolvem o Rust, uma nova edição fornece um ponto de encontro para o projeto como um todo.

No momento da escrita, quatro edições do Rust estão disponíveis: Rust 2015, Rust 2018, Rust 2021 e Rust 2024. Este livro foi escrito usando os idiomas da edição Rust 2024.

A chave `edition` em *Cargo.toml* indica qual edição o compilador deve usar para seu código. Se a chave não existir, o Rust usa `2015` como o valor da edição por motivos de compatibilidade com versões anteriores.

Cada projeto pode optar por uma edição diferente da edição padrão de 2015. As edições podem conter alterações incompatíveis, como a inclusão de uma nova palavra-chave que conflita com identificadores no código. No entanto, a menos que você opte por essas alterações, seu código continuará a compilar mesmo que você atualize a versão do compilador Rust que você usa.

Todas as versões do compilador Rust suportam qualquer edição que existia antes do lançamento desse compilador e podem vincular crates de quaisquer edições suportadas. As alterações de edição afetam apenas a maneira como o compilador analisa inicialmente o código. Portanto, se você estiver usando o Rust 2015 e uma de suas dependências usar o Rust 2018, seu projeto será compilado e poderá usar essa dependência. A situação oposta, em que seu projeto usa Rust 2018 e uma dependência usa Rust 2015, também funciona.

Para ser claro: a maioria dos recursos estará disponível em todas as edições. Os desenvolvedores que usam qualquer edição do Rust continuarão a ver melhorias à medida que novas versões estáveis forem feitas. No entanto, em alguns casos, principalmente quando novas palavras-chave são adicionadas, alguns novos recursos podem estar disponíveis apenas em edições posteriores. Você precisará alternar as edições se quiser aproveitar esses recursos.

Para obter mais detalhes, o *Guia de Edição* é um livro completo sobre edições que enumera as diferenças entre as edições e explica como atualizar automaticamente seu código para uma nova edição por meio de `cargo fix`.
