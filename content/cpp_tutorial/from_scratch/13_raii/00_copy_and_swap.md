---
layout: article
---

Let's recall a dynamic array once again. There is an importan thing to be shown.

```c++
#include <iostream>

class dynamic_array
{
public:
    dynamic_array() : data(nullptr), size(0) {}
    dynamic_array(int size);
    ~dynamic_array();

    int  operator[](int pos) const { return data[pos]; }
    int& operator[](int pos)       { return data[pos]; }

    int length() const { return size; }
    void print() const;

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

void dynamic_array::print() const
{
    for (int i = 0; i < length(); ++i)
        std::cout << "[" << i << "] = " << data[i] << "\n";
}

int main()
{
    dynamic_array a1(10);
    for (int i = 0; i < a1.length(); ++i)
        a1[i] = i * 2;

    std::cout << "first array\n";
    a1.print();

    dynamic_array a2 = a1;
    std::cout << "second array\n";
    a2.print();
}
```

Looks like another simple program calling constructors, but upon running there is a surprise:

~~~
$ ./program
first array
[0] = 0
[1] = 2
[2] = 4
[3] = 6
[4] = 8
[5] = 10
[6] = 12
[7] = 14
[8] = 16
[9] = 18
second array
[0] = 0
[1] = 2
[2] = 4
[3] = 6
[4] = 8
[5] = 10
[6] = 12
[7] = 14
[8] = 16
[9] = 18
Segmentation fault (core dumped)
~~~

**The program has crashed.** The error message is different on each system, on Windows most common is "program has stopped working" popup, on Unix systems memory segmentation errors or SIGSEGV signal.

So what has gone wrong? The program successfully printed contents of both arrays but something bad happened later.

The cause might not be clear at the first sight, so lets outline what actually happens when the program is run:

- first array is created
  - memory is allocated by the constructor
- first array is modified
- first array is printed
- second array is copy-created from first array
  - `data` is copied to second array
  - `size` is copied to second array
- second array is printed
- end of scope, run destructors (in reverse order)
  - destructor of second array frees memory pointed by `data`
  - destructor of first array frees memory pointed by `data` - **the same memory is freed again**

The cause of the problem lies here:

```c++
    dynamic_array a2 = a1;
```

So in short, both arrays were sharing the same memory block because the copy constructor copied the pointer. At the end of the program, each object was not aware that there is another object that also has access to that memory and is going to free it.

You can notice that memory is shared by modifying any data of one of array objects:

```c++
int main()
{
    dynamic_array a1(10);
    for (int i = 0; i < a1.length(); ++i)
        a1[i] = i * 2;

    dynamic_array a2 = a1;
    std::cout << "first  array pos 1: " << a1[1] << "\n";
    std::cout << "second array pos 1: " << a2[1] << "\n";
    a2[1] = 31;
    std::cout << "first  array pos 1: " << a1[1] << "\n";
    std::cout << "second array pos 1: " << a2[1] << "\n";
}
```

~~~
$ ./program
first  array pos 1: 2
second array pos 1: 2
first  array pos 1: 31
second array pos 1: 31
Segmentation fault (core dumped)
~~~

Because both arrays have the same value of the pointer, any modification of one also affects another - there is only one array is memory but 2 objects have pointers to it.

## resource ownership

The problem above occurs due to lack of clear ownership semantics. 2 objects try to manage the same resource (here: dynamically allocated block of memory) without knowledge of another.

Additionally, any modification to that resource (here: changing values in the array) also affects other objects which (unaware) share this resource.

By creating a second object, we would like to have 2 separate arrays.

## shallow vs deep copying

Copy constructors by default blindly copy each member. It works well for built-in types and structures of them, but creates problems when copying an owning pointer. This is because the pointer is copied, not the resource managed by it.

What we want is **deep copying** - instead of copying the pointer itself (**shallow copy**), we want to copy contents of the pointed memory (**deep copy**). This is achieved by writing a custom copy constructor:

```c++
// inside the class
dynamic_array(const dynamic_array& other);

// outside
dynamic_array::dynamic_array(const dynamic_array& other)
: data(new int[other.size]), size(other.size) // allocate block of the same length
{
    for (int i = 0; i < size; ++i)
        data[i] = other[i]; // copy array contents
}
```

Now any object created by copy constructor will have a separate memory block. If you rerun the second example after adding custom copy ctor it will correctly show that both array objects manage different memory (changes are unrelated):

~~~
$ ./program
first  array pos 1: 2
second array pos 1: 2
first  array pos 1: 2
second array pos 1: 31
~~~

Now, each object has it's own memory block. The program no longer crashes ... untill we use `operator=`:

```c++
int main()
{
    dynamic_array a1(10);
    for (int i = 0; i < a1.length(); ++i)
        a1[i] = i * 2;

    std::cout << "first array\n";
    a1.print(); // should be 0, 2, 4, 6, ...

    dynamic_array a2 = a1; // copy construction
    for (int i = 0; i < a2.length(); ++i)
        ++a2[i];
    std::cout << "second array\n";
    a2.print(); // should be 1, 3, 5, 7, ...

    a1 = a2; // assignment
    std::cout << "first array after assignment\n"
    a1.print(); // should be 1, 3, 5, 7, ...
}
```

~~~
first array
[0] = 0
[1] = 2
[2] = 4
[3] = 6
[4] = 8
[5] = 10
[6] = 12
[7] = 14
[8] = 16
[9] = 18
second array
[0] = 1
[1] = 3
[2] = 5
[3] = 7
[4] = 9
[5] = 11
[6] = 13
[7] = 15
[8] = 17
[9] = 19
first array after assignment
[0] = 1
[1] = 3
[2] = 5
[3] = 7
[4] = 9
[5] = 11
[6] = 13
[7] = 15
[8] = 17
[9] = 19
Segmentation fault (core dumped)
~~~

The problem is the same, this time the source of the problem is defaultly-generated assignment operator.

## implementing assignment operator

... is very similar to copy constructor but it's prone to code duplication.

```c++
dynamic_array& dynamic_array::operator=(const dynamic_array& other)
{
    // free currently owned array
    delete[] data;

    // allocate
    data = new int[other.size]; // this might throw an exception
    size = other.size;

    // copy data from other
    for (int i = 0; i < size; ++i)
        data[i] = other[i]; // copy array contents

    return *this;
}
```

Additionally, the above code is not exception-safe as much as it could be. Exceptions are covered later, all that you need to know now is that `new` can throw (unless given `std::nothrow`) which creates here a possibility of `delete`d `data` but no allocated one.

So, how should `operator=` be implemented without sacrificing safety or causing code duplication?

## copy and swap

The **copy and swap** idiom solves exception safety problem while also allowing code reuse.

First, we need a swap function that would do what its name suggests.

```c++
// can be inside class definition
friend void swap(dynamic_array& lhs, dynamic_array& rhs)
{
    using std::swap;
    swap(lhs.data, rhs.data);
    swap(lhs.size, rhs.size);
}
```

The function has explicit using which causes it to pull `std::swap` into current scope.

- If ADL finds `::swap(member_type&, member_type&)` for any member it will call it (and then members can also do this for their own members...).
- If there is no such function, it will call `std::swap` instead.

In this case, members of `dynamic_array` are not user-defined types so they can't have their own swap functions, so the standard library swap is used instead.

There are some concerns regarding whether swap function should be a member or non-member friend. I wrote is as non-member friend to be in line with symmetric operator overloads.

Once we have swap function, we can **copy and swap**:

```c++
// could also be written inside the class definition since it takes only 2 lines
dynamic_array& dynamic_array::operator=(dynamic_array other) // copy
{
    swap(*this, other); // swap
    return *this;
}
```

Note the difference - copy ctor takes argument by const reference, assignment operator takes argument by copy - effectively reusing copy ctor.

`other` is a temporary object that lives until the function returns. We swap current object with it and then `other` dies invoking destructor which will release the resource if `*this` had any. This is important, because creation happens first (argument passed by copy), then there is release - if creation fails (exception) operation is aborted and code is strongly exception safe.

## summary

- Classes which manage a resource (eg memory) need custom dtor, custom copy ctor and assignment operator to work properly.
- Assignment operator should be implemented by reusing copy ctor.
