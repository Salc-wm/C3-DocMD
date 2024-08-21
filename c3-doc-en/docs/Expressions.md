Expressions work like in __C__, with one exception: it is possible to take the address of a temporary.
This uses the operator `&&` rather than `&`.

Consequently, this is valid:
```c
fn void test(int* x) { ... }

test(&&1);

// In C:
// int x = 1;
// test(&x);
```

# Well-defined evaluation order
Expressions have a well-defined evaluation order:

  1. __Binary__ expressions are evaluated from _left_ to _right_.
  2. Assignment occurs _right_ to _left_, so `a = a++` would result in `a` being _unchanged_.
  3. Call arguments are evaluated in _parameter_ order.
  4. For named parameters, evaluation is in __parameter__ order, not argument order. So the evaluation order of `foo(.a = x++, .b = x--)` depends on the declaration order of `a` and `b`.

# Compound literals
__C3__ has __C__’s compound _literals_, but unlike __C__’s cast _style syntax_ `(MyStruct) { 1, 2 }`, it uses __C++__ syntax: `MyStruct { 1, 2 }`.
```go
struct Foo
{
    int a;
    double b;
}

fn void test1(Foo x) { ... }

...

test1(Foo { 1, 2.0 });
```

### Arrays follow the same syntax:
```rust
fn void test2(int[3] x) { ... }

...

test2(int[3] { 1, 2, 3 });
```

Note that when it’s possible, inferring the type is allowed, so we have for the above examples:
```c
test1({ 1, 2.0 });
test2({ 1, 2, 3 });
```

One may take the _address_ of _temporaries_, using `&&` (_rather than `&` for normal variables_).
This allows the following:

Passing a slice
```c
fn void test(int[] y) { ... }

// Using &&
test(&&int[3]{ 1, 2, 3 });

// Explicitly slicing:
test(int[3]{ 1, 2, 3 }[..]));

// Using a slice directly as a temporary:
test(int[]{ 1, 2, 3 }));
```

Passing the pointer to an array
```rust
fn void test1(int[3]* z) { ... }
fn void test2(int* z) { ... }

test1(&&int[3]{ 1, 2, 3 }));
test2(&&int[3]{ 1, 2, 3 }));
```

# Constant expressions
In __C3__ all constant expressions are guaranteed to be calculated at compile time.
The following are considered constant expressions:

  1. The `null` literal.
  2. _Boolean_, _floating_ point and _integer_ __literals__.
  3. The result of arithmetics on constant expressions.
  4. Compile time variables (_prefixed with `$`_)
  5. Global constant variables with initializers that are constant expressions.
  6. The result of macros that does not generate code and only uses constant expressions.
  7. The result of a cast if the value is cast to a _boolean_, _floating_ __point__ or _integer_ type and the value that is converted is a constant expression.
  8. _String_ literals.
  9. Initializer lists containing constant values.

Some things that are _not_ __constant__ expressions:

  1. Any pointer that isn’t the `null` literal, even if it’s derived from a constant expression.
  2. The result of a cast except for casts of constant expressions to a numeric type.
  3. Compound literals - even when values are constant expressions.

## Including binary data
The `$embed(...)` function includes the contents of a file into the compilation as a _constant array_ of bytes:

```c
char[*] my_image = $embed("my_image.png");
```
The result of an embed work similar to a _string literal_ and can implicitly convert to a `char*`, `void*`, `char[]`, `char[*]` and `String`.

### Limiting length
It’s possible to limit the length of included with the optional second parameter.
```rust
char[4] my_data = $embed("foo.txt", 4);
```

### Failure to load at compile time and defaults
Usually it’s a compile time error if the file can’t be included, but sometimes it’s useful to only optionally include it.
If this is desired, declare the left hand side to be an optional:
```c
char[]! my_image = $embed("my_image.png");
```

`my_image` with be an optional `IoError.FILE_NOT_FOUND?` if the image is missing.

This also allows us to _pass_ a default value using `??`:
```c
char[] my_image = $embed("my_image.png") ?? DEFAULT_IMAGE_DATA;
```