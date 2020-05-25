C++ is a complex language and is well-known for its quirks. This article presents some lesser known features that may be useful in some situations.

Note that some of these features are not actually complicating the language - sometimes it's exactly the opposite. People coming mostly from higher-level languages may view these as additional features but in fact, the language they are coming from might aswell support it - some languages simply directly forbid certain uses that could potentially be misused (Java is the primal example of a language that explicitly restricts many features).

## returning void

You can directly return result of another expression, even if it's of void type.

```c++
void foo();

void bar()
{
	if (something)
		return foo();
}
```

This particular "feature" is very useful in some templates, especially the ones that wrap callbacks. They simply do not have to care about the return type, even if the callback returns `void`.

For many languages, `void` (or equivalent) is a very special type that has many explicit restrictions. In C++, `void` is an incomplete type that can not be completed, which restricts its usage to only what is available to forward-declared names. There are some further restrictions, eg. forbidding references to void (void pointers are allowed).

Some languages have an equivalent type but it's fully defined and instances of it can be created. Such example is `NoneType` in Python. [There is a proposal for C++ to make void type a regular type](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0146r1.html); **this proposal is not a new language feature - it's actually a language simplification because it proposes to remove explicit restrictions**.

## copy/move constructors with additional parameters

Copy/move ctors do not have to take exactly 1 parameter. They only have to:

- take first parameter which is the instance that is copied/moved (const reference for copy ctors, rvalue reference for move ctors)
- be able to be called with just the first parameter (any additional parameters must have default values)

## multiple inheritance

This is a primal example where C++ is a simpler language than other languages which support object-oriented programming. In C++ there is no `interface` keyword. Want an interface? Write `class`, just don't add any member variables and make all functions `virtual`.

There are no restrictions on inheritance in C++ simply because they were never necessary. There are no barriers in implementation of MI, in fact, in other languages classes that inherit from multiple interfaces may have multiple virtual table pointers.

Note that the existence of the feature is not an encouragement to write inheritance-heavy code. C++ simply follows the design of giving wide possibilities, some of which may drag high responsibility. C++ design is against approaches like "we should not have this feature because it can be easily abused or misused".

## covariant return types

TODO

## comma operator

Nothing special, right? Well, not really in C++. Just a normal operator. **Can be overloaded.** By default evaluates all expressions (in left-to-right order) and returns the result of the last expression. Such code is valid:

```c++
if (something)
	++x, return y;
else
	++y, return x;
```

Obviously overloading `,` or abusing it in way like the above is discouraged.

TODO example of , used to "implement" C++17 folds in C++11 code

## pure virtual destructor

Most other languages don't have destructors. In C++ you may sometimes want to have a class that acts as an interface but don't need any functions. To workaround the lack of `interface` or `abstract` keyword, pure virtual destructors can be used to force type to be abstract.

```c++
class abstract_interface
{
public:
	virtual ~abstract_interface() = 0;
};
```

Note that in this special case, even though the function is pure (abstract) it has to be defined:

```c++
// in a source file
abstract_interface::~abstract_interface() {}
```

## unary plus

These expressions are valid:

```c++
c = a + b; // binary +
c = a - b; // binary -
c = -c;    // unary  -
```

and so is this one:

```c++
c = +c;    // unary  +
```

There is no reason not to support unary plus. Again, even though removing this operator is trivial it's an unnecessary language complication. By default, it does nothing except promoting the value to `int` if it's smaller (remnant of backwards compatibility) but it's a very good opporunity for overloading.

Uses of unary plus in C++:

- Boost Spirit (for specifying grammar using operators)
- promoting characters to integers (`std::cout << 'a'` will print `a` but `std::cout << +'a'` will print the value of this character in the encoding used by the implementation)
- lambda to pointer convertion: `+lambda` forces the use of unary `operator+(T*)` - non-capturing lambdas can be converted to function pointers
