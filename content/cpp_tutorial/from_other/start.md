---
layout: article
---

This is the start of accelerated tutorial. It is aimed to go faster and not bother you with boring things such as functions, scoping, basic OO which you know already from other language(s).

Note: this tutorial is aimed only at users of similar languages. Check FAQ right below.

## I have experience in C, should I start here?

No. Use the beginner (from scratch) tutorial. C++ has significantly different conventions than C. Many features have been removed or replaced by better alternatives. There is no worse thing than assuming everything is still the same and that C++ is just a superset of features. C-ish C++ is bad C++.

## I have experience in C# or Java, should I start here?

Yes. These are the languages that are most similar to C++ (actually there are more similar ones, but I do not want to talk about non-standard C++ forks which do not have any wide use). This tutorial is aimed primarly to users of these languages. They both share very similar syntax and language rules. Up to certain point, most features will be identical or have only minor syntax differences.

There will be some code samples that compare equivalent C# or Java code to C++.

## I have experience in Python, should I start here?

No. Python is just too different in too many ways (including the type system, syntax and conventions like error handling). If you don't have any experience with statically-typed languages, use the from scratch tutorial.

## I have experience in JavaScript, should I start here?

No. JS is a completely different word.

## What about TypeScript?

I have no knowledge about TS, can not guuarantee that nothing important was missed. Go with this tutorial at your own risk.

## PHP?

Version 7 significantly refreshed the language but I really have no idea how close PHP 7 is to C++.

## What when I have experience in \<insert-other-language-here\>?

Use your knowledge about this language and decide on your own. In case of doubt, I recommend going with the from scratch tutorial because **C++ can be really frustrating when you assume some things and they turn out to work differently**.

## overview

This tutorial assumes that **you know** (in general):

- basic control flow rules (conditionals, loops, `continue` and `break` keywords)
- how strongly typed languages work
- function overloading
- OOP concepts (abstraction, inheritance, polymorphism)
- exceptions (how the work)
- lambda expressions (C++ lambdas are very powerful so there will be lots of examples)
- what is an IDE, OS command line etc
- good practices when writing code (meaningful naming, no code duplication, no hidden side effects, no global data etc)

This tutorial is written assuming that **you may not know**:

- C++ grammar, syntax, conventions
- header vs source files
- preprocessor (macros and conditional compilation)
- **how resources (not necessarily memory) are managed in a language without garbage collection**
- **anything related to pointers**
- C++ OOP: **destructors**, **copy/move constructors**, multiple inheritance
- operator overloading
- `const` keyword (which is extremely powerful in C++)

Additionally, this tutorial presents:

- snippets comparing C++ code with equivalent code in other language(s)
- common C++ mistakes and misconceptions
- some common build errors and how to deal with them
- how to acomplish basic tasks and common exerices (formatted I/O, random numbers, data structures and algorithms, etc)

## before you start

Make sure you have installed compiler, IDE and/or any other tools necessary to work. If not, go to TODO link to tools.

For the compiler, I recommend using GCC or Clang because:

- **unlike Microsoft's Visual compiler, they strictly verify code's compliance by default** (Microsoft has a complex history of various C++ forks and technically-invalid-but-accepted-code)
- these 2 are most up to date with language specification
- their errors are much easier to search about than the ones given by Microsoft Visual compiler
- they are free and open-source software
- they work practically everywhere (almost any unix system, for Windows there are GCC forks under name MinGW)
