---
layout: article
---

## A collection of common misconceptions about C++

### "C++ requires manual memory management"

TODO describe RAII, perhaps Thor library article

### "C is faster than C++"

TODO move to separate article

It is not. Both languages should compile to equally fast (and often the same) machine code.

The myth appears to be true because:

**1. There is an assumption that going lower level brings better performance because you have more power.**

This can often be reverse - correctly done high-level abstractions are better for performance because compilers gain more information about the code. Take for example the sort algorithm:

```c++
// C
void qsort(
	void* ptr,
	size_t count,
	size_t size,
	int (*comp)(const void*, const void*));

// C++
template <typename RandomAccessIterator, typename BinaryPredicate>
constexpr void std::sort(
	RandomAccessIterator first,
	RandomAccessIterator last,
	BinaryPredicate func);
```

The C version takes a pointer to the data (`ptr`), amount of elements (`count`) and size (in bytes) of each element (`size`). The algorithm swaps elements depending on the result of the provided comparison function.

The C++ version takes 2 iterators (pointers are the simplest iterators) and accepts any function object that implements `operator()` which compares 2 elements provided by dereferencing iterators.

It turns out that the C++ version is much better optimized. Why?

- C version uses function pointer. It is unknown what will be put here so it's an optimization barrier. Compiler must insert function call. It can not be inlined.
- Comparisons  are usually very short and C++ template takes advantage of this and inlines practically every closure coming from lambda expressions or other comparators. This avoids large overhead of calling a function for a very simple operation.
- C version loses type information. By treating everything by `void*`, it limits any type to trivially copyable block of bytes. C++ sort implementation supports any sequence which offers random access to elements

**2. C doesn't provide very advanced abstractions and so C done wrong crashes; C++ has many advanced abstractions but wrongly choosen ones usually end in lower performance instead of crahses.**

This is a thing which is easily overlooked. Performance is never a necessity in every part of the program and so in the places where top performance is not required projects prefer C++ abstractions which are eg 10% slower than C but 10x easier and safer to use (values choosen arbitrarly).

### "C++ compilers are just transpilers to C which then call actual compiler for the C code"

This was true like ... 30 years ago. It is no doubt that all of major compilers reuse their implementation for both C and C++ but by no means C++ is translated to C and then compiled. Newer standards of C++ provide so much useful information to the compiler that translating C++ to C would be a huge waste of various optimization opportunities.

### "`const char*` is faster than `std::string`"

The entire comparison is flawed by the choice of and usage of these types.

The first one is used for compile-time provided strings, while the second one is used for dynamically allocated strings which contents are run-time dependent.

For honest comparison, one should do:

- `const char*` vs `std::string_view`
- `char*` + `malloc()` + `free()` vs `std::string`

And in both of these comparisons both approaches are pretty much even, although none are ideally equivalent for all cases.

- merging 2 `const char*` strings requires 1 call to the allocator and 2 calls to `strlen()`, which has $O(n)$ complexity
- merging 2 `std::string_view` strings requires 1 call to the allocator and 2 calls to `std::string_view::size()` which has $O(1)$ complexity

Generally, C++ classes store more information than plain character pointers (so they use more memory) but thanks to cached length information they can avoid redundant `strlen()` calls.

Also, in C++ string class there is short string optimization which I doubt is very often used in C (what is SSO - [link 1](https://stackoverflow.com/questions/10315041/), [link 2](https://stackoverflow.com/questions/21694302/)).

### "the type of `"abc"` is `const char*`"

Let's test it:

```c++
#include <type_traits>

template <typename T>
void test(T&& arg)
{
    static_assert(std::is_same<const char*, decltype(arg)>::value);
}

int main()
{
    test("abc");
}
```

```c++
main.cpp: In instantiation of 'void test(T&&) [with T = const char (&)[4]]':
main.cpp:11:15:   required from here
main.cpp:6:19: error: static assertion failed
     static_assert(std::is_same<const char*, decltype(arg)>::value);
                   ^~~
```

**The type of quoted string literals is `const char[N]`** where `N` is the amount of characters + 1 (for the null-terminating character) (other types than `char` when appropriate).

The myth comes from very often forgotten implicit convertion:

```c++
const char str1[] = "abc"; // array initialization
const char* str2 = "abc"; // implicit convertion from const char[4] to const char*
```

This specific implicit convertion is known as *array decay*.

If you remove universal reference (`&&`) from the example, it will compile because non-reference templates also perform implicit convertions.

In C, this convertion happens every time an array is passed to a function - the array decays to pointer to the first element. The following 2 functions are equivalent:

```c++
// NEVER do such things when you want more than 1 element
void func(const int* ptr);

// This is more informative to the user, but mind that for
// both C and C++ compilers this is still const int* parameter.
// The size for C-style arrays is never checked.
void func(const int ptr[4]);

// proper type-safe way
void func(std::span<int, 4> s);
```
