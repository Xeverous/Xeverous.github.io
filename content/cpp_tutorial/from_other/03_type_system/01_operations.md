---
layout: article
---

C++, like practically every language, has many built-in arithmetic operations.

Remainer: **integral types** = `bool` + **character types** + **integer types**.

Remainder: **arithmetic types** = **integral types** + **floating-point types**.

## built-in integral math operators

For every **integer type** there exist following operators: `+`, `-`, `*`, `/`, `%`.

### unary plus

Both `+` and `-` have unary and binary variants:

```c++
c = a + b; // binary +
c = a - b; // binary -

c = -c; // unary - (negates integers)
c = +c; // unary + (does nothing for integers)
```

Unary `+` may seem unnecessary but it exists for following reasons:

- consistency with unary `-`
- it can be overloaded
- it can be used to force integral promotions

### modulus

Until C++11, if one or both operands to binary operator `%` were negative, the sign of the remainder was implementation-defined, as it depends on the rounding direction of integer division. The function `std::div` (which returns quotient and remainder) provided well-defined behavior in that case.

Since C++11, there is no difference between the function and built-in operators `/` and `%`.

On many platforms, a single CPU instruction obtains both the quotient and the remainder, and this function may leverage that, although compilers are generally able to merge nearby / and % where suitable.

### integral promotions

Some other important aspects:

- When an operation is performed on values of different types, the smaller one will be promoted.
- All arithmetic operations on integral types produce a type that is at least as large as `int`. In other words, built-in operators accept only `int` and larger types - anything smaller is promoted.

This means that:

- `int` + `int` = `int`
- `int` + `long` = `long`
- `char` + `char` = `int`
- `char` + `bool` = `int`
- `bool` + `bool` = `int`
- `-false` is `0`
- `-true` is `-1`
- `-'A'` is `int`
  - value implementation defined, but practically every implementation uses ASCII encoding
  - sign of resulting integer is dependent whether `char` is signed or unsigned on particular implementation
- TODO there is some corner case of signed + unsigned

## built-in bitwise operations

- bitwise NOT: `~x`
- bitwise OR: `x | y`
- bitwise AND: `x & y`
- bitwise XOR: `x ^ y`

It is recommended to perform bitwise operations only on (fixed-width) unsigned types.

## bitshifts

- left shift of `x` by `n` bits: `x << n`
- right shift of `x` by `n` bits: `x >> n`

As with bitwise operations, it is recommended to perform bitwise operations only on (fixed-width) unsigned types. The rules for bitwise shifts on signed types were/are (depending on C++ standard) undefined or implementation-defined.

TODO important: the value `n` can not:

- be negative
- be equal or larger than the amount of bits in `x`

## built-in floating point operators

For every floating-point type, there exist following operators: `+`, `-`, `*`, `/`.

If at least one of operands is of a floating-point type, the expression is evaluated using floating-point arithmetic.

There is no built-in `%` for floating-points. Checkout `std::remainder` and `std::fmod` from `<cmath>` instead.

## floating-point errors

TODO

## restrictions on operations

For **integral types** only:

- `a / 0` is undefined behaviour.
- `a % 0` is undefined behaviour.
- If the quotient `a / b` is not representable in the result type, the behavior of both `a / b` and `a % b` is undefined.

## common mistakes

- Using `x ^ y` to compute powers. There is no power operator in C++, `x ^ y` performs bitwise XOR. For simple cases, write `x * x` etc. Otherwise use `std::pow()` from `<cmath>`
- `if (some_check() & some_other_check())` will bitwise AND 2 `bool`eans (casted first to `int`) - this has the same eventual effect as `&&` but is considered bad code
- using unsigned types for other purpose than bitshifts, bitwise operations or memory inspection
