---
layout: article
---

C++ has some very unique keywords which may not have clear purposes.

This list is not exhaustive.

## alignment

Specify and query type's alignment in memory. Not a really often used stuff, likely for some HPC and cache-oriented optimizations.

```c++
struct alignas(16) sse_t
{
    float sse_data[4];
};
```

```c++
#include <iostream>
#include <cstddef>
int main()
{
	// print alignment that is as strict as of every scalar type
	std::cout << alignof(std::max_align_t) << '\n';
}
```

## size

Query the size (in bytes) of an object at compile-time. C++ guuarantees that `char` types (any signess) occupy 1 byte and any other type has size >= 1.

Pointers typically occupy architecture length. There are some guuarantees on integer types lengths for specific platforms. TODO link cppreference

```c++
#include <iostream>

// empty base optimization applies, but minimal size is 1
struct empty {};
struct A : empty {};
struct B : empty { int x; };
struct C : B { int y; };

struct polymorphic { virtual ~polymorphic() = default; };

int main()
{
	std::cout << "size of char      : " << sizeof(char) << "\n";
	std::cout << "size of int       : " << sizeof(int) << "\n";
	std::cout << "size of long      : " << sizeof(long) << "\n";
	std::cout << "size of long long : " << sizeof(long long) << "\n";

	std::cout << "size of pointer   : " << sizeof(void*) << "\n";

	std::cout << "size of empty type: " << sizeof(empty) << "\n";
	std::cout << "size of A         : " << sizeof(A) << "\n";
	std::cout << "size of B         : " << sizeof(B) << "\n";
	std::cout << "size of C         : " << sizeof(C) << "\n";

	std::cout << "minimal size of polymorphic type: " << sizeof(polymorphic) << "\n";
}
```

`sizeof(void)` is invalid.

## inline assembly

I doubt you will ever need it. Support, syntax and rules regarding inline assembly are implementation-defined.

```c++
// works only in x86_64 Linux, compile with GCC
#include <iostream>

extern "C" int func();
// the definition of func is written in assembly language
// raw string literal could be very useful
asm(R"(
.globl func
    .type func, @function
    func:
    .cfi_startproc
    movl $7, %eax
    ret
    .cfi_endproc
)");

int main()
{
    int n = func();
    // extended inline assembly
    asm ("leal (%0,%0,4),%0"
         : "=r" (n)
         : "0" (n));
    std::cout << "7*5 = " << n << std::endl; // flush is intentional

    // standard inline assembly
    asm ("movq $60, %rax\n\t" // the exit syscall number on Linux
         "movq $2,  %rdi\n\t" // this program returns 2
         "syscall");
}
```

## automatic type

Basically C++'s `var` or `let`. Type deduced at compile-time - requires some expression on the right side.

```c++
auto x = 0ul;          // unsigned long
auto b = true;         // bool
auto s1 = "abc";       //
auto s2 = "abc"s;      // std::string (uses custom suffix)
auto c = 1.0f + 1.0if; // std::complex<float>
```

TODO cases with references?

## null

Since C++11 there is a special keywords for null pointers - `nullptr`. It is a compile-time constant that can be used as a null representation of any pointer type.

<div class="note pro-tip" markdown="block">

Do not ever use macro `NULL`. It is not type-safe, does not work in some templates and can create various bugs related to text replacement.
</div>

## compile-time assertion

Forces compilation fail if triggered. Useful for testing templates and veryfing code relying on platform-specific characteristics.

Expression must be able to be evaluated at compile-time and have convertion to `bool`.

Syntax:

```c++
static_assert(constexpr bool, error_msg);
// since C++17 message no longer required
// in earlier standards you can write ""
```

Examples:

```c++
static_assert(__cplusplus > 201703, "at least C++17 required");

static_assert(
	std::is_same<
		typename std::decay<decltype(*it)>::type,
		typename std::iterator_traits<It>::vale_type,
	>::value,
	"Wrongly implemented iterator!"
);
```
