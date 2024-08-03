# You first C3 Project
Let’s create our first C3 project.
We will create a simple calculator app that will add two numbers.

But before that we need to install C3C, the compiler for C3.

## Creating a new project
The `c3c init` comamnd will create a new directory containing your project structure.
```bash
$ c3c init <project-name>
```

## Project structure
If you check the directory that was created you might find it a bit confusing with a bunch of different
directories but worry not because if you expand them you will realise that most of them are actually empty!

Let’s go over some of the useful ones;
  - `./build` is where your temporary files will end up,
  - `./docs` are the overall documentations for your code,
  - `./lib` is where C3 libraries (with the .c3l suffix) end up
  - `./resources` are for any resources like images, sound effects etc.
  - `./scripts` is for various scripts, including .c3 scripts that can run at compile time to generate code
  - The `./src` folder is what we care about as it’s where a majority of our code will go.
  - And lastly the project.json file is where you add more data about your project, sort of like package.json for node.js

For now we can ignore all of these and jump into `src/main.c3`

# You first Hello World
Let’s start with a simple “Hello World” program.
The simplest Hello World you can write in C3 looks like this:
```rust
import std::io;

fn void main()
{
    io::printn("Hello, World!");
}
```
The `import` statement imports other modules, and we want `printn` which is in `std::io`
> [!NOTE]
> Import is always recursive, so `import std::io` actually imports both `std::io` and its sub-modules,
> such as `std::io::path` (which handles file paths). You could even do `import std;`, which would import the entire standard library into your project!

Next we define a function, which starts with the `fn` keyword followed by the return type.
In this case we don’t need to return anything, so we use `void`. Then follows the name of the function, `main` followed by the parameter list, which is empty.

```rust
fn void main() {}
```

> [!TIP]
> The name `main` is a bit special as it is also the entry point to the program.
> For Unix-like OSes there are a few different variants, for example we might declare it as `fn void main(String[] args)`.
> In that case the parameter “args” contains a slice of strings, which correspond to the command line arguments, with the first one being the name of the application itself.

`{` and `}` signifies the start and end of the function respectively. Inside we have a single call to the function `printn` in `std::io`. We use the last part of the path “io” in front of the function to identify what module it belongs to.

> [!TIP]
> We could also have used `std::io::printn` if we wanted. Just a part of the module path, like “io::printn”
> is known as “path-shortening” and is the common way of referring to functions (avoid `std::io::printn`, it’s not idiomatic).

The io::printn function takes a single argument and prints it, followed by a line feed After this the function ends and the program terminates.
```rust
fn void main()
{
    io::printn("Hello, World!");
}
```

## Compiling the program
Let’s take the above program and put it in a file called `hello_world.c3`.

We can then compile it:
```bash
$ c3c compile hello_world.c3
```
And run it:
```bash
$ ./hello_world
```

It should print `Hello, World!` and return back to the command line prompt. If you are on Windows, you will have `hello_world.exe` instead. Call it in the same way.

## Compiling and running
When we start out it can be useful to compile and then have the compiler start the program immediately.
We can do that with compile-run:
```bash
$ c3c compile-run hello_world.c3
> Program linked to executable 'hello_world'.
> Launching hello_world...
> Hello, World
```
If you followed along so far: Congratulations! You’re now up and running with C3.

# Basic types and values
C3 provides a similar set of fundamental data types as C: integers, floats, arrays and pointers.
On top of this it expands on this set by adding slices and vectors, as well as the `any` and `typeid` types for advanced use.

## Integers
C3 has signed and unsigned integer types.
The built-in signed integer types are `ichar`, `short`, `int`, `long`, `int128`, `iptr` and `isz`.
`ichar` to `int128` have all well-defined power-of-two bit sizes, whereas `iptr` has the same number of bits as a `void*` and `isz` has the same number of bits as the maximum difference between two pointer.
For each signed integer type there is a corresponding unsigned integer type: `char`, `ushort`, `uint`, `ulong`, `uint128`, `uptr` and `usz`.

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

On 64-bit machines `iptr`/`uptr` and `isz`/`usz` are usually 64-bits, like `long`/`ulong`.
On 32-bit machines on the other hand they are generally `int`/`uint`.

## Integer constants
Numeric constants typically use decimal, e.g.
`234`, but may also use hexadecimal (base 16) numbers by prefixing the number with `0x` or `0X`, e.g. `int a = 0x42edaa02;`. There is also octal (base 8) using the `0o` or `0O` prefix, and `0b` for binary (base 2) numbers:

Numbers may also insert underscore `_` between digits to improve readability, e.g. `1_000_000`.
```c
a = -2_000;
b = 0o770;
c = 0x7f7f7f;
```
For decimal numbers, the value is assumed to be a signed `int`, unless the number doesn’t fit in an `int`, in which case it is assumed to be the smallest signed type it does fit in (`long` or `int128`).

For hexadecimal, octal and binary, the type is assumed to be unsigned.

A integer literal can implicitly convert to a floating point literal, or an integer of a different type provided the number fits in the type.

## Constant suffixes
If you want to ensure that a constant is of a certain type, you can either add an explicit cast like:
`(ushort)345`, or use an integer suffix: `345u16`.

The following integer suffixes are available:
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

Note how `uint` also has the `u` suffix.

## Booleans
A bool will be either `true` or `false`. Although a bool is only a single bit of data, it should be noted that it is stored in a byte.
```c
bool b = true;
bool f = false;
```

## Character literals
A character literal is a value enclosed in '``'. Its value is intepreted as being its ASCII value for a single character.

It is also possible to use 2, 4 or 8 character wide character literals. Such are interpreted as ushort, `uint` and `ulong` respectively and are laid out in memory from left to right. This means that the actual value depends on the [endianess](https://en.wikipedia.org/wiki/Endianness) of the target.

  - 2 character literals, e.g. 'C3', would convert to an ushort.
  - 4 character literals, e.g. 'TEST', converts to an uint.
  - 8 character literals, e.g. 'FOOBAR11' converts to an ulong.

The 4 character literals correspond to the layout of [FourCC](https://en.wikipedia.org/wiki/FourCC) codes. It will also correctly arrange unicode characters in memory. E.g. `Char32 smiley = '\u1F603'`

## Floating point types
As is common, C3 has two floating point types: `float` and `double`. float is the 32 bit floating point type and `double` is 64 bits.

## Floating point constants
Floating point constants will at least use 64 bit precision.
Just like for integer constants, it is possible to use `_` to improve readability, but it may not occur immediately before or after a dot or an exponential.

C3 supports floating points values either written in decimal or hexadecimal formats. For decimal, the exponential symbol is e (or E, both are acceptable), for hexadecimal p (or P) is used: `-2.22e-21 -0x21.93p-10`

While floating point numbers default to `double` it is possible to type a floating point by adding a suffix:
*Suffix* | *type?*
:------ | :------:
f32 or f | float
f64 | double

## Arrays
Arrays have the format `Type[size]`, so for example: `int[4]`.
An array is a type consisting of the same element repeated a number of times. Our `int[4]` is essentially four `int` values packed together.

For initialization it’s sometimes convenient to use the wildcard `Type[*]` declaration, which infers the length from the number of elements:
```c
int[3] abc = { 1, 2, 3 }; // Explicit int[3]
int[*] bcd = { 1, 2, 3 }; // Implicit int[3]
```
Read more about initializing arrays in the [chapter on arrays](https://c3-lang.org/references/docs/arrays/).

## Slices
Slices have the format `Type[]`. Unlike the array, a slice does not hold the values themselves but instead presents a view of some underlying array or vector.

Slices have two properties: `.ptr`, which retrieves the array it points to, and `.len` which is the length of the slice - that is, the number of elements it is possible to index into.

Usually we can get a slice by taking the address of an array:
```c
int[3] abc = { 1, 2, 3 };
int[] slice = &abc;       // A slice pointing to abc with length 3
```

Because indexing into slices is range checked in safe mode, slices are vastly more safe providing pointer + length separately.

## Vectors
Similar to arrays, vectors use the format `Type[<size>]`, with the restriction that vectors may only form out of integers, floats and booleans. Similar to arrays, wildcard can be used to infer the size of a vector:
```c
int[<*>] a = { 1, 2 };
```
Vectors are based on hardware SIMD vectors, and supports many different operations that work on all elements in parallel, including arithmetics:
```c
int[<2>] b = { 3, 8 };
int[<2>] c = { 7, 2 };
int[<2>] d = b * c;    // d is { 21, 16 }
```
Vector initialization and literals work the same way as arrays, using `{ ... }`

## String literals
Like C, string literals is text enclosed in `" "` just like in C.
These support escape sequences like `\n` for line break and need to use `\"` for any `"` inside of the string.

C3 also offers raw strings which are enclosed in ``` ` ` ```. Inside of a raw string, no escapes are available, and to write a ``` ` ```, simply double the character:
```rust
char* foo = `C:\foo\bar.dll`;
char* bar = `"Say ``hello``"`;
// Same as
char* foo = "C:\\foo\\bar.dll";
char* bar = "\"Say `hello`\"";
```

String literals are special in that they can convert to several different types: `String`, `char` and `ichar` arrays and slices and finally `ichar*` and `char*`.

## Base64 and hex data literals
Base64 literals are strings prefixed with `b64` to containing [Base64 encoded](https://en.wikipedia.org/wiki/Base64) data, which is converted into a char array at compile time:
```c
// The array below contains the characters "Hello World!"
char[*] hello_world_base64 = b64"SGVsbG8gV29ybGQh";
```
The corresponding hex data literals convert a hexadecimal string rather than Base64:
```c
// The array below contains the characters "Hello World!"
char[*] hello_world_hex = x"4865 6c6c 6f20 776f 726c 6421";
```

## Pointer types
Pointers have the syntax `Type*`.
A pointer is a memory address where one or possibly more elements of the underlying address is stored. Pointers can be stacked: `Foo*` is a pointer to a `Foo` while `Foo**` is a pointer to a pointer to `Foo`.

The pointer type has a special literal called null, which is an invalid, empty pointer.

## `void*`
The `void*` type is a special pointer which implicitly converts to any other pointer. It is not “a pointer to void”, but rather a wildcard pointer which matches any other pointer.

## Printing values
Printing values can be done using `io::print`, `io::printn`, `io::printf` and `io::printfn`.
This requires importing the module `std::io`.

> [!NOTE]
> The n variants of the print functions will add a newline after printing, which is what we’ll often use in the examples, but `print` and `printf` work the same way.
```rust
import std::io; // Get the io functions.

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
If you run this program you will get:

```c
1234
71963842633920255
0.001303
[72, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100, 33]
```

To get more control we can format the output using `printf` and `printfn`:
```rust
import std::io;
fn void main()
{
    int a = 1234;
    ulong b = 0xFFAABBCCDDEEFF;
    double d = 13.03e-04;
    char[*] hex = x"4865 6c6c 6f20 776f 726c 6421";
    io::printfn("a was:                        %d", a);
    io::printfn("b in hex was:                 %x", b);
    io::printfn("d in scientific notation was: %e", d);
    io::printfn("Bytes as string:              %s", (String)&hex);
}
```

We can apply the [standard printf formatting rules](https://en.cppreference.com/w/c/io/fprintf), but unlike in C/C++ there is no need to indicate the type when using `%d` - it will print unsigned and signed up to `int128`, in fact there is no support for `%u`, `%lld` etc in `io::printf`. Furthermore, `%s` works not just on strings but on any type:
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

This prints:
```h
a: 1234, b: 340193404210632335760508365704335069440, foo: BCD
```
