---
layout: article
---

## preface

The following tutorial is aimed at people who already know programming principles (best suited for people who have experience with other C-family languages, especially Java and C#) and would like to learn another language without the need to go through all the basic stuff while at the same time not being afraid they accidentally skipped something important.



I will still show a lot of basic stuff just to make you familiar with the syntax - while most languages have for-each loop each has a different syntax for it. Explicit use of pointers might also be something new for you.

I will also point out many common mistakes that people coming from other languages may commit. A lot of mistakes can be classified as "writing C in C++".

## "C/C++"

You may very often encounter this phrase. It has as much meaning as "Ham/Hamster" or "Java/JavaScript" (btw this pair does share some history too).

C++ is much different from C not only by the amount of language features but also by conventions. A good C code may often be terrible C++ code. Having over 35 years of backwards compatibility does not mean we should write C-ish C++ today.

C++ is not "C with classes". C++ is not just object-oriented language. It is a multi-paradigm language that allows multiple styles of programming to be mixed, including but not limited to OOP. In fact, vast majority of C++ standard library does not rely on runtime polymorphism but generic programming. You would need to dig quite deep to even find a virtual function. A large part of standard library are free (not belonging to any class) function templates.

<div class="note warning">
Do not copy C code without understanding to a C++ project. There is a lot of C that is not legal C++ or legal C++ that invokes undefined behaviour or behaves differently.
</div>

## Some facts

C++ is:

- statically typed, NOT dynamically like in JavaScript or Python
- compiled, NOT interpreted
- having own (almost unique) memory management system, NO garbage collection
- different than C (code which works in both may often be a good C but bad C++)
- ISO standardized, NOT owned by any company

## low level access

C++ gives access to low-level stuff which manifests as:

- ability to directly call underlying system interface
- ability to run in kernel space (compared to user space) (in other words being excluded from memory protection)
- ability to be the system (OS) itself (freestanding executable, running "on bare metal")
- ability to use pointers and perform pointer arithmetics
- ability to perform manual resource management
- ability to explicitly violate type system (with restrictions)
- ability to write inline assembly code that co-operates with high-level code

Which can have following consequences:

- high performance
- **undefined behaviour**

Despite being able to do all of this, it does not mean it should be done. Actually, C++ strives to provide type-safe high-level zero-overhead abstractions and discourages low-level code that is not necessary. During this tutorial, you will learn what abstractions C++ has put on C and how to use them correctly.

## ~~memory~~  resource management

C++ discourages manual resource management which is done in C on a daily basis. C++ does not rely on garbage collection either - instead it relies on RAII idiom which so far only 2 languages support (C++ and Rust).

RAII is a completely different concept: it is a **deterministic resource management mechanism that is enforced at compile time**. RAII has different variations, some are completely overhead free, some have but still less than garbage collectors. RAII in C++ relies on **destructors**.

The tutorial will focus strongly on RAII as it is one of fundamental language pillars. There will be also lots of examples of bad code that is very easy to find on the internet - mostly C code that had minimal edits just to compile as C++.

Using `new` in C++ is considered low-level and discouraged. It should be used only to implement data structures, smart pointers and allocators.

## inheritance and polymorphism

C++ offers typical dynamic dispatch mechanism (more known as the use of virtual functions or simply polymorphism). Apart from virtual destructors, there is no difference and same OOP principles apply.

What C++ does differently are inheritance rules:

- There are no *interfaces*. A class in C++ can be said to be an interface if it consists only of body-less virtual functions.
- There are no restrictions on inheritance itself:
  - You can inherit multiple times from the same class
  - You can have multiple parent classes that have member variables
  - Parent classes may share a parent (inheritance graph does not have to be a tree)
  - A class can be abstract despite all of its parents not being abstract
  - Class templates can perform recursive inheritance and recursive class definitions

Again, being able to do all of this does not mean it should be done. General OOP recommendation applies (max 1 parent class with state).

## generic and metaprogramming

C++ would not be C++ without templates. Around 95% of standard library are templates - there is also a term STL (standard template library).

C++ *templates* much differ from *generics* which are found in other languages - a very good coincidence that other languages use a different term.

In C++, templates are first instantiated and then compiled. Each use (instantiation) with different parameters can produce vastly different assembly. This makes C++ probably the harest ever programming language to reverse-engineer.

Templates in C++ form a functional Turing-complete language itself that can be used to run any metaprogram at compile time to create a type or value. There are very few restrictions, so if you are used to facts like "this expression using `T` is not legal" you can assume it is legal in C++.

It is hard to showcase the power of templates in a short description, but for example - templates let to write generic algorithms than can run on any data structure that satisfies certain criteria. Together with lambdas, they can result in massive code reuse that achieves better performance than hand-written C.
