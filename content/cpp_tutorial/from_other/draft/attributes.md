---
layout: article
---

Prerequisities: header vs source

Modern C++ introduced attributes which can be used as "labels" on practically any part of code (a type, a function, statement, etc). They provide a unified standard syntax for implementation-defined language extensions, such as the GNU and IBM language extensions `__attribute__((...))`, Microsoft extension `__declspec()`, etc.

```c++
// built-in attrbiutes: [[attribute(args), ...]]
[[nodiscard, noreturn, unlikely]]

// implementation-defined attributes
// [[namespace_name::attribute(args), ...]]
[[gnu::hot, gnu::cold]]
// [[using namespace_name: scoped_attribute(args), ...]]
[[using gnu: hot, cold]]
```

Attribute names come either from the language itself (global scope attributes) or are supported by the implementation as an extension (which must use a namespace). Implementations are required to ignore unknown non-standard attributes.

In most other situations, attributes apply to the directly preceding entity.

Attributes should not be repeated. If there is an attribute applied to some function declaration in a header file, the source file should not mention it (the same holds for function default arguments).

## comparison with C# attributes

- C++ attributes are not classes.
- C++ attributes do not embed any metadata. They are merely a signs for the compiler.
- You can not implement any new attributes.

## `[[noreturn]]`

Indicates that a function does not return (that is, the function throws, terminates the program or performs a non-local jump). The behaviour is undefined if the function actually returns.

This attribute is hardly ever needed for user code. Some standard library functions are marked with this attribute for clarity.

```c++
#include <iostream>
#include <csetjmp>

std::jmp_buf jump_buffer;

[[noreturn]] void a(int count)
{
    std::cout << "a(" << count << ") called\n";
    std::longjmp(jump_buffer, count+1);  // setjmp() will return count+1
}

int main()
{
    volatile int count = 0; // local variables must be volatile for setjmp
    if (setjmp(jump_buffer) != 9) {
        a(count++);  // This will cause setjmp() to exit
    }
}
```

`longjmp` is the mechanism used in C to handle unexpected error conditions where the function cannot return meaningfully. Because non-local jumps do not execute destructors of automatic objects, in C++ prefer exception handling for this purpose.

## `[[carries_dependency]]`

Indicates that dependency chain in release-consume `std::memory_order` propagates in and out of the function, which allows the compiler to skip unnecessary memory fence instructions.

See https://en.cppreference.com/w/cpp/language/attributes/carries_dependency for example.
