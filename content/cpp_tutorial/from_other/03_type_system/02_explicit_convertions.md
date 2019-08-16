---
layout: article
---

Practically no question regarding C++ has an easy answer and so it is with explicit convertions.

TODO mention `explicit`.

## C-style casts

The casts in the form of `(type)expr` are prevalent in many languages but *discouraged* in C++.

*Discouraged* here means it is being kept for backwards compatibility. This cast is not good because there are situations in C++ where there is more than 1 possibile method of convertion - C casts can be ambiguous. Compilers also offer to set up a warning (GCC/Clang: `-Wold-style-cast`).

The only widely accepted cast of this form is explicit discard of the result: `(void) expression`. It relies on the fact that the result of any expression can be converted to void type - this cast should be used purely to shut warnings about unused funtion parameters.

## functional casts

First edition of C++-favoured casts were functional casts: `type(expr)`. They have cleaner syntax: `int(x + y)` vs `(int) x + y`.

This cast when used with a class type, does not perform any special convertion - it is just a constructor call.

## modern casts

These are the only casts that you should use in modern C++. New, named casts solve these problems:

- ambiguous C-style casts (reinterpretation or actual convertion)
- problems related to old C that had no `const` keyword
- be explicit - new casts clearly describe intention

```c++
     static_cast<T>(expression)
    dynamic_cast<T>(expression)
      const_cast<T>(expression)
reinterpret_cast<T>(expression)
```

## static cast

There are 2 purposes:

- for built-in types: arithmetic convertions (between arithmetic types)
- for user-defined types:
  - moving across inheritance tree
  - integral-to-enum and enum-to-integral

### built-in types

There is no need for explicit casts when doing a promotion (lossless convertion) but narrowing in many contexts will require it.

```c++
double x = 3.3;
double y = 1.6;

// floating-point division, result truncated to 2 when saved to int
int a = x / y;
// integer division: 3 / 1, result saved to int
int b = static_cast<int>(x) / static_cast<int>(y);
```

Static casts can not be used for int-to-string or string-to-int convertions since strings are not a built-in type. Standard library offers multiple parsing functions instead. See parsing tutorial TODO link

### user-defined types: upcast

Every derived type is also a base type. C++ allows to implicitly convert derived type pointers and references to base type.

```c++
struct base {};
struct derived : base {};

void foo(const base&);

void bar()
{
    derived d;
    func(d); // ok, implicit upcast is always safe
    func(static_cast<base&>(d)); // verbose, unnecessary
}
```

Note that if `foo` took the parameter by value, **object slicing** would happen, which is hardly ever wanted - see OOP chapter for explanation and recommendations.

### user-defined types: downcast

Static casts can be used to perform downcasts on pointers and references. **No runtime check is performed.** This cast has no overhead but invokes undefined behaviour if the object is not of derived type.

```c++
static_cast<derived&>(base_obj)
```

Like in other languages, downcasts smell of bad code. Notable exception: C++-template-specific design patterns like CRTP use static downcasts but their correctness is enforced at compile time so there is no risk of undefined behaviour.

### user-defined types: sidecast

Multiple inheritance brings the possibility of sidecasts. They can not be performed directly though.

```c++
struct A {};
struct B1 : A {};
struct B2 : A {};
struct C : B1, B2 {};

C c;
B1& b1 = c;
B2& b2 = static_cast<B2&>(b1); // error: B2 is not a parent of or descendant of B1
B2& b2 = static_cast<B2&>(static_cast<C&>(b1)); // ok (first down, then up)
B2& b2 = static_cast<B2&>(static_cast<A&>(b1)); // ok (first up, then down)
```

Sidecasts are of course a big code smell.

## dynamic cast

This cast is intended for safe downcasting and sidecasting - for this to work the type must be polymorphic. The cast can not remove costness or volatility.

Static casts have no difference when casting over references or pointers, but for dynamic casts:

- when casting over pointers and cast fails, null pointer is returned
- when casting over references and cast fails, `std::bad_cast` is thrown

```c++
class animal
{
public:
    virtual ~animal() = default;
};

class cat : public animal {};

void f1(animal& a)
{
    // success: pointer to cat
    // failure: null pointer
    if (auto c = dynamic_cast<cat*>(&a); c != nullptr)
    {
        // use c...
    }
}

void f2(animal& a)
{
    // success: reference to cat
    // failure: std::bad_cast is thrown
    auto& c = dynamic_cast<cat&>(a);
    // use c...
}
```

comparison with static cast:

cast | down (success) | down (failure) | side cast
-----|----------------|----------------|---------
static | no overhead | undefined behaviour | not allowed
dynamic (over pointers) | pointer to derived | null pointer | allowed
dynamic (over references) | reference to derived | `std::bad_cast` thrown | allowed

```c++
// example implementation of Java keyword
// usage: if (instanceof<T>(obj))
template <typename Derived, typename Base>
bool instanceof(const Base& base) noexcept
{
    return dynamic_cast<Derived*>(&base) != nullptr;
}
```

## const cast

All types can have implicitly added constness, but not removed. This cast allows to explicitly remove constness or volatility.

```c++
int legacy_syscall(char*);

void func()
{
    const char* path = "/bin/sh";
    legacy_syscall(const_cast<char*>(path));
}
```

The cast allows to workaround C++ type system rules, but still invokes undefined behaviour if the const-casted object is attempted to be modified.

This cast is purely for interacting with legacy code (mostly OS interfaces) that date back to times preceeding `const` keyword (`const` initially appeared in C++; after some resistance the feature was also added to C).

## reinterpretation

Like `const_cast`, `reinterpret_cast` does not compile to any CPU instructions (except when converting between integers and pointers or on obscure architectures where pointer representation depends on its type). It is purely a compile-time directive which instructs the compiler to treat expression as if it had a different type.

Reinterpretation can be used to:

- **perform casts between two arbitrary pointer types** (including different pointer levels but excluding casting away constness or volatility or types without allowed aliasing)
- perform pointer-to-integer and integer-to-pointer convertions (converting between numerical address of the object and the pointer type)
- perform casts between (member) function pointer types
- perform casts between pointer-to-member-A-of-T1 and pointer-to-member-B-of-T2 (with restriction that T2's alignment can not be looser than T1's)

The last 2 casts are practically never used.

The second one is useful for some embedded devices where addresses of some registers are fixed (pointers to these registers could often be `volatile`).

```c++
#include <cstdint>

std::uint64_t read_register(std::uintptr_t addr)
{
    auto reg = reinterpret_cast<const volatile std::uint64_t*>(addr);
    return *reg;
}
```

```c++
#include <iostream>
#include <iomanip> // std::hex
#include <cstdint> // std::uintptr_t

int main()
{
    int x;
    // note that this is already done inside the stream if given a pointer
    std::cout << "address of x: " << std::hex << std::showbase
        << reinterpret_cast<std::uintptr_t>(&x) << '\n';
}
```

The first cast allows object inspection as it was of a different type **but only if the resulting pointer does not violate aliasing rules.** C and C++ compilers perform many load/store optimizations, some of which rely on the fact that pointers to unrelated types never have the same value (in other words, they can not alias). **Failure to satisfy aliasing requirements results in undefined behavior.**

A common case of aliasing bugs is an attempt to read a floating point value as an integer:

```c++
#include <cstdint>

std::uint32_t float_as_uint(float f)
{
    static_assert(sizeof(std::uint32_t) == sizeof(float), "use different integer type");
    return *reinterpret_cast<std::uint32_t*>(&f);
}
```

This is undefined behavior, because under no circumstances a valid C++ program may have 2 pointers of very different types that point to the same memory location.

For such simple cases, they might be detected by a compiler. More complex cases (eg passing address of an object as `void*` to a callback and casting the pointer inside the callback to an object of different type that was actually given) are either very hard to detect or require specific tools which perform complex static code analysis or search for optimization-on-UB-code symptomps like misaligned reads.

```
main.cpp: In function 'uint64_t float_as_uint(float)':
main.cpp:5:13: warning: dereferencing type-punned pointer will break strict-aliasing rules [-Wstrict-aliasing]
     return *reinterpret_cast<std::uint64_t*>(&f);
             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Read the article about strict aliasing for a detailed explanation of aliasing problems and solutions to tasks like object-as-bytes introspection. TODO link
