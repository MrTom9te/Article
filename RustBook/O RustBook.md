# 1.0 Introdução

> Nota: Esta edição do livro é a mesma que "The Rust Programming Language" disponível em formato impresso e ebook pela No Starch Press.

Bem-vindo a _A Linguagem de Programação Rust_, um livro introdutório sobre Rust. A linguagem de programação Rust ajuda você a escrever software mais rápido e confiável. Alta ergonomia e controle de baixo nível frequentemente estão em conflito no design de linguagens de programação; Rust desafia esse conflito. Através do equilíbrio entre poderosa capacidade técnica e uma ótima experiência de desenvolvedor, Rust dá a você a opção de controlar detalhes de baixo nível (como uso de memória) sem todas as dificuldades tradicionalmente associadas a tal controle.

## Para Quem o Rust É Indicado

Rust é ideal para muitas pessoas por diversas razões. Vamos examinar alguns dos grupos mais importantes.

### Equipes de Desenvolvedores

Rust está se mostrando uma ferramenta produtiva para colaboração entre grandes equipes de desenvolvedores com níveis variados de conhecimento em programação de sistemas. Código de baixo nível é propenso a vários bugs sutis, que na maioria das outras linguagens só podem ser detectados através de testes extensivos e revisão cuidadosa de código por desenvolvedores experientes. Em Rust, o compilador desempenha o papel de guardião, recusando-se a compilar código com esses bugs elusivos, incluindo bugs de concorrência. Ao trabalhar junto com o compilador, a equipe pode dedicar seu tempo focando na lógica do programa em vez de caçar bugs.

Rust também traz ferramentas contemporâneas de desenvolvimento para o mundo da programação de sistemas:
- Cargo, o gerenciador de dependências e ferramenta de build incluída, torna a adição, compilação e gerenciamento de dependências indolor e consistente em todo o ecossistema Rust.
- A ferramenta de formatação Rustfmt garante um estilo de codificação consistente entre desenvolvedores.
- O rust-analyzer potencializa a integração com Ambientes de Desenvolvimento Integrado (IDEs) para autocompletar código e exibir mensagens de erro inline.

Usando estas e outras ferramentas no ecossistema Rust, os desenvolvedores podem ser produtivos enquanto escrevem código de nível de sistema.

### Estudantes

Rust é para estudantes e aqueles que estão interessados em aprender sobre conceitos de sistemas. Usando Rust, muitas pessoas aprenderam sobre tópicos como desenvolvimento de sistemas operacionais. A comunidade é muito acolhedora e feliz em responder perguntas de estudantes. Através de esforços como este livro, as equipes do Rust querem tornar os conceitos de sistemas mais acessíveis para mais pessoas, especialmente aquelas novas em programação.

### Empresas

Centenas de empresas, grandes e pequenas, usam Rust em produção para uma variedade de tarefas, incluindo ferramentas de linha de comando, serviços web, ferramentas de DevOps, dispositivos embarcados, análise e transcodificação de áudio e vídeo, criptomoedas, bioinformática, motores de busca, aplicações de Internet das Coisas, aprendizado de máquina e até mesmo partes importantes do navegador web Firefox.

### Desenvolvedores de Código Aberto

Rust é para pessoas que querem construir a linguagem de programação Rust, comunidade, ferramentas de desenvolvimento e bibliotecas. Adoraríamos que você contribuísse para a linguagem Rust.

### Pessoas que Valorizam Velocidade e Estabilidade

Rust é para pessoas que desejam velocidade e estabilidade em uma linguagem. Por velocidade, queremos dizer tanto quão rapidamente o código Rust pode ser executado quanto a velocidade com que Rust permite que você escreva programas. As verificações do compilador Rust garantem estabilidade através de adições de recursos e refatoração. Isto contrasta com o código legado frágil em linguagens sem essas verificações, que os desenvolvedores frequentemente têm medo de modificar. Ao buscar abstrações de custo zero, recursos de alto nível que compilam para código de baixo nível tão rápido quanto código escrito manualmente, Rust se esforça para fazer com que código seguro também seja código rápido.

A linguagem Rust espera apoiar muitos outros usuários também; os mencionados aqui são meramente alguns dos maiores interessados. No geral, a maior ambição do Rust é eliminar as compensações que os programadores aceitaram por décadas, fornecendo segurança _e_ produtividade, velocidade _e_ ergonomia. Experimente Rust e veja se suas escolhas funcionam para você.

## Para Quem Este Livro É Destinado

Este livro assume que você já escreveu código em outra linguagem de programação, mas não faz suposições sobre qual. Tentamos tornar o material amplamente acessível para aqueles de uma ampla variedade de origens de programação. Não gastamos muito tempo falando sobre o que programação _é_ ou como pensar sobre isso. Se você é completamente novo em programação, seria melhor ler um livro que especificamente forneça uma introdução à programação.

## Como Usar Este Livro

Em geral, este livro assume que você está lendo-o em sequência do início ao fim. Capítulos posteriores se baseiam em conceitos de capítulos anteriores, e capítulos anteriores podem não se aprofundar em detalhes sobre um tópico específico, mas revisitarão o tópico em um capítulo posterior.

Você encontrará dois tipos de capítulos neste livro: capítulos de conceito e capítulos de projeto. Em capítulos de conceito, você aprenderá sobre um aspecto do Rust. Em capítulos de projeto, construiremos pequenos programas juntos, aplicando o que você aprendeu até agora. Os capítulos 2, 12 e 21 são capítulos de projeto; o resto são capítulos de conceito.

O Capítulo 1 explica como instalar Rust, como escrever um programa "Olá, mundo!" e como usar o Cargo, o gerenciador de pacotes e ferramenta de build do Rust. O Capítulo 2 é uma introdução prática à escrita de um programa em Rust, fazendo você construir um jogo de adivinhação de números. Aqui cobrimos conceitos em alto nível, e capítulos posteriores fornecerão detalhes adicionais. Se você quer colocar a mão na massa imediatamente, o Capítulo 2 é o lugar para isso. O Capítulo 3 cobre recursos do Rust que são semelhantes aos de outras linguagens de programação, e no Capítulo 4 você aprenderá sobre o sistema de ownership do Rust. Se você é um aprendiz particularmente meticuloso que prefere aprender cada detalhe antes de seguir em frente, talvez queira pular o Capítulo 2 e ir direto para o Capítulo 3, retornando ao Capítulo 2 quando quiser trabalhar em um projeto aplicando os detalhes que aprendeu.

O Capítulo 5 discute structs e métodos, e o Capítulo 6 aborda enums, expressões `match` e a construção de controle de fluxo `if let`. Você usará structs e enums para criar tipos personalizados em Rust.

No Capítulo 7, você aprenderá sobre o sistema de módulos do Rust e sobre as regras de privacidade para organizar seu código e sua Interface de Programação de Aplicativos (API) pública. O Capítulo 8 discute algumas estruturas de dados de coleção comuns que a biblioteca padrão fornece, como vetores, strings e hash maps. O Capítulo 9 explora a filosofia e técnicas de tratamento de erros do Rust.

O Capítulo 10 se aprofunda em generics, traits e lifetimes, que lhe dão o poder de definir código que se aplica a múltiplos tipos. O Capítulo 11 é todo sobre testes, que mesmo com as garantias de segurança do Rust são necessários para garantir que a lógica do seu programa esteja correta. No Capítulo 12, construiremos nossa própria implementação de um subconjunto de funcionalidades da ferramenta de linha de comando `grep`, que busca texto dentro de arquivos. Para isso, usaremos muitos dos conceitos que discutimos nos capítulos anteriores.

O Capítulo 13 explora closures e iteradores: recursos do Rust que vêm de linguagens de programação funcional. No Capítulo 14, examinaremos o Cargo com mais profundidade e falaremos sobre as melhores práticas para compartilhar suas bibliotecas com outros. O Capítulo 15 discute ponteiros inteligentes que a biblioteca padrão fornece e os traits que permitem sua funcionalidade.

No Capítulo 16, percorreremos diferentes modelos de programação concorrente e falaremos sobre como o Rust ajuda você a programar em múltiplas threads sem medo. No Capítulo 17, construiremos sobre isso explorando a sintaxe async e await do Rust e o modelo de concorrência leve que eles suportam.

O Capítulo 18 examina como os idiomas do Rust se comparam aos princípios de programação orientada a objetos com os quais você pode estar familiarizado.

O Capítulo 19 é uma referência sobre padrões e pattern matching, que são maneiras poderosas de expressar ideias em programas Rust. O Capítulo 20 contém uma variedade de tópicos avançados de interesse, incluindo Rust inseguro, macros e mais sobre lifetimes, traits, tipos, funções e closures.

No Capítulo 21, completaremos um projeto no qual implementaremos um servidor web multithread de baixo nível!

Finalmente, alguns apêndices contêm informações úteis sobre a linguagem em um formato mais parecido com uma referência. O Apêndice A aborda as palavras-chave do Rust, o Apêndice B aborda os operadores e símbolos do Rust, o Apêndice C aborda traits deriváveis fornecidos pela biblioteca padrão, o Apêndice D aborda algumas ferramentas de desenvolvimento úteis e o Apêndice E explica as edições do Rust. No Apêndice F, você pode encontrar traduções do livro, e no Apêndice G abordaremos como o Rust é feito e o que é o Rust nightly.

Não há maneira errada de ler este livro: se você quiser pular adiante, vá em frente! Talvez você precise voltar a capítulos anteriores se sentir alguma confusão. Mas faça o que funcionar para você.

Uma parte importante do processo de aprendizado do Rust é aprender a ler as mensagens de erro que o compilador exibe: elas irão guiá-lo em direção ao código funcional. Como tal, forneceremos muitos exemplos que não compilam junto com a mensagem de erro que o compilador mostrará a você em cada situação. Saiba que se você inserir e executar um exemplo aleatório, ele pode não compilar! Certifique-se de ler o texto ao redor para ver se o exemplo que você está tentando executar destina-se a dar erro. Ferris também ajudará você a distinguir código que não deve funcionar:

| Ferris | Significado                                      |
| ------ | ------------------------------------------------ |
| 🦀     | Este código não compila!                         |
| 💥     | Este código entra em pânico!                     |
| ❌      | Este código não produz o comportamento desejado. |

Na maioria das situações, levaremos você à versão correta de qualquer código que não compile.

## Código-Fonte

Os arquivos-fonte a partir dos quais este livro é gerado podem ser encontrados no GitHub.