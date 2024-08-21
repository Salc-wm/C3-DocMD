Precedence _rules_ in __C3__ differs from __C/C++__.
Here are all precedence levels in __C3__, listed from highest (_1_) to lowest (_11_):

  1. `()`, `[]`, `.`, `!!` postfix `!`, `++` and `--`
  2. `@`, _prefix_ `-`, `~`, _prefix_ `*`, `&`, prefix `++` and `--`
  3. _infix_ `*`, `/`, `%`, `*%`
  4. `<<`, `>>`
  5. `^`, `|`, _infix_ `&`
  6. `+`, _infix_ `-`
  7. `==`, `!=`, `>=`, `<=`, `>`, `<`
  8. `&&`
  9. `||`
  10. __ternary__ `?:` `??`
  11. `=`, `*=`, `/=`, `%=`, `+=`, `-=`, `<<=`, `>>=`, `&=`, `^=`, `|=`


The main difference is that _bitwise_ operations and _shift_ has higher precedence than __addition/subtraction__ and __multiplication/division__ in __C3__.
_Bitwise_ operations also have higher precedence than the relational operators.
Also, there is no difference in precedence between `&&` `||` or between the _bitwise operators_.

Examples
```
a + b >> c + d

(a + b) >> (c + d) // C (+ - are evaluated before >>)
a + (b >> c) + d   // C3 (>> is evaluated before + -)


a & b == c

a & (b == c)       // C  (bitwise operators are evaluated after relational)
(a & b) == c       // C3 (bitwise operators are evaluated before relational)


a > b == c < d

(a > b) == (c < d) // C  (< > binds tighter than ==)
((a > b) == c) < d // C3 Error, requires parenthesis!


a | b ^ c & d

a | ((b ^ c) & d)  // C  (All bitwise operators have different precedence)
((a | b) ^ c) & d  // C3 Error, requires parenthesis!
```

The change in precedence of the bitwise operators corrects a long standing issue in the __C__ specification.
The change in precedence for shift operations goes towards making the precedence less surprising.

Conflating the precedence of relational and equality operations, and all bitwise operations was motivated by simplification: few remember the exact internal differences in precedence between bitwise operators. Parenthesis are required for those conflated levels of precedence.

__Left-to-right__ offers a very simple model to think about the internal order of operations, and encourages use of explicit ordering, as best practice in __C__ is to use parentheses anyway.