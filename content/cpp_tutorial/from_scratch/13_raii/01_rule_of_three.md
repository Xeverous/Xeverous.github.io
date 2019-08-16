---
layout: article
---

Previous lesson presented an example problem which occurs as a consequence of improper resource management. RAII (resource acquisition is initialiation) is a C++ programming technique which relies on binding a resource to the lifetime of an object and clear notion of *owners* and *observers*.

## owners vs observers

The `data` member pointer of the `dynamic_array` class is an **owning pointer**. It's owning because it is used to allocate and deallocate a resource (here: memory).

This is not an owning pointer:

```c++
{
    int x;
    const int* p = &x; // not an owner (does not manage memory)
    // p is an *observer*
}
```

**Observers** may be created in various ways, but they should not release the resource.

It's important to have clear distinction between owners and observers as otherwise resource management becomes very unclear which results either in crashes or resource leaks.

**RAII is one of fundamental C++ concepts.**

## core RAII principles

**Owner** is an object (usually a pointer) which represents a runtime-obtained resource.

- Only owners should be used in resource acquisition and release.
- Owners must not be lost (by being overwritten or going out of scope) before the resource is released.

**Observer** is an object (also usually a pointer) which represents access to the resource.

- Observers should never acquire or release the resource.
- Observers should be outlived by the resource or have a deterministic mechanism to safely check that the resource has been released (otherwise we get dangling pointers/references).

Resources should be tied to the lifetime of an object which itself uses automatic allocation.

- Only RAII types should manage owners.
- Each RAII type should manage exactly 1 resource.

RAII types may additionally offer (if possible):
- obtaining a replica of the resource and deep copying the contents
- moving/swapping resources between RAII-type objects
- releasing the resource before it would normally be released

## the rule of 3

TODO def block

If a class contains any of the following:

- custom destructor
- custom copy constructor
- custom copy assignment operator (`operator=` overload)

it almost certainly needs all 3.

The presence of any of mentioned 3 special member functions almost always is a consequence of having to deal with resource management. Writing one but forgetting to write the rest usually ends in bugs causing crashes like the one presented in this lesson.

Therefore, you should follow the rule of 3 - watch out when writing any class that holds owning pointers - you almost always don't want to copy them but the contents of managed resource.

## relation to the example

`data` member of `dynamic_array` class is an owner. The pointer is used to allocate and deallocate a resource (here: memory).

There are no observers, although we could think of `operator[]` as one.

We can say that after the addition of:

- a custorm dtor
- a custom copy ctor,
- a custom assignment operator

`dynamic_array` class is a RAII class - the lifetime of the dynamically allocated array inside the class is tied to the lifetime of the class object.

```c++
{
    dynamic_array da(/* ... */);

    // use da...

} // no memory leaks, destructor frees resources
```

This way, we are back to very simple syntax. We can now build more complex types, which consist of types such as `dynamic_array` and we do not need to write more custom special member functions for them. All of the memory management is self-contained within the class and any other (enclosing) class can use `dynamic_array` objects as its members just like they were plain `int`s.

#### Question: Example of a different resource than memory?

Practically all resources will be more or less related to memory but many of them will be more than just dynamically allocated block.

Example other resources:

- POSIX file descriptor (given by the OS when opening a file) (this is an integer)
- network socket
- available threads to execute some tasks

## more on RAII

Later in the tutorial, you will learn:

- how to deal with resources that are unique (can not be copied, only observed)
- how to share a resource to multiple users without breaking it's cleanup
- how to transfer (move) a resource between 2 objects (effectively changing the owner)
- resource-managing tools that are offered by C++ standard library, especially:
  - containers
  - smart pointers

If you find the term name unclear (resource acquisition is initialiation) - don't worry, C++ has a lot of idioms that have weird names, some of which have unpronounceable abbreviations. The alternative names for RAII are SBRM (scope-based resorce management) and CARD (constructor acquires, destructor releases).

## recommendations

- Have a clear notion of owning and observing pointers - whenever you create one, think of it whether it should be used for resource management or not. A good way is to always allocate and deallocate through the same pointer.
- Encapsulate a resource by writing a class for it (more examples soon).
- Follow the rule of 3 (note: there is also rule of 5 and of 0 - these will be presented in next lessons).
