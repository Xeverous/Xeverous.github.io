
- a **prvalue** if the function return type is an **object** (`T`)
- an **lvalue** if the function return type is an **lvalue reference** (`T&`)
- an **xvalue** if the function return type is an **rvalue reference** (`T&&`)


## value categories since C++11



![value types in C++](https://i.stack.imgur.com/GNhBF.png)

Now, instead of 2 we have 3 mutually exclusive groups: **lvalues**, **xvalues** and **prvalues**.

## prvalue

A **prvalue expression** is an expression that denotes a temporary or initializes an object. Expressions returning `void` are also prvalues. prvalues represent middle-computation values that do not have any defined origin - **prvalues live only to the end of expression and do not have any connection to where they came from**.

It's the ultimate form of a temporary.

example prvalues (pure right values):

- non-string literals (so called "hardcoded values"): `1337`, `true`, `nullptr`, `3.14`
- funcion calls where the function returns by value (`T`) - this rule also applies to overloaded operators, built-in operators and comparisons: `a + b`, `a > b`, `a || b`, `a % b`, `a << b`, `!a`
- the address-of operator: `&a` and the this pointer: `this` (we can say these expressions return addresses by value)
- an enum (value of an integer type) - just like hardcoded values
- member of object: `a.m` if { `m` is an enum or non-static member function } or { `a` in an rvalue and `m` is a non-static data member of non-reference type }
- member of pointer: `p->m` if `m` is a member enumerator or a non-static member function
- pointer to member of object: `a.*mp` if `mp` is a pointer to member function
- pointer to member of pointer: `p->*mp` if `mp` is a pointer to member function
- a cast to non-reference type: `static_cast<T>(expr)` (can can be considered a function which returns by value)
- a **lambda expression**
- (since C++20) a requires expression (template stuff)
- (since C++20) a specialization of a concept (template stuff)

## lvalue

An **lvalue expression** is an expression that denotes an accessible object, bit field or a function. **Every name is an lvalue.**

example lvalues (left values):

- name of a variable/constant/function: `a`, `func`, `std::cout`
- assignment expression: `a = b`, `a += 1` (it returns reference to left operand)
- preincrement/predecrement operation: `++a`, `--b`
- function calls where the function return type is an lvalue reference (`T&`) - this rule also applies to overloaded operators
- pointer dereference: `*p`
- member of object: `a.m` if { `a` is an lvalue and `m` is not a member enum or a non-static member function } or { `a` is an rvalue and `m` is a non-static data member of non-reference type }
- member of pointer: `p->m` if `m` is not a member enum or a non-static member function
- pointer to member of object: `a.*mp` if `a` is an lvalue and `mp` is a pointer to data member
- pointer to member of pointer: `p->*mp` if `mp` is a pointer to data member
- array subscript: `a[n]` if `a` is an lvalue
- comma expression: `foo, bar` where `bar` is an lvalue (comma returs last thing)
- a string literal: `"hello world"`, `L"wide string"` (these return references to arrays)
- casts to lvalue reference types: `static_cast<T&>(expr)`
- casts to rvalue reference to function type: `static_cast<void (&&)(int)>(x)`

## xvalue

An **xvalue expression** in an expression that denotes a **unnamed temporary object that has connection to where it has came from**. xvalues denote an object (which has some lifetime) or bit field which resources can be reused.

example xvalues (expiring values):

- casts to rvalue reference: `static_cast<T&&>(expr)`
- funcion calls where the function returns by rvalue reference (`T&&`) - this rule also applies to overloaded operators
- array subscript: `a[n]` if `a` is an array rvalue
- member of object: `a.m` if `a` is an rvalue and `m` is a non-static data member of non-reference type
- pointer to member of object: `a.*mp` if `a` is an rvalue and `mp` is a pointer to data member
- (since C++17) all expressions designating temporary objects after temporary materialization: `X().n`

## about `a ? b : c`

Now, [this is complicated](http://en.cppreference.com/w/cpp/language/operator_other#Conditional_operator). Generally it will return lvalue if both `b` and `c` are lvalues and a prvalue if both `b` and `c` are prvalues. However, there are multiple possibilities for implicit convertions and the fact that you can do things like `str = x ? "ok" : throw std::logic_error()` where `"ok"` returns a literal and second expression technically returns `void` (but actually throws) makes it even more complicated. Especailly when we combine it with bit fields. Convertions for both operands must be decided at compile time as the value category is a specification formed while parsing code, not a runtime decision.

Basically avoid this ternary operator in unclear contexts. It's a pile of C backwards compability layers and multiple implicit convertion rules. If you are going to conditionally use copy/move constructor better wrap everything in standard `if`s and use explicit casts.

## properties

**glvalue** (general left value) (an object)

- can be polymorphic
- type can be incomplete (only declared) if the full definition is not needed in the expression
- may fall into implicit convertions (which result in prvalues, as the convertions copy-construct the object)

**rvalue** (right value) (a temporary)

- address of can not be taken
- can not be assigned to
- can be used to initialize an object, a const lvalue reference or an rvalue reference

**lvalue** (left value) (an object with owner)

- (all that glvalue has)
- has well defined scope (lifetime)
- can be assigned to
- address of can be taken

**prvalue** (pure right value) (a pure temporary)

- (all that rvalue has)
- has no scope - dies instantly at nearest `;` if not used
- has no connection to where it came from

**xvalue** (expiring value) (an object that just lost it's owner)

- (all that glvalue has)
- (all that rvalue has)


### other notes

- expressions (functions, casts, hardcods, enums, operators, addresses) that return non-reference (by value) are **prvalues**
- every name (as a sole expression) is an **lvalue**, additionally expressions returning references to objets are also **lvalues**
- every named reference is an **lvalue**, since whatever has been bound to it has now a name and well-defined scope
- function name is an lvalue (`func`) but value type of function call expression (`func()`) depends on it's return type
- value type of comma expression `a, b` is the value type of `b`
- expressions returning rvalue reference are **xvalues**

## tl;dr and what you need to remember

<div class="note info">
This is important. These rules will be repeated in furter lessons with examples.
</div>

![value types in C++](https://i.stack.imgur.com/GNhBF.png)

Shortest value types descriptions ("can be reused" == "can be **moved from**")

- **prvalue** - just the pure resource (value) (can be reused), has no lifetime
- **lvalue** - some identity with an associated resource, has a well-defined lifetime
- **xvalue** - some identity with expired resource (can be reused)

**move semantics**

**xvalues** are the values which originally were **lvalues** but lost original owner. Now their resources can be reused (or stolen).

**Binding rules**

- non-const lvalue references (`T&`) accept only **lvalue**s
- rvalue references (`T&&`)  accept only **rvalue**s
- const lvalue references (`const T&`) accept everything

In terms of the code, almost all expressions will be either **lvalue** (objects) or **prvalues** (pure temporaries). Expressions are never **xvalues** unless applied specific convertions.

TODO bucket of water example? Move this to next lesson?
