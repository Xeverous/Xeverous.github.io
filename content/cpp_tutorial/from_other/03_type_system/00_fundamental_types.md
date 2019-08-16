---
layout: article
---

## preface

C++ has strong typing. When an object of one type is passed to an expression expecting an object of another type, convertion is needed.

## the void type

Objects of type `void` and references to `void` can not be created.

### functions taking `void`

Sometimes you may see mysterious function declarations.

```c++
// C  : function has unspecified amount of arguments (this line only introduces function name)
// C++: function takes 0 arguments
void func();

// C  : function takes 0 arguments
// C++: function takes 0 arguments (just ugly)
void func(void);
```

in C, `()` alone means that "this function has unspecified argument types and amount" and `(void)` has to be used to express that a function does not take any arguments. This was a design mistake in C, even acknowledged by Ritche.

C++ corrected that mistake by treating `()` as 0-argument function. `(void)` is still valid, but discouraged and kept only for backwards compatibility.

## the boolean type

A type that can hold only 2 values: `false` and `true`.

- When converted to any integer type, `false` becomes `0` and `true` becomes `1`.
- When converted from any integer type, `0` becomes `false` and any other value becomes `true`.
- When converted from any pointer type, null pointers become `false` and any other pointer becomes `true`

Prior to C99, the language did not have actual `bool` type, even though the language had operator `!`. Various libraries and user code performed all kinds of tricks to simulate it, mostly by aliasing/macroing `bool` to `int` and `false`/`true` to `0`/`1`. Since C99 there is `_Bool` type, also available as (redefinable) `bool` macro. The keyword is a different, ugly word to avoid name conflicts with existing code which uses manually-implemented `bool`.

C++ has proper `bool` type since the beginning and there are no such problems. Interaction with C that has `bool`/`false`/`true` macros may result in compilation warnings (keywords should never be text-replaced).

## integer types

- `short`
- `int`
- `long`
- `long long`
- `unsigned short`
- `unsigned`
- `unsigned long`
- `unsigned long long`

Integers are signed by default. Signed types can hold both positive and negative values. Unsigned types have twice as large range, but can only hold non-negative values.

The order of keywords doesn't matter, but obviously the form \<signedness\> \<length\> \[int\] is preferred over something like `long int unsigned long`.

You can omit unnecessary keywords when the integer type name is unambiguous:

- `signed` is same as `int`
- `short` is same as `signed short int`
- `unsigned long` is same as `unsigned long int`

TODO implicit int - border

In very old code, even sole `int` could be omitted, allowing function declarations like `f(x)`. This was a big grammar mistake, corrected in C89. C++ from the beginning requries `int f(int x)`.

Lengths and ranges of integer types are implementation defined. The choices made by each implementation about the sizes of the fundamental types are collectively known as data model. Four data models found wide acceptance:

32 bit systems:

- LP32 or 2/4/4 (int is 16-bit, long and pointer are 32-bit)
  - Win16 API
- ILP32 or 4/4/4 (int, long, and pointer are 32-bit);
  - Win32 API
  - Unix and Unix-like systems (Linux, Mac OS X)

64 bit systems:

- LLP64 or 4/4/8 (**int and long are 32-bit**, pointer is 64-bit)
  - Win64 API
- LP64 or 4/8/8 (int is 32-bit, long and pointer are 64-bit)
  - Unix and Unix-like systems (Linux, Mac OS X)

Obviously 32-bit long sucks. When a fixed-width integer is needed, you should use names from the standard library, found in `<cstdint>`:

- `std::int8_t`
- `std::int16_t`
- `std::int32_t`
- `std::int64_t`
- `std::uint8_t`
- `std::uint16_t`
- `std::uint32_t`
- `std::uint64_t`
- (for more, rarely used names see https://en.cppreference.com/w/cpp/types/integer)

These names are not separate types. They are only type aliases to corresponding types (appropriate for any given platform) - this means that you should not write code assuming they are different. For example, don't overload a function for `std::int32_t` and `long` - this might work on one platform but create compilation error (identical arguments for 2 overloads) on another. Either use keyword-only types or fixed-width type aliases - don't mix them.

Most of fixed-width type aliases are optional for the implementation - they are only provided when the target platform actually support them. So don't expect `std::int64_t` to exist on 32-bit hardware.

## floating-point types

- `float` - single precision floating point type. Usually IEEE-754 32 bit floating point type.
- `double` - double precision floating point type. Usually IEEE-754 64 bit floating point type.
- `long double` - extended precision floating point type. Does not necessarily map to types mandated by IEEE-754. Usually 80-bit x87 floating point type on x86 and x86-64 architectures.

Floating-point types **may** support special values:

- Infinity, available as `INFINITY` in `<cmath>`. If floating-point infinities are not supported by the platform, `INFINITY` is guuaranteed to overflow at compile time, trigerring a warning on any reasonable compiler.
- Negative zero, which is different from positive zero. This may impact result of some math functions (most notably trigger errors on functions which do not accept negative values). Also `1.0/0.0 == INFINITY` but `1.0/-0.0 == -INFINITY`.
- Not-a-number (NaN), which does not compare equal with anything (including itself). Available as `NAN` in `<cmath>`, only if supported by the implementation.

On x86 and x86-64 all of special values are supported.

C++ takes no special notice of signalling NaNs other than detecting their support. This means that you will never get exceptions etc for invalid math operations - any operation involving a NaN will simply produce another NaN.

TODO floating-point operations and errors. Perhaps in a separate article.

## character types

Ordinary character types:

- `char`
- `signed char`
- `unsigned char`

Unlike integers, `char` is not the same as `signed char`. `char` has the same representation and alignment as one of the other 2, but at the language level is a distinct type. The signedness of `char` depends on the compiler and the target platform: the defaults for ARM and PowerPC are typically unsigned, the defaults for x86 and x64 are typically signed.

Since C++14, character types are guuaranteed to be large enough to represent any UTF-8 eight-bit code unit.

Additional character types:

- `char16_t` - character type capable of holding any UTF-16 code unit
- `char32_t` - character type capable of holding any UTF-32 code unit
- `wchar_t` - wide character type capable of holding any supported character code point; a type abomination which is 32-bit everywhere except Windows, on which UTF-16 is used

C++20 introduces another character type: `char8_t`. TODO describe?

## null pointer type

The value `nullptr` (basically C++ `null` keyword) is of its own, separate type named `std::nullptr_t` that is implicitly convertible to any other pointer type. This all is for type safety reasons.

You can think of null-pointer-type similar to `bool` type - but unlike `bool` which has 2 possible values, `std::nullptr_t` can hold only 1 value - `nullptr`.

Because the value of `nullptr` has a separate type, it is possible to overload a function for a pointer and null-pointer-literal type. The second overload will only be matched when `nullptr` (they keyword) is passed directly to the function.

## summary

During the course, you might see some terms like **arithmetic type**, **integer type** or **integral type**. Many of these terms are supersets of others. Here is a graph of the entire C++ type system:

- **fundamental types**:
  - the type `void`
  - the type `std::nullptr_t`
  - **arithmetic types**:
    - **floating-point types** (`float`, `double`, `long double`)
    - **integral types**
      - the type `bool`
      - **character types**:
        - **narrow character types**:
          - **ordinary character types** (`char`, `signed char`, `unsigned char`)
          - the type `char8_t` (since C++20)
        - **wide character types** (`char16_t`, `char32_t`, `wchar_t`)
      - **integer types**:
        - **signed integer types** (`short`, `int`, `long`, `long long`)
        - **unsigned integer types** (`unsigned short/int/long/long long`)
- **compound types** (types encapsulating other types):
  - **reference types**:
    - **lvalue reference types** (`T&`):
      - **lvalue reference to object types**
      - **lvalue reference to function types**
    - **rvalue reference types** (`T&&`):
      - **rvalue reference to object types**
      - **rvalue reference to function types**
  - **pointer types** (`T*`):
    - **pointer-to-object types**
    - **pointer-to-function types**
  - **pointer-to-member types** (`C::T*`):
    - **pointer-to-data-member types**
    - **pointer-to-member-function types**
  - **array types** (`T[]`)
  - **function types** (`R F(Ts...)`)
  - **user-defined types**:
    - **enumeration types**
    - **class types**:
      - **non-union types**
      - **union types**

## cv-qualifications

For every type `T` other than reference and function, C++ type system supports three additional cv-qualified versions of that type: `const T`, `volatile T`, and `const volatile T`.

Note that C++ type system treats these as distinct types, meaning that (apart from few exceptions in overloading):

- assignment from `int` to `const int` technically requires convertion
- you can overload a function for `int*`, `volatile int*` and `const int*`
- class template instantiated with `int` is considered a completely different class than the one with `const int` - this means than an object of type `std::vector<int>` will not match function expecting `std::vector<const int>`
