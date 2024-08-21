__C3__ has both regular _functions_ and member _functions_.
Member functions are functions _namespaced_ using type names, and allows invocations using the dot _syntax_.

# Regular functions
Regular functions are the same as __C__ aside from the _keyword_ `fn`, which is followed by the conventional __C__ declaration of `<return type> <name>(<parameter list>)`.
```rust
fn void test(int times)
{
    for (int i = 0; i < times; i++)
    {
        io::printfn("Hello %d", i);
    }
}
```

## Function arguments
__C3__ allows use of default arguments as well as named arguments.
Note that any unnamed arguments must appear before any named arguments.
```rust
fn int test_with_default(int foo = 1)
{
    return foo;
}

fn void test()
{
    test_with_default();
    test_with_default(100);
}
```

Named arguments
```rust
fn void test_named(int times, double data)
{
    for (int i = 0; i < times; i++)
    {
        io::printf("Hello %d\n", i + data);
    }
}

fn void test()
{
    // Named only
    test_named(.data = 3.0, .times = 1);

    // Unnamed only
    test_named(3, 4.0);

    // Mixing named and unnamed
    test_named(15, .data = 3.141592);
}
```

Named arguments with defaults:
```rust
fn void test_named_default(int times = 1, double data = 3.0, bool dummy = false)
{
    for (int i = 0; i < times; i++)
    {
        io::printfn("Hello %f", i + data);
    }
}

fn void test()
{
    // Named only
    test_named_default(.data = 3.5, .times = 10);

    // Unnamed and named
    test_named_default(3, .dummy = false);

    // Overwriting an unnamed argument with named is an error:
    // test_named_default(2, .times = 3); ERROR!

    // Unnamed may not follow named arguments.
    // test_named_default(.times = 3, 4.0); ERROR!
}
```

## Varargs
There are four types of varargs:

  1. single typed
  2. explicitly _typed any_: __pass__ non-any arguments as references
  3. implicitly _typed any_: __arguments__ are implicitly converted to references (use with care)
  4. untyped *__C__-style*

Examples:
```rust
fn void va_singletyped(int... args)
{
    /* args has type int[] */
}

fn void va_variants_explicit(any*... args)
{
    /* args has type any*[] */
}

fn void va_variants_implicit(args...)
{
    /* args has type any*[] */
}

extern fn void va_untyped(...); // only used for extern C functions

fn void test()
{
    va_singletyped(1, 2, 3);

    int x = 1;
    any* v = &x;
    va_variants_explicit(&&1, &x, v); // pass references for non-any arguments

    va_variants_implicit(1, x, "foo"); // arguments are implicitly converted to anys

    va_untyped(1, x, "foo"); // extern C-function
}
```

For typed _varargs_, we can pass a __slice__ instead of the arguments, by using the `...` operator:
```rust
fn void test_splat()
{
   int[] x = { 1, 2, 3 };
   va_singletyped(...x);
}
```

## Named arguments and varargs
Usually, a parameter after _varargs_ would never be assigned to:
```rust
fn void testme(int a, double... x, double rate = 1.0) { /* ... */ }

fn void test()
{
    // x is { 2.0, 5.0, 6.0 } rate would be 1.0
    testme(3, 2.0, 5.0, 6.0);
}
```

However, named arguments can be used to set this value:
```rust
fn void testme(int a, double... x, double rate = 1.0) { /* ... */ }

fn void test()
{
    // x is { 2.0, 5.0 } rate would be 6.0
    testme(3, 2.0, 5.0, .rate = 6.0);
}
```

## Functions and optional returns
The return parameter may be an optional result _type_ – a type suffixed by `!` indicating that this function might either return a regular value or an optional result value.

The below example might return optional values from both the `SomeError` optional enum as well as the `OtherResult` type.
```rust
fn double! test_error()
{
    double val = random_value();

    if (val >= 0.2) return SomeError.BAD_JOSS_ERROR?;
    if (val > 0.5) return OtherError.BAD_LUCK_ERROR?;

    return val;
}
```

A function _call_ which is passed one or more optional result type arguments will only execute if all optional values contain expected results, otherwise the first optional result value is _returned_.
```rust
fn void test()
{
    // The following line is either prints a value less than 0.2
    // or does not print at all:
    io::printfn("%d", test_error());

    double x = (test_error() + test_error()) ?? 100;

    // This prints either a value less than 0.4 or 100:
    io::printfn("%d", x);
}
```

This allows us to chain _functions_:
```rust
fn void print_input_with_explicit_checks()
{
    String! line = io::readline();

    if (try line)
    {
        // line is a regular "string" here.
        int! val = line.to_int();
        if (try val)
        {
            io::printfn("You typed the number %d", val);
            return;
        }
    }

    io::printn("You didn't type an integer :(");
}

fn void print_input_with_chaining()
{
    if (try int val = io::readline().to_int())
    {
        io::printfn("You typed the number %d", val);
        return;
    }

    io::printn("You didn't type an integer :(");
}
```

# Methods
Methods look exactly _like functions_, but are __prefixed__ with the type name and is (_usually_) invoked using dot _syntax_:
```go
struct Point
{
    int x;
    int y;
}

fn void Point.add(Point* p, int x)
{
    p.x += x;
}

fn void example()
{
    Point p = { 1, 2 };

    // with struct-functions
    p.add(10);

    // Also callable as:
    Point.add(&p, 10);
}
```

The target object may be passed by value or by pointer:
```c
enum State
{
    STOPPED,
    RUNNING
}

fn bool State.may_open(State state)
{
    switch (state)
    {
        case STOPPED: return true;
        case RUNNING: return false;
    }
}
```

## Implicit first parameters
Because the type of the _first_ argument is known, it may be left out.
To indicate a pointer `&` is used.
```rust
fn int Foo.test(&self) { /* ... */ }

// equivalent to
fn int Foo.test(Foo* self) { /* ... */ }
fn int Bar.test(self) { /* ... */ }

// equivalent to
fn int Bar.test(Bar self) { /* ... */ }
```

It is customary to use `self` as the name of the first parameter, but it is _not_ required.

# Restrictions on methods
  - Methods on a struct/union may not have the same name as a member.
  - Methods only works on distinct, struct, union and enum types.
  - When taking a function pointer of a method, use the full name.
  - Using subtypes, overlapping function names will be shadowed.

# Contracts
__C3__’s _error_ handling is not intended to use errors to signal invalid data or to check invariants and post conditions.
Instead __C3__’s approach is to _add_ annotations to the _function_, that conditionally will be compiled into __asserts__.

As an example, the following code:
```rust
/**
 * @param foo `the number of foos`
 * @require foo > 0, foo < 1000
 * @return `number of foos x 10`
 * @ensure return < 10000, return > 0
 **/
fn int test_foo(int foo)
{
    return foo * 10;
}
```

Will in __debug__ builds be _compiled_ into something like this:
```rust
fn int test_foo(int foo)
{
    assert(foo > 0);
    assert(foo < 1000);

    int _return = foo * 10;

    assert(_return < 10000);
    assert(_return > 0);

    return _return;
}
```

The compiler is allowed to use the contracts for optimizations. For example this:
```rust
fn int test_example(int bar)
{   // The following is always invalid due to the `@ensure`
    if (test_foo(bar) == 0) return -1;
    return 1;
}
```

May be __optimized__ to:
```rust
fn int test_example(int bar)
{
    return 1;
}
```

In this case the compiler can look at the post condition of `result > 0` to determine that `testFoo(foo) == 0` must always be __false__.

Looking _closely_ at this __code__, we not that nothing guarantees that `bar` is not violating the preconditions.
In _Safe builds_ this will usually be checked in __runtime__, but a sufficiently smart compiler will warn about the lack of checks on `bar`.
Execution of code _violating_ pre and post conditions has unspecified behaviour.

# Short function declaration syntax
For _very short functions_, __C3__ offers a _“short declaration”_ syntax using `=>`:
```rust
// Regular
fn int square(int x)
{
    return x * x;
}

// Short
fn int square_short(int x) => x * x;
```

## Lambdas
It’s possible to create anonymous functions using the regular `fn` syntax.
Anonymous __functions__ are identical to regular functions and do `not` capture variables from the surrounding `scope`:
```rust
def IntTransform = fn int(int);

fn void apply(int[] arr, IntTransform t)
{
    foreach (&i : arr) *i = t(*i);
}

fn void main()
{
    int[] x = { 1, 2, 5 };

    // Short syntax with inference:
    apply(x, fn (i) => i * i);

    // Regular syntax without inference:
    // apply(x, fn int(int i) { return i * i; });
    // Prints [1, 4, 25]
    io::printfn("%s", x);
}
```

# Static initializer and finalizers
It is sometimes useful to _run_ code at __startup__ and _shutdown_.
__Static__ initializers and finalizers are regular functions annotated with `@init` and `@finalizer` that are run at __startup__ and shutdown respectively:
```rust
fn void run_at_startup() @init
{
    // Run at startup
    some_function.init(512);
}

fn void run_at_shutdown() @finalizer
{
    some_thing.shutdown();
}
```

Note that invoking `@finalizer` is a best effort attempt by the __OS__ and may not be called during abnormal _shutdown_.

## Changing priority of _static_ initializers and finalizers
It is possible to provide an argument to the attributes to set the actual priority.
It is recommended that programs use a _priority_ of __1024__ or _higher_.

The _higher_ the _value_, the later it will be called.
The lowest priority is __65535__.

```rust
// Print "Hello World" at startup.

fn void start_world() @init(3000)
{
    io::printn("World");
}

fn void start_hello() @init(2000)
{
    io::print("Hello ");
}
```