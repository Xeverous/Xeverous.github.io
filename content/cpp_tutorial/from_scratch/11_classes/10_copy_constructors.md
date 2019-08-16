---
layout: article
---

## copy constructor

There is a special constructor overload which takes another object of it's type as parameter.

```c++
class point
{
public:
    // always present unless you write your own with = delete or custom body
    point(const point& p) = default; // copy ctor

private:
    int x = 0;
    int y = 0;
};
```

If you don't explicitly remove copy constructor, it will be automatically added to the class, like in the example above.

By default, copy ctor copies value of each member in the order they are declared.

## example

Copy constructors are called every time an object of the same type is created from another object. Copy constructors are called only at initialization.

```c++
#include <iostream>

class point
{
public:
    point(int x, int y);
    void print() const;

    // we rely on automatically added copy ctor here

private:
    int x = 0;
    int y = 0;
};

point::point(int x, int y)
: x(x), y(y)
{
}

void point::print() const
{
    std::cout << "(" << x << ", " << y << ")\n";
}

int main()
{
    point p1(2, 3);
    p1.print();

    point p2(p1); // calls copy ctor
    p2.print();

    point p3 = p2; // calls copy ctor
    p3.print();

    point p4(4, 5);
    // p2 = p4; // error: no match for operator=(point, point)
}
```

## initialization vs assignment

In the example above first line using `=` compiles but second does not.

This is because first line is an initialization but second is an assignment. Initialization happens only upon object creation - the second line does not create an object.

Initialization in C++ has a variety of syntax options (including `=`) and regardless of the choosen one, it will call one of available constructors or directly initialize some members.

In order to make second line work, we need to overload `operator=`. This is covered in next chapter.

## custom copy constructor

Custom copy constructors are written just like any other constructors. The only difference is that they must take a const reference to another object of the same type as the only parameter.

```c++
// inside class body
point(const point& other); // no = default/delete means compiler will expect custom definition

// outside
point::point(const point& other)
: x(other.x), y(other.y)
{
    std::cout << "copy ctor called\n";
}
```

## copy constructors vs pointers

The are some problems when a class contains pointers:

- What if some pointers are dangling?
- What should happen to allocated memory?
- Should copy constructor copy pointers or the contents pointed by them? 
- What if we would like to swap resources between 2 objects?

These will be covered later, in RAII chapter. Right now watch out or better not write any classes containing pointers. There are some hard-to-spot sources of UB for beginners but they are easily dealt with once RAII idiom is understood.
