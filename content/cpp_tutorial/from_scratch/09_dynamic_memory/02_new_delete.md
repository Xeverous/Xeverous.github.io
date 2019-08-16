---
layout: article
---

The first C++ abstractions that make allocation easier are operators `new` and `delete`.

Similarly to C functions, they work in pairs. They do not solve all problems with allocation but make it somewhat more convenient.

## syntax

```c++
// allocate/deallocate an object
T* ptr = new T;
T* ptr = new T(initializer); // (optional initial value)

delete ptr;

// allocate/deallocate an array of objects
T* arr = new T[N];
delete[] arr;
```

## notes

Type `T` can be put in parentheses if it's complex and creates syntax ambiguities.

If there is no initial value provided, object is default-constructed (for integers this means they will be 0). There is no way to provide initializers for an array.

Allocation/deallocation should not be mixed - all operations must be done in relevant pairs:

- `malloc()` - `free()`
- `new` - `delete`
- `new[]` - `delete[]`

Passing pointer from allocation to an incompatible deallocation is undefined behaviour.

## example

TODO test

```c++
#include <iostream>
#include <new> // needed for std::nothrow

int main()
{
    int n;
    std::cout << "How many numbers would you like to type? ";
    std::cin >> n;
    int* ptr = new (std::nothrow) int[n];

    if (ptr == nullptr)
    {
        std::cout << "Error: memory could not be allocated";
    }
    else
    {
        for (int i = 0; i < n; ++i)
        {
            std::cout << "Enter number: ";
            std::cin >> ptr[i];
        }

        std::cout << "You have entered: ";
    
        for (int i = 0; i < n; ++i)
            cout << ptr[i] << ", ";
    
        delete[] ptr;
    }
}
```

## advantages over C functions

- New-expression allocates memory for N objects of given type, not bytes - no need for calculating size.
- New-expression is type-safe - no void pointers, no casting.
- New-expression initializes data - `malloc()` is usually followed by an assignment; there is `calloc()` but it offers only zero-initialization while new-expression allows any initial value.
- New-expression is an operator and can be overloaded - this allows custom allocation for specific custom types
- Automatic failure propagation: if allocation fails, instead of returning a null pointer an exception is thrown. When desired, this can be disabled by no-throw argument:

```c++
#include <new> // std::nothrow
new (std::nothrow) T
new (std::nothrow) T(initializer)
new (std::nothrow) T[N]
```

- `delete` and `delete[]` doesn't require explicit null pointer checks

```c++
if (ptr != nullptr) // redundant, delete already checks it
    delete[] ptr;   // calls deallocation only if ptr is non-null
```

## problems with new

New expression solves these problems:

- initialization of allocated memory
- type safety

New-expression still shares some problems with C allocation functions:

- it's easy to forget to detele (memory leak)
- it's easy to delete twice (undefined behaviour)
- `delete[]` can be mistakenly written as `delete` causing undefined behaviour
