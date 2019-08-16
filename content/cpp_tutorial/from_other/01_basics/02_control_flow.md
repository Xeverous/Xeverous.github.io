---
layout: article
---

# control flow

## jumps

Classical ever-hated goto. Very strongly discouraged. C++ forbids it in few contexts.

```c++
	code();
label:
	code();
	if (x)
		goto label;
```

Obviously, don't use it.

## conditions - switch

Due to backwards compatibility switch in C++ has few restrictions:

- cases can only test equality
- value for any case must be a constant expression

Cases fallthrough by default. Use breaks to stop it.

```c++
switch (x)
{
case 1:
	// ...
	break;
case 2:
	// ...
	// fallthrough, may generate a warning
case 3:
	// ...
	break;
default:
	// ...
	break;
}
```

Braces are required if cases enter variable's scope ommiting initialization.

```c++
switch (x)
{
    case 1:
        int x = 0; // initialization
        break;
    default:
    // {
        // compilation error: jump to default would enter the scope of 'x'
        // without initializing it
        break;
    // }
}
```

In case of doubt (pun intended) add braces around every case.

Default case is not required. Compilers will generate a warning when an defaultless enumeration switch does not cover all possible values.

## conditions - if

Braces not required for 1-statement bodies.

```c++
if (condition)
	statement();
else
	statement();
```

Chaining in the form of `else if` keyword pair is possible due to lack of brace requirement.

```c++
if (cond1)
	statement();
else if (cond2) // else { if ... } not required
	statement();
else if (cond2)
	statement();
else
	statement();
```

Since C++17 you can put an additional statement before actual condition. This can be useful to create an object only for if-else scope.

```c++
if (std::string str = func(); !str.empty()) {
	statement();
}
else {
	statement(str); // str also lives here
}
// but not here
```

This is also possible for switches.

## while

As with others, braces not required.

```c++
while (condition)
	loop_body();

while (statement; condition) // since C++20
	loop_body();
```

```c++
do
	loop_body();
while (condition); // <-- watch out, this semicolon is often forgotten
```

The only difference between `while` and `do ... while` loop is that `do ... while` loops execute condition after loop body, which means that even if condition fails all the time the loop will still be executed once.

TODO border prefer while to do-while loops

## for

Classical for loop:

```c++
// on ++i (preincrement) vs i++ (postincrement) later
for (int i = 0; i < arr.size(); ++i) {
	// use arr[i]
}
```

Iterator-based loop:

```c++
for (auto it = arr.begin(); it != arr.end(); ++it) {
	// use  it to access iterator
	// use *it to access element
}

// more on iterators in STL chapter
```

For-each loops:

```c++
std::vector<T> v = /* ... */;
// copy each element - obj modifications in the loop will not have visible effects
for (T obj : v)
// const (read-only) reference
for (const T& obj : v)
// non-const reference - when modification is intended
for (T& obj : v)
// rvalue reference - when resource ownership (lifetime) changes are intended
for (T&& obj : v)

// more on references in type system chapter
```
