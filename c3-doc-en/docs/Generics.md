Generic modules are parameterized modules that allow functionality for arbitrary types.

For generic modules, the generic parameters follows the module name:

```cpp
// TypeA, TypeB, TypeC are generic parameters.
module vector(<TypeA, TypeB, TypeC>);
```

Code inside a generic module may use the generic parameters as if they were well-defined symbols:
```cpp
module foo_test(<Type1, Type2>);

struct Foo
{
   Type1 a;
}

fn Type2 test(Type2 b, Foo *foo)
{
   return foo.a + b;
}
```

Including a generic module works as usual:
```rust
import foo_test;

def FooFloat = Foo(<float, double>);
def test_float = foo_test::test(<float, double>);

...

FooFloat f;
Foo(<int, double>) g;

...

test_float(1.0, &f);
foo_test::test(<int, double>)(1.0, &g);
```

Just like for __macros__, optional constraints may be added to improve _compile errors_:
```cpp
/**
 * @require $assignable(1, TypeB) && $assignable(1, TypeC)
 * @require $assignable((TypeB)1, TypeA) && $assignable((TypeC)1, TypeA)
 */
module vector(<TypeA, TypeB, TypeC>);

/* .. code * ../
```
```python
def testFunction = vector::testFunc(<Bar, float, int>);

// This would give the error
// --> Parameter(s) failed validation:
//     @require "$assignable((TypeB)1, TypeA) && $assignable((TypeC)1, TypeA)" violated.
```