---
layout: article
---

A common problem experienced by those who come from other languages is how to acomplish basic tasks using C++ standard library. I can admit that some things from the standard library can be surprising.

## overview

TODO idea - each point a link to respective tutorial?

The C++ standard library (sometimes referred to as the STL, for historial reasons) contains the following facilities:

- OS-related:
  - I/O (`<*stream>`, `<cstdio>`)
  - error handling and program termination (`<cstdlib>`)
  - sending/subscribing to [signals](https://en.wikipedia.org/wiki/Signal_(IPC)) (`<csignal>`)
  - (since C++17) `<filesystem>`
- error handling:
  - C-style assertion (`<cassert>`)
  - C-style error number (`<cerrno>`)
  - C++-style error facilities (`<system_error>`)
  - standard library exception types (`<stdexcept>`)
  - exception utilities (`<exception>`)
  - contract violation info (`<contract>`)
- memory management:
  - lowest-level memory allocation (`<cstdlib>`)
  - low-level memory allocation (`<new>`)
  - high-level memory management (`<memory>`)
  - nested allocators (`<scoped_allocator>`)
  - polymorphic allocators and memory resources (`<memory_resource>`)
- strings:
  - some legacy C cruft (`<cctype>`, `<cwctype>`, `<cwchar>`)
  - C-style low-level string handling such as `strcpy()`, `strlen()`, `memcpy()`, `memset()` (`<cstring>`)
  - null-terminated character buffer class (`<string>`)
  - non-null-terminated character sequence observer (`<string_view>`)
  - (since C++17) string-to-int/float and int/float-to-string converters (`<charconv>`)
  - `<regex>`
- containers (collections, data structures):
  - stack-allocated array (`<array>`)
  - heap-allocated expandable array (`<vector>`)
  - double-ended expandable array (`<dequeue>`)
  - doubly linked list (`<list>`)
  - singly linked list (`<forward_list>`)
  - ordered dictionaries (trees) (`<set>`, `<map>`)
  - unordered dictionaries (hash tables) (`<unordered_set>`, `<unordered_map>`)
  - container adaptors (`<stack>`, `<queue>`)
  - continuous memory view (`<span>`)
- generic algorithms (working on any data structure and memory layout)
  - `<algorithm>`
  - `<iterator>`
  - `<ranges>`
- concurrency:
  - `<atomic>`
  - `<thread>`
  - `<mutex>`
  - `<shared_mutex>`
  - `<future>`
  - `<condition_variable>`
  - parallel execution of algorithms (`<execution>`)
- numeric:
  - C-style macros of integral types limits (`<climits>`)
  - C-style macros of floating-point types limits (`<cfloat>`)
  - C++-style numeric type properties (`<limits>`)
  - fixed-width types such as `std::uint32_t` (`<cstdint>`)
  - common math functions (`<cmath>`)
  - other numeric functions such as GCD, LCM (`<numeric>`)
  - complex number (`<complex>`)
  - array of values (SIMD wrapper) (`<valarray>`)
  - floating-point environment (`<cfenv>`)
  - random number generation (`<random>`)
  - (since C++20) bit manipulation (`<bit>`)
- utility:
  - swap, forward, exchange (`<utility>`)
  - function/lamba/reference wrappers (`<functional>`)
  - pairs and tuples (`<tuple>`)
  - (since C++17) any type holder (`<any>`)
  - (since C++17) optional (`<optional>`)
  - (since C++17) type-safe union (`<variant>`)
- date/time:
  - C-style time (`<ctime>`)
  - C++-style time (`<chrono>`)
  - (since C++20) C++-style date (`<chrono>`)
- meta:
  - compile-time type information (`<type_traits>`)
  - standard types (`std::size_t`, `std::ptrdiff_t`, `std::byte`) (`<cstddef>`)
  - (since C++20) implementation information (`<version>`)

Standard library (neither the C++ itself) does not contain the following features:

- automatic (de)serialization
- networking (use Boost ASIO and Boost Beast instead)
- process-related OS utilities (IPC, invoking other programs)
- GUI, 2D/3D graphics, audio
- command-line parsing (use Boost Program Options instead)
- reading/writing of common file formats like JSON, XML (use dedicated external libraries)
