# The _“def”_ statement
The `def` statement in __C3__ is intended for aliasing identifiers and types.

## Defining a type alias
`def <type alias> = <type>` creates a type alias.
Type aliases need to follow the name convention of user defined types (i.e. _capitalized names with at least one lower case letter_).
```python
def CharPtr = char*;

def Numbers = int[10];
```

Function pointers must be aliased in __C3__.
The _syntax_ is somewhat different from __C__:
```python
def Callback = fn void(int a, bool b);
```

This defines an alias to _function pointer_ type of a function that returns nothing and requires two arguments: an `int` and a `bool`.
Here is a sample usage:
```cpp
Callback cb = my_callback;
cb(10, false);
```

# Distinct types
Similar to `def` aliases are `distinct` which create distinct new _types_.
Unlike type _aliases_, they do not implicitly convert to or from any other type.
Literals will convert to the `distinct` types if they would convert to the _underlying_ type.
```go
distinct Foo = distinct int;

Foo f = 0; // Valid since 0 converts to an int.
f = f + 1;
int i = 1;

// f = f + i Error!
f = f + (Foo)i; // Valid
```

## Distinct inline
When interacting with various __APIs__ it is sometimes desirable for `distinct` types to implicitly convert to its _base type_, but not from that type.

Behaviour here is analogous how _structs_ may use `inline` to create _struct subtypes_.
```rust
distinct CString = char*;
distinct ZString = inline char*;
...
CString abc = "abc";
ZString def = "def";

// char* from_abc = abc; // Error!
char* from_def = def; // Valid!
```

# Function and variable aliases
`def` can also be used to create aliases for functions and variables.

The _syntax_ is `def <alias> = <original identifier>`.
```c
fn void foo() { ... }
int foo_var;

def bar = foo;
def bar_var = foo_var;

fn void test()
{
  // These are the same:
  foo();
  bar();

  // These access the same variable:
  int x = foo_var;
  int y = bar_var;
}
```

## Using def to create generic types, functions and variables
It is recommended to favour using `def` to create _aliases_ for parameterized types, __functions__ and __variables__:
```go
 import generic_foo;

 // Parameterized function aliases
 def int_foo_call = generic_foo::foo_call(<int>);
 def double_foo_call = generic_foo::foo_call(<double>);

 // Parameterized type aliases
 def IntFoo = Foo(<int>);
 def DoubleFoo = Foo(<double>);

 // Parameterized global aliases
 def int_max_foo = generic_foo::max_foo(<int>);
 def double_max_foo = generic_foo::max_foo(<double>);
```

For more information, see the chapter on generics.

## Function pointer default arguments and named parameters
It is possible to attach default arguments to __function__ _pointer aliases_.
There is no requirement that the function has the same default arguments.
In fact, the __function pointer__ may have default arguments where the function doesn’t have it and _vice-versa_.
Calling the function directly will then use the _function’s default_ arguments, and calling through the function _pointer_ will __yield__ the function _pointer alias’_ default argument.

Similarly, named parameter arguments follow the _alias_ definition when calling _through_ the __function pointer__:
```rust
def TestFn = fn void(int y = 123);

fn void test(int x = 5)
{
    io::printfn("X = %d");
}

fn void main()
{
    TestFn test2 = &test;

    test();        // Prints X = 5
    test2();       // Prints X = 123

    test(.x = 3);  // Prints X = 3
    test2(.y = 4); // Prints X = 4
}
```