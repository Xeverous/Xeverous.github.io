---
layout: article
---

## custom allocation

It's possible to overload `new` and `delete` to provide custom memory allocation for the given type. Since this is a C++ tutorial, we are far away from cases where something so advanced could be easily explained.

No type in the standard library overloads these operators.

## unary `*` and `->`

These operators allow to replicate pointer interface and in fact are used by smart pointers and iterators. TODO order of articles? Should operator overloading be later?

## other operators

Operators `&&`, `||`, unary `&` (the address of operator) and `,` are hardly ever overloaded.

The first 2 doesn't signal any concrete behaviour - what would be the purpose of `a && b` when they are not boolean?

Overloading `&` makes obtaining address of the object harder, assuming one knows about `std::address_of` which can bypass it.

Overloading comma only asks for more problems - imagine passing arguments to function: `f(a, b)` suddenly becomes `f(operator,(a, b))` (and calls the function overload for 1 argument).

#### Question: Are there any places in which these operators are overloaded?

Yes. C++ is a fun language where even the most obscure "features" will be found by someone to be usable.

Operator `,` is overloaded in [Boost Assign](http://www.boost.org/doc/libs/release/libs/assign/doc/index.html#intro) to extend interface of STL containers. This library is somewhat old (pre modern C++ standards), since C++11 there are much better (and less confusing) ways to do things what the library provides.

A true mastery of extraordinary operator overloads is achieved in Boost Spirit. [This short article](https://en.wikipedia.org/wiki/Spirit_Parser_Framework) gives an overview of the library purpose - operators overload templates are used to form grammar specification allowing to parse and lex any text. This sort of solution is known as DSL (domain specific language) where a different language happens to be inside another (here EBNF grammar inside C++).

## summary

Don't overload `&&`, `||`, `&` (address of) and `,`. Just don't.
