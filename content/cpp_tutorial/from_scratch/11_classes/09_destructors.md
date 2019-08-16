---
layout: article
---

Ok, you probably surmise what destructors do. They are called when an object is destroyed (goes out of scope).

Destructors are written like constructors, but the class name is prepended with `~`.

```c++
class foo
{
public:
    foo();  // ctor
    ~foo(); // dtor
};
```

One thing comes to mind - what would be a possible use of parameters in destructors? How would I call a destructor on an expiring object?

The answer is simple: you can't. **Destructors are always 0-argument and can not be overloaded.** So every class always has exactly 1 destructor (implicit or custom one) and you can at most customize it's body.

## rules

Most rules for destructors are the same as for constructors:

- Like constructors, destructors have a fixed name (`~class_name`).
- Like constructors, destructors do not have a return type - not even `void`.
- Like constructors, destructors can have an early `return` statement like in a function returning `void`.
- Like constructors, if you don't write any destructor a default one is implicitly added to the class (it's public and does nothing).
- Like constructors, destructors are affected by access specifiers.
- Like constructors, destructors can be made `= default` and `= delete`.
- Unlike constructors, destructors can not be overloaded and always take 0 arguments.
- Unlike constructors, destructors are not inevitable - but the ways to avoid destruction are not that trivial (note: destruction is always wanted, in rare cases you just have to do it manually)

1 rule requires knowledge from further chapters (not explained in this lesson)

- Unlike constructors, destructors can be `virtual`.

## examples

There is not that much to present since most rules apply in the same fashion.

## syntax

Names for constructors and destructors are fixed.

```c++
class foo
{
public:
    foo();  // ctor
    ~foo(); // dtor
};

foo::foo() // ctor
{
}

foo::~foo() // dtor
{
}
```

## inevitability

Destructors for stack-allocated objects are inevitable.

```c++
{
    foo f; // public 0-argument ctor required

    // some code
    // ...

} // public dtor required
```

For heap-allocated objects - they are called when you `delete` the pointer that points to them.

For other types of allocation - well, it's complicated. Nothing you should be worried now though.

As you see, for some cases it's possible to avoid (read: forget) destruction. You will soon learn about standard library classes that help with this problem.

## purpose of destructors

Usually custom destructors are not needed - we do not care with member objects that die - there is no point in resetting them to zero or something similar because the memory will be overwritten by another program.

**But what if memory is dynamically allocated?** Destructors are then the best way to always enforce cleanup!

Look how useful destructors can become when dealing with dynamic allocation:

```c++
class dynamic_array
{
public:
    dynamic_array() : data(nullptr), size(0) {}
    dynamic_array(int size);
    ~dynamic_array();

    // other methods...

private:
    int* data;
    int size;
};

dynamic_array::dynamic_array(int size)
: data(new int[size]), size(size)
{
}

dynamic_array::~dynamic_array()
{
    // remainder: delete checks for non-null pointers
    // you don't need to write if (data != nullptr)
    delete[] data;
}
```

I used const pointer and const size as it does not change through the lifetime of objects of this class. This also forces us to correctly use member initializer list - and as you see, you can put arbitrary expressions there like `new int[size]` (don't forget about memberwise initialization order).

In this simple class ctor + dtor form a pair that is hard to break through - if someone creates a local object of type `dynamic_array`, it's guaranteed to execute both ctor and dtor and therefore both allocation and deallocation of memory.

Thanks to such class, dynamic allocation works like stack-allocated objects:

```c++
{
    dynamic_array da(1000); // ctor called, allocates memory for 1000 integers

    // ...da is used...

} // dtor called here, releases memory
```

Note that destructor calls are not written - you don't write `da.~dynamic_array();` or something similar - it's done automatically. It's actually possible to do this but the current level of this chapter is far behind situations in which you would need to manually call destructors.

## resource management

The class presented above encapsulates a resource (allocated memory) which prevents from resource leaks. This specific usage of ctor + dtor pair is a part of RAII idiom. More about RAII in future chapters.

## going further

We can now write a class that safely manages dynamic memory. By adding more public functions, we can extend it's functionality while preserving invariants. With the help of operator overloading, we can even make `da[i]` work just like it does with plain pointers.

There is a similar class in the standard library - `std::vector` (it's actually a class template) but it has much greater functionality. If you can already understand it's documentation and use it (even partially) - that's awesome.

There is a vector tutorial later. Now, continue with learning OOP stuff. A lof of further lessons showcase features which are used to write classes like vector.
