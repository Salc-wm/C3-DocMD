# A quick primer on C3
This primer is intended as a guide to how the C syntax – and in some cases C semantics – is different in C3.
It is intended to help you take a piece of C code and understand how it can be converted manually to C3.

## Struct, enum and union declarations
Don’t add a `;` after enum, struct and union declarations, and note the slightly different syntax for declaring a named struct inside of a struct.

```c
// C
typedef struct
{
    int a;
    struct
    {
        double x;
    } bar;
} Foo;

// C3
struct Foo
{
    int a;
    struct bar
    {
        double x;
    }
}
```
Also, user defined types are used without a struct, union or enum keyword, as if the name was a C typedef.

### Arrays
Array sizes are written next to the type and arrays do not decay to pointers, you need to do it manually:
```c
// C
int x[2] = { 1, 2 };
int *y = x;

// C3
int[2] x = { 1, 2 };
int* y = &x;
```

You will probably prefer slices to pointers when passing data around:
```c
// C
int x[100] = ...;
int y[30] = ...;
int z[15] = ...;

sort_my_array(x, 100);
sort_my_array(y, 30);

// Sort part of the array!
sort_my_array(z + 1, 10);
```
```c
// C3
int[100] x = ...;
int[30] y = ...;

sort_my_array(&x); // Implicit conversion from int[100]* -> int[]
sort_my_array(&y); // Implicit conversion from int[30]* -> int[]

sort_my_array(z[1..10]); // Inclusive ranges!
```

Note that declaring an array of inferred size will look different in C3:
```c
// C
int x[] = { 1, 2, 3 }; // x is int[3]
```
```c
// C3
int[*] x = { 1, 2, 3 }; // x is int[3]
```

Arrays are trivially copyable:
```c
// C
int x[3] = ...;
int y[3];

for (int i = 0; i < 3; i++) y[i] = x[i];
```
```c
// C3
int[3] x = ...;
int[3] y = x;
```
See more _[here](https://c3-lang.org/references/docs/arrays/)_.

### Undefined behaviour
C3 has less undefined behaviour, in particular integers are defined as using 2s complement and signed overflow is wrapping.
See more _[here](https://c3-lang.org/references/docs/undefinedbehaviour/)_.

### Functions
Functions are declared like _C_, but you need to put `fn` in front:
```c
// C:
int foo(Foo *b, int x, void *z) { ... }
```
```rust
// C3
fn int foo(Foo* b, int x, void* z) { ... }
```
See more about functions, like named and default arguments _[here](https://c3-lang.org/references/docs/functions/)_.

### Calling C functions
Declare a function (or variable) with `extern` and it will be possible to access it from __C3__:
```c
// To access puts:
extern fn int puts(char*);
...

puts("Hello world");
```

Note that currently only the _C_ standard library is automatically passed to the linker.
In order to link with other libraries, you need to explicitly tell the compiler to link them.

If you want to use a different identifier inside of your __C3__ code compared to the function or variable’s external name, use the `@extern` attribute:
```c
extern fn int _puts(char* message) @extern("puts");
...

_puts("Hello world"); // <- calls the puts function in libc
```

### Identifiers
Name standards are enforced:
```c
// Starting with uppercase and followed somewhere by at least
// one lower case is a user defined type:
Foo x;
M____y y;

// Starting with lowercase is a variable or a function or a member name:

x.myval = 1;
int z = 123;
fn void fooBar(int x) { ... }

// Only upper case is a constant or an enum value:

const int FOOBAR = 123;
enum Test
{
  STATE_A = 0,
  STATE_B = 2
}
```

### Variable declaration
Multiple declarations together with initialization isn’t allowed in __C3__:
```c
// C
int a, b = 4; // Not allowed in C3
```
```c
// C3
int a;
int b = 4;
```

In __C3__, variables are always zero initialized, unless you explicitly opt out using `@noinit`:
```c
// C
int a = 0;
int b;
```
```c
// C3
int a;
int b @noinit;
```

### Compound literals
Compound literals use _C++_ style brace initialization, not cast style like in __C__.
For convenience, assigning to a struct will infer the type even if it’s not an initializer.
```c++
// C
Foo f = { 1, 2 };
f = (Foo) { 1, 2 };
callFoo((Foo) { 2, 3 });
```
```rust
// C3
Foo f = { 1, 2 };
f = { 1, 2 };
callFoo(Foo{ 2, 3 });
```

### typedef and #define becomes ‘def’
`typedef` is replaced by `def`:
```c
// C
typedef Foo* FooPtr;
```
```python
// C3
def FooPtr = Foo*;
```

`def` also allows you to do things that __C__ uses `#define` for:
```c
// C
#define println puts
#define my_excellent_string my_string

char *my_string = "Party on";
...

println(my_excellent_string);
```
```python
// C3
def println = puts;
def my_excellent_string = my_string;

char* my_string = "Party on";
...

println(my_excellent_string);
```

Read more about `def` _[here](https://c3-lang.org/references/docs/define/)_.

### Basic types
Several __C__ types that would be variable sized are fixed size, and others changed names:
```c
// C
int16_t a;
int32_t b;
int64_t c;
uint64_t d;
size_t e;
ssize_t f;
ptrdiff_t g;
intptr_t h;
```
```c
// C3
short a;    // Guaranteed 16 bits
int b;      // Guaranteed 32 bits
long c;     // Guaranteed 64 bits

ulong d;    // Guaranteed 64 bits
int128 e;   // Guaranteed 128 bits
uint128 f;  // Guaranteed 128 bits
usz g;      // Same as C size_t, depends on target
isz h;      // Same as C ptrdiff_t
iptr i;     // Same as intptr_t depends on target
uptr j;     // Same as uintptr_t depends on target
```

Read more about types _[here](https://c3-lang.org/references/docs/types/)_.

### Instead of #include: Modules and import
Declaring the module name is not mandatory, but if you leave it out the file name will be used as the module name. Imports are recursive.
```rust
module otherlib::foo;

fn void test() { ... }
struct FooStruct { ... }

module mylib::bar;
import otherlib;
fn void myCheck()
{
  foo::test(); // foo prefix is mandatory.
  mylib::foo::test(); // This also works;
  FooStruct x; // But user defined types don't need the prefix.
  otherlib::foo::FooStruct y; // But it is allowed.
}
```

### Comments
The `/* */` comments are nesting
```js
/* This /* will all */ be commented out */
```

Note that doc comments, starting with `/**` has special rules for parsing it, and is not considered a regular comment.
See [contracts](https://c3-lang.org/references/docs/contracts/) for more information.

### Type qualifiers
Qualifiers like `const` and `volatile` are removed, but `const` before a constant will make it treated as a compile time constant. The constant does not need to be typed.
```c
const A = false;
// Compile time
$if A:
  // This will not be compiled
$else
  // This will be compiled
$endif
```

`volatile` is replaced by macros for volatile load and store.

### Goto removed
`goto` is removed, but there is labelled `break` and `continue` as well as `defer` to handle the cases when it is commonly used in __C__.
```c
// C
Foo *foo = malloc(sizeof(Foo));

if (tryFoo(foo)) goto FAIL;
if (modifyFoo(foo)) goto FAIL;

free(foo);
return true;

FAIL:
free(foo);
return false;
```
```rust
// C3, direct translation:
do FAIL:
{
    Foo *foo = malloc(sizeof(Foo));

    if (tryFoo(foo)) break FAIL;
    if (modifyFoo(foo)) break FAIL;

    free(foo);
    return true;
}
free(foo);
return false;

// C3, using defer:
Foo *foo = malloc(Foo);
defer free(foo);

if (tryFoo(foo)) return false;
if (modifyFoo(foo)) return false;

return true;
```

### Changes in `switch`
`case` statements automatically break.
Use `nextcase` to fallthrough to the next statement, but empty case statements have implicit fallthrough:
```c
// C
switch (a)
{
    case 1:
    case 2:
        doOne();
        break;
    case 3:
        i = 0;
    case 4:
        doFour();
        break;
    case 5:
        doFive();
    default:
        return false;
}
```
```rust
// C3
switch (a)
{
    case 1:
    case 2:
        doOne();
    case 3:
        i = 0;
        nextcase;
    case 4:
        doFour();
    case 5:
        doFive();
        nextcase;
    default:
        return false;
}
```

Note that we can jump to an arbitrary case using __C3__:
```c
// C
switch (a)
{
    case 1:
        doOne();
        goto LABEL3;
    case 2:
        doTwo();
        break;
    case 3:
        LABEL3:
        doThree();
    default:
        return false;
}
```
```rust
// C3
switch (a)
{
    case 1:
        doOne();
        nextcase 3;
    case 2:
        doTwo();
    case 3:
        doThree();
        nextcase;
    default:
        return false;
}
```

## Bitfields are replaced by explicit bitstructs
A `bitstruct` has an explicit container _type_, and each field has an exact bit range.
```c
bitstruct Foo : short
{
    int a : 0..2; // Exact bit ranges, bits 0-2
    uint b : 3..6;
    MyEnum c : 7..13;
}
```

There exists a simplified form for a bitstruct containing only booleans, it is the same except the ranges are left out:
```rust
struct Flags : char
{
    bool has_hyperdrive;
    bool has_tractorbeam;
    bool has_plasmatorpedoes;
}
```
For more information see the _[section on bitstructs](https://c3-lang.org/references/docs/types/#bitstructs)_.

# Other changes
The following things are enhancements to __C__, that does not have a direct counterpart in __C__.
  - Expression blocks
  - Defer
  - Methods
  - Optionals
  - Semantic macros
  - Generic modules
  - Contracts
  - Compile time evaluation
  - Reflection
  - Operator overloading
  - Macro methods
  - Static initialize and finalize functions
  - Dynamic interfaces

__For the full list of all new features__ see the _feature list_.

__Finally__, the _[FAQ](https://c3-lang.org/references/docs/faq/)_ answers many questions you might have as you _start_ out.
