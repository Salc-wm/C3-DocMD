__Vectors__ - where possible - based on underlying hardware vector implementations.
A _vector_ is similar to an array, but with additional functionality.

The _restriction_ is that a vector may only consist of elements that are _numerical types_, __boolean__ or __pointers__.

A __vector__ is declared similar to an array but uses `[<>]` rather than `[]`, e.g. `int[<4>]`.

(If you are searching for the counterpart of __C++__‘s `std::vector`, look instead at the standard library __List type__.)

# Arithmetics on vectors
__Vectors__ support all arithmetics and other operations supported by its underlying type. The operations are always performed elementwise.
```cpp
int[<2>] a = { 23, 11 };
int[<2>] b = { 2, 1 };
int[<2>] c = a * b;     // c = { 46, 11 }
```

For integer and boolean types, bit operations such as `^` `|` `&` `<< >>` are available, and for _pointers_, _pointer_ arithmetic is supported.

## Scalar values
Scalar values will implicitly widen to vectors when used with __vectors__:
```cpp
int[<2>] d = { 21, 14 };
int[<2>] e = d / 7;      // e = { 3, 2 }
int[<2>] f = 4;          // f = { 4, 4 }
```

# Additional operations
The `std::math` module contains a wealth of additional operations available on vectors using __dot-method__ syntax.

  - `.sum()` - sum all vector elements.
  - `.product()` - multiply all vector elements.
  - `.max()` - get the maximum element.
  - `.min()` - get the minimum element.

  - `.dot(other)` - return the dot product with the other __vector__.
  - `.length(other)` - return the square root of the dot product (not available on _integer vectors_).
  - `.distance(other)` - return the length of the difference of the two vectors (not available on _integer vectors_).
  - `.normalize()` - return a normalized vector (not available on _integer vectors_).

  - `.comp_lt(other)` - return a boolean vector with a component wise __”<”__
  - `.comp_le(other)` - return a boolean vector with a component wise __”<=”__
  - `.comp_eq(other)` - return a boolean vector with a component wise __”==”__
  - `.comp_gt(other)` - return a boolean vector with a component wise __”>”__
  - `.comp_ge(other)` - return a boolean vector with a component wise __”>=”__
  - `.comp_ne(other)` - return a boolean vector with a component wise __”!=”__

Dot methods available for scalar values, such as `ceil`, `fma` etc are in general also available for _vectors_.

# Swizzling
__Swizzling__ using dot notation is supported, using __*x, y, z, w or r, g, b, a*__:
```cpp
int[<3>] a = { 11, 22, 33 };
int[<4>] b = a.xxzx;                         // b = { 11, 11, 33, 11 }

int c = b.w;                                 // c = 11;
char[<4>] color = { 0x11, 0x22, 0x33, 0xFF };
char red = color.r;                          // red = 0x11
```

# Array-like operations
Like _arrays_, it’s possible to make __slices__ and _iterate_ over _vectors_.

It should be noted that the storage alignment of _vectors_ are often different from _arrays_, which should be taken into account when storing __vectors__.