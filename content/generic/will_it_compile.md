---
layout: article
title: "The greatest C++ game ever: Will it compile?"
---

## abstract

This article **is not** a C++ hate wall. It's just the collection of examples of obscure language rules in action.

<details>
<summary>just this</summary>
<p markdown="block">

![I know C++](https://pbs.twimg.com/media/C0shWwKXcAAXVvA.jpg:large)
</p>
</details>

You can play the game but also use this article to test your knowledge (or your cppreference browsing skills). At least once. Or ... until new standard arrives!

## backwards compatibility

A lot of this is caused by **the best and simultaneously the worst part of C++: backwards compatibility** (even down to C89).

**Note:** if you think C++ should do large breaking change (similar to Python 2 => 3 move) I have to tell you that we should also:

- speak [Esperanto](https://en.wikipedia.org/wiki/Esperanto)
- replace decimal system with base 12 ([easier counting](https://www.youtube.com/watch?v=U6xJfP7-HCc) and non-irrational 1/3 1/6 fractions) or base 16 (because computers)
- replace Gregorian Calendar with [Fixed 13-month calendar](https://en.wikipedia.org/wiki/International_Fixed_Calendar) where every month is exactly 4 weeks + 1 day for Year Day ($13 * 28 + 1 = 365$)
- reform/replace world-wide used currencies

## game notes

Examples are mostly asorted. For most examples, classifying them would reveal the answer (or give a strong hint).

The possible answers are (**Note:** if you do not know these terms, [read this](https://en.cppreference.com/w/cpp/language/ub)):

- does not compile (ill-formed program)
- may not compile (undefined behaviour)*
- compiles, but does not link
- compiles, but has implementation-defined behaviour
- compiles, but has unspecified behaviour
- compiles, but has undefined behaviour
- compiles and has well-defined behaviour

**may not compile** - undefined behaviour, depending on the source of the problem may be easy detectable or not. Standard does not impose any requirements on code that falls under this category. Compilers may rejest such code because they can't understand such nonsense or just accept it silently assuming everything works as expected. The simplest example: complex control flow ommiting assignments: may be understood and compiled but hard or impossible to prove that variable will or will not be always initialized.

For every question it's also possible that:

- emits a warning (**note:** I'm GCC and Clang biased)
- answer depends on the language standard

**Note:** multiple code snippets may expose bugs in compilers. Answers are judged by the holy ISO standard, not compiler implementations.

All examples will have an answer (one of above terms) and an explanation (full description of the problem). Some examples may have hints.

Try to answer all questions for the given code until opening spoilers - answers or hints for one question will often help with all.

## some struct

source: old.reddit.com/r/cpp_questions/comments/9087l1

TODO fix templates and precise aggregates. Answers are wrong.

```c++
template <typename T>
struct some_struct {
    int val = gen_val();

private:
    // int weird = gen_val();

    static int gen_val() {
        return 42;
    }
};

int main() {
    return some_struct<void>{}.val;
}
```

- Variant 1: with the `weird` variable
- Variant 2: without `weird` variable
- Variants 3, 4: variants 1, 2 but without template (remove first line and `<void>`)

<details>
    <summary>hint 0</summary>
    <p>Nothing related to templates. Answer for V3 is the same as for V1. Answer for V2 is the same as for V4.</p>
</details>

<details>
    <summary>hint 1</summary>
    <p>`{}`</p>
</details>

<details>
    <summary>hint 2</summary>
    <p>TODO good unicorn image (mb some deviant art?)</p>
</details>

<details>
<summary>answer</summary>
<p markdown="block">

- V1, V3: compiles, well-defined behavior
- V2, V4: does not compile (ill-formed program)
</p>
</details>

<details>
<summary>explanation</summary>
<p markdown="block">
    
Perfect example of **Unicorn Initialization** (formerly known as Uniform Initialization).

`some_struct<void>{}` involves [**aggregate initialization**](https://en.cppreference.com/w/cpp/language/aggregate_initialization) if type has (incomplete list):

- **no private or protected non-static data members**
- no user-declared constructors (until C++11)
- no user-provided constructors (explicitly defaulted or deleted constructors are allowed) (since C++11) (until C++17)
- no user-provided, inherited, or explicit constructors (explicitly defaulted or deleted constructors are allowed (since C++17)
 (until C++20)
- no user-declared or inherited constructors (since C++20)

Otherwise it's [list initialization](https://en.cppreference.com/w/cpp/language/list_initialization) which ends up calling default constructor (because the type is not an aggregate).

V1, V3 use list initialization (default constructor).

V2, V4 use aggregate initialization which wants to use private member function.

`some_struct<void>()` works in all variants. 
</p>
</details>

## recursive main

source: stackoverflow.com/questions/2532912/

```c++
int main()
{
    return main();
}
```

<details>
    <summary>answer</summary>
    <p>may not compile (undefined behaviour)</p>
</details>

<details>
    <summary>explanation</summary>
    <p>Main function has multiple restrictions. Address of main function can not be taken, the function can not be overloaded and called directly. In reality, compilers have no problems calling main function again although they might do unexpected "optimization" in this case.</p>
</details>

## non-function main

https://stackoverflow.com/questions/7660100/what-is-happening-here-in-this-c-code
