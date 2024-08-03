# Seu primeiro projeto C3!
Vamos criar nosso primeiro projeto C3.
Criaremos um aplicativo de calculadora simples que adicionará dois números.

Mas antes disso precisamos instalar o C3C, O compilador do C3.

## Criando um novo projeto
O comando `c3c init` criará um novo diretório contendo a estrutura do seu projeto.
```bash
$ c3c init <project-name>
```

## Estrutura do projeto
Se você verificar o diretório que foi criado, poderá achar um pouco confuso ao ver diferentes
diretórios, mas não se preocupe, porque ao expandi-los, perceberá que a maioria deles está realmente vazia!

Vamos ver suas utilidades;
  - `./build` é onde seus arquivos temporários irão,
  - `./docs` são as documentações gerais do seu código,
  - `./lib` é onde as bibliotecas C3(sufixo .c3l) terminam
  - `./resources` são para recursos como imagens, efeitos sonoros, etc..
  - `./scripts` para vários scripts, incluindo scripts .c3 que podem ser executados em tempo de compilação, e gerar código
  - `./src` é o que realmente importa, pois é para onde irá a maior parte do nosso código.
  - Por último, o arquivo `project.json` que você adiciona mais dados sobre o seu projeto, parecido com package.json para node.js

Por enquanto vamos ignorar e pular para `src/main.c3`.

# Seu primeiro Olá mundo!
Vamos começar com um simples “Olá, Mundo!”.
O `Olá, Mundo!` mais simples que você pode escrever em C3:
```rust
import std::io;

fn void main()
{
    io::printn("Olá, Mundo!");
}
```

A instrução `import` importara outros módulos, usaremos `printn` que está em `std::io`
> [!NOTE]
> A importação é sempre recursiva, então `import std::io` na verdade importa `std::io` e seus submódulos,
> como `std::io::path`(lida com caminhos dos arquivos). Você poderia fazer `import std;`, o que importara toda a biblioteca padrão para o seu projeto!

A seguir definimos uma função, que começa com `fn` seguida pelo tipo de retorno.
Neste caso não precisamos retornar nada, então usamos `void`. Logo o nome da função, `main` seguido da lista de parâmetros, que está vazia.
```rust
fn void main() {}
```

> [!TIP]
> O `main` é um pouco especial, já que também é o ponto de entrada para o programa.
> Para sistemas operacionais do tipo Unix, existem variantes, por exemplo, podemos declará-lo como `fn void main(String[] args)`.
> Nesse caso o parâmetro _“args”_ contém uma fatia de strings, que correspondem aos argumentos da linha de comando, sendo o primeiro, nome da própria aplicação.

`{` - `}` significam o início e o fim da função, respectivamente. Dentro temos uma única chamada para a função `printn` em `std::io`. Usamos a última parte do caminho _“io”_ na frente da função para identificar qual módulo ela pertence.

> [!TIP]
> Poderíamos ter usado `std::io::printn` se quiséssemos. Só uma parte do caminho do módulo, como _“io::printn”_
> é conhecido como __“path-shortening”__, ou seja encurtamento do caminho, a forma comum de se referir a funções (Evite `std::io::printn`, não é idiomático).

A Função `io::printn` pega um único argumento e o imprime, seguido por um avanço de linha. Apos isso, a função termina/programa termina.
```rust
fn void main()
{
    io::printn("Olá, Mundo!");
}
```

## Compilando
Vamos pegar o código acima e colocar no arquivo `ola_mundo.c3`.

Podemos compilá-lo:
```bash
$ c3c compile ola_mundo.c3
```
E rode:
```bash
$ ./ola_mundo
```
Deve imprimir `Olá, Mundo!` e retornar ao prompt da linha de comando. Se você estiver no Windows, terá `ola_mundo.exe`. Chame-o da mesma maneira.

## Compilando e Executando
Quando começamos, pode ser útil compilar e fazer o compilador iniciar o programa imediatamente.
Podemos fazer isso com _compile-run_:
```bash
$ c3c compile-run ola_mundo.c3
> Program linked to executable 'ola_mundo'.
> Launching hello_world...
> Olá, Mundo!
```
Se você acompanhou até aqui: Parabéns! Agora você está preparado e pronto para utilizar o C3.

# Tipos basicos e Valores
C3 fornece um conjunto semelhante de tipos de dados fundamentais como C: integers, floats, arrays and pointers.
Também, ele expande esse conjunto adicionando fatias e vetores, bem como os tipos `any` e `typeid` para um uso avançado.

## Inteiros
C3 possui tipos inteiros(signed) assinados e não assinados(unsigned).

Tipos inteiros(signed) assinados integrados são `ichar`, `short`, `int`, `long`, `int128`, `iptr` e `isz`.
`ichar` a `int128` têm todos os tamanhos de bits de potência de dois bem definidos, enquanto `iptr` tem o mesmo tamanho de bits que um `void*` e `isz` que tem o mesmo tamanho de bits entre a diferença máxima de dois ponteiros.

Para cada tipo inteiro(signed) assinado existe um tipo inteiro(unsigned) não assinado correspondente: `char`, `ushort`, `uint`, `ulong`, `uint128`, `uptr` e `usz`.

*type* | *signed?* | *min* | *max* | *bits*
:--------- | :------: | -------: | -------: | -------:
ichar | yes | -128 | 127 | 8
short | yes | -32768 | 32767 | 16
int | yes | -2^31 | 2^31 - 1 | 32
long | yes | -2^63 | 2^63 - 1 | 64
int128 | yes | -2^127 | 2^127 - 1 | 128
iptr | yes | varies | varies | varies
isz | yes | varies | varies | varies
char | no | 0 | 255 | 8
ushort | no | 0 | 65535 | 16
uint | no | 0 | 2^32 - 1 | 32
ulong | no | 0 | 2^64 - 1 | 64
uint128 | no | 0 | 2^128 - 1 | 128
uptr | no | 0 | varies | varies
usz | no | 0 | varies | varies

Em máquinas de 64 bits `iptr`/`uptr` e `isz`/`usz` geralmente serão 64 bits, como `long`/`ulong`.
Por outro lado, em máquinas de 32 bits, geralmente serão `int`/`uint`.

## Inteiros Constantes
Constantes numéricos normalmente usam decimal, por ex. `234`,
Também podemos usar números hexadecimais(base 16) adicionando um prefixo `0x` ou `0X`, por exemplo `int a = 0x42edaa02;`. Também á octal(base 8) com o prefixo `0o` ou `0O`, e `0b` para números binários(base 2):

Os números também podem conter sublinhado `_` entre os dígitos para melhorar a legibilidade, por exemplo. `1_000_000`.
```c
a = -2_000;
b = 0o770;
c = 0x7f7f7f;
```
Para números decimais, o valor é assumido como sendo um signed `int`, a menos que não caiba em um `int`, caso que é assumido como o menor tipo de sinal em que cabe(`long` ou `int128`).

Para hexadecimal, octal e binário, o tipo é considerado unsigned.

Um inteiro literal pode ser convertido implicitamente em um literal de ponto flutuante ou em um número inteiro de um tipo diferente, desde que o número caiba no tipo.

## Sufixos constantes
Se quiser garantir que uma constante seja de um determinado tipo, você pode adicionar uma conversão explícita como:
`(ushort)345` ou usar um sufixo inteiro: `345u16`.

Os sufixos inteiros que estão disponíveis:
*suffix* | *type?*
:------ | :------:
i8 | ichar
i16 | short
i32 | int
i64 | long
i128 | int128
u8 | char
u16 | ushort
u32 | uint
u | uint
u64 | ulong
u128 | uint128

Observe que `uint` também tem o sufixo `u`.

## Booleanos
O booleano(bool) será `true` ou `false`. Embora um bool seja apenas um bit de dados, deve-se observar que ele é armazenado em um byte.
```c
bool b = true;
bool f = false;
```

## Caracteres Literais
Um  caracterer literal é o valor entre ``` '``' ```. Seu valor é interpretado como sendo o valor ASCII para um único caractere.

Também é possível usar literais de 2 caracteres, 4 ou 8. Eles são interpretados como `ushort`, `uint` e `ulong` respectivamente e são dispostos na memória da esquerda para a direita. Isso significa que o valor real depende da [endianess](https://en.wikipedia.org/wiki/Endianness) do alvo.

  - literais de 2 caracteres, Ex. 'C3' seria convertido em `ushort`.
  - Literais de 4 caracteres, Ex. 'TEST', converte para um `uint`.
  - Literais de 8 caracteres, Ex. 'FOOBAR11' é convertido em um `ulong`.

Os literais de 4 caracteres correspondem ao layout do código [FourCC](https://en.wikipedia.org/wiki/FourCC). Ele também organizará corretamente os caracteres Unicode na memória. Por exemplo. `Emoticon Char32 = '\u1F603'`

## Tipos de ponto flutuante
Comulmente como outras linguagens, C3 possui dois tipos de ponto flutuante: `float` e `double`. float é o tipo de 32 bits e `double` é de 64 bits.

## Constantes de ponto flutuante
Constantes de ponto flutuante usarão pelo menos uma precisão de 64 bits.
Assim como para constantes inteiras, é possível usar `_` para melhorar a legibilidade, mas pode não ocorrer imediatamente antes ou depois de um ponto ou exponencial.

C3 suporta valores de pontos flutuantes escritos no formato _decimal_ ou _hexadecimal_. Para decimal, o símbolo exponencial é _e_ (ou _E_, ambos são aceitáveis), para hexadecimal _p_ (ou _P_) é usado: `-2.22e-21 -0x21.93p-10`

Embora os números de ponto flutuante sejam padronizados como `double`, é possível digitar um ponto flutuante adicionando um sufixo:
*Suffix* | *type?*
:------ | :------:
f32 or f | float
f64 | double

## Arrays
Arrays têm o formato `Type[size]`, então por exemplo: `int[4]`.
Uma matriz é um tipo que consiste no mesmo elemento repetido várias vezes. Nosso `int[4]` é essencialmente quatro valores `int` agrupados.

Para inicialização, às vezes é conveniente usar a declaração curinga `Type[*]`, que infere o comprimento a partir do número de elementos:
```c
int[3] abc = { 1, 2, 3 }; // Explícito int[3]
int[*] bcd = { 1, 2, 3 }; // Explícito int[3]
```
Leia mais sobre como inicializar arrays em [capítulo sobre arrays](https://c3-lang.org/references/docs/arrays/).

## Slices
Os Slices|Fatias têm o formato `Type[]`. Ao contrário da matriz, uma fatia não contém os valores em si, mas apresenta uma visão de alguma matriz ou vetor subjacente.

Slices|Fatias têm duas propriedades: `.ptr`, que recupera o array para o qual aponta, e `.len` que é o comprimento da fatia - ou seja, o número de elementos nos quais é possível indexar.

Normalmente podemos obter uma fatia pegando o endereço de um array:
```c
int[3] abc = { 1, 2, 3 };
int[] slice = &abc;       // Fatia apontando para abc, com o comprimento 3
```

Como a indexação de fatias é verificada no modo de segurança, as fatias são muito mais seguras fornecendo _pointer + length_ separadamente.

## Vectors
Semelhante aos arrays, os vetores usam o formato `Tipo[<tamanho>]`, com uma restrição de que os vetores só podem ser criados a partir de números inteiros, flutuantes e booleanos. Semelhante aos arrays, o curinga pode ser usado para inferir o tamanho de um vetor:
```c
int[<*>] a = { 1, 2 };
```
Os vetores são baseados em SIMD vectors de hardware e suportam muitas operações diferentes que funcionam em todos os elementos em paralelo, incluindo aritmética:
```c
int[<2>] b = { 3, 8 };
int[<2>] c = { 7, 2 };
int[<2>] d = b * c;    // d é { 21, 16 }
```
Inicialização de vetores e literais funcionam da mesma maneira que arrays, usando `{ ... }`

## String literais
Como C, literais de string são textos entre `" "` assim como em C.
Eles suportam sequências de escape como `\n` para quebra de linha e precisam usar `\"` para qualquer `"` dentro da string.

C3 oferece strings brutas que são colocadas entre ``` ` ` ```. Dentro de uma string bruta, nenhum escape está disponível, e para escrever um ``` ` ```, simplesmente dobre o caractere:
```rust
char* foo = `C:\foo\bar.dll`;
char* bar = `"Say ``hello``"`;
// É igual a
char* foo = "C:\\foo\\bar.dll";
char* bar = "\"Say `hello`\"";
```

 String literais são especiais porque podem ser convertidos em vários tipos diferentes: arrays e slices de `String`, `char` e `ichar` e finalmente `ichar*` e `char*`.

## Literais de dados Base64 e Hexadecimais
Literais Base64 são strings prefixadas com `b64` para conter dados [codificados em Base64](https://en.wikipedia.org/wiki/Base64), que são convertidos em um _array char_ em tempo de compilação:
```c
// A matriz abaixo contém os caracteres "Hello World!"
char[*] hello_world_base64 = b64"SGVsbG8gV29ybGQh";
```
Literais de dados hexadecimais correspondentes convertem uma string hexadecimal em vez de Base64:
```c
// A matriz abaixo contém os caracteres "Hello World!"
char[*] hello_world_hex = x"4865 6c6c 6f20 776f 726c 6421";
```

## Tipos de ponteiro
Os ponteiros possuem a sintaxe `Type*`.
Um ponteiro é um endereço de memória onde um ou possivelmente mais elementos do endereço subjacente são armazenados. Os ponteiros podem ser empilhados: `Foo*` é um ponteiro para `Foo` enquanto `Foo**` é um ponteiro para um ponteiro para `Foo`.

O tipo de ponteiro possui um literal especial chamado `null`, que é um ponteiro vazio e inválido.

## `void*`
O tipo `void*` é um ponteiro especial que converte implicitamente em qualquer outro ponteiro. Não é “a pointer to void”, mas sim um ponteiro curinga que corresponde a qualquer outro ponteiro.

## Imprimindo valores
A impressão de valores pode ser feita usando `io::print`, `io::printn`, `io::printf` e `io::printfn`.
Isso requer a importação do módulo `std::io`.

> [!NOTE]
> As variantes n das funções de impressão adicionarão uma nova linha após a impressão, que é o que usaremos frequentemente nos exemplos, mas `print` e `printf` funcionam da mesma maneira.
```rust
import std::io; // Obtenha as funções io.

fn void main()
{
    int a = 1234;
    ulong b = 0xFFAABBCCDDEEFF;
    double d = 13.03e-04;

    char[*] hex = x"4865 6c6c 6f20 776f 726c 6421";

    io::printn(a);
    io::printn(b);
    io::printn(d);
    io::printn(hex);
}
```

Se você executar este programa, obterá:
```c
1234
71963842633920255
0.001303
[72, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100, 33]
```

Para obter mais controle, podemos formatar a saída usando `printf` e `printfn`:
```rust
import std::io;
fn void main()
{
    int a = 1234;
    ulong b = 0xFFAABBCCDDEEFF;
    double d = 13.03e-04;

    char[*] hex = x"4865 6c6c 6f20 776f 726c 6421";

    io::printfn("a era:                       %d", a);
    io::printfn("b hexadecimal é:             %x", b);
    io::printfn("d em notação científica foi: %e", d);
    io::printfn("Bytes como string:           %s", (String)&hex);
}
```

Podemos aplicar as [regras de formatação padrão do printf](https://en.cppreference.com/w/c/io/fprintf), mas diferentemente do C/C++ não há necessidade de indicar o tipo ao usar `%d` - ele irá imprimir sem assinatura e assinado em `int128`, na verdade não há suporte para `%u`, `%lld` etc, em `io::printf`. Além disso, `%s` funciona não apenas em strings, mas em qualquer tipo:
```rust
import std::io;

enum Foo
{
    ABC,
    BCD,
    EFG,
}
fn void main()
{
    int a = 1234;
    uint128 b = 0xFFEEDDCC_BBAA9988_77665544_33221100;

    Foo foo = BCD;
    io::printfn("a: %s, b: %d, foo: %s", a, b, foo);
}
```

Imprime:
```h
a: 1234, b: 340193404210632335760508365704335069440, foo: BCD
```
