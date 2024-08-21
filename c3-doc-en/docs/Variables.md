# Zero init by default
Unlike __C__, __C3__ local variables are zero-initialized by default. To avoid _zero-init_, you need to explicitly _opt-out_.
```cpp
int x;               // x = 0
int y @noinit;       // y is explicitly undefined and must be assigned before use.

AStruct foo;         // foo is implicitly zeroed
AStruct bar = {};    // bar is explicitly zeroed

AStruct baz @noinit; // baz is explicitly undefined
```

Using a variable that is explicitly undefined before will __trap__ or be initialized to a specific value when compiling _“safe”_ and is undefined behaviour in _“fast”_ builds.