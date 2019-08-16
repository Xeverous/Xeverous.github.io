---
layout: article
---

Prerequisities: const member functions, noexcept, perfect forwarding

Each of C++ standard library containers (collections, data structures) has a separate header.

All containers:

- are class templates
- can work with any data type (any restrictions are propagated, eg if a type is not copyable the container of it is also not copyable)
- automatically manage memory
- share more or less common API (`size()`, `capacity()`, `operator[]`, `clear()`)

Each container stores elements differently (different memory layout and allocation strategy) which impacts the complexity of operations such as insertion, erasure and lookup.

All forms of indexed access in C++ use 0-based numering. That is, for any container of `N` elements valid indexes are `0...N-1`.

## stack-allocated array

https://en.cppreference.com/w/cpp/container/array

```c++
#include <array>

namespace std {

template <typename T, std::size_t N>
class array;

}
```

A zero-overhead wrapper around plain C array - `std::array<T, N> arr` wraps `T arr[N]` and offers the following advantages:

- is a type-safe class, which unlike C arrays does not decay to pointers losing important type information
- has `size()` which removes the need of manual, bug-prone size computation through `sizeof`
- supports for-each loops
- supports iterators and algorithms
- unlike C arrays, it can be copied/moved (has copy/move ctors)

`std::array` is the only container which stores its elements on the stack. In other words, stored elements are a part of array object representation. `std::array` never allocates any memory.

Because the required stack space is computed at compilation time, the array is not expandable and the size must be a compile time constant.

Default-constructed array will attempt to default construct all `N` elements.

- If you want to only construct specific elements, you can use `std::array<std::optional<T>, N>`
- If you want 0 elements by default and be able to insert up to `N` elements, use `boost::container::static_vector<T, N>`

Example usage:

```c++
#include <array>
#include <utility>
#include <iostream>

int main()
{
    std::array<int, 3> arr = { 1, 2, 3 };
    arr[1] *= 2;
    std::swap(arr.front(), arr.back());

    for (int x : arr)
        std::cout << x << ' '; // 3 4 1
}
```

### common problems

- `std::array` wraps underlying C array. The constructor is defined implicitly, which performs aggregate initialization. For classes, this will call their default constructors but for built-in types, they will not be initialized.

```c++
std::array<int, 10> arr;      // bad: elements not initialized
std::array<int, 10> arr = {}; // ok: all elements 0

std::array<my_class, 10> arr; // ok: will call 10 default constructors
```

- The aggregate initialization of array is used to initialize underlying C array (this is the only container which has such initialization). If you get a compiler error (or warning in C++14 or above), just add another layer of braces

```c++
// main.cpp:7:32: warning: suggest braces around initialization of subobject [-Wmissing-braces]
std::array<int, 4> arr = { 1, 2, 3, 4 };
//                         ^~~~~~~~~~
//                         {         }

// ok
std::array<int, 4> arr = {{ 1, 2, 3, 4 }};
```

## vector (expandable array)

Your every day go to dynamic array class in C++. Basically the thing you would use as an equivalent of indexed arrays in any higher-level language. One of the most often used C++ standard library types.

Vector is the simplest implementation of a dynamic buffer. Can store up to some number of elements, after which expansion will happen - a larger buffer will be reserved and elements will be moved to the new buffer. Elements are always stored continuously in memory.

API overview (simplified):

```c++
#include <vector>

namespace std {

template <typename T, typename Allocator = std::allocator<T>>
class vector
{
public:
    // default ctor, vector is empty, no memory is reserved
    vector();
    // construct vector with n default-constructed elements
    explicit vector(size_t n);
    // construct vector with n copies of 'val'
    explicit vector(size_t n, const T& val);
    // construct vector from a read-only sequence
    template <typename InputIterator>
    vector(InputIterator first, InputIterator last);
    // construct vector from init list
    vector(initializer_list<T> list);
    // create a deep copy of other vector (copy ctor)
    vector(const vector& other);
    // create a deep copy of other vector (copy assignment)
    vector& operator=(const vector& other);
    // assign list of elements to vector
    vector& operator=(initializer_list<T> list);
    // steal elements (and underlying buffer) from other vector (move ctor)
    vector(vector&& other) noexcept;
    // steal elements (and underlying buffer) from other vector (move assignment)
    vector& operator=(vector&& other) noexcept;
    // destructor - releases any owned memory
    ~vector();

    // check if vector is empty
    bool empty() const noexcept;

    // get number of currently stored elements
    size_t size() const noexcept;

    // get number of available space
    // (insertion up to this amount will be O(1) and not cause relocation)
    // capacity() is always >= size()
    size_t capacity() const noexcept;

    // reserve enough memory for n elements
    // if n <= capacity(), nothing happens
    // otherwise reallocation happens (elements are move-constructed) and all iterators are invalidated
    // in any case, size() does not change
    void reserve(size_t n);

    // remove all elements (will call destructor of each)
    // size() will be 0, capacity() will not change
    void clear() noexcept;

    // access first / last element
    // if vector is empty the behavior is undefined!
          T& front();
    const T& front() const;
          T& back();
    const T& back() const;

    // access nth element
          T& operator[](size_t n);
    const T& operator[](size_t n) const;

    // get a pointer to underlying buffer
    // if vector is empty, the pointer is not guuaranteed to be null
          T* data()       noexcept;
    const T* data() const noexcept;

    // TODO insertion/erasure
};

}
```

## common mistakes

- writing `if (c.size() == 0)` instead of `if (c.empty())`
- passing containers to functions by value instead of by const reference (this causes very expensive copies)
- using container of base class for storing objects of derived class (object slicing will happen)
