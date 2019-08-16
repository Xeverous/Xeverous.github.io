---
layout: article
---

TODO prerequisities - static cast, dynamic cast, auto (type interference)

The title of this article is not actually a C++ keyword. **There is no `typeof` keyword in C++**, mostly for historical (backwards compatibility) reasons.

There are 2 keywords which have typeof-related functionality:

- `typeid`, which is the actual C++ "typeof" keyword which queries the type of an object at runtime
- `decltype`, which queries the type of an object at compile time

## `typeid`

Syntax:

- `typeid(type)`
- `typeid(expression)`

Requirements:

- The header `<typeinfo>` must be included before any use of the keyword, otherwise the program is ill-formed.
- The type must be a complete type.

Behaviour:

- If `typeid` is applied to a reference type, the result is as if it was applied to the referenced type (that is, `typeid(T&) == typeid(T)`).
- Top-level cv-qualifiers are ignored (that is, `typeid(const T) == typeid(T)`)
- If `typeid` is applied to an object non-polymorphic type (a class that does not declare or inherit virtual functions), the type information is resolved at compile time.
- If `typeid` is applied to a polymorphic type:
  - the evaluation incurs a runtime overhead (virtual table lookup)
  - if the expression is an unary operator `*` applied to a pointer and the pointer is a null pointer, an exception of `std::bad_typeid` or an implementation-defined derived type is thrown

Result:

lvalue reference to `const std::type_info` (or any other, implementation-defined type derived from it). The address of the returned object is not guuaranteed to be always the same, but the object will outlive the program.

```c++
namespace std {
    class type_info
    {
    public:
        type_info() = delete;            // (type info can not be constructed)
        type_info& operator=() = delete; // (type info can not be assigned)
        virtual ~type_info();

        // use these to compare types at runtime
        bool operator==(const type_info&) const;
        bool operator!=(const type_info&) const;

        /*
         * Returns true if the type of this type_info precedes the type of other in the
         * implementation's collation order. No guarantees are given; in particular,
         * the collation order can change between the invocations of the same program.
         */
        bool before(const type_info& other) const;

        /*
         * Returns an unspecified value such that for all type_info objects
         * referring to the same type, their hash_code() is the same.
         *
         * No other guarantees are given: type_info objects referring to different
         * types may have the same hash_code (although the standard recommends that
         * implementations avoid this as much as possible), and hash_code for
         * the same type can change between invocations of the same program.
         */
        size_t hash_code() const;

        /*
         * Returns an implementation defined null-terminated character string
         * containing the name of the type. No guarantees are given; in particular,
         * the returned string can be identical for several types and change between
         * invocations of the same program.
         */
        const char* name() const;
    };
}
```

Runtime type information (RTTI) is required for `typeid` to work on polymorphic types. For some compilers, RTTI can be disabled or is not available on particular platform.

### comparing types

While hash code is guuaanteed to be the same for the same types, it can also be the same for other types. The only proper way to compare types is to use operators.

```c++
void f(const animal& a)
{
    if (typeid(a) == typeid(cat))
    {
        const auto& c = static_cast<const cat&>(a); // safe
    }
}
```

Note that this comparison does not work like `instanceof` in Java. It will be true only for instances of exactly `cat` and result as `false` for both parent types and `cat`-derived types. If you want the `instanceof` behaviour (checking whether type at runtime is a specific type or a type derived from it), use `dynamic_cast`. TODO link

### printing type name

There is no simple way in C++ to print the actual type name - C++ does not require any type metadata so `std::type_info::name()` is implementation-defined. Some implementations (GCC, Clang) reuse names from symbol tables as if the type was loaded from a shared library on the given platform.

GCC offers to demangle the name through their ABI functions (see example below), you can also use [`boost::core::demangle`](http://www.boost.org/doc/libs/release/libs/core/doc/html/core/demangle.html).

The lifetime of the string returned by `std::type_info::name()` is unspecified but in practice it is the lifetime of the RTTI which either lasts the entire program or the (dynamically loaded) shared library from which the type comes.

```c++
#include <vector>
#include <string>
#include <iostream>

#ifdef __GNUC__
    #include <cxxabi.h>
#endif

int main()
{
    std::cout << "name of std::vector<std::string>:\n" << typeid(std::vector<std::string>).name() << "\n";

    #ifdef __GNUC__
        int status = 0;
        char* demangled_name = abi::__cxa_demangle(typeid(std::vector<std::string>).name(), nullptr, nullptr, &status);
        if (status == 0) {
            std::cout << "demangled name:\n" << demangled_name << "\n";
            std::free(demangled_name);
        }
    #endif
}
```

```
name of std::vector<std::string>:
St6vectorINSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEESaIS5_EE
demangled name:
std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >
```

### ordering

The `before` member function has very few guuarantees but they are enough to implement an ordering, eg for associative containers.

There is a wrapper class that does exactly this:

```c++
namespace std {
    class type_index
    {
    public:
        // construct an instance of type_index from type information
        type_index(const type_info&);

        size_t hash_code() const noexcept;
        const char* name() const noexcept;

        // implemented using underlying type_info::operator==
        bool operator==(const type_index&) const noexcept;
        bool operator!=(const type_index&) const noexcept;

        // implemented using underlying type_info::before
        bool operator< (const type_index&) const noexcept;
        bool operator<=(const type_index&) const noexcept;
        bool operator> (const type_index&) const noexcept;
        bool operator>=(const type_index&) const noexcept;
    };

    // hash support for unordered containers
    template <>
    struct hash<type_index>
    {
        size_t operator()(const type_index&) const noexcept;
    };
}
```

Example usage:

```c++
#include <iostream>
#include <typeinfo>
#include <typeindex>
#include <unordered_map>
#include <string>
#include <memory>

struct A {
    virtual ~A() = default;
};

struct B : A {};
struct C : A {};

int main()
{
    std::unordered_map<std::type_index, std::string> type_names;

    type_names[std::type_index(typeid(int))] = "int";
    type_names[std::type_index(typeid(double))] = "double";
    type_names[std::type_index(typeid(A))] = "A";
    type_names[std::type_index(typeid(B))] = "B";
    type_names[std::type_index(typeid(C))] = "C";

    int i;
    double d;
    A a;

    // note that we are storing pointer to type A
    std::unique_ptr<A> b = std::make_unique<B>();
    std::unique_ptr<A> c = std::make_unique<C>();

    std::cout << "i is " << type_names[std::type_index(typeid(i))] << '\n';
    std::cout << "d is " << type_names[std::type_index(typeid(d))] << '\n';
    std::cout << "a is " << type_names[std::type_index(typeid(a))] << '\n';
    std::cout << "b is " << type_names[std::type_index(typeid(*b))] << '\n';
    std::cout << "c is " << type_names[std::type_index(typeid(*c))] << '\n';
}
```

```
i is int
d is double
a is A
b is B
c is C
```

## `decltype`

In contast to `typeid`, this keyword queries the compile-time type information. **The result of the expression is not an object but a type.**

- if supplied with an **unparenthesized identifier**, returns the type of that identifier
  - if the identifier matches a function with multiple overloads, the program is ill-formed
- otherwise:
  - if the value category of supplied expression is **prvalue**, yields `T`
  - if the value category of supplied expression is **lvalue**, yields `T&`
  - if the value category of supplied expression is **xvalue**, yields `T&&`

The expression does not need to have a complete type - only enough to resolve subexpressions in unevaluated context. Thus for `decltype(f(g(), h()))`, `g` and `h` must have a complete type, but `f` need not.

`decltype` keyword is extremely sensitive to top-level parentheses, which decide whether the code inside it is treated as an identifier or an expression:

```c++
int x;
decltype(x) y;     // decltype(x)   = int
decltype((x)) ref; // decltype((x)) = int&
```

`decltype` is useful when declaring types that are difficult or impossible to declare using standard notation, like lambda-related types or types that depend on template parameters.

```c++
template <typename T>
void f(const T& t)
{
    // create a type alias that is of the same type as T::foo
    using U = delctype(t.foo); // can also be: decltype(T::foo)

    auto lambda = [](int n) -> int
    {
        return n * sizeof(U);
    };

    // copy lambda instance
    // since each lambda has unique, unnamed type you need to use aliases
    decltype(lambda) l2 = lambda; // can also be: auto l2 = lambda;
}
```
