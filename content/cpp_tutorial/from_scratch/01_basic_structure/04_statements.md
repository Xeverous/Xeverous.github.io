---
layout: article
---


Every C++ (and C) program consists of **statements** and **functions** (note: if you heard the term *class* already it can be seen as a definition statement). Statements consist of **expressions**.

Example expressions:

```c++
1 + 2
3 * 4
5 % 6
[](){}
struct s
foo::bar
3.14 * (r * r)
y = f(x)
(x - a) + (y - b)
arr[k] = v
std::cout << "hello, world"
```

Statements are analogy to human language sentences and expressions are analogy to words and punctuation.

```c++
int x = 5; // this code is a statement
// "x = 5" is an expression, just like "a + b" 
```

**Every statement ends with `;`.** Statements may be empty or contain multiple expressions.

```c++
; // This is a valid empty statement. This instruction does nothing (may trigger a compiler warning because it's useless)
int i = x + y + z; // this statement has multiple (sub)expressions
f(a, b, c, d); // this statement uses , to separate arguments
```

It's possible to place multiple statements on one line

```c++
int a = 1; int b = 2; int c = 3;
```

However, such style is not recommended for readability reasons. **Write at most 1 statement per line and separate different groups:**

```c++
int a = 1;
int b = 2;
int c = 3;

double x = 1.0;
double y = -1.0;
double z = 1.5;

std::string str = "Hello, world!";
```

Whitespace characters (spaces, tabs, new lines, etc) are ignored by the compiler. This means that `x+y` works the same as `x + y` or `x   +y`. Obviously whitespace separates names so `ab` will be one name "ab" while `a b` will represent 2 names: "a" and "b".

Whitespace characters are not ignored in quoted strings such as `" "` or `' '`.
