---
layout: article
---

# program behavior

C++ standard precisely defines observable behaviour of any program. It is important to understand consequences of these rules.

- **well-defined** - there is only 1 way a program can behave in the given situation. This way satisfies all language requirements. For example, the `sizeof(char)` is always `1`.
- **ill-formed** - code has syntax errors or diagnosable semantic errors. Compiler is required to issue a diagnostic. This is practically always a compiler error resulting in the build failure.
- **ill-formed no diagnostic required** - code has semantic errors which are not diagnosable at compilation time (eg violations of **ODR**, ABI incompatibility, missing symbol definitions). Compilers are not required to issue any diagnostic, although in practice these errors are always caught by the linker. If such program is build and executed, the behaviour is undefined.
- **implementation-defined behavior** - program behavior varies between implementations. They are still required to satisfy language requirements and required to document effects of such behaviour. For example: the number of bits in a byte, the size of built-in types, the text message in standard exception classes. A subset of implementation-defined behavior is **locale-specific behavior**, which depends on the implementation-supplied **locale**.
- **unspecified behavior** - program behavior varies between implementations but they are not required to document it. Code should never rely on such behaviour, because implementations are allowed to change it any time and without any information. For example, the order of evaluation of arguments is unspecified which means that for `foo(bar(), baz())` `foo` is guuaranteed to be executed last but the order of `bar` and `baz` calls is unknown and may change in each compilation.
- **undefined behavior** - the most famoust of them all. There are no requirements for any implementation and the program is not required to do anything meaningful. For example: accessing array out of bounds, dereferencing null pointer, signed integer overflow, violations of aliasing rules (accesing objects of pointers of unexpected types), infinite loop without side effects. Compilers have no requirements for dealing with UB, although they catch many simple cases.

# optimizations

## the as-if rule

C++ contains actually very simple optimization rules, collectively referred to as "the as-if rule".

From the C++ standard (emphasis mine):

> An implementation is **free to disregard any requirement of this document as long as the result is as if the requirement had been obeyed**, as far as can be determined from the observable behavior of the program. For instance, an actual implementation need not evaluate part of an expression if it can deduce that its value is not used and that no side effects affecting the observable behavior of the program are produced.

This allows the compiler to do absolutely everything as long as the requirements of the observable behaviour are satisfied. This permits the compiler to:

- reorder objects in memory that are not belonging to the same array
- cache results and remove calls to other functions if they have no side effects or the side effects are the same
- remove unused/dead code
- transform/add/remove (member) functions
- reorder reads/writes if the effect is the same
- precompute predictable results
- merge/extract/unroll loops

The as-if rule has 2 exceptions:

- An implementation (as a part of optimization) is allowed to remove temporary (intermediate) and unused objects - this can cause removal of ctor/copy ctor/move ctor + destructor function pairs even if they have side effects. In other words, compiler can assume that any respective ctor + destructor are complementary and not creating an object is also valid behavior even if object creation has side effects.
- Reads/writes from/to objects of `volatile`-qualified type can never be reordered or optimized out.

## undefined behaviour vs optimizations

A correct C++ program is completely free of UB. Compilers are allowed to optimize the program assuming no UB is invoked (otherwise there would be no point in optimizing broken program). **If the program actually does contain UB, the results may be very surprising.**

### example 1

Unsigned integer overflow is well-defined to wrap-around the value.

For code like this:


```c++
int f(unsigned x)
{
    return x + 1 > x;
}
```

the x86_64 assembly can be:

```
f(unsigned int):
        cmp     edi, 0xffffffff
        setne   al
        ret
```

Which effectively returns `false` for `4294967296` and `true` for any other number.

Signed integer overflow is UB. Therefore, a compiler can assume it never happens, for code like this:

```c++
int f(int x)
{
    return x + 1 > x;
}
```

The assembly can be:

```
f(int):
        mov     eax, 1
        ret
```

No comparison is performed.

### example 2

```c++
int f(int x)
{
    int result; // (uninitialized)

    if (x == 0)
        result = 42;

    return result;
}
```

This function has 2 possible paths:

- `x == 0` and the function returns `42`
- `x != 0` and the function returns value of uninitialized variable (UB)

Compiler is allowed to issue such assembly (assusing `x` is always `0`):

```
f(int):
        mov     eax, 42
        ret
```

All of major compilers issue a warning for this code.

### example 3

```c++
int foo(int* p)
{
    int x = *p; // p must be non-null

    if (p) // chekcing whether p is non-null
        return x;
    else
        return 0;
}
```

This function has 2 possible paths:

- `p` is non-null and the value of pointed integer is returned
- `p` is null and the value `0` is returned

But there is a catch. The pointer is dereferenced before checking. Since accessing null pointers is UB, the first line causes the compiler to assume that the function expects valid pointer - otherwise the read operation would invoke UB. Since the pointer must be valid, the `else` branch is considered a dead code and the resulting assembly is a read from given address without any checking:

```
foo(int*):
        mov     eax, DWORD PTR [rdi]
        ret
```

## undefined behaviour in practice

Code which contain UB can result in:

- very reproducible crashes (accessing pointers not belonging to the program address space or null pointers is easily caught on all major OSes)
- very random, hard to reproduce crashes (accessing arrays of out of bounds may overwrite nearby pointers which can cause very arbitrary jumps) - this behavior is commonly used for security exploits
- crashes on memory (de)allocation - when UB causes memory corruptions
- crashes on program startup/shutdown (this is especially true for global objects)
- OS errors while the executable is being loaded
- compilation/link failures
- broken optimizations (eg missing reads/writes, program freezing in infinite loops)
- nothing (the program seems to work fine) - the assembly may contain undiscovered bugs (often security exploits)

