# Roteiro de apresentação

Este documento serve como um roteiro/resumo da apresentação destinada à SECOMP-UFSJ de 2022 sobre a linguagem Rust. O documento pode não ser extensivo, mas serve como referência geral sobre os temas a serem abordados bem como uma breve explicação de cada um.

A apresentação parte do pressuposto que o espectador já sabe os conceitos básicos de programação, de preferência em C, a fim de pular os passos mais introdutórios e poder discutir diretamente o que faz a linguagem única, que para os fins desta apresentação são os tipos definidos por usuários (structs e enums), o sistema de traços e as regras de empréstimo.

A apresentação também não está estruturada como um tutorial ou um "follow along", que reduziria drasticamente o conteúdo apresentado. Ao invés disso, o objetivo da apresentação é detalhar as *features* únicas da linguagem, como elas podem facilitar o desenvolvimento de software e sistemas, e como podem tornar a linguagem referência no mercado de trabalho nos próximos anos.

Finalmente, este documento e a apresentação que o segue foram fortemente baseados em *"The Book"* de Rust, o livro oficial e mantido pela comunidade com o objetivo de documentar e ensinar Rust. Este e muitos outros "livros" de Rust estão oficialmente disponíveis [aqui](https://www.rust-lang.org/learn).

# Linguagem Rust: programação baixo nível com segurança e confiabilidade

## Introdução: O que é Rust?

Rust é uma linguagem de programação multi-paradigma <sub><sup>(imperativo, orientado a objetos\*, funcional)</sup></sub> estaticamente tipada que compila para código de máquina. Seus objetivos primários são eficiência e confiabilidade no código escrito.

A linguagem consegue alcançar a confiabilidade graças a uma série de fatores que a distinguem de linguagens de programação convencionais. O principal destes fatores são as ["regras de empréstimo" (borrowing rules)](#mutabilidade-e-empréstimo), que causa confusão no início do aprendizado e exige uma mudança de raciocínio durante o desenvolvimento com esta.

Para balancear a complexidade que esse sistema traz, Rust inclui o melhor de qualquer compilador existente entre as linguagens de programação, detalhando com exatidão os motivos pelo qual o programa não compilou, dicas de como alterar seu código para que seja possível compilar e referências sobre o erro.

<sub><sup>* A "orientação a objetos" de Rust difere do tradicionl. Enquanto linguagens como C++, Java e Python possuem herança de classes para a propagação de métodos para subclasses, Rust traz um sistema de traços (ou características), no qual definem-se a quais traços uma struct implementa. Mais sobre isso [nesta seção](#impl-implementando-métodos-em-structs-e-enums)</sup></sub>

## Por que Rust? Quem usa?

Rust, desenvolvido em 2011 pela Mozilla, já é utilizado por todas as gigantes de tecnologia (Google, Microsoft, Amazon, Dropbox, etc.).

Durante a escrita deste documento, estão havendo experimentos para inserir Rust no kernel do Linux, como confirmado por Linus Torvalds, que se demonstrou otimista sobre o assunto. Isto porque, diferente de C, Rust possui um único padrão, inclui um gerenciador de pacotes, garante que a tipagem seja respeitada e torna impossível condições de corridas em programas multi-thread.

A fim de elaborar sobre o penúltimo ponto, considere o programa em C a seguir:

```c
#include <stdbool.h>
#include <stdio.h>

int main() {
	bool a = true;

	printf("a = %d\n", a);

	if (a == 1)
		printf("a = true é igual a 1\n");

	a = 10;
	printf("a = %d\n", a);

	if (a == 1)
		printf("a ainda é igual a 1\n");

  else
      printf("a é diferente de 1\n");
}
```

Este programa, quando executado, tem a seguinte saida:

```
a = 1
a = true é igual a 1
a = 1
a ainda é igual a 1
```

Este é só um exemplo banal de como o C não é confiável. Não só o compilador nos deixou atribuir 10 a um suposto tipo booleano, como ele desconsiderou a atribuição `a = 10;`. Além disso, ele permitiu a comparação entre tipos de dados completamente distintos e, possivelmente, incompatíveis. Isso tudo sem nem mesmo mostrar um *warning* durante a compilação.

A atribuição e comparação exemplificadas acima muito provavelmente teriam sido causadas erroneamente pelo programador, que não tinha, de fato, a intenção de atribuir um inteiro a um booleano, e poderia ter sido evitada pelo compilador. Abaixo segue o mesmo exemplo escrito em Rust.

```rust
fn main() {
	let a = true;

	println!("a = {}", a);
	
	if a == 1 {
		println!("a = true é igual a 1");
	}

	a = 10;
	println!("a = {}", a);

	if a == 1 {
		println!("a ainda é igual a 1");
	}

	else {
		println!("a é diferente de 1");
	}
}
```

E o erro de compilação

```
error[E0308]: mismatched types
 --> t.rs:6:10
  |
6 |     if a == 1 {
  |             ^ expected `bool`, found integer

error[E0308]: mismatched types
  --> t.rs:10:6
   |
2  |     let a = true;
   |             ---- expected due to this value
...
10 |     a = 10;
   |         ^^ expected `bool`, found integer

error[E0308]: mismatched types
  --> t.rs:13:10
   |
13 |     if a == 1 {
   |             ^ expected `bool`, found integer

error: aborting due to 3 previous errors

For more information about this error, try `rustc --explain E0308`.
```

### Onde posso usar Rust?

Rust possui uma comunidade que cresce cada vez mais rapidamente. Por 7 anos seguidos, Rust foi votado como [a linguagem mais amada pelos desenvolvedores](https://survey.stackoverflow.co/2022/?utm_source=so-owned&utm_medium=announcement-banner&utm_campaign=dev-survey-2022&utm_content=results#section-most-loved-dreaded-and-wanted-programming-scripting-and-markup-languages) (com uma bela margem, inclusive). Por este motivo e sua alta eficiência (otimizada com a LLVM do Clang), Rust pode ser usado em cada vez mais aplicações.

- Para desenvolvimento web *backend*, Rust pode ser usado com [actix](https://actix.rs/), o 5º framework mais rápido segundo o [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r21&test=fortune).

- Para desenvolvimento web *frontend*, Rust pode ser usado com [yew](https://yew.rs/), um framework que permite escrever HTML dentro do código, como o React faz, e exporta funções para interações com JavaScript. Este framework é possível pelo fato de que Rust foi feito para compilar em WebAssembly também. Afinal de contas, foi desenvolvido pela Mozilla.

- Para desenvolvimento de jogos, Rust pode ser usado em conjunto com [Godot](https://godotengine.org/) por meio da extensão [Godot-Rust](https://godot-rust.github.io/), provendo uma eficiência ainda maior que a linguagem de script nativa do motor de jogos.

- Para desenvolvimento de microcontroladores, o suporte é nativo (vide *"The Embedded Rust Book"*).

- Possui interoperabilidade com C (vide *"The Embedded Rust Book"*, capítulos *"A little C with your Rust"* e *"A little Rust with your C"*).

---

## Prólogo: Tipos de Dados

Antes de chegar nas *features* mencionadas, uma pequena etapa necessária é a apresentação dos tipos de dados de Rust, apenas para confirmar que todos estão na mesma página.

Um detalhe importante a ser mencionado é que, apesar de Rust ser uma linguagem estaticamente tipada, raramente é necessário dizer o tipo de uma variável explicitamente. Um dos favores que o compilador te faz é inferir o tipo de dado de uma variável analisando o contexto em sua volta. Alguns dos meios que o compilador pode inferir o tipo de uma variável são:

- Atribuição direta

    ```rust
    let x = 2; // x é inferido como i32 (inteiro de 32 bits)
    let y = 10.5; // y é inferido como f32 (float de 32 bits)
    ``` 

- Utilização da variável como parâmetros de funções

    ```rust
    let x = 2; // x __seria__ inferido como i32 somente com esta linha;

    let y = soma_2(x); // Porém, considerando o uso de x aqui e a anotação da função abaixo, x é tido como i64 ao invés

    fn soma_2(a: i64) -> i64 { a + 2 }
    ``` 

### Tipos primitivos e compostos

| Com sinal | Sem sinal | Significado           | Equivalente em C                                  |
|:---------:|:---------:|:----------------------|:--------------------------------------------------|
| `i8`      | `u8`      | Inteiro de 8 bits     | `[unsiged] char`                                  |
| `i16`     | `u16`     | Inteiro de 16 bits    | `[unsigned] short int \| uint16_t, int16_t`       |
| `i32`     | `u32`     | Inteiro de 32 bits    | `[unsiged] int`                                   |
| `i64`     | `u64`     | Inteiro de 64 bits    | `[unsiged] long int \| [unsiged] long long int \| uint64_t, int64_t`    |
| `i128`    | `u128`    | Inteiro de 128 bits   | `__uint128_t, __int128_t` *                       |
| `isize`   | `usize`   | Inteiro de 32 ou 64 bits, a depender da sua arquitetura | Não existe |
| `f32`     | --        | Float de 32 bits      | `float`                                           |
| `f64`     | --        | Float de 64 bits      | `double`                                          |
| `bool`    | --        | Booleano              | `bool`                                            |
| `char`    | --        | Caractere Unicode     | Não existe, vide explicação abaixo                |

<sub><sup>* Somente quando compilado no GCC ou CLang em máquinas 64bit</sup></sub>

Note que os tipos `char` do C e do Rust não representam a mesma coisa. Um `char` em C é composto por um inteiro de 1 byte, com 255 possíveis valores, o suficiente para as tabelas ASCII e ASCII estendida. Enquanto isso, um `char` em Rust é composto por um inteiro de 4 bytes, podendo representar toda a magnitude de caracteres Unicode, incluindo emojis que utilizamos em smartphones.

Além dos tipos primitivos, existem os tipos compostos e os tipos definidos pelo usuário. Dentre os tipos compostos existem apenas dois: *arrays* e tuplas.

```rust
let array: [i32; 4] = [1, 2, 3, 4];
let tuple: (i32, f32, &str) = (1, 2.5, "asd");

let array_first = array[0];
let tuple_first = tuple.0;
```

A principal diferença entre estes tipos é que *arrays* são homogêneos, enquanto tuplas são heterogêneas. Em ambos os casos, estes tipos têm tamanho fixo. Para coleções de tamanho variado, possuímos os tipos definidos por usuários: `struct`s.

Por exemplo, temos as seguintes `struct`s com métodos para vetores dinâmicos e strings:

```rust
// Vetor -------------------------------
let mut v = Vec::new(); // Vetor vazio
v.push(1); // Adiciona o número 1
v.push(2);
v.push(3);

// Alternativa (mesma struct)
let v = vec![1, 2, 3]; // Não precisa ser mutável!

// String ------------------------------
let mut s = String::new();
s.push_str("Olá mundo!");

// Alternativa
let s = String::from("Olá mundo!"); // Não precisa ser mutável!
```

Discutiremos mais a fundo `struct`s e seus métodos [aqui](#tipos-definidos-pelo-usuário-structs-e-enums) e [aqui](#impl-implementando-métodos-em-structs-e-enums), e discutiremos sobre a mutabilidade mencionada [aqui](#mutabilidade-e-empréstimo).

### E os ponteiros?

Apesar de Rust possuir ponteiros, não é comum utilizá-los como os usamos em C. Ao invés, é muito mais comum utilizarmos as referências (como em C++).

```rust
let x = 10;
let y = &x; // Referência ao valor de x

assert_eq!(x, y);
// Note que não é necessário escrever `*y`.
// Rust irá auto-desreferenciar ponteiros em
// tempo de compilação quando for conveniente.
```

Note, contanto, que ambos x e y são imutáveis. Em Rust, toda variável é declarada como imutável por padrão. No próximo capítulo entraremos em mais detalhes sobre isso.

---

## Mutabilidade e Empréstimo

Dois detalhes muito importantes de Rust e que requerem mudança no raciocínio durante o desenvolvimento com a linguagem são os conceitos de mutabilidade e empréstimo.

Mutabilidade é o atributo que define se o valor que uma variável segura pode ser alterado ou não. A variável `x` do exemplo anterior, não poderia ter seu valor alterado, isto é, as expressões abaixo são inválidas:

```rust
// Aviso: o código não compila!
let x = 10;

x = 20; // Inválido
x += 10; // Inválido
```

Ao tentarmos compilar, obtemos o seguinte erro:

```
error[E0384]: cannot assign twice to immutable variable `x`
 --> m.rs:4:2
  |
2 |     let x = 10;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     
4 |     x = 20; // Inválido
  |     ^^^^^^ cannot assign twice to immutable variable

error[E0384]: cannot assign twice to immutable variable `x`
 --> m.rs:5:2
  |
2 |     let x = 10;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
...
5 |     x += 10; // Inválido
  |     ^^^^^^^ cannot assign twice to immutable variable

error: aborting due to 2 previous errors; 2 warnings emitted

For more information about this error, try `rustc --explain E0384`.
```

Para solucionar este problema, duas abordagens podem ser aplicadas: ou deixemos que `x` seja mutável em sua declaração (`let mut x = 10;`) ou "sombreamos" a variável, criando outra com o mesmo nome, como abaixo:

```rust
// Podemos fazer que `x` seja mutável ----

let mut x = 10;
x = 20;
x += 10;

// OU podemos sombrear a variável ----

let x = 10;
let x = 20;
let x = x + 10;
```

Sabendo disso, você assumiria que o bloco abaixo estaria correto:

```rust
// Aviso: o código não compila!

let mut x = 10;
let mut y = &x;

*y = 20; // Inválido
```

Ao compilar, obtemos o seguinte erro:

```
error[E0594]: cannot assign to `*y`, which is behind a `&` reference
 --> m.rs:5:2
  |
3 |     let mut y = &x;
  |                 -- help: consider changing this to be a mutable reference: `&mut x`
4 |     
5 |     *y = 20; // Inválido
  |     ^^^^^^^ `y` is a `&` reference, so the data it refers to cannot be written

error: aborting due to previous error; 2 warnings emitted

For more information about this error, try `rustc --explain E0594`.
```

E, aparentemente contraditoriamente, o código não compila. Isto porque, declarar `y` como mutável não garante o direito de alterar o valor de `x`. Ao invés, permite que `y` referencie outro endereço.

Para que `y` possa alterar o valor de `x`, devemos fazer uma referência mutável, como abaixo:

```rust
let mut x = 10;
let y = &mut x;

*y = 20;
```

Neste trecho, `y` não é mutável, porém segura o valor de uma referência mutável. Quando anotado, o tipo de dado de `y` é `&mut i32`.

### Empréstimos e suas vantagens

Ainda no tópico de referências, temos que discutir sobre o sistema mais importante de Rust, as regras de empréstimo. Em Rust, apenas uma variável pode ser dona de um dado.

```rust
// Aviso: o código não compila!
let mut v1 = vec![1, 2, 3, 4, 5]; // Novo vetor dinâmico
v1.push(6);

let mut v2 = v1; // Os dados de `v1` são movidos* para `v2`
v2.push(7);

v1.push(8); // Inválido! `v1` não é mais o dono do vetor!
```

Ao tentar compilar, o compilador de Rust nos informa do erro:

```
error[E0382]: borrow of moved value: `v1`
 --> m.rs:8:2
  |
2 |     let mut v1 = vec![1, 2, 3, 4, 5]; // Novo vetor dinâmico
  |         ------ move occurs because `v1` has type `Vec<i32>`, which does not implement the `Copy` trait
...
5 |     let mut v2 = v1; // Os dados de `v1` são movidos* para `v2`
  |                  -- value moved here
...
8 |     v1.push(8); // Inválido! `v1` não é mais o dono do vetor!
  |     ^^^^^^^^^^ value borrowed here after move

error: aborting due to previous error

For more information about this error, try `rustc --explain E0382`.
```

É relevante informar ao leitor que, assim como notado pelo compilador, o Rust só não permitiu que esta operação fosse feita porque o tipo `Vec<i32>` não implementa o traço `Copy`. Em Rust, tipos que não possuem tamanhos fixos não implementarão o traço mencionado e causarão este mesmo erro.

Tipos com tamanhos conhecidos e fixos, por outro lado, como os tipos primitivos e compostos, implementam este traço e, numa atribuição, seriam copiados para a outra variável. Isto porque os tipos mencionados são alocados na ***stack***, enquanto estes tipos dinâmicos como `Vec<T>` e `String` são ponteiros para dados alocados na ***heap*** e copiá-los não somente poderia levar muito tempo como não deixaria claro ao programador que esta operação custaria mais memória. 

Por serem ponteiros, assumiríamos que o valor copiado para a outra variável seria o endereço, enquanto o dado apontado seria o mesmo, como acontece em C, C++, Java, Python, etc. Diferentemente destas linguagens, contudo, Rust não permite que duas variáveis sejam donas do mesmo dado.

### Por que não?

Dois dos maiores problemas quando falamos de ponteiros são valores nulos e condições de corrida. Estes problemas acontecem exclusivamente quando mais de um local possui a referência de escrita e/ou leitura para o mesmo valor.

Valores nulos, muitas vezes usados para sinalizar que ocorreu algum erro com a função, podem causar falhas de segmentação quando não tratados corretamente. De fato, o inventor do valor nulo, Tony Hoare, chama o *null type* de "erro de um bilhão de dólares", referenciando quanto dinheiro foi gasto com tempo de desenvolvedores e erros de software pelos valores nulos. Contudo, valores nulos ainda têm uma boa motivação por trás, e por isso Rust traz sua própria versão deles, que discutiremos [aqui](#padrões-de-acesso-para-structs-e-enums).

Por estes e outros motivos, um dos objetivos do Rust é garantir que todos os ponteiros apontem para endereços válidos de memória. Essa abordagem vem com uma série de vantagens, sendo algumas:

1) Um código desenvolvido em Rust jamais causará a falha de segmentação;
2) Tipos que envolvem alocação dinâmica são desalocados sozinhos, sem coletor de lixo envolvido, evitando tanto vazamentos de memória quanto *hits* de performace;
3) Condições de corrida se tornam impossíveis, dado que nenhuma duas variáveis poderiam escrever/ler de um valor ao mesmo tempo.

### Emprestando valores para leitura e escrita

Sabendo da motivação e das vantagens proporcionadas pelo sistema de empréstimo, basta aprender como usá-lo. Na seção [anterior](#mutabilidade-e-empréstimo), foi demonstrado como pode-se criar referências para valores e definí-las como somente leitura ou leitura e escrita. Quando criamos uma referência imutável para um valor, dizemos que aquele valor foi emprestado para leitura, enquanto referências mutáveis são empréstimos para leitura e escrita.

Mas por que chamamos isto de empréstimo? Simples, a variável dona do dado não poderá usá-lo enquanto este foi emprestado. Assim como se você emprestasse o seu carro, você não poderia dirigí-lo durante o empréstimo. Uma regra que foge a esta analogia, contudo, é que podem haver múltiplos empréstimos para leitura por vez, contanto que não exista nenhuma variável escrevendo no valor simultaneamente.

Uma melhor analogia, portanto, seria como se "escrever" um valor fosse o mesmo que dirigir o carro, enquanto "ler" o valor fosse o mesmo que lavar o carro. Múltiplas diferentes pessoas podem estar lavando o carro enquanto ele está parado, mas enquanto alguém dirige - e só pode haver um motorista - ninguém poderá lavá-lo.

É importante ressaltar que não é necessário escrever ou definir manualmente quando você parou de emprestar uma referência. Rust consegue inferir e forçar as regras de empréstimo em tempo de compilação.

---

## Tipos definidos pelo usuário: Structs e Enums

Voltando, agora, a um tópico menos complexo e mais comum em outras linguagens, temos os tipos definidos pelo usuário. No C existem 2 maneiras diferentes de definir tipos: `struct`s e `union`s. `Struct`s contêm múltiplos campos e têm seu tamanho definido, em tempo de compilação, como a soma dos tamanhos de seus elementos. `Union`s, em contrapartida, possuem seu tamanho como o maior de seus elementos.

Em Rust, as `struct`s funcionam igualmente, pelo menos na parte de dados. 

```c
// Exemplo em C
struct Cachorro {
  char *nome;
  int idade;
  bool eh_macho;
};

// ...

char nome[] = "Barney";
struct Cachorro barney = {nome, 2, true};
```

```rust
// Exemplo em Rust
struct Cachorro {
  nome: String,
  idade: i32,
  eh_macho: bool
}

// ...

let barney = Cachorro {
  nome: String::from("Barney"),
  idade: 5,
  eh_macho: true 
};
```

Onde as duas linguagens diferem, contudo, são nos `enum`s. Em C, um `enum` serve para criar nomes para diferentes valores de inteiros relacionados ou não. Existem alguns problemas com esta definição, mais notavelmente que os nomes internos são definidos em escopo global, impedindo que outros `enum`s tenham campos com nomes iguais, mas isto está além desta discussão.

Em Rust, `enum`s possuem um propósito geral e se assemelham mais às `union`s de C. Considere o seguinte exemplo de implementação dos diferentes tipos de mensagem que encontramos em aplicativos mensageiros como WhatsApp e Telegram:

```rust
enum Mensagem {
  Ligacao,
  Texto(String),
  Reacao {
    emoji: char,
    id_mensagem_reagida: i32
  }
}

// ...

// Métodos da struct App
impl App {
  fn usuario_reagiu(&self) -> bool;
  fn get_id_mensagem_reagida(&self) -> i32;
  fn get_emoji_reacao(&self) -> char;
  fn envia_mensagem(&self, mensagem: Mensagem);
}

// ...

if app.usuario_reagiu() {
  let id_mensagem_readiga = app.get_id_mensagem_reagida();
  let emoji = app.get_emoji_reacao();

  let nova_mensagem = Mensagem::Reacao {
    id_mensagem_reagida,
    emoji
  };

  app.envia_mensagem(nova_mensagem);
}
```

No exemplo, o `enum` **Mensagem** discrimina entre três diferentes `struct`s:
- **Chamada**: uma `struct` sem nenhum campo;
- **Texto**: uma `struct`-tupla, com um único elemento do tipo `String`, contendo o texto digitado;
- **Reacao**: uma `struct` convencional com os campos `emoji`, contendo um caractere unicode, e `id_mensagem_reagida`, contendo o identificador da mensagem que foi reagida.

Os `enum`s do Rust, assim como as `union`s do C, assumem um tamanho como o maior de seus elementos. Neste caso, sabemos que **Reacao** possui 4 (char) + 4 (i32) = 8 bytes, sendo assim o maior sub-elemento e, portanto, definindo o tamanho do `enum`. Como o compilador consegue saber o tamanho da estrutura, ela pode ser utilizada como parâmetro da função `App::envia_mensagem()`

### Padrões de acesso para Structs e Enums

Como podemos ver, uma variável de `enum` pode assumir diferentes `struct`s, então como podemos saber qual é o subtipo para poder acessar o campo correto?

Para responder a esta pergunta e, aproveitando, elaborar sobre os valores nulos discutidos [aqui](#por-que-não), vamos olhar para um `enum` extremamente útil da bilbioteca padrão: `Option<T>`:

```rust
// Nota: esta é apenas parte da implementação do enum
enum Option<T> {
  Some(T),
  None
}
```

Uma variável deste `enum` pode assumir duas formas: `Some(T)`, uma `struct`-tupla que contém um elemento do tipo `T` como único valor, ou `None`, que não possui nada. Este `enum` é utilizado em toda função ou método que pode ou não retornar algum dado, como os ponteiros nulos de C. Abaixo segue um exemplo de uso:

```rust
// Assumindo a implementação completa da função abaixo:
fn get_valor_campo_nome() -> Option<String>;

fn main() {
  let entrada = get_valor_campo_nome();

  match entrada {
    Some(nome) => println!("O nome do usuário é {}", nome),
    None => println!("O usuário não digitou seu nome!")
  }
}
```

Neste exemplo, que serviria para uma implementação de interface gráfica, nós temos a função `get_valor_campo_nome()` que pode, ou não, retornar uma string contendo o nome do usuário. Rust não permite que retiremos o valor de dentro da `Option` sem, primeiro, discriminá-la. Como `None` não possui este campo, acessá-lo causaria uma falha de segmentação.

Para discriminá-la, usamos o bloco `match`, que funciona como um `switch` com superpoderes. Nele colocamos o "braço" `Some(nome)`, que compara se a variável `entrada` é do subtipo `Some(T)` e, se sim, move o valor interno do tipo `T` para a variável `nome`.

Como a função `get_valor_campo_nome()` foi declarada a retornar `Option<String>`, o compilador sabe que `entrada` também é deste tipo e que `T` é `String`. Neste caso, ele consegue inferir que `nome` deve ser do tipo `String`. Este é mais um exemplo de como o compilador de Rust consegue te auxiliar durante o desenvolvimento.

---

## `impl`: Implementando métodos em Structs e Enums

Agora que `struct`s e `enum`s foram explicados, podemos falar das capacidades de orientação a objetos\* de Rust. Em Rust, podemos iniciar um "bloco de implementação" para uma `struct`. Este bloco definirá os métodos que poderão ser executados a partir de uma variável deste tipo.

<sub><sup>\* Mais uma vez, a OO de Rust é incomum, e, inclusive, existe quem diria que Rust não é realmente orientado a objetos. Independente de quem está certo, Rust possui alguns detalhes comumente associados a OO, assim como Lua possui.</sup></sub>

```rust
struct Retangulo {
  altura: f32,
  largura: f32
}

impl Retangulo {
  fn new(altura: f32, largura: f32) -> Self {
    Retangulo { altura, largura }
  }

  fn area(&self) -> f32 {
    self.altura * self.largura
  }
}

fn main() {
  let ret = Retangulo::new(3.5, 10f32);
  println!("A área é {}", ret.area());
}
```

Note que, no exemplo, definimos dois métodos: um método de classe `new` e um método de instância `area`. Distinguimos os dois pelo primeiro parâmetro: métodos de instância precisam receber um de três parâmetros iniciais:

1) `&self`: uma referência imutável ao objeto. Este tipo de acesso permite leitura de todos os campos;
2) `&mut self`: uma referência mutável ao objeto. Este tipo de acesso permite leitura e escrita de todos os campos;
3) `self`: o próprio objeto. Este tipo de acesso "move" o valor da variável original para a variável `self`, consumindo o objeto e deixando a variável originalmente dona inutilizável após o método retornar. Se isto te confundiu, refira-se aos primeiros exemplos [deste capítulo](#mutabilidade-e-empréstimo).

### Traços

Em Rust, ao invés de herança entre classes, `struct`s podem definir quais traços possuem. Traços nada mais são do que blocos de implementação com métodos especificamente nomeados e parametrados. Para exemplificar, vamos tentar imprimir nosso retângulo na tela.

```rust
let ret = Retangulo::new(3.5, 10f32);
println!("Meu retângulo: {}", ret);
```

O compilador prontamente nos aponta o erro:

```
error[E0277]: `Retangulo` doesn't implement `std::fmt::Display`
  --> t.rs:14:32
   |
14 |     println!("Meu retângulo: {}", ret);
   |                                   ^^^ `Retangulo` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Retangulo`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
```

Este erro vem acompanhado de possíveis soluções, mas vamos explorar a solução de implementar o traço `std::fmt::Display` para Retangulo.

```rust
struct Retangulo {
  // ... Omitido
}

impl Retangulo {
  // ... Omitido
}

// Novo codigo ---

// Traz o módulo `fmt` da biblioteca padrão para o escopo
use std::fmt;

impl fmt::Display for Retangulo {
	fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
		write!(f, "Ret {} x {}", self.altura, self.largura)?;
		Ok(())
	}
}

fn main() {
  let ret = Retangulo::new(3.5, 10f32);
  println!("Meu retângulo: {}", ret);
}

```

E então, quando executado, temos a saída:

```
Meu retângulo: Ret 3.5 x 10
```

Esse bloco trouxe vários detalhes ainda não discutidos, como o parâmetro genérico em `Formatter<'_>`, o operador `?` na chamada do macro `write!()` e `Ok(())`. Explicá-los foge do escopo deste documento, mas você pode encontrar explicações detalhadas direto de *"The Book"* respectivamente nos links: 

- `<'_>`: [A sintaxe de *lifetimes* para referências](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html);
- `?`: [Um atalho para a propagação de erros](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator);
- `Ok(())`: [Erros recuperáveis com `Result`](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)

---

## Epílogo: Considerações finais

Rust é uma linguagem relativamente nova, com apenas uma década de existência, enquanto C possui 5. Computadores e sistemas operacionais 50 anos atrás eram consideravalmente diferentes e suas limitações são responsáveis por terem moldado C e suas funcionalidades. Rust, por outro lado, é desenvolvido para resolver os problemas modernos, causados, principalmente, por linguagens mais antigas.

Por este motivo, aprender Rust pode parecer difícil, visto que a linguagem traz diversos operadores, etruturas e conceitos que não são encontrados em outras linguagens. É fácil, inicialmente, passar muito tempo sem conseguir compilar um código por conta das regras de empréstimos e *lifetimes*. Por que, então, Rust é tão amado?

Simples: o tempo que é gasto em compilar o código, não é gasto em debuggá-lo. É relativamente fácil desenvolver um código em C e compilá-lo. O que não é trivial, contudo, é descobrir onde acontece um *double free*, vazamento de memória, condição de corrida ou comportamento inesperado causado pelos famosos *"undefined behaviors"*.

Já em Rust, pode ser mais complicado<sup>**inicialmente**</sup> compilar o código, mas você terá uma garantia de que estes problemas mencionados não ocorrerão em tempo de execução. O usuário não descobrirá um problema causado pela falta de tratamento de erros por parte do programador. De fato, é um sentimento comum entre a comunidade de Rust de que, quando o código compila, ele funciona.

---

## Bônus: mais algumas funcionalidades

Nesta breve seção serão apresentadas algumas funcionalidades de Rust sem muita explicação ou exemplificação, puramente para título de curiosidade. Todas terão suas páginas no *"The Book"* propriamente *linkadas*.

### [Quebra de múltiplos loops](https://doc.rust-lang.org/book/ch03-05-control-flow.html#loop-labels-to-disambiguate-between-multiple-loops)

```rust
fn main() {

	'outer:
	for i in 0..10 {
		for j in 0..10 {
			
      println!("{} {}", i, j);

			if i == 2 && j == 2 {
        println!("Encontrei!");
				break 'outer;
			}
		}
	}
}
```

Em Rust, é possível quebrar múltiplos loops aninhados definindo um label para o loop alvo. Assim, o comando `break 'label` quebrará todos os laços até seu alvo. Essa funcionalidade é muito útil, por exemplo, quando buscando dados em uma matriz.

Adicionalmente, evitam-se variáveis booleanas e `if`s para quebrar os loops de dentro. Quando compilado para assembly, Rust simplesmente realiza um `jump`, aumentando a performance.

### [Desestruturação de tuplas e structs](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-to-break-apart-values)

Tuplas e `struct`s podem ser desestruturadas de múltiplas maneiras e em vários locais diferentes. Abaixo seguem alguns exemplos.


```rust
// Desestruturação de tuplas
let p = (1, 2);
let (x, y) = p; // x = 1, y = 2
```


```rust
// Desestruturando tuplas como parâmetros de funções
type Ponto = (i32, i32);

fn soma_pontos((x, y): Ponto, (xt, yt): Ponto) -> Ponto {
  (x + xt, y + yt)
}

fn main() {
  let p1 = (3, 4);
  let p2 = (10, 20);

  assert_eq!(soma_pontos(p1, p2), (13, 24));
}
```

```rust
// Desestruturando membros de struct
struct Aluno {
	nome: String,
	idade: u8,
	matricula: i32
}

impl Aluno {
	fn new(nome: &str, idade: u8, matricula: i32) -> Self {
		Aluno { nome: String::from(nome), idade, matricula }
	}
}

fn main() {
	let brenno = Aluno::new("Brenno Lemos", 22, 182050000);

  // Note que podemos ignorar a matrícula, por exemplo, como poderíamos ignorar qualquer campo
	let Aluno { nome, idade, .. } = brenno;

	println!("Nome: {}, idade: {}", nome, idade);
}
```

### [`derive`: implementações automáticas](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#appendix-c-derivable-traits)

```rust
#[derive(PartialEq, PartialOrd)]
struct Massa {
  kilos: i32,
  gramas: i32,
}

fn main() {
  let a = Massa { kilos: 1, gramas: 200 };
  let b = Massa { kilos: 2, gramas: 100 };
  let c = Massa { kilos: 2, gramas: 200 };

  assert!(a < b);
  assert!(b < c);
  assert!(a != c);
}
```

O código acima implementa os traços `PartialEq` e `PartialOrd`, responsáveis pelos operadores de igualdade e comparação automaticamente para a `struct Massa` utilizando o macro `derive`. Quando derivados, os traços criam uma implementação baseada na ordem de definição dos campos da `struct`, de mais importantes para menos importantes. Uma lista de todos os traços deriváveis da biblioteca padrão pode ser encontrada [aqui](https://doc.rust-lang.org/rust-by-example/trait/derive.html).

### Discriminação sem `match`: [`if let`](https://doc.rust-lang.org/book/ch06-03-if-let.html) e [`while let`](https://doc.rust-lang.org/rust-by-example/flow_control/while_let.html)

Quando `match` é utilizado para discriminar um `enum`, o compilador de Rust força o programador ou a escrever um caso para cada subtipo possível ou a definir um braço base (`_ => ..`). Contudo, existem casos nos quais faria mais sentido realizar tratamentos para alguns casos específicos ao invés de tratar o todo. Esse é um caso de uso para o bloco `if let`, que possui a seguinte sintaxe:

```rust
let input = pega_tecla_apertada();
if let Some(tecla) = input {
  println!("O usuário apertou a tecla {}", tecla);
}
```

De maneira similar, o `while let` pode ser utilizado para manter um laço enquanto o valor retornado casar com expressão.

```rust
let string_digitada = String::new();

while let Some(c) = pega_tecla_apertada() {
  string_digitada.push(c);
}

println!("O usuário digitou {}", string_digitada);
```

### [Programação funcional: Iterators e Closures](https://doc.rust-lang.org/book/ch13-00-functional-features.html)

```rust
fn main() {
  println!("O quadrado de todos os números pares de 0 a 20:");

  let quadrados_pares: Vec<i32> =
    (0..=20)
      .filter(|i| i&1 == 0)
      .map(|i| i * i)
      .collect();

    for i in quadrados_pares {
      print!("{} ", i);
    }
}
```

Lembre-se de que Rust, mesmo com métodos sofisticados, continua sendo uma linguagem para programação de sistemas e, portanto, é altamente otimizada. Isto faz parte da promessa *zero-cost abstractions*. Quando possível, Rust irá desenrolar os laços e realizar métodos *inline* para evitar trocas de contexto. A fim de exemplificar isto, observe os códigos de C e Rust abaixo:

```c
// C
// Compilado em clang, flags: -O2
int soma_quadrados_pares() {
  int s = 0;
  for (int i = 0; i <= 20; i++)
    if ((i&1) == 0)
      s += i * i;

  return s;
}
```

```rust
// Rust
// Compilado em rustc, flags: -C -opt-level=2
fn soma_quadrados_pares() -> i32 {
  (0..=20)
    .filter(|i| i&1 == 0)
    .map(|i| i * i)
    .sum()
}
```

Compilam para exatamente o mesmo código em assembly:

```asm
soma_quadrados_pares:
        mov     eax, 1540
        ret
```

Confira no [Compiler Explorer](https://godbolt.org).