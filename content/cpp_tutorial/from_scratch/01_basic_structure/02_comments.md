---
layout: article
---

### comments

It is possible to insert comments into the program. Such text will be completely ignored by the compiler.

There are 2 types of comments in C++:

```c++
// this is a single-line comment
// <- you need to start it with // and the comment will span to the end of line

/* this is a multiline comment - it can span multiple lines
notice the order of / and * a the beginning and end
multi-line comments must end with * and / as it is here: */
```

Single–line comments can start after code:

```c++
int x = 5; // this comment spans to the end of this line
```

<div class="note warning">
Watch out for accidental `\` at the end of a singleline comment. It extends the comment to another physical line.

```c++
// text \
this text is also a part of the comment
``` 

It's advised to just write 2 comments instead:

```c++
// text
// text
```
</div>

Multi-line comments can start and end any time — in such scenario they can be shorter than `//` comments:

```c++
int /* some comment */ x = /* other comment */ 5;
```

Often you can find asterisk–stylized multi–line comments:

```c++
/*
 * text
 * text
 * text
 */
```

Some IDEs do it automatically. If you have a comment that spans many lines it's recommended to start lines with `*`, just like in the example above.

Sometimes you may find documentation comments:

```c++
/*
 * @brief some short description
 * @param x 
 * @param y
 * @return value or error
 * 
 * @details long description...
 */
```

Tags like `@brief`, `@param` are searched by tools like Doxygen which generate documentation from the comments. Some IDEs may also display help boxes relying on these tags.

**Nesting**

Multi-line comments can not be nested. You can't write a comment inside a comment.

```c++
/* text1 /* text2 */ text3 */
```

An error will appear at the `text3`.

### recommendations

Do not comment **what** happens — it's visible in the code. Describe **why**.

bad comment

```c++
// Calculate the cost of the items.
cost = items / 2 * price;
```

good comment

```c++
// We need to divide items by 2 here because they are bought in pairs.
cost = items / 2 * price;
```

Multiple comments in this tutorial will break this rule, as here they must explain what's actually happening. They are comments for learning, not how actual comments should be written.

### Commenting out the code

You can disable code by making it comments. Use it for experiments.

```c++
// std::cout << "hello, world";
std::cout << "my" << "own" << "text";
```

Multiple IDEs and text editors offer shortcuts to comment/uncomment multiple lines at once. Usually `ctrl`+`/` which changes selected text.

## Exercise

Put some comments in random places in the hello world program, experiment and check which lines are executed and which not.
