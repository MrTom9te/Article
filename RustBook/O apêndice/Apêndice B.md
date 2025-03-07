# Operadores e Símbolos

Este apêndice contém um glossário da sintaxe do Rust, incluindo operadores e outros símbolos que aparecem sozinhos ou no contexto de caminhos, genéricos, limites de trait, macros, atributos, comentários, tuplas e colchetes.
### Operadores

A Tabela B-1 contém os operadores em Rust, um exemplo de como o operador apareceria no contexto, uma breve explicação e se esse operador é sobrecarregável. Se um operador for sobrecarregável, o trait relevante para usar para sobrecarregar esse operador é listado.

Tabela B-1: Operadores

| Operador     | Exemplo                 | Explicação                                                                                                                                | Sobrecarga?     |
| :----------- | :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------- | :-------------- |
| `!`          | `ident!(...)`, `ident!{...}`, `ident![...]` | Expansão de macro                                                                                                                |                 |
| `!`          | `!expr`                | Complemento bit a bit ou lógico                                                                                                          | `Not`           |
| `!=`         | `expr != expr`          | Comparação de não igualdade                                                                                                              | `PartialEq`     |
| `%`          | `expr % expr`          | Resto aritmético                                                                                                                  | `Rem`           |
| `%=`         | `var %= expr`          | Resto aritmético e atribuição                                                                                                         | `RemAssign`     |
| `&`          | `&expr`, `&mut expr`    | Empréstimo (Borrow)                                                                                                                     |                 |
| `&`          | `&type`, `&mut type`, `&'a type`, `&'a mut type` | Tipo ponteiro emprestado                                                                                                             |                 |
| `&`          | `expr & expr`          | E bit a bit                                                                                                                        | `BitAnd`        |
| `&=`         | `var &= expr`          | E bit a bit e atribuição                                                                                                               | `BitAndAssign`  |
| `&&`         | `expr && expr`          | E lógico de curto-circuito                                                                                                             |                 |
| `*`          | `expr * expr`          | Multiplicação aritmética                                                                                                              | `Mul`           |
| `*=`         | `var *= expr`          | Multiplicação aritmética e atribuição                                                                                                   | `MulAssign`     |
| `*`          | `*expr`                | Desreferência                                                                                                                      | `Deref`         |
| `*`          | `*const type`, `*mut type` | Ponteiro bruto                                                                                                                        |                 |
| `+`          | `trait + trait`, `'a + trait` | Restrição de tipo composto                                                                                                           |                 |
| `+`          | `expr + expr`          | Adição aritmética                                                                                                                    | `Add`           |
| `+=`         | `var += expr`          | Adição aritmética e atribuição                                                                                                         | `AddAssign`     |
| `,`          | `expr, expr`          | Separador de argumento e elemento                                                                                                         |                 |
| `-`          | `- expr`               | Negação aritmética                                                                                                                    | `Neg`           |
| `-`          | `expr - expr`          | Subtração aritmética                                                                                                                  | `Sub`           |
| `-=`         | `var -= expr`          | Subtração aritmética e atribuição                                                                                                       | `SubAssign`     |
| `->`         | `fn(...) -> type`, `|…| -> type` | Tipo de retorno de função e closure                                                                                                       |                 |
| `.`          | `expr.ident`           | Acesso a membro                                                                                                                        |                 |
| `..`         | `..`, `expr..`, `..expr`, `expr..expr` | Literal de intervalo exclusivo à direita                                                                                             | `PartialOrd`    |
| `..=`        | `..=expr`, `expr..=expr` | Literal de intervalo inclusivo à direita                                                                                               | `PartialOrd`    |
| `..`         | `..expr`               | Sintaxe de atualização de literal de struct                                                                                             |                 |
| `..`        | `variant(x, ..)`, `struct_type { x, .. }` | Vinculação de padrão "E o resto"                                                                                                   |                 |
| `...`        | `expr...expr`          | (Descontinuado, use `..=` em vez disso) Em um padrão: padrão de intervalo inclusivo                                               |                 |
| `/`          | `expr / expr`          | Divisão aritmética                                                                                                                  | `Div`           |
| `/=`         | `var /= expr`          | Divisão aritmética e atribuição                                                                                                       | `DivAssign`     |
| `:`          | `pat: type`, `ident: type` | Restrições                                                                                                                          |                 |
| `:`        |   `ident: expr`           |  Inicializador de campo de struct                                                                                         |      |
| `:`          | `'a: loop {...}`        | Rótulo de loop                                                                                                                      |                 |
| `;`          | `expr;`                | Terminador de declaração e item                                                                                                        |                 |
| `;`          | `[...; len]`           | Parte da sintaxe de array de tamanho fixo                                                                                             |                 |
| `<<`         | `expr << expr`         | Deslocamento à esquerda                                                                                                             | `Shl`           |
| `<<=`        | `var <<= expr`         | Deslocamento à esquerda e atribuição                                                                                                  | `ShlAssign`     |
| `<`          | `expr < expr`          | Comparação menor que                                                                                                                    | `PartialOrd`    |
| `<=`         | `expr <= expr`          | Comparação menor ou igual a                                                                                                               | `PartialOrd`    |
| `=`          | `var = expr`, `ident = type` | Atribuição/equivalência                                                                                                               |                 |
| `==`         | `expr == expr`          | Comparação de igualdade                                                                                                                 | `PartialEq`     |
| `=>`         | `pat => expr`          | Parte da sintaxe de braço de match                                                                                                       |                 |
| `>`          | `expr > expr`          | Comparação maior que                                                                                                                    | `PartialOrd`    |
| `>=`         | `expr >= expr`          | Comparação maior ou igual a                                                                                                               | `PartialOrd`    |
| `>>`         | `expr >> expr`         | Deslocamento à direita                                                                                                             | `Shr`           |
| `>>=`        | `var >>= expr`         | Deslocamento à direita e atribuição                                                                                                  | `ShrAssign`     |
| `@`          | `ident @ pat`          | Vinculação de padrão                                                                                                                     |                 |
| `^`          | `expr ^ expr`          | OU exclusivo bit a bit                                                                                                                | `BitXor`        |
| `^=`         | `var ^= expr`          | OU exclusivo bit a bit e atribuição                                                                                                     | `BitXorAssign`  |
| `|`          | `pat | pat`            | Alternativas de padrão                                                                                                                 |                 |
| `|`          | `expr | expr`          | OU bit a bit                                                                                                                           | `BitOr`         |
| `|=`         | `var |= expr`          | OU bit a bit e atribuição                                                                                                               | `BitOrAssign`   |
| `||`         | `expr || expr`          | OU lógico de curto-circuito                                                                                                             |                 |
| `?`          | `expr?`                | Propagação de erro                                                                                                                      |                 |

### Símbolos Não Operadores

A lista a seguir contém todos os símbolos que não funcionam como operadores; ou seja, eles não se comportam como uma chamada de função ou método.

A Tabela B-2 mostra símbolos que aparecem sozinhos e são válidos em uma variedade de locais.

Tabela B-2: Sintaxe Independente

| Símbolo   | Explicação                                                                                               |
| :-------- | :------------------------------------------------------------------------------------------------------- |
| `'ident`  | Tempo de vida nomeado ou rótulo de loop                                                                    |
| `...u8`, `...i32`, `...f64`, `...usize`, etc. | Literal numérico de tipo específico                                                                         |
| `"..."`   | Literal de string                                                                                            |
| `r"..."`, `r#"..."#`, `r##"..."##`, etc. | Literal de string bruto, caracteres de escape não processados                                               |
| `b"..."`  | Literal de string de bytes; constrói um array de bytes em vez de uma string                              |
| `br"..."`, `br#"..."#`, `br##"..."##`, etc. | Literal de string de bytes bruto, combinação de literal de string bruto e de bytes                        |
| `'...'`   | Literal de caractere                                                                                         |
| `b'...'`  | Literal de byte ASCII                                                                                        |
| `|…| expr` | Closure                                                                                                   |
| `!`       | Tipo inferior sempre vazio para funções divergentes                                                          |
| `_`       | Vinculação de padrão "Ignorado"; também usado para tornar literais inteiros legíveis                     |

A Tabela B-3 mostra símbolos que aparecem no contexto de um caminho através da hierarquia de módulos para um item.

Tabela B-3: Sintaxe Relacionada a Caminho

| Símbolo                             | Explicação                                                                                                                      |
| :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------ |
| `ident::ident`                      | Caminho de namespace                                                                                                          |
| `::path`                            | Caminho relativo ao prelúdio externo, onde todos os outros crates estão enraizados (ou seja, um caminho explicitamente absoluto, incluindo o nome do crate)                        |
| `self::path`                         | Caminho relativo ao módulo atual (ou seja, um caminho explicitamente relativo).                                             |
| `super::path`                       | Caminho relativo ao pai do módulo atual                                                                                      |
| `type::ident`, `<type as trait>::ident` | Constantes, funções e tipos associados                                                                                      |
| `<type>::...`                        | Item associado para um tipo que não pode ser nomeado diretamente (por exemplo, `<&T>::...`, `<[T]>::...`, etc.)           |
| `trait::method(...)`                | Desambiguando uma chamada de método nomeando o trait que a define                                                           |
| `type::method(...)`                 | Desambiguando uma chamada de método nomeando o tipo para o qual ela é definida                                                |
| `<type as trait>::method(...)`       | Desambiguando uma chamada de método nomeando o trait e o tipo                                                                 |

A Tabela B-4 mostra símbolos que aparecem no contexto do uso de parâmetros de tipo genérico.

Tabela B-4: Genéricos

| Símbolo                    | Explicação                                                                                                                    |
| :------------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                | Especifica parâmetros para tipo genérico em um tipo (por exemplo, `Vec<u8>`)                                                  |
| `path::<...>`, `method::<...>` | Especifica parâmetros para tipo genérico, função ou método em uma expressão; frequentemente referido como turbofish (por exemplo, `"42".parse::<i32>()`) |
| `fn ident<...> ...`        | Define função genérica                                                                                                       |
| `struct ident<...> ...`    | Define estrutura genérica                                                                                                    |
| `enum ident<...> ...`      | Define enumeração genérica                                                                                                     |
| `impl<...> ...`            | Define implementação genérica                                                                                                 |
| `for<...> type`           | Limites de tempo de vida de ordem superior                                                                                 |
| `type<ident=type>`        | Um tipo genérico onde um ou mais tipos associados têm atribuições específicas (por exemplo, `Iterator<Item=T>`)               |

A Tabela B-5 mostra símbolos que aparecem no contexto de restringir parâmetros de tipo genérico com limites de trait.

Tabela B-5: Restrições de Limite de Trait

| Símbolo              | Explicação                                                                                                                                                                                                      |
| :-------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                | Parâmetro genérico `T` restrito a tipos que implementam `U`                                                                                                                                               |
| `T: 'a`               | Tipo genérico `T` deve sobreviver ao tempo de vida `'a` (o que significa que o tipo não pode conter transitivamente nenhuma referência com tempos de vida menores que `'a`)                                 |
| `T: 'static`         | Tipo genérico `T` não contém referências emprestadas além das `'static`                                                                                                                                      |
| `'b: 'a`               | Tempo de vida genérico `'b` deve sobreviver ao tempo de vida `'a`                                                                                                                                             |
| `T: ?Sized`           | Permitir que o parâmetro de tipo genérico seja um tipo de tamanho dinâmico                                                                                                                             |
| `'a + trait`, `trait + trait` | Restrição de tipo composto                                                                                                                                                                                 |

A Tabela B-6 mostra símbolos que aparecem no contexto de chamar ou definir macros e especificar atributos em um item.

Tabela B-6: Macros e Atributos

| Símbolo          | Explicação        |
| :--------------- | :----------------- |
| `#[meta]`        | Atributo externo  |
| `#![meta]`       | Atributo interno   |
| `$ident`         | Substituição de macro |
| `$ident:kind`    | Captura de macro  |
| `$(…)…`          | Repetição de macro |
|`ident!(...)`, `ident!{...}`, `ident![...]`|chamado a macro |

Tabela B-7: Comentários

| Símbolo   | Explicação                                |
| :-------- | :---------------------------------------- |
| `//`      | Comentário de linha                        |
| `//!`     | Comentário de linha de documentação interno |
| `///`     | Comentário de linha de documentação externo |
| `/*...*/` | Comentário de bloco                         |
| `/*!...*/` | Comentário de bloco de documentação interno |
| `/**...*/` | Comentário de bloco de documentação externo |

A Tabela B-8 mostra símbolos que aparecem no contexto do uso de tuplas.

Tabela B-8: Tuplas

| Símbolo       | Explicação                                                                   |
| :------------ | :--------------------------------------------------------------------------- |
| `()`          | Tupla vazia (também conhecida como unidade), tanto literal quanto tipo    |
| `(expr)`      | Expressão entre parênteses                                                  |
| `(expr,)`     | Expressão de tupla de um único elemento                                   |
| `(type,)`     | Tipo de tupla de um único elemento                                        |
| `(expr, ...)` | Expressão de tupla                                                        |
| `(type, ...)` | Tipo de tupla                                                             |
|`expr(expr, ...)` | Expressão de chamada de função; também usado para inicializar `struct`s de tupla e variantes de `enum` de tupla |
|`expr.0`,`expr.1`, etc.| Indexação de tupla|

A Tabela B-9 mostra os contextos em que chaves são usadas.

Tabela B-9: Chaves

| Contexto      | Explicação            |
| :------------ | :--------------------- |
| `{...}`       | Expressão de bloco    |
| `Type {...}` | Literal de `struct` |

A Tabela B-10 mostra os contextos em que colchetes são usados.

Tabela B-10: Colchetes

| Contexto                             | Explicação                                                                                                                               |
| :----------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                              | Literal de array                                                                                                                          |
| `[expr; len]`                        | Literal de array contendo `len` cópias de `expr`                                                                                        |
| `[type; len]`                        | Tipo de array contendo `len` instâncias de `type`                                                                                      |
| `expr[expr]`                         | Indexação de coleção. Sobrecarga (`Index`, `IndexMut`)                                                                                  |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexação de coleção fingindo ser fatiamento de coleção, usando `Range`, `RangeFrom`, `RangeTo` ou `RangeFull` como o "índice" |
