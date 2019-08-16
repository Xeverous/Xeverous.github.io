---
layout: article
---

This is a brief article showcasing most impactul differences between C++ and other C-family languages (mostly C# and Java).

<div class="note warning">
<h4>Warning</h4>

This is only overview article, it does not replace the tutorial.
</div>

### naming style

Likely you know, but just in case: C++ followed C naming conventions and uses beatiful lowercase_snake_style for most things, including class names.

Macros are written using UPPERCASE, template aliases and concepts - PascalCase.

What style you use is your personal choice (many C++ libraries use various styles) but from the Core Guidelines, it's recommended to be consistent with the standard library.

Some studies show it's better for reading (compare `find_first_not_of` with `findFirstNotOf`). DoYouAlsoFindSnakeCaseToBeMoreReadable?

Snake case also resolves some conflicts: `convertMDToHTML` or `convertMdToHtml`? `convert_md_to_html`! There are numeric distribution types in `<random>`. How would you write `student_t_distribution` and `fisher_f_distribution` as PascalCase?

Do not fear verbosity. Things can be hard to name, but I don't think you will beat `std::hardware_constructive_interference_size` and `std::allocator_traits<Alloc>::propagate_on_container_move_assignment`.

### access operators

C++ splits `.` into 3 operators which have mutually-exclusive meaning:

- `::` (scope resolution) - accessing namespaces, definitions, static methods
- `.` (dot) - accessing member variables, functions through value/reference
- `->` (arrow) - accessing member variables, functions through pointers/smart pointers/iterators

This may sound unnecessary but once you notice that something can both be a treated as an object (eg smart pointer object) and as an reference-like access (object managed by smart pointer) it makes sense.

```c++
// ::      namespace access      ::
std::unique_ptr<animal> ptr = std::make_unique<cat>("meow");
ptr. // access smart pointer stuff (memory info, allocator state, change/swap resource)
ptr-> // access animal class public methods
```

### typing

All types in C++ are value types (passed by value by default). All pointers and references must be explicitly declared. This is in contrast to C# and Java where classes are reference types - C++ treats absolutely everything by value by default.

This is why most functions in C++ take parameters by (const) reference - it's very normal to append `&` to every bigger object to avoid expensive (often allocating) copies.

```c++
void func1(std::string str);        // function allocates temporary string
void func2(const std::string& str); // function works on the given object, guaranteed no modification
```

If you don't use (const) references when passing objects, C++ will perform deep copies every time (unless the compiler can elide them which is not always possible).

### object copying

There is no `clone()` in C++ because objects perform deep copies of themselves by default. C++ differentiates copy constructors (which perform deep copies) and move constructors (which retake the ownership of resources). References can be viewed as shallow copies - under the hood they are just pointers to the same object.

There is no copy-on-write:

- if you use a const reference, you can't write (modify the object)
- if you use non-const reference, you modify the same former object
- if you don't use references, you deep copy instantly
- if you use move operations, you perform resource ownership changes - the moved-from object releases its contents and moved-to object takes care of them

```c++
std::string foo1 = "foo";
std::string foo2 = foo1; // deep copy, duplicates memory use
// both strings have own memory, changes to one are unrelated to another

std::string bar1 = "bar";
std::string bar2 = std::move(bar1); // bar1 passes it's resources to bar2
// bar2 now owns memory of text "bar", bar1 is empty

std::string qux = "qux";
std::string& ref = qux;

// these 2 lines are the same - they both modify the same object
qux = "text";
ref = "text";
```

Copy-on-write idiom would require different string implementation.

### control flow

These are just examples of commonly unknown features for common instructions.

C++ offers for each loops (technically named range-based loop), the syntax is identical to the one in Java:

```c++
for (const std::string& str : arr) // note that by const/reference applies just like with function arguments
```

It is possible to write an own type which supports range-based iteration (requires certain member functions and their properties).

Ifs (and since C++20 also while) can have an initializer statement before the condition:

```c++
if (const type val = func(); val != expected_val)
// val scope only to if and else blocks
```

### iterators

C++ iterators are different than the ones from Java. There are few Java-like but the majority are more of pointer abstractions than data structure control. What iterators can do depends on their type (different data structures offer different iterators). They practically always come in pairs (beginning, end) and are most commonly used with standard algorithms.

### type convertions

C++ has 2 types of convertions: implicit and explicit.

Implicit convertions happens under certain conditions, some these convertions exists as backwards compatibility. The most often used ones are integer convertions - pretty straightforward things such as promotion from `int` to `long`.

Explicit convertions are the ones you have to write in order to perform them. Note that C syntax (`(new_type)val`) is discouraged as it can violate type system.

C++ has own set of convertion keywords, each doing it differently and for different purpose. In practically every scenario, only 1 of them is the right choice - it's not that hard to learn them.

```c++
static_cast<new_type>(expr)
dynamic_cast<new_type>(expr)
const_cast<new_type>(expr)
reinterpret_cast<new_type>(expr)
```

Read the convertions tutorial for full "how do I cast in C++".

### inheritance

C++ has no notion of interfaces, however there are no restrictions on class members and inheritance - the border is very fluid. If you want to create an interface, simply don't add any member variables. If one or more function is pure virtual (has no defined body) the class implicitly becomes abstract (there is no `abstract` keyword).

Multiple inheritance is allowed, including situations where more than 1 parent class has member variables, although rarely used in practice - almost everywhere regular recommendation is followed (0 or 1 parent class and 0+ interface-class parents).

There are also:
- virtual inheritance (solves [diamiond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem))
- protected/private inheritance (blocks upcasting, parent class becomes non-public implementation detail)
- recursive-like inheriting from own template (used in specific template pattern to implement compile-time polymorphism with no virtual call overhead - CRTP)

Sticking to simplest OOP while learning is a good thing. Virtual inheritance and non-public inheritance is hardly ever used.

### resource management

C++ does not have garbage collector, instead [RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization) idiom is used. These are mostly zero-overhead abstractions over memory allocation, which crucial part are destructors.

RAII is used mostly for memory management (smart pointers) but other resources can also be encapsulated.

It is a big mistake to use raw owning pointers and `new`/`delete` in modern C++ (unless you are writing own data structure, smart pointer or an allocator). In many places of C++, pointers are not needed - many beginners may overuse them.

In GC languages, `new` is the way to create objects. In C++, this is just one of the lower layers:

*from the top (highest level) to the bottom (lowest level)*

- smart pointers (eg `std::unique_ptr`), containers (eg `std::vector`)
- allocators (eg `std::allocator`)
- `new`/`delete`/`new[]`/`delete[]` expressions (there are 20+ overloads)
- `std::malloc()`, `std::free()`
- OS implementation of allocation (`malloc()`, `free()`) or your own (eg [jemalloc library](http://jemalloc.net))

The core how-to of RAII is rather simple - you don't use lower-level layers and you don't care. Destructors of standard types do it for you. Smart pointers are basically pointer wrappers which just call freeing functions when they are destroyed (go out of scope).

TODO link?
Read the smart pointer tutorial for full examples and more in-depth expalantions.

### templates

C++ offers very advanced template metaprograming. Everything is resolved at compile-time and optimized for each use case. Template specialization, SFINAE and concepts can be used to further customize behaviour depending on the properties of types.

Multiple templates exist:
- of functions
- of classes
- of aliases
- of variables
- of other templates

C++ templates can be viewed as a separate, Turing-complete purely functional language. This can be mixed with imperative programming and other compile-time features of C++.

It's hard to briefly showcase the full power of templates, just note that
the tutorial about templates is one of (if not #1) the longest tutorials on this website. Functions and classes accepting `T` is not even a 1 complete chapter.

### callbacks and concurrency

There are no keywords for such things in C++ (`delegate` or `async`/`await`). Instead, very advanced templates are used.

C++ implementation has to cope with underlying OS API, which so far is always C. It may look hard, but once you learn how to use templates it feels definitely better than standard C or POSIX functions. You don't have to implement threading yourself - just use standard library.

Just to ilustrate:

```c++
// C++ (thread class constructor)
template <typename Function, typename... Args>
std::thread(Function&& f, Args&&... args); // Ok, pass function name and any arguments it takes (including "this")

// standard C
typedef int (*thrd_start_t)(void*);
int thrd_create(thrd_t* thr, thrd_start_t func, void* arg); // What? Only 1 type of function possible? How do I void* my arguments?

// POSIX (pthreads)
typedef void* (*thrd_start_t)(void*); // How do I void*(*)(void*)?
int pthrd_create(thrd_t* thr, thrd_start_t func, void* arg);
```

Standard library covers threads, parallel algorithms, atomics, async/await, callbacks and more. Even more is also available in Boost.

### exceptions

Exceptions in C++ are quite rare and the standard library is very minimal in this regard. Many projects don't use them (disable with compilation flags) for performance reasons and the standard library rarely can throw.

The base standard class for every exception is `std::exception` which has only one method to override:

```c++
virtual const char* what() const noexcept;
```

You can derive directly from `std::exception` or [it's descendants](http://en.cppreference.com/w/cpp/error/exception).

Note that this is not a requirement. C++ allows to throw anything, including things not derived from standard exception class or even plain non-class types such as integers. Of course if exceptions are used, it's recommended to derive from standard exception class or create own tree of classes for this purpose.

It's strongly recommended to catch by const reference

```c++
catch (const std::exception& e)
{
    std::cout << e.what() << '\n';
}
```

It's possible rethrow (write `throw;` inside a catch block) and catch everything possible (`catch (...)`)

<div class="note info">
<h4> finally </h4>

C++ does not have `finally`. Use RAII idiom (destructor-encapsulated resource management) instead.
</div>

**Trivia**

Exceptions are disputed, their existence in C++ is controversial. This is the only language feature which does not adhere to the rule "you don't pay for what you don't use". There is some work under progress to change this situation. Herb Sutter has proposed to refactor exceptions into static error handling by using same-return-channel implementation - essentially each exception would be of the same type (implemented as an integer). Such approach allows to get rid of allocation and provide compile-time guarantees about handling. I don't know any programming language which has that sort of error handling. Other people has suggested to change where and how standard library throws - specifically in the failure of memory allocation. We will see what will be done in the future - as of now, the committee is clear that the current situation of exceptions is not satisfactory.

### reflection

C++ does not have reflection (although there are some plans for it for 2023/2026). There is no `typeof` keyword, but the language offers two other:

- `decltype` which deduces type from the expression at compile time (no polymorphism here). Used mostly in templates.
- `typeid` which queries type information at runtime - the keyword returns a struct containing implementation-defined type name and hash. Hardly ever used.

The lack of reflection and related runtime features is simply because it's not the aim of C++. The entire language is designed to form zero-overhead abstractions and discard all type information during the build. Most performance-oriented C++ projects don't use RTTI.

### documentation

C++ has no standarized documentation model, however practically every project uses [Doxygen](https://en.wikipedia.org/wiki/Doxygen) which has became the unofficial standard tool.

Doxygen is just a replica of JavaDoc with minor changes to accomodate to C++ language (eg `@tparam` for template parameters). If you have worked previously with JavaDoc, you already know how to document C++.

Doygen offers multiple options, and some alternative syntax but in most cases defaults are used for intuitive compability with JavaDoc.

### build system

Likely you know, C++ uses header + source project structure. It's a remnant of C build system. There is work undergoing on modules (including proposals from Microsoft and Google), but it will take few years to be fully standarized and implemented.

The build itself is automated. C++ IDEs either generate CMake lists which then generate makefiles or directly generate makefiles. From makefiles, compiler, linker and assembler are invoked with relevant comamnds. The part you have to do is just to select commands in the IDE (language standard, warnings, project/library paths, etc).

There are 2 major build systems:

- any IDE except VS => CMake (optional step) => makefile => GNU make => GCC/Clang compiler
- CMake (optional) => Visual Studio => makefile => MS nmake => Microsoft C++ compiler

### libraries

C++ has available an enormous amount of both old and newer libraries. Also, almost every C library is C++ compatible. Thankfully to the backward-compatibility aims - even the code from 1980s can still be compiled/run with the code from newest standards!

Unfortunately there is no standarized process of adding libraries to a project nor any official package manager. Recently there have been big changes in this regard - modules are finally comming into the language but it's just the beginning of automatizing library integration.

For this reason, multiple libraries are header-only (especially the smaller ones) - projects copy-paste them to repositories and just use includes. If not, they have a guide how to build and integrate - usually with a meta build tool such as CMake (generates project and make files).

## ???

Features you are likely to know, but have important differences in C++:

- OOP (eg multiple inheritance, virtual inheritance, no interfaces)
- `const`/`final`/`readonly` (`const` has much bigger power in C++)
- `var`/`let`
- operator overloading (C++ offers much more than C#)
- exceptions

Features that are very different in C++:

- Memory management: if you previously worked only with garbage-collected languages, this will be a whole new thing. It's not that hard as you may expect. Surprise fact: `new` in C++ is not used for what you may think it is.
- Multithreading (reason: look point above).
- Templates. It's a very different thing from dynamically typed languages and from generics in C# and Java. Functions or classes accepting `T` are only the beginning of metaprogramming, not the feature itself.
- Build system: header + source files (reason: C compability, but this is changing, Modules in 2020)

Features that are not (yet) in C++

- advanced RTTI
- reflection
- pattern matching
