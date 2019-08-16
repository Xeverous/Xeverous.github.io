---
layout: article
---

## preface

<div class="note info">
This lesson contains very important theory. Make sure you pass it with no doubts.
</div>

<div class="note info" markdown="block">
Further lessons in this chapter will cover the most important aspects of modern C++:

- **lvalue reference** (`T&`) vs **rvalue reference** (`T&&`)
- **copy constructor/assignment** vs **move constructor/assignment**
- **RAII** idiom
- **smart pointer** classes
</div>

## the purpose

Value categories describe what can be done with them and which expressions they support.

Does any of these make sense to you?

```c++
a + b = c;           // Save c to what? Does it even compile?
const int* p = &123; // Address of a constant? Is this legit?
int* ptr = &(a + b); // Is this a dangling pointer?
```

Of course, none of these expressions are valid. It's obvious that you can not put `a + b` on the left side or take address of a hardcoded number or something that has questionable lifetime.

But how would you explain it?

The answer is: **value categories**. **Every expression is either an lvalue or an rvalue.**

Note: the formal terms are *lvalue expression*, *rvalue expression* and such.

### lvalue (left value)

An **lvalue expression** is an expression that can appear **both on the left and the right** side of an assignment.

Suppose we have integers `a`, `b` and `c`.

All of the following expressions are valid:

```c++
a = b;
b = a;
a = a + 1;
c = a + b;
int* p = &c;
```

Don't overthink it, it's that simple. `a`, `b` and `c` can be on both sides.

Thus, all of `a`, `b` and `c` are **lvalue expressions**.

### rvalue (right value)

An **rvalue expression** is an expression that can appear **only on the right** side of an assignment.

The following expressions are valid:

```c++
a = a + 1;
c = a + b;
int* p = &c;
```

But all of the following are not:

```c++
a + 1 = a;
a + b = c;
int* p = &(a + b);
```

Thus, `a + 1` and `a + b` are **rvalue expression**s. They can not be put on the left.

In other words, rvalues are temporary objects. **You can not assign to temporaries and can not obtain their address.** Temporaries have almost no lifetime - they die instantly as the expression ends.

`1` is also an rvalue expression:

```c++
// both do not compile
1 = a;
const int* p = &1;
```

and so are writes to temporaries returned by functions:

```c++
int func();
func() = 5; // can't assign to a temporary
```

## short summary

- **lvalue expressions**:
  - can appear on both left and right side of an assignment
  - have well-defined scope and lifetime, as they represent variables
- **rvalue expressions**:
  - can appear only on the right side of an assignment
  - are (unnamed) temporary objects, they live only as long as necessary (to the end of statement)
  - can not be assigned to
  - have no obtainable address

### prefix vs postfix

You might already came up with this question: Is `++a` and `a++` an lvalue or an rvalue expression?

Compare what is happening for `a++`:

```c++
int postincrement(int x)
{
    int temp = x;
    x += 1;
    return temp;
}
```

and what for `++a`:

```c++
int& preincrement(int& x)
{
    x += 1;
    return x;
}
```

Expression `a++` does not have it's results visible immediately. `a` is changed but the old value is returned. However, `++a` changes `a` in-place **and returns a reference to an existing object**.

The answer is:

- `++a` is an lvalue expression
- `a++` is an rvalue expression

The following expressions are valid:

```c++
b = ++a; // lvalue = lvalue (saves new a to b)
b = a++; // lvalue = rvalue (saves old a to b)
++a = b; // lvalue = lvalue (increments a, then writes b into it)
```

but the following is not:

```c++
a++ = b; // rvalue = lvalue (increment a but can't save b to temporary old a)
```

and produces such error message when run by GCC:

```c++
main.cpp: In function 'int main()':
main.cpp:9:11: error: lvalue required as left operand of assignment
     a++ = b;
           ^
```

Why it happens:

- `++a` increments `a` and then returns it back (you still get the same object), hence lvalue - it's the same object, just after modification.
- `a++` makes a copy of `a`, increments `a` and then returns the copy - the copy is a temporary object - hence rvalue.

### functions

- preincrement returns a reference back to the original object
- potsincrement returns a temporary (old value)

This drives to the question: How does function's return type impact value category?

- If the function returns element by value, it's an **rvalue**.
- If the function returns element by reference, it's an **lvalue**.

## value categories in C and before C++11

TODO improve, bucket of water example

In C, every expression in an lvalue or an rvalue. There are no features above pointers and structs and this is enough.

In C++, a problem was discovered once RAII started to be widely used - there was no clear, idiomatic way to:

- move the resource from one object to another (instead of a deep copy) - everyone had their own, inconsistent function for it
- indicate that a resource is non-copyable - before C++11 there was no `= default/delete` so the workaround was to make copy ctor and assignment private
- optimize out wasted allocations - pass a pointer from A to B instead of allocating in B, copying A to B and releasing A

Before C++11, the standard had only notion of left and right values. But it was not consistent - multiple contexts used these terms but applied long exceptions, thus making each situation a unique set of rules. It was clear that new terms need to be made to avoid exceptions in rules and provide more consistent writing.

## value categories after C++11

Rvalue has been split to **prvalue** and **xvalue**.

~~~
    value category
          / \
         /   \
        /     \
       /     rvalue
      /        / \
     /        /   \
    /        /     \
   /        /       \
lvalue   xvalue   prvalue
~~~
