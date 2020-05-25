---
layout: article
---

All of the programs below have exactly the same behaviour - they print information about 2 types of animals, but each program is written differently.

This article is to showcase how various programming styles can be. Each has different tradeoffs - mostly around convenience/easyness, extensibility/customizability and performance.

**Note 1**

Do not compare performance and generated assembly from these programs. They contain no user-dependent control flow and are therefore abnormal optimization cases. Also, modern compilers tend to do weird (read: abusive aggressive optimization) things with very short programs.

**Note 2**

None of these styles is a panacea to every problem. It is to showcase different styles of solving similar problems. None of these styles are always appropriate for any problem.

### classic structural

Most commonly seen in C. Very basic and simple, but lacking advanced functionality. Note: vector is not structural style, written only for comparison.

```c++
#include <iostream>
#include <vector>

enum class animal_type { cat, dog };

struct animal
{
    animal_type type;
};

void print_animal(const animal& a)
{
    // commonly written using switch
    if (a.type == animal_type::cat)
        std::cout << "meow\n";
    else if (a.type == animal_type::dog)
        std::cout << "whoof\n";
}

int main()
{
    std::vector<animal> v;
    v.push_back(animal{animal_type::cat});
    v.push_back(animal{animal_type::dog});

    for (animal a : v)
        print_animal(a);
}
```

### classic OOP

Very prominent style in Java. The simplest of OOP styles, but unfortunately the worst in regards to performance (virtual function call overhead). Still, classic OOP has the widest possibilities when it comes to the features.

```c++
#include <iostream>
#include <vector>
#include <memory>

class animal
{
public:
    virtual ~animal() = default;

    virtual void sound() const = 0;
};

class cat : public animal
{
public:
    void sound() const override
    {
        std::cout << "meow\n";
    }
};

class dog : public animal
{
public:
    void sound() const override
    {
        std::cout << "whoof\n";
    }
};

void print_animal(const animal& a)
{
    a.sound(); // virtual call
}

int main()
{
    std::vector<std::unique_ptr<animal>> v;
    v.push_back(std::make_unique<cat>());
    v.push_back(std::make_unique<dog>());

    for (const auto& a : v)
        print_animal(*a);
}
```

### monadic

Style encouraged in Haskell, also part of modern C++ trends. Since C++17 multiple monadic types are present in the standard library. More monads are proposed.

Monadic visitor has 1 core difference to classic OOP approach - here the visitor implements functionality for each type. In classic OOP the base class would provide an interface for each operation.

```c++
#include <variant>
#include <vector>
#include <iostream>

template <typename... Ts>
struct overloaded : Ts... { using Ts::operator()...; };

template <typename... Ts>
overloaded(Ts...) -> overloaded<Ts...>;

struct cat {};
struct dog {};

using animal = std::variant<cat, dog>;

void print_animal(const animal& a)
{
    std::visit(overloaded {
            [](cat c) { std::cout << "meow\n"; },
            [](dog d) { std::cout << "whoof\n"; }
        }, a);
}

int main()
{
    std::vector<animal> animals;
    v.push_back(cat{});
    v.push_back(dog{});

    for (const auto& a : v)
        print_animal(a);
}
```

### metaprogramming

CRTP (Curiously Recurring Template Pattern). Limited to compile-time features (and people skilled in template sorcery) but guaranteed to be zero-overhead. This style is complicated but it does wonders in C++ standard library and other libraries which require maximum performance with great list of compile-time features - such as in Eigen - linear algebra library which draws it's power from expression templates.

```c++
#include <iostream>

template <typename Derived>
class animal
{
private:
    Derived& derived() { return *static_cast<Derived*>(this); }
    const Derived& const_derived() const { return *static_cast<const Derived*>(this); }

public:
    void sound() const
    {
        const_derived().sound();
    }
};

class cat : public animal<cat>
{
public:
    void sound() const { std::cout << "meow\n"; }
};

class dog : public animal<dog>
{
public:
    void sound() const { std::cout << "whoof\n"; }
};

template <typename Derived>
void print_animal(const animal<Derived>& a)
{
    a.sound(); // NOT a virtual call
}

int main()
{
    cat c;
    dog d;
    print_animal(c);
    print_animal(d);
}
```

## "row vs column" problem

All these options have different tradeoffs

approach | classic OOP | functional | CRTP
---------|-------------|------------|-----
implementation | each object has a pointer to its table of functions which implement behaviour | each object has a discriminant value which states how it should be interpreted | compile-time evaluation
overhead per object | 1 pointer for each non-virtual interface | 1 integer | (none)
overhead per use | indirect function call (terrible for cache, can not be inlined) | jump table on integer (can be heavily inlined) | (none)
pros | adding new type is easy | adding new behavior is easy | best performance
cons | adding new behavior is hard | adding new type is hard | input must be compile-time
memory layout | absolutely terrible unless custom allocator is used | very-cache fiendly if subvariants are value types | depends on the underlying type
additional info | ABI stability | strong type safety for behaviour implementers | requires language with metaprogramming features
