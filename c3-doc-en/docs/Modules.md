__C3__ groups functions, types, variables and macros into namespaces called modules.
When doing builds, any __C3__ file must start with the `module` _keyword_, specifying the module.
When compiling single files, the module is not needed and the module name is assumed to be the file name, converted to lower case, with any invalid characters replaced by underscore (`_`).

A module can consist of multiple files, e.g.
`file_a.c3`
```c
module foo;

/* ... */
```

`file_b.c3`
```c
module foo;

/* ... */
```

`file_c.c3`
```c
module bar;

/* ... */
```

Here `file_a.c3` and `file_b.c3` belong to the same module, __foo__ while `file_c.c3` belongs to to __bar__.

# Details
Some details about the __C3__ module system:

  - Modules can be arbitrarily nested, e.g. `module foo::bar::baz;` to create the sub module baz in the sub module `bar` of the module `foo`.
  - Module names must be alphanumeric lower case letters plus the underscore character: `_`.
  - Module names are limited to _31 characters_.
  - Modules may be spread across multiple files.
  - A single file may have multiple module declarations.
  - Each declaration of a distinct module is called a _module section_.

# Importing modules
Modules are imported using the `import` statement.
Imports always recursively _import sub-modules_.
Any module will automatically import all other modules with the same parent module.

`foo.c3`
```rust
module some::foo;
fn void test() {}
```

`bar.c3`
```rust
module bar;
import some;

// import some::foo; <- not needed, as it is a sub module to "some"
fn void test()
{
    foo::test();
    // some::foo::test() also works.
}
```

In some cases there may be ambiguities, in which case the full path can be used to resolve the ambiguity:
`abc.c3`
```cpp
module abc;

struct Context
{
    int a;
}
```

`def.c3`
```cpp
module def;

struct Context
{
    void* ptr;
}
```

`test.c3`
```cpp
module test1;

import def, abc;
// Context c = {} <- ambiguous
abc::Context c = {};
```

# Implicit imports
The module system will also implicitly import:

  1. The `std::core` module (_and sub modules_).
  2. Any other module sharing the same top module. E.g. the module `foo::abc` will implicitly also import modules `foo` and `foo::cde` if they exist.

# Visibility
All files in the same module share the same global declaration namespace.
By default a symbol is visible to all other modules.
To make a symbol only visible inside the module, use the `@private` attribute.
```rust
module foo;

fn void init() { .. }

fn void open() @private { .. }
```

In this example, the other modules can use the __init()__ function after importing _foo_, but only files in the foo module can use __open()__, as it is specified as `private`.

It’s possible to further restrict visibility: `@local` works like `@private` except it’s only visible in the _local_ context.
```rust
// File foo.c3
module foo;

fn void abc() @private { ... }
fn void def() @local { ... }

// File foo2.c3
module foo;

fn void test()
{
    abc(); // Access of private in the same module is ok
    // def(); <- Error: function is local to foo.c3
}
```

# Overriding symbol visibility rules
By using `import <module> @public`, it’s possible to access another module´s private symbols.
Many other module systems have hierarchal visibility rules, but the `import @public` feature allows visibility to be manipulated in a more __ad-hoc__ manner without imposing hard rules.

For example, you may provide a library with two __modules__: _“mylib::net”_ and _“mylib::file”_ - which both use functions and types from a common __“mylib::internals”__ module.

The two libraries use `import mylib::internals @public` to access this module’s private functions and type.
To an external user of the library, the _“mylib::internals”_ does not seem to exist, but inside of your library you use it as a shared dependency.

A simple example:
```cpp
// File a.c3
module a;

fn void a_function() @private { ... }

// File b.c3
module b;

fn void b_function() @private { ... }

// File c.c3
module c;
import a;
import b @public;

fn void test()
{
  a::a_function(); // <-- error, a_function is private
  b::b_function(); // Allowed since import converted it to "public" in this context.
}
```

*Note: `@local` visibility cannot be overridden using a __“@public”__ import.*

# Changing the default visibility
In a normal module, _global_ declarations will be public by default.
If some other visibility is desired, it’s possible to declare `@private` or `@local` after the module name.
It will affect all declaration in the same section.
```cpp
module foo @private;

fn void ab_private() { ... } // Private

module foo;

fn void ab_public() { ... } // Public

module bar;
import foo;

fn void test()
{
    foo::ab_public(); // Works
    // foo::ab_private(); <- Error, private method
}
```

If the default visibility is `@private` or `@local`, using `@public` sets the visibility to public:
```cpp
module foo @private;

fn void ab_private() { ... }        // Private
fn void ab_public() @public { ... } // Public
```

# Linker visibility and exports
A function or global prefixed `extern` will be assumed to be linked in later.
An _“extern”_ function may not have a body, and global variables are prohibited from having an init expression.

The attribute `@export` explicitly marks a function as being exported when creating a (_static or dynamic_) library.
It can also change the linker name of the __function__.

# Using functions and types from other modules
As a __rule__, __functions__, __macros__, __constants__, variables and types in the same module do not need any namespace prefix.
For imported modules the following rules hold:

  1. Functions, macros, constants and variables require at least the (sub-) module name.
  2. Types do not require the module name unless the name is ambiguous.
  3. In case of ambiguity, only so many levels of module names are needed as to make the symbol unambiguous.

```cpp
// File a.c3

module a;

struct Foo { ... }
struct Bar { ... }
struct TheAStruct { ... }

fn void anAFunction() { ... }

// File b.c3
module b;

struct Foo { ... }
struct Bar { ... }
struct TheBStruct { ... }

fn void aBFunction() { ... }

// File c.c3
module c;
import a, b;

struct TheCStruct { ... }
struct Bar { ... }

fn void aCFunction() { ... }

fn void test()
{
    TheAStruct stA;
    TheBStruct stB;
    TheCStruct stC;

    // Name required to avoid ambiguity;
    b::Foo stBFoo;

    // Will always pick the current module's
    // name.
    Bar bar;

    // Namespace required:
    a::aAFunction();
    b::aBFunction();

    // A local symbol does not require it:
    aCFunction();
}
```

This means that the rule for the common case can be summarized as
  Types are used without _prefix_; __functions__, __variables__, __macros__ and _constants_ are prefixed with the sub module name.

# Module sections
A single file may have multiple module declarations, even for the same module. This allows us to write for example:
```cpp
// File foo.c3
module foo;
fn int hello_world()
{
    return my_hello_world();
}

module foo @private;
import std::io;         // The import is only visible in this section.

fn int my_hello_world() // @private by default
{
    io::printn("Hello, world\n");
    return 0;
}

module foo @test;
fn void test_hello() // @test by default
{
    assert(hello_world() == 0);
}
```

# Versioning and dynamic inclusion
NOTE: _This feature may significantly change_.

When including dynamic libraries, it is possible to use optional functions and globals. This is done using the `@dynamic` attribute.

An example library could have this:

`dynlib.c3i`
```cpp
module dynlib;

fn void do_something() @dynamic(4.0)
fn void do_something_else() @dynamic(0, 5.0)
fn void do_another_thing() @dynamic(0, 2.5)
```

Importing the dynamic library and setting the base version to __4.5__ and minimum version to __3.0__, we get the following:

`test.c3`
```cpp
import dynlib;

fn void test()
{
    if (@available(dynlib::do_something))
    {
        dynlib::do_something();
    }
    else
    {
        dynlib::do_someting_else();
    }
}
```

In this example the code would run `do_something` if available (_that is, when the dynamic library is 4.0 or higher_), or fallback to `do_something_else` otherwise.

If we tried to conditionally add something not available in the compilation itself, that is a compile time error:
```cpp
if (@available(dynlib::do_another_thing))
{
    dynlib::do_another_thing(); // Error: This function is not available with 3.0
}
```

Versionless dynamic loading is also possible:
`maybe_dynlib.c3i`
```cpp
module maybe_dynlib;
fn void testme() @dynamic;
```

`test2.c3`
```cpp
import maybe_dynlib;

fn void testme2()
{
    if (@available(maybe_dynlib::testme))
    {
        dynlib::testme();
    }
}
```
This allows things like optionally loading dynamic libraries on the platforms where this is available.

# Textual includes

## $include
It’s sometimes useful to include an entire file, doing so employs the `$include` function. Includes are only valid at the top level.

File `Foo.c3`
```cpp
module foo;

$include("Foo.x");

fn void test()
{
    io::printf("%d", testX(2));
}
```

File `Foo.x`
```cpp
fn testX(int i)
{
    return i + 1;
}
```

The result is as if `Foo.c3` contained the following:
```cpp
module foo;

fn testX(int i)
{
    return i + 1;
}

fn void test()
{
    io::printf("%d", testX(2));
}
```

The include may use an absolute or relative path, the relative path is always relative to the source file in which the include appears.

Note that to use it, the __trust level__ of the compiler must be set to at least 2 with the —trust option (i.e. use `--trust=include` or `--trust=full` from the command line).

## $exec
An alternative to `$include` is `$exec` which is similar to include, but instead includes the output of an external program as the included text.

An example:
```cpp
import std::io;

// On Linux or MacOS this will insert 'String a = "Hello world!";'
$exec("echo", "String a = \\\"Hello world!\\\"\\;");

fn void main()
{
  io::printn(a);
}
```

Using `$exec` requires full __trust level__, which is enabled with `-trust=full` from the command line.

_‘$exec’_ will by default run from the `/scripts` directory for projects, for _non-project_ builds, the current directory is used as well.

### $exec scripting
`$exec` allows a special scripting mode, where one or more __C3__ files are compiled on the fly and run by `$exec`.
```cpp
import std::io;

// Compile foo.c3 and bar.c3 in the /scripts directory, invoke the resulting binary
// with the argument 'test'
$exec("foo.c3;bar.c3", "test");

fn void main()
{
  ...
}
```