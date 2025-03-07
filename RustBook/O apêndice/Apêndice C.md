# Traits Deriváveis (Derivable Traits)

Em vários lugares do livro, discutimos o atributo `derive`, que você pode aplicar a uma definição de struct ou enum. O atributo `derive` gera código que implementará um trait com sua própria implementação padrão no tipo que você anotou com a sintaxe `derive`.

Neste apêndice, fornecemos uma referência de todos os traits na biblioteca padrão que você pode usar com `derive`. Cada seção cobre:

*   Quais operadores e métodos derivar este trait habilitará
*   O que a implementação do trait fornecida por `derive` faz
*   O que implementar o trait significa sobre o tipo
*   As condições em que você pode ou não implementar o trait
*   Exemplos de operações que requerem o trait

Se você quiser um comportamento diferente daquele fornecido pelo atributo `derive`, consulte a documentação da biblioteca padrão para cada trait para obter detalhes de como implementá-los manualmente.

Esses traits listados aqui são os únicos definidos pela biblioteca padrão que podem ser implementados em seus tipos usando `derive`. Outros traits definidos na biblioteca padrão não têm um comportamento padrão sensato, então cabe a você implementá-los da maneira que fizer sentido para o que você está tentando realizar.

Um exemplo de trait que não pode ser derivado é `Display`, que lida com a formatação para usuários finais. Você deve sempre considerar a maneira apropriada de exibir um tipo para um usuário final. Quais partes do tipo um usuário final deve ter permissão para ver? Quais partes eles achariam relevantes? Qual formato dos dados seria mais relevante para eles? O compilador Rust não tem essa percepção, então ele não pode fornecer um comportamento padrão apropriado para você.

A lista de traits deriváveis fornecida neste apêndice não é abrangente: as bibliotecas podem implementar `derive` para seus próprios traits, tornando a lista de traits que você pode usar com `derive` verdadeiramente aberta. Implementar `derive` envolve o uso de uma macro procedural, que é abordada na seção "Macros" do Capítulo 20.

### `Debug` para Saída do Programador

O trait `Debug` habilita a formatação de depuração em strings de formato, que você indica adicionando `:?` dentro de marcadores de posição `{}`.

O trait `Debug` permite que você imprima instâncias de um tipo para fins de depuração, para que você e outros programadores que usam seu tipo possam inspecionar uma instância em um ponto particular na execução de um programa.

O trait `Debug` é necessário, por exemplo, ao usar a macro `assert_eq!`. Esta macro imprime os valores das instâncias fornecidas como argumentos se a asserção de igualdade falhar para que os programadores possam ver por que as duas instâncias não eram iguais.

### `PartialEq` e `Eq` para Comparações de Igualdade

O trait `PartialEq` permite que você compare instâncias de um tipo para verificar a igualdade e habilita o uso dos operadores `==` e `!=`.

Derivar `PartialEq` implementa o método `eq`. Quando `PartialEq` é derivado em structs, duas instâncias são iguais apenas se *todos* os campos forem iguais, e as instâncias não são iguais se algum campo não for igual. Quando derivado em enums, cada variante é igual a si mesma e não igual às outras variantes.

O trait `PartialEq` é necessário, por exemplo, com o uso da macro `assert_eq!`, que precisa ser capaz de comparar duas instâncias de um tipo para igualdade.

O trait `Eq` não tem métodos. Seu propósito é sinalizar que para cada valor do tipo anotado, o valor é igual a si mesmo. O trait `Eq` só pode ser aplicado a tipos que também implementam `PartialEq`, embora nem todos os tipos que implementam `PartialEq` possam implementar `Eq`. Um exemplo disso são os tipos de número de ponto flutuante: a implementação de números de ponto flutuante declara que duas instâncias do valor não-é-um-número (`NaN`) não são iguais entre si.

Um exemplo de quando `Eq` é necessário é para chaves em um `HashMap<K, V>` para que o `HashMap<K, V>` possa dizer se duas chaves são iguais.

### `PartialOrd` e `Ord` para Comparações de Ordenação

O trait `PartialOrd` permite que você compare instâncias de um tipo para fins de ordenação. Um tipo que implementa `PartialOrd` pode ser usado com os operadores `<`, `>`, `<=` e `>=`. Você só pode aplicar o trait `PartialOrd` a tipos que também implementam `PartialEq`.

Derivar `PartialOrd` implementa o método `partial_cmp`, que retorna um `Option<Ordering>` que será `None` quando os valores fornecidos não produzirem uma ordenação. Um exemplo de valor que não produz uma ordenação, mesmo que a maioria dos valores desse tipo possa ser comparada, é o valor de ponto flutuante não-é-um-número (`NaN`). Chamar `partial_cmp` com qualquer número de ponto flutuante e o valor de ponto flutuante `NaN` retornará `None`.

Quando derivado em structs, `PartialOrd` compara duas instâncias comparando o valor em cada campo na ordem em que os campos aparecem na definição da struct. Quando derivado em enums, as variantes do enum declaradas anteriormente na definição do enum são consideradas menores que as variantes listadas posteriormente.

O trait `PartialOrd` é necessário, por exemplo, para o método `gen_range` do crate `rand` que gera um valor aleatório no intervalo especificado por uma expressão de intervalo.

O trait `Ord` permite que você saiba que para quaisquer dois valores do tipo anotado, uma ordenação válida existirá. O trait `Ord` implementa o método `cmp`, que retorna um `Ordering` em vez de um `Option<Ordering>` porque uma ordenação válida sempre será possível. Você só pode aplicar o trait `Ord` a tipos que também implementam `PartialOrd` e `Eq` (e `Eq` requer `PartialEq`). Quando derivado em structs e enums, `cmp` se comporta da mesma maneira que a implementação derivada para `partial_cmp` faz com `PartialOrd`.

Um exemplo de quando `Ord` é necessário é ao armazenar valores em um `BTreeSet<T>`, uma estrutura de dados que armazena dados com base na ordem de classificação dos valores.

### `Clone` e `Copy` para Duplicar Valores

O trait `Clone` permite que você crie explicitamente uma cópia profunda de um valor, e o processo de duplicação pode envolver a execução de código arbitrário e a cópia de dados da heap. Consulte a seção "Maneiras como Variáveis e Dados Interagem: Clone" no Capítulo 4 para obter mais informações sobre `Clone`.

Derivar `Clone` implementa o método `clone`, que quando implementado para todo o tipo, chama `clone` em cada uma das partes do tipo. Isso significa que todos os campos ou valores no tipo também devem implementar `Clone` para derivar `Clone`.

Um exemplo de quando `Clone` é necessário é ao chamar o método `to_vec` em uma slice. A slice não possui as instâncias de tipo que contém, mas o vetor retornado de `to_vec` precisará possuir suas instâncias, então `to_vec` chama `clone` em cada item. Assim, o tipo armazenado na slice deve implementar `Clone`.

O trait `Copy` permite que você duplique um valor copiando apenas os bits armazenados na stack; nenhum código arbitrário é necessário. Consulte a seção "Dados Apenas na Stack: Copy" no Capítulo 4 para obter mais informações sobre `Copy`.

O trait `Copy` não define nenhum método para impedir que os programadores sobrecarreguem esses métodos e violem a suposição de que nenhum código arbitrário está sendo executado. Dessa forma, todos os programadores podem assumir que copiar um valor será muito rápido.

Você pode derivar `Copy` em qualquer tipo cujas partes todas implementem `Copy`. Um tipo que implementa `Copy` também deve implementar `Clone`, porque um tipo que implementa `Copy` tem uma implementação trivial de `Clone` que executa a mesma tarefa que `Copy`.

O trait `Copy` raramente é necessário; tipos que implementam `Copy` têm otimizações disponíveis, o que significa que você não precisa chamar `clone`, o que torna o código mais conciso.

Tudo o que é possível com `Copy` você também pode realizar com `Clone`, mas o código pode ser mais lento ou ter que usar `clone` em alguns lugares.

### `Hash` para Mapear um Valor para um Valor de Tamanho Fixo

O trait `Hash` permite que você pegue uma instância de um tipo de tamanho arbitrário e mapeie essa instância para um valor de tamanho fixo usando uma função hash. Derivar `Hash` implementa o método `hash`. A implementação derivada do método `hash` combina o resultado de chamar `hash` em cada uma das partes do tipo, o que significa que todos os campos ou valores também devem implementar `Hash` para derivar `Hash`.

Um exemplo de quando `Hash` é necessário é ao armazenar chaves em um `HashMap<K, V>` para armazenar dados de forma eficiente.

### `Default` para Valores Padrão

O trait `Default` permite que você crie um valor padrão para um tipo. Derivar `Default` implementa a função `default`. A implementação derivada da função `default` chama a função `default` em cada parte do tipo, o que significa que todos os campos ou valores no tipo também devem implementar `Default` para derivar `Default`.

A função `Default::default` é comumente usada em combinação com a sintaxe de atualização de struct discutida na seção "Criando Instâncias de Outras Instâncias com Sintaxe de Atualização de Struct" no Capítulo 5. Você pode personalizar alguns campos de uma struct e, em seguida, definir e usar um valor padrão para o resto dos campos usando `..Default::default()`.

O trait `Default` é necessário quando você usa o método `unwrap_or_default` em instâncias de `Option<T>`, por exemplo. Se o `Option<T>` for `None`, o método `unwrap_or_default` retornará o resultado de `Default::default` para o tipo `T` armazenado no `Option<T>`.
