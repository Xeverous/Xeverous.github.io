---
layout: article
---

Comparing integer class would be fairly simple - I guess you can already deduce how to write overloads for comparison operators. To showcase more complex problems this (and the next lesson) will use structures consisting of multiple members.

## convention

Overloads of comparison operators do not have any constraints on the return type. But it would be pointless to return any other type than `bool`.

For a such type:

```c++
struct package
{
    int floor;
    int shelf;
    int position;
    double weight;
};
```

We would like to sort an array of these - we need a way to compare them - namely **strict weak ordering** (more about all relation terms in next lesson).

Let's say we would like to sort packages by their place (in order: floor, shelf, position). We do not care for the weight member (it will not be used in comparisons).

The convention in C++ is to implement `==` and `<` first and then reuse them in implementing other operators.

Comparison operators are implemented as non-member since both arguments are treated exactly the same.

## equality

```c++
bool operator==(const package& lhs, const package& rhs)
{
    return lhs.floor == rhs.floor && lhs.shelf == rhs.shelf && lhs.position == rhs.position;
}
```

Remaining operators can be implemented in terms of others:

```c++
bool operator!=(const package& lhs, const package& rhs)
{
    return !(lhs == rhs); // reuse == and reverse the result
}
```

## ordering

Don't make the mistake: `<` is less-than operator. It answers whether left operand is *less than* the second. If they are equal, result should be false.

```c++
bool operator<(const package& lhs, const package& rhs)
{
    if (lhs.floor != rhs.floor)
        return lhs.floor < rhs.floor;

    if (lhs.shelf != rhs.shelf)
        return lhs.shelf < rhs.shelf;

    return lhs.position < rhs.position;
}
```

This is **lexicographical comparison** - elements are compared in priority order. This method is short circuited - if high-priority elements are different, the comparison stops on them. In our case floor is checked first, if it's the same we move to shelf and if shelf is the same we move to position.

There is a simpler way to implement above comparison:

```c++
bool operator<(const package& lhs, const package& rhs)
{
    return std::tie(lhs.floor, lhs.shelf, lhs.position)
         < std::tie(rhs.floor, rhs.shelf, rhs.position);
}
```

`std::tie` creates a tuple of references to elements. `std::tuple` has already overloaded comparison operators (thanks to templates) - so we just reuse it's lexicographical comparison implementation.

Other comparison operators reuse `<`:

```c++
// swap order (rhs is on the left) and reuse <
bool operator> (const package& lhs, const package& rhs) { return   rhs < lhs;  }
// less-than-or-equal is not-greater-than
bool operator<=(const package& lhs, const package& rhs) { return !(lhs > rhs); }
// greater-than-or-equal is not-less-than
bool operator>=(const package& lhs, const package& rhs) { return !(lhs < rhs); }
```

Thanks to these, we need only to implement actual comparison in `==` and `<`. Other operators (`!=`, `>`, `<=`, `>=`) are then easy to boilerplate. This also reduces space for potential mistakes since remaining operators inherit behaviour from first two.

#### Question: Isn't `!(lhs < rhs)` slower than manual implementation?

No, it is not. Compilers today do very advanced optimizations and they easily inline and remove redundant operations. There are also some hardware specific quirks (eg whether `<` takes less cycles than `<=`). I suggest not to care about performance now and focus on writing good, idiomatic code. You need to first understand language and conventions, then can move into not-so-clear-and-straight topics such as optimization.

## 3-way comparison reuse

Some functions (mostly from C library) implement 3-way comparison. An example of this is **str**ing **c**o**mp**are function:

```c++
int std::strcmp(const char* lhs, const char* rhs);
```

It returns:

- negative number if `lhs` is lexicographically less-than `rhs`
- zero if both sequences are equal
- positive number if `lhs` is lexicographically greater-than `rhs`

Given function that does such 3-way comparison, operators can be overloaded this way:

```c++
bool operator==(const X& lhs, const X& rhs){ return cmp(lhs, rhs) == 0; }
bool operator!=(const X& lhs, const X& rhs){ return cmp(lhs, rhs) != 0; }
bool operator< (const X& lhs, const X& rhs){ return cmp(lhs, rhs) <  0; }
bool operator> (const X& lhs, const X& rhs){ return cmp(lhs, rhs) >  0; }
bool operator<=(const X& lhs, const X& rhs){ return cmp(lhs, rhs) <= 0; }
bool operator>=(const X& lhs, const X& rhs){ return cmp(lhs, rhs) >= 0; }
```

## `std::rel_ops`

There are few convenience templates in standard library that are supposed to automatically implement remaining operators. The core purpose of them was to reduce boilerplate code. Unfortunately, these function templates have their own problems which causes to require different boilerplate code (issues with ADL and namespaces).

Because of this, it is not adviced to use them and implement operators like in the examples above.

Utilimately one can use [Boost Operators library](http://www.boost.org/doc/libs/release/libs/utility/operators.htm) that offers various functionalities to simply and automate operator overloads generation (not only for comparison).

`std::rel_ops` is deprecated in C++20 (in favour of new operator `<=>`).

## traps

Remainder: if `<` returns `false` for `a < b` it does not mean that `a > b`, they can also be equal.

<div class="table-responsive">
    <table class="table table-bordered table-dark">
        <tbody>
            <tr>
                <th>operator a</th>
                <th>operator with exactly reverse result</th>
            </tr>
            <tr>
                <td>==</td>
                <td>!=</td>
            </tr>
            <tr>
                <td>&lt;</td>
                <td>&gt;=</td>
            </tr>
            <tr>
                <td>&gt;</td>
                <td>&lt;=</td>
            </tr>
        </tbody>
    </table>
</div>
