---
layout: article
---

<div class="note info">
This article is aimed to explain the entry point of most complex C++ templates to those who are not very familar with the language.

If you are familiar with Java or C# generics or generally understand core principles in generic programming you should understand well what makes C++ templates different from generic programming in other languages.
</div>

SFINAE is one of the most arcane C++ "features" which allows to enable/disable *certain code* depending on the properties of a type or value. That *certain code* can be (non-exhaustive list):

- function overload
- function template specialization
- class template specialization
- variable template specialization

SFINAE is not a direct language feature, hence the quotes. It's more of a consequence of certain language design decisions that led to its possibility. SFINAE itself comes from very clever use of language rules regarding templates.

The acronym comes from *substitution failure is not an error* which is one of major rules that led to its existence.

Of course to fully understand it's power we need an example. I will start with simple problem, move on and then explain where SFINAE kicks in.

## starting point

We have a classical animal example. Focus on the `print_info` function - currently it's overloaded for any animal type and integers.

```c++
#include <iostream>

class animal
{
public:
    virtual ~animal() = default;
    virtual std::string sound() const = 0;
};

class cat : public animal
{
public:
    std::string sound() const override
    {
        return "meow";
    }
};

void print_info(int x)
{
    std::cout << "Object is not an animal and has value " << x << ".\n";
}

void print_info(const animal& a)
{
    std::cout << "Object is an animal and does " << a.sound() << ".\n";
}

int main()
{
    int x = 7;
    cat c;
    print_info(x);
    print_info(c);
}
```

Everything works as expected:

~~~
Object is not an animal and has value 7.
Object is an animal and does meow.
~~~

## unexpected overload

Now, suppose the integer overload has been changed to a function template.

```c++
#include <iostream>

class animal
{
public:
    virtual ~animal() = default;
    virtual std::string sound() const = 0;
};

class cat : public animal
{
public:
    std::string sound() const override
    {
        return "meow";
    }
};

template <typename T>
void print_info(const T& t)
{
    std::cout << "Object is not an animal and has value " << t << ".\n";
}

void print_info(const animal& a)
{
    std::cout << "Object is an animal and does " << a.sound() << ".\n";
}

int main()
{
    int x = 7;
    cat c;
    print_info(x); // expecting template function with [T = int]
    print_info(c); // expecting non-template function
}
```

It seems reasonable that the expected output would be the same.


Instead, we get a compiler error spam starting with something like this (everything further down is about all failed attempts to match operators with various stream << overloads):

```c++
main.cpp: In instantiation of 'void print_info(const T&) [with T = cat]':
main.cpp:34:17:   required from here
main.cpp:21:59: error: no match for 'operator<<' (operand types are 'std::basic_ostream<char>' and 'const cat')
     std::cout << "object is not an animal and has value " << t << '\n';
     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~
```

It seems that overload resolution went troll mode and choose the function template (substituting `T` for `cat`) over a ready function overload which would work on any type derived from `animal`. **Why template overload was choosen when a cat is an animal?**

## type deduction vs overloading

To understand why this happened, we need to dig into language rules.

When a function call is encountered, the following happens:

- the name (here: `print_info`) is looked up in all relevant scopes (the global scope, the current namespace scope and relevant namespace scopes if any argument comes from foreign namespace)
- all overloads are put into a set of all possible overloads
- template type deduction takes place
- the best matching overload is choosen (overloads with less required convertions have higher priority)

Reviewing this process with example code:

- name lookup: all the code here uses global scope and none of included files declare a function with such name
- 2 potential macthes are found:

```c++
template <typename T>
void print_info(const T&);      // (1)

void print_info(const animal&); // (2)
```

- type deduction:
    - (1) function was called with object `c` which is of type `cat`. `T` is deduced to `cat`
    - (2) is not a template - no type deduction takes place
- Overload resolution has 2 candidates to consider:

```c++
void print_info(const cat&);    // (1)
void print_info(const animal&); // (2)
```

Given such functions, it should be obvious that (1) is preferred for cats. The big thing is that overload resolution does not know that (1) came from a template.

## the core cause

**Template type deduction happens before overload resolution.**

Overload resolution never examines templates because it's always presented with concrete functions.

It's an impactful C++ language design decision. If the reverse was true, there would be many problems regarding specializations and overloads. Instead of good matches with (potentially specialized) templates, various types would pretend very weird convertions to fit into non-template overloads potentially losing data or accuracy while being converted. In this scenario, there would be a dilemma:

- use non-template overload which requires implicit upcast (losing information during convertion)
- use template overload which has perfect type match (`T` deduced to `cat`)

To avoid such dilemma language creators decided it would be better to deduce types first and then, attempt to match the argument to any of concrete functions. Because template type deduction is pretty powerful it usually ends in perfectly selected types which are better matches than non-template overloads (which typically require implicit convertions).

For this reason, it's is adviced not to mix function templates with overloads which are expected to match multiple types (eg derived classes). It's better to TODO (gotw?).

## the secret hole

> because template type deduction is pretty powerful it **usually** ends in perfectly selected types

Yes, usually.

**Because template type deduction occurs before overload resolution, it has been allowed to fail.**

What happens when dedution fails? SFINAE. *Substitution failure is not an error* means that if a template type can not be deduced, an error does not occur. The template is simply ignored.

By forcing deduction to fail in certain situations SFINAE is used to effectively disable certain templates.

**Note:** if deduction fails in all templates and there is no remaining overload to choose it's a compilation error. Quite long error because the compiler will output detailed information about each template and why it was thrown out.

## when deduction can fail

SFINAE only takes place during deduction, never after. In the example above compilation error happened because a correctly deduced template was used with incompatible type.

```c++
template <typename T>       // SFINAE can happen here
void print_info(const T& t) // and here
{
    // but not here (the body of the function is never examined)
    std::cout << "Object is not an animal and has value " << t << ".\n";
}
```

Template type deduction only considers function arguments - never it's body. This makes it consistent with overload resolution - only function prototype/signature (TODO choose right) is examined.

## how deduction can fail

A substitution failure occurs when the **type** or **expression** in the function's prototype/signature (TODO choose right) would be ill-formed.

Type SFINAE examples:

- forming an array of
    - `void`
    - references
    - functions (not function pointers)
    - abstract types (not abstract type pointers)
    - negative size or size 0
- attempting to use a member type that does not exist
- attempting to use a member type/non-type/template where a different entity is encountered
- forming a pointer to a reference
- forming a reference to `void`
- attempting to perform a convertion which does not exist
- forming a function which would take object of type `void` as an argument
- forming a function which would take or return object of abstract type by value

Expression SFINAE examples:

- ill-formed expression used in a template parameter type
- ill-formed expression used in the function type

In short, any deduction that would result in creating nonsensical type or expression results in SFINAE.

Code examples:

```c++
template <typename T>
int func(T);              // (1)

template <typename T>
int func(typename T::X*); // (2)

// 0 is of type int
// (1): deduction succeeds: T = int
// (2): there is no int::X*, deduction fails, SFINAE
int i = func(0); // calls (1)
```

```c++
template <typename T>
T& func();              // (1)

template <typename T>
int func(T x = T());    // (2)

template <typename T>
T func(double x = 0.0); // (3)

// (1): SFINAE because there is no void&
// (2): SFINAE because a funcion can not take argument of type void
// (3): void as T succeeds, function is well-formed
func<void>(); // calls (3)
```

```c++
struct X {};
struct Y { Y(X){} }; // X is convertible to Y

template <typename T>
auto f(T t1, T t2) -> decltype(t1 + t2); // (1)

X f(Y, Y);                               // (2)

X x1;
X x2;
// (1): T unanimously deduced to X, but expression (t1 + t2) is ill-formed
//      because there is no operator+(X, X); SFINAE
// (2): requires implicit convertion which lowers this overload
//      priority in resolution but it's the only candidate left
X x3 = f(x1, x2); // calls (2)
```

## how to use SFINAE

Because using SFINAE is rather tricky (I doubt anyone clearly remembers all the cases where it does happen) a convention has been established in the C++ standard library for type and expression SFINAE.

**type SFINAE - enable-if metafunction**

```c++
// possible implementation inside standard library
template <bool B, typename = void>
struct enable_if {};

template <typename T>
struct enable_if<true, T> { using type = T; };
```

How it works: the core idea is to make `enable_if<true>::type` a valid entity and nonexistent `enable_if<false>::type`.

How it is achieved: by default, the structure is empty, which means that there is no member `type` entity. This default definition applies when the specialization below does not match (`B` is not `true`), otherwise specialized definition is used. Since `bool` has only 2 possible values `enable_if` can aswell be implemented in reverse way (`type` in default definition and empty struct specialization for `false`).

**expression SFINAE - void variadic alias**

```c++
template <typename...>
using void_t = void;
```

This very simple construct aliases any `void_t<zero_or_more_types>` to `void`. Of course, as long as `zero_or_more_types` are valid. If not, `void_t` is ill-formed and triggers SFINAE.

We can obtain type of an expression through a keyword `decltype`, so `void_t<decltype(arbitrary_expression)>` can be used for expression SFINAE.

## type traits

Standard library type traits exhibit meta information about any type, for example:

- is it trivial to construct/copy
- is it abstract type
- is it a pointer/reference/array type
- is it const/volatile
- is it lvalue/rvalue reference

They can also be used to transform types:

- add/remove const/volatile/lvalue reference/rvalue reference
- add/remove pointer
- add/remove array dimension

Type traits can be used in type SFINAE, eg `std::enable_if<std::is_pointer<T>::value>::type` enables only if `T` is a pointer type.

Transformation traits can be used in expression SFINAE, eg `std::void_t<std::add_reference<T>::type>` disables if `T` is `void`.

Traits can also be used as building blocks for other traits.

## the solution

We fix the example by throwing away templates which should not be considered. We feed `std::enable_if` with boolean which comes from certain type traits. Traits exhibit member `value` which is `true` or `false` dependig on the provided type. If `enable_if` gets `false`, it's member `type` entity does not exist making the template ill-formed.

```c++
#include <iostream>
#include <type_traits>

class animal
{
public:
    virtual ~animal() = default;
    virtual std::string sound() const = 0;
};

class cat : public animal
{
public:
    std::string sound() const override
    {
        return "meow";
    }
};

template <
    typename T,
    typename = typename std::enable_if<!std::is_base_of<animal, T>::value>::type
>
void print_info(const T& t)      // (1)
{
    std::cout << "Object is not an animal and has value " << t << ".\n";
}

void print_info(const animal& a) // (2)
{
    std::cout << "Object is an animal and does " << a.sound() << ".\n";
}

int main()
{
    int x = 7;
    cat c;
    print_info(x); // (A)
    print_info(c); // (B)
}

// A:
// deduction succeeds:
//     T deduced to int
//     std::is_base_of<int, animal>::value is false (becomes true by !)
//     std::enable_if<true>::type is valid
//     the result of enable-if is assigned to anonymous typename (unused template parameter)
// possible overloads:
//     void print_info(const int&);
//     void print_info(const animal&);
// overload (1) is choosen

// B:
// deduction fails:
//     T deduced to cat
//     std::is_base_of<cat, animal>::value is true (becomes false by !)
//     std::enable_if<false>::type does not exist, SFINAE takes place
// possible overloads:
//     void print_info(const animal&);
// overload (2) is choosen

```

## applications

Is SFINAE used in practice? Yes, mostly in the standard library. It's usually used when a function template has multiple overloads and a way to select one is needed - otherwise all overloads would deduce the same type and end being equal (compilation error due to ambiguous call).

Example - sorting algorithm. Like many standard algorithms:

- simplest overload takes just a pair of iterators (1)
- has an overload letting to specify concurrent execution (2)
- has an overload letting to specify custom comparison (3)
- has an overload letting both custom comparison and concurrent execution (4)

```c++
template <typename RandomIt>
void sort(RandomIt first, RandomIt last);                                       // (1)
template <typename ExecutionPolicy, typename RandomIt>
void sort(ExecutionPolicy policy, RandomIt first, RandomIt last);               // (2)
template <typename RandomIt, typename Compare>
void sort(RandomIt first, RandomIt last, Compare comp);                         // (3)
template <typename ExecutionPolicy, typename RandomIt, typename Compare>
void sort(ExecutionPolicy policy, RandomIt first, RandomIt last, Compare comp); // (4)
```

Everything seems ok, right? Well, the problem is that overloads (2) and (3) are actually the same. Both take 3 arguments.

SFINAE is used on (2) to throw it away if the type of first argument is not an execution policy type.

(2) actually looks like this:

```c++
template <
    typename ExecutionPolicy,
    typename RandomIt
    typename = typename std::enable_if<std::is_execution_policy<
        std::decay<ExecutionPolicy>::type>::value>::type>
void sort(ExecutionPolicy policy, RandomIt first, RandomIt last);
```


## alternatives

Of course, there are better alternatives. **Tag dispatch** relies on the type of first function argument. Instead of complex analysis inside template deduction we simply rely on the type of first argument (which exists only for this purpose).

```c++
struct use_foo {};
struct use_bar {};
// 2 overloads which expect the same amount and types of arguments (except the first)
func(use_foo{}, arguments...);
func(use_bar{}, arguments...);
```

> Wouldn't it be simpler to just use different function name for each overload?

Impossible for constructors since they must use class name.

## the future

SFINAE is surely tricky and quite awkward to use. Concepts are meant to solve this problem by allowing to define any set of requirements using regular code rather than template shenanigans. In short, concept is satisfied if it's code would compile.

Example (before concepts):

```c++
// 1. std::declval<T>() returns object of type T
// 2. decltype checks whether the expression of calling hash with an instance of T would produce a valid type
// 3. std::is_same checks whether that function call would correctly return value of type std::size_t
// 4. std::enable_if will SFINAE if std::is_same<...>::value is false
template <
    typename T,
    typename = typename std::enable_if<std::is_same<std::size_t, decltype(std::hash<T>{}(std::declval<T>()))>::value>::type>
void save(const T&);
// if we fail to call any save() overload with valid T we get long template errors
```

With concepts:

```c++
template <typename T>
concept Hashable = requires(T t) // this code never gets executed
{
    // expression std::hash<T>{}(t) must be valid
    // expression's result must be convertible to std::size_t
    { std::hash<T>{}(t) } -> std::size_t;
};

template <Hashable T> // T constrained by concept
void save(const T&);
// if deduction results in T that does not satisfy Hashable, compilation error happens
```
