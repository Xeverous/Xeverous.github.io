---
layout: article
---

Below is a (hopefully complete) list of differences between C and C++.

I do not list features that are only in one language, only stuff that exists in both but has different meaning.

## size of characters

In C, character expressions are of integer type.

```c++
// C
sizeof('a') == sizeof(int)
// C++
sizeof('a') == sizeof(char)
```

## string literals

C allows to assign string literals to non-const pointers

```c++
char* str = "abc"; // valid C, invalid C++
```

Attempting to modify such string is still undefined behaviour.

Note that both languages allow to assign a string literal to a non-const array:

```c++
char arr[] = "abc";
```

There is no UB when modifying such array.

TODO link `const char*` vs `const char[]` article

## type definitions

C requires to prefix every non-built in type name with a keyword that describes what it is.

```c
struct foo { int x; };
void func(struct foo* f);

enum some_enum { e1, e2 };
void func_enum(enum some_enun e);
```

It's possible to create an alias to avoid this requirement. A very strong convention is to use exactly the same name.

```c
struct foo { int x; };
typedef struct foo foo;
void func(foo* f);

enum some_enum { e1, e2 };
typedef enum some_enum some_enum;
void func_enum(some_enun e);
```

A lot of code combines the type definition and an alias into one statement:
```c
typedef struct { int x; } foo;
typedef enum { e1, e2 } some_enum;
```

All of the above is allowed in C++ but not required. All types defined by `struct`, `class`, and `enum` do not require these keywords whenever type names are mentioned.

## empty types

C does not allow empty types.

```c++
struct empty {}; // invalid C, valid C++
```

Empty types in C++ are commonly used in empty base optimization, tag dispatching and other tricks that leverage strong typing.

## empty parameter list

In C, function with no parameters is treated only as a name declaration - arguments are still unspecified.

```c++
// C  : 'func' declaration with unspecified amount and types of arguments
// C++: 'func' declaration that takes 0 arguments
void func();

// C  : 'func' declaration that takes 0 arguments
// C++: 'func' declaration that takes 0 arguments, just ugly (keeped for backwards compatibility)
void func(void);
```

## no return

In both languages it is valid to have a function with non-void return that does not return on some control flow path.

```c++
int func(int a, int b, int n)
{
	if (n > 0)
		return a / b;
	if (n < 0)
		return b / a;
}
```

However:

- C: it is UB to read the value returned from such function if it reached non-return path.
- C++: it is UB to just reach the non-return path when executing the function (the stricter requirement is an effect of return value optimation which C does not have).

Writing such functions is obviously discouraged, all major compilers generate a warning.

## keywords

- C++ has no `restrict` keyword. There were some attempts to bring it but it is already complicated in C - in C++ due to language complexity it could very easily become a source of bug-generating optimizations if used incorrectly. Note that every major compiler offers `__restrict`.
- C++ has no meaning for `register`, the keyword remains reserved.

## linkage

Names in the global scope that are `const` and not `extern` have external linkage in C, but internal linkage in C++.

```c++
const int n = 1;
// C  : can     be referred from other translation units
// C++: can not be referred from other translation units (requires extern)
```

## unions

C allows unions for type punning.

C++ has constructors and destructors which complicate the situation. Unions allows only to access last assigned member and any other access is undefined behaviour.

```c++
// valid C, UB in C++
union {
	int n;
	char bytes[4];
} packet;

packet.n = 1;
if (packet.bytes[0] == 1) // accessing other member
	// ...
```

## aliasing

In both languages any (potentially cv-qualified) `void*` may alias.

In C, (potentially cv-qualified) signed/unsigned/unspecified `char*` may alias.

In C++, only (potentially cv-qualified) unsigned/unspecified `char*` may alias.

See strict aliasing article for term explanation with examples. TODO link
