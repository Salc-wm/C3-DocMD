Attributes are compile-time annotations on functions, types, global constants and variables.
Similar to Java annotations, a decoration may also take arguments.
A attribute can also represent a bundle of attributes.

# Built in attributes

## @align(alignment)
Used for: _struct, bitstructs, union, var, function_

This attribute sets the minimum alignment for a field or a variable.

## @benchmark
Used for: _function_

Marks the function as a benchmark function.
Will be added to the list of benchmark functions when the benchmarks are run, otherwise the function will not be included in the compilation.

## @bigendian
Used for: _bitstruct_

Lays out the bits as if the data was stored in a big endian type, regardless of host system endianness.

## @builtin
Used for: _function, macro, global, const_

Allows a macro, function, global or constant be used from another module without the module path prefixed.
Should be used sparingly.

## @callc
Used for: _function_

Sets the call convention, which may be ignored if the convention is not supported on the target.
Valid arguments are veccall, ccall, stdcall.

## @deprecated
Used for: _types, function, macro, global, const, member_

Marks the particular type, global, const or member as deprecated, making use trigger a warning.

## @export
Used for: _function, global, const, enum, union, struct, fault_

Marks this declaration as an export, this ensures it is never removed and exposes it as public when linking.
The attribute takes an optional string value, which is the external name.
This acts as if `@extern` had been added with that name.

## @extern
Used for: _function, global, const, enum, union, struct, fault_

Sets the external (_linkage_) name of this declaration.

## @finalizer
Used for: _function_

Make this function run at shutdown. See `@init` for the optional priority.
Note that running a finalizer is a _“best effort”_ attempt by the __OS__.
During abnormal termination it is not guaranteed to run.

The function must be a void function taking no arguments.

## @if
Used for: _all declarations_

Conditionally includes the declaration in the compilation.
It takes a constant compile time value argument, if this value is `true` then the declaration is retained, on false it is removed.

## @init
Used for: _function_

Make this function run at startup before main.
It has an optional priority _1 - 65535_, with lower being executed earlier.
It is not recommended to use values less than _128_ as they are generally reserved and using them may _interfere_ with standard program initialization.

The function must be a void function taking no arguments.

## @inline
Used for: _function, call_

Declares a function to always be `inlined` or if placed on a call, that the call should be `inlined`.

##  @littleendian
Used for: _bitstruct_

Lays out the bits as if the data was stored in a little `endian type`, regardless of host system endianness.

## @local
Used for: _any declaration_

Sets the visibility to _“local”_, which means it’s only visible in the current module section.

## @maydiscard
Used for: _function, macro_

Allows the `return` value of the function or _macro_ to be discarded even if it is an optional.
Should be used sparingly.

## @naked
Used for: _function_

This attribute disables _prologue / epilogue_ emission for the function.

## @nodiscard
Used for: _function, macro_

The return value may not be discarded.

## @noinit
Used for: _global, local variable_

Prevents the compiler from zero initializing the variable.

## @noreturn
Used for: _function_

Declares that the function will never return.

## @nostrip
Used for: _any declaration_

This causes the declaration never to be stripped from the executable, even if it’s not used.
This also transitively applies to any dependencies the declaration might have.

## @obfuscate
Used for: _any declaration_

Removes any string values that would identify the declaration in some way.
Mostly this is used on faults and enums to remove the stored names.

## @operator
Used for: _method, macro method_

This attribute has arguments `[]` `[]=` `&[]` and len allowing operator overloading for `[]` and _foreach_.
By implementing `[]` and `len`, _foreach_ and `foreach_r` is enabled.
In order to do _foreach_ by reference, `&[]` must be implemented as well.

## @overlap
Used for: _bitstruct_

Allows bitstruct fields to have overlapping bit ranges.

## @packed
Used for: _struct, union_

Causes all members to be packed as if they had alignment 1.
The alignment of the _struct/union_ is set to 1.
This alignment can be overridden with `@align`.

## @private
Used for: _any declaration_

Sets the visibility to _“private”_, which means it is visible in the same module, but not from other modules.

## @pure
Used for: _call_

Used to annotate a non pure function as _“pure”_ when checking for conformance to `@pure` on functions.

## @packed
Used for: _struct, union, enum_

If used on a `struct` or `enum`: packs the type, including any components to minimum size.
On an _enum_, it uses the smallest representation containing all its values.

## @reflect
Used for: _any declaration_

Adds additional reflection information. Has no effect currently.

## @section(name)
Used for: _function, const, global_

Declares that a global variable or function should appear in a specific section.

## @test
Used for: _function_

Marks the function as a test _function_.
Will be added to the list of test functions when the tests are run, otherwise the function will not be included in the compilation.

## @unused
Used for: _any declaration_

Marks the declaration as possibly unused (_but should not emit a warning_).

## @used
Used for: _any declaration_

Marks a _parameter_, _value_ etc. as must being used.

## @weak
Used for: _function, const, global_

Emits a weak symbol rather than a global.

# User defined attributes
User defined attributes are intended for conditional application of built-in attributes.

```c
def @MyAttribute = { @noreturn @inline };

// The following two are equivalent:
fn void foo() @MyAttribute { ... }
fn void foo() @noreturn @inline { ... }
```

A user defined attribute may also be completely empty:
```python
def @MyAttributeEmpty = {};
```