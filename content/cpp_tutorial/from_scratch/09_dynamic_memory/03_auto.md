---
layout: article
---

Sometimes you can encounter very long type names or code that repeats them. There is a special feature to deal with it - **auto**matic type.

```c++
auto a = 1;    // int
auto b = 2ul;  // unsigned long
auto c = 3.0f; // float
auto d = 4.0l; // long double
auto c = '3';  // char
```

The type is deduced from initializing expression. You can not ommit it:

```c++
auto val; // error: no expression to deduce from
```

Automatic type deduction does not preserve cv-qualification and references:

```c++
const int x = 10;
const int& y = x;
auto a1 = y;        // int
const auto& a2 = y; // const int&
```

Unless it has to:

```c++
const int x = 10;
const int& y = x;
auto& a = y; // not an error, will add hidden const
```
