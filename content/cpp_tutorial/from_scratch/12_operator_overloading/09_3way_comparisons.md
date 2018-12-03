---
layout: article
---

<div class="note info">
This lesson describes a feature added in C++20.
</div>

## defaulted comparisons

Since C++20 it's possible to default comparison just like ctor/dtors.

```c++
struct package
{
    int floor;
    int shelf;
    int position;
    double weight;

    // note the return type
    auto operator<=>(const package& rhs) const = default;
};
```

New operator `<=>` represents 3-way comparison and behaves similarly to string compare function.

It has it's own language rules and can be safely declared as a member function.

Compiler automatically generates all comparison operators. By default all members of the type are compared in order of their appearance, making it a usual lexicographical comparison. Basically it applies operators to consecutive members which may have operator overloads themselves.

Operator `<=>` can also be used directly: `if ((a <=> b) < 0)` (but better not do it). The return type of `<=>` can only be compared with literal `0` or specific relational terms from the standard library.

## custom (user-defined) comparisons

When default semantics are not suitable, eg:

- we would like to compare fields in a different order
- we would like to ignore some fields
- we would like to support only `==` and `!=`

operator `<=>` can be defined with specific enumeration return type and have custom implementation.

Return types that supports all comparison operators:

- `std::strong_ordering`
- `std::weak_ordering`
- `std::partial_ordering`

Return types that supports only `==` and `!=`:

- `std::strong_equality`
- `std::weak_equality`

Each of these expresses a different level of support:

- **strong** means that if two objects compare equal, they are indistinguishable
- **weak** means that two objects may compare equal but be somewhat different
- **partial** means that a comparison of two objects may not be possible

There is no **partial equality** (at least in C++).

If the return type is left `auto` it will be the most powerful category that is supported by all members (strong > partial > weak). If there is a member that has no defined comparison category - compilation error.

### mathematical example

Given a set of numbers:

$$
S = \{ 1, 3, -7, 5, -10, -13, 8, -3, 10 \}
$$

we can sort it using various methods that implement different ordering guarantees:

<div class="table-responsive">
    <table class="table table-bordered table-dark">
        <tbody>
            <tr>
                <th>sorting criteria</th>
                <th>result</th>
                <th>term</th>
            </tr>
            <tr>
                <td>$f(x) => x$</td>
                <td>${-13, -10, -7, -3, 1, 3, 5, 8, 10}$</td>
                <td>strong ordering</td>
            </tr>
            <tr>
                <td>$g(x) => abs(x)$</td>
                <td>${1, 3, -3, 5, -7, 8, -10, 10, -13}$</td>
                <td>weak ordering</td>
            </tr>
            <tr>
                <td>$h(x) => \sqrt{x}$</td>
                <td>${1, 3, 5, 8, 10}$</td>
                <td>partial ordering</td>
            </tr>
        </tbody>
    </table>
</div>

$f(x)$ (identity function) results in strong ordering. If 2 values are equal in this ordering, they are exactly the same numbers.

$g(x)$ results in weak ordering. Different numbers may compare equal ($abs(3) = abs(-3)$).

$h(x)$ results in partial ordering. Some values can not be compared (here because they are out of domain of square root function).

### code examples

Some types do not make sense in ordering, but we should be able compare whether they are equal.

```c++
struct point
{
    std::strong_equaliy operator<=>(const point& rhs) const = default;

    int x;
    int y;
};
```

There is no point (pun intended) in sorting 2D numbers so this type should only support operators `==` and `!=`. To constrain it, we simply change the return type from `auto` to **strong equality**.

If the point had more members (eg color) but we wanted to compare only X, Y coordinates we would need to write custom body of the operator. Because some information would be ignored, that would be **weak equality**.

Here we have a class which represents people's ancestry. Not all people can be proved to be parents/childs of others so it's patial ordering.

```c++
class person
{
public:
    std::partial_ordering operator<=>(const person& that) const
    {
        if (this->is_the_same_person_as ( that)) return std::partial_ordering::equivalent;
        if (this->is_transitive_child_of( that)) return std::partial_ordering::less;
        if (that. is_transitive_child_of(*this)) return std::partial_ordering::greater;
        return std::partial_ordering::unordered; // unrelated people
    }

    // ...
};
```

## recommendation

Strong and weak orderings should always satisfy:

- irreflexivity: `x < x` is always `false`.
- asymmetry: if `x < y` is `true` then `y < x` is `false`.
- transitivity: if `x < y` and `y < z` then `x < z`.
- exactly one of `x < y`, `x > y`, `x == y` is `true`

A type that implements strong ordering/equality should compare all of it's members - otherwise substitutability can be compromised (operator `==` returns `true` but some member variables can be different). If this is the case then it should be weak ordering/equality.

## summary

<div class="table-responsive">
    <table class="table table-bordered table-dark">
        <tbody>
            <tr>
                <th>category</th>
                <th>supported operators</th>
                <th>any pair of objects is comparable</th>
                <th>equal objects are indistinguishable</th>
                <th>examples</th>
            </tr>
            <tr>
                <td>`std::strong_ordering`</td>
                <td>&lt; &gt; &lt;= &gt;= == !=</td>
                <td>&#10004;</td>
                <td>&#10004;</td>
                <td>whole numbers</td>
            </tr>
            <tr>
                <td>`std::weak_ordering`</td>
                <td>&lt; &gt; &lt;= &gt;= == !=</td>
                <td>&#10004;</td>
                <td></td>
                <td>case-insensitive strings</td>
            </tr>
            <tr>
                <td>`std::partial_ordering`</td>
                <td>&lt; &gt; &lt;= &gt;= == !=</td>
                <td></td>
                <td></td>
                <td>floating-point numbers</td>
            </tr>
            <tr>
                <td>`std::strong_equality`</td>
                <td>== !=</td>
                <td>&#10004;</td>
                <td>&#10004;</td>
                <td>pointers</td>
            </tr>
            <tr>
                <td>`std::weak_equality`</td>
                <td>== !=</td>
                <td>&#10004;</td>
                <td></td>
                <td>hashes of different inputs, check sums</td>
            </tr>
        </tbody>
    </table>
</div>

In other words:

- **strong** guuarantees that 2 objects that compare equal have exactly the same members
- **weak** allows to objects that compare equal to have somewhat different members (they might just not matter for comparison)
- **partial** allows comparison to signal a failed attempt

In later tutorials you will be using STL data structures and algorithms - they offer certain operation complexity guarantees but require certain conditions to be satisfied - for example `std::sort` requires at least weak ordering, `std::find` requires at least weak equality.

## exercise

Which comparison categories are in the following situations?

- points by the sum of their X, Y coordinates
- files by their size
- files by their paths
- files by their directories
- users by their ID
- people by their birth date
- game results by the score of each player

<details>
<summary>answers</summary>
<p markdown="block">

- weak ordering (points 3,5 and 4,4 are equal)
- weak ordering (files with same size can be different)
- weak equality (symlinks can create multiple paths for the same file)
- partial ordering (files may not have common parent directory) - this example is similar to persons in family trees
- strong equality (IDs are unique) OR strong ordering if IDs are treated as numbers
- weak ordering (date can be the same for different people)
- weak ordering (the same score can be achieved by multiple players)
</p>
</details>
