---
layout: article
---

There are some myths going around which are simply wrong.

## "In C++, you need to manually free any resources"

Since its creation (as an upgrade to C) one of C++ aims was to get rid exactly of this issue. Every resource should be encapsulated in a class that enforces its invariants (constructor acquires, destructor releases, copy/move constructor change ownership) (RAII or SBRM idiom). Once a resource is encapsulated, there should never be a need to release it manually.

People who claim there is such problem are usually not aware that C++ has compile-time enforced destructors and/or are not aware of the simplest resource-managing classes from the standard library (containers + strings). There was never such problem in C++, and the modern C++ update (post 2011) with *smart pointer* class templates and move ctors made resource encapsulation and management significantly easier.

In other words if you:

- allocate/release resources outside designated member functions (ctor, dtor, copy ctor, move ctor)
- use `new` and `delete` keywords (which are considered low-level in C++) for something other than internal implementations of data structures / smart pointers / allocators (which hardly ever anyone writes)
- constantly check which pointers are observers and which are used free resources
- are resistant to use exceptions "because they will prevent code from reaching resource release lines"
- manually write all control flow paths for each error condition (eg `if (rc != 0) goto err;`)
- think that there is no other resource to manage than memory

...then you are doing C++ wrong.

## "classes imply overhead"

C++ classes that do not implement polymorphism are completely zero-overhead. Any member function is equivalent to a free (global) function that takes a pointer to the instance. There is no type information or any other metadata embedded in objects. The size of a class object in memory is equivalent to the sum of sizes of its members + any padding/alignment required by the target architecture.

Example C code:

```c
#include <math.h>

struct point
{
    int x;
    int y;
};

double distance_from_origin(const struct point* p)
{
    return hypot(p->x, p->y);
}
```

Example C++ code:

```c++
#include <cmath>

struct point
{
    int x;
    int y;

    double distance_from_origin() const
    {
        return std::hypot(x, y);
    }
};
```

x86_64 assembly for C:

```asm
distance_from_origin:
  pxor xmm0, xmm0
  pxor xmm1, xmm1
  cvtsi2sd xmm0, DWORD PTR [rdi]
  cvtsi2sd xmm1, DWORD PTR [rdi+4]
  jmp hypot
```

x86_64 assembly for C++:

```asm
point::distance_from_origin() const:
  pxor xmm0, xmm0
  pxor xmm1, xmm1
  cvtsi2sd xmm0, DWORD PTR [rdi]
  cvtsi2sd xmm1, DWORD PTR [rdi+4]
  jmp hypot
```

In the case of classes which have virtual functions, the resulting object stores additional (hidden) data. For any of major implementations, this is 1 virtual table pointer for each unique base "interface" class. This does not make C++ any worse - C code that implements the same behaviour runs at the same speed but is usually manually written which only increases the chance of a mistake.

## "C is faster than C++"

... is just a false claim. **Properly written code** in both languages should work fast.

Advantages of C for optimization:

- `restrict` keyword

Advantages of C++ for optimization:

- **significantly more type information** (particulary for optimizations relying on *strict aliasing* which has the same effect as `restrict` in C)
- more constrained language rules
- generic code (being able to adapt to more than 1 type), compile time code generation
- contracts (checked in debug builds for runtime errors, used to optimize release builds)

## "C++ == lower level code compared to higher-level languages (eg Java, C#)"

This is just wrong. In fact, C++ contains very powerful template metaprogramming features which let you write even higher-level and more reusable code than in the languages C++ is ususally compared to.

## "You need to learn C before C++"

It is undoubtly true that there is some common subset between C and C++ but:

- this subset is still not complete C
- this subset is stricter and more expressive in C++ (errors are catched earlier)
- a significant part of this subset is also inside any other programming language
- some parts of this subset have better alternatives in modern C++ (either built into the language or inside the standard library)
- code regarding this subset has different conventions in C and C++
- this subset is covered by this (and likely, any other) C++ tutorial

So no, there is no reason to learn C before C++.

## "C++ is a past language and is no longer maintained"

This wrong, and even more wrong than ever since "modern C++" (post-2011 standards) came out.

The language is mainatined by ISO WG21 committee (with increasing number of members) and so far issued new standards in 1998, 2011, 2014, 2017 and prepares upcoming 2020 standard.

In 2013, CppCon started, gathering experts of various fields annually to a conference with open entry for speaking. Videos from public lectures are available on YouTube.

## "C++ is platform-dependent"

Some of the constructs are platform-dependent (eg the size of built-in types, padding, alignment and support for arithmetic operations) but with a little help of the standard library and some language features offering generic code, you can very easily write platform-independent code.

Also, Java is not platform independent. Actually, it runs only on 1 platform - JVM.

## "C is better than C++ for small projects"

There is no overhead for "just using C++" like program startup cost. C++ does not require more code to acomplish the same tasks as in C. If you need to acomplish any of tasks that are not covered by C language or its standard library (even just string or array manipulations) you will need to write a lot of bug-prone low-level code to solve problems which are already solved in the C++ standard library.

## "You need to declare something before using it."

As in the following code:

```c++
double average(const int* arr, int len)
{
    int sum;
    int i;

    sum = 0;
    for (i = 0; i < len; ++i)
        sum += arr[i];

    return double(sum) / len;
}
```

The statement in the myth is just wrong.

- First, statements like `int sum;` are actually a definition, not a declaration.
- The requirement to list all used objects at the start of the scope only existed in C89. Any later C standard has no such rule. **There was never such requirement in any C++ standard.**

Also:

- This myth encourages code which introduces unnecessary object lifetime (this complicated code for the reader). In the above example, `i` outlives the loop.
- This myth encourages code that does not initialize objects (no assigned value upon creation) - this is bug-prone
- This myth encourages code with unnecessarily complex flow, which may prevent applying `const` where it could be done otherwise.
