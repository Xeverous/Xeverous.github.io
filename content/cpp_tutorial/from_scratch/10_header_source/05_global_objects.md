---
layout: article
---

So far I have presented how to define a function in a separate source file. The process of defining a global variable is very similar.

For function, we provided definition in source file and declaration in header file. How are variables declared without definitions?

## variable declarations

If we try to put `int x;` into a header file, we will get build errors once multiple source files include that header. The problem is that such statement is also a definition.

First thing that might come to your mind is to use `inline` - in fact, this is a valid solution but supported only since C++17.

There is a different keyword that specifies that given statement is only a declaration:

```c++
extern int x; // defined externally, here just a declaration
```

Now if you try to build such project, you will get a different error - this time not duplicated definition but missing definition.

The correct way is to declare using `extern` in a header file and define without this keyword in any choosen source file.

`extern` can also be used for function declarations, but since functions have a distiction between declaration and definition it is not necessary. The keyword is required only for things that have no syntax for pure declaration.

## example project

**global.hpp**

```c++
#pragma once

extern int x;
```

**global.cpp**

```c++
#include "global.hpp"

int x = 1; // initializers must be at the point of definition
```

**functions.hpp**

```c++
#pragma once
#include <iostream> // see notes

void print_x();
void assign_x(int val);
```

**functions.cpp**

```c++
#include "functions.hpp"
#include "global.hpp"
#include <iostream>

void print_x()
{
    std::cout << "x is now: " << ::x << "\n"; // see notes on ::
}

void assign_x(int val)
{
    ::x = val;
}
```

**main.cpp**

```c++
#include "functions.hpp"
#include "global.hpp"
#include <iostream>

int main()
{
    print_x();
    assign_x(2);
    print_x();
    std::cout << "x is now: " << ++::x << "\n";
}
```

## recommendations

### accessing global objects

`::x` was used to explicitly state that it's a global object. There is no namespace name before scope resolution operator - such syntax makes sure that a globally visible entity it used. It's used for clarity - global variables are bad and it's better to clearly express that we want to access one.

### include redundancy

**main.cpp** includes **iostream** even though it's already provided by **functions.hpp**. This is good because if at any point you realize that **iostream** is not actually needed for **functions.hpp** such removal would not break the build.

You should always include headers if you use any stuff from them. Do not rely on other headers including headers you need because if at any point dependencies of your dependencies change, you will get build errors. In other words, think of headers as necessary and not rely on what they include themselves.

### include order

There are no requirements for specific order of include directives (we just list dependencies and if they have their own dependencies header guards make redundant includes empty) but there are some benefits for inside-out order - mostly hitting any build errors sooner and preventing code from accidental relying on dependencies of dependencies.

Therefore, I advice to list headers in the following order:

- associated header file with the same name (if it's a source file) - see first inclusion in **functions.cpp**
- any header closely related to the code (usually headers from your own project)
- any external library headers if needed in this file
- any standard library headers if needed in this file

This way your project headers will be always parsed first, making sure they are self-contained. If they need some library that you did not include in themselves, build will appropriately fail.

## include paths

In most projects, directory structure resembles namespaces used in the code and groups of classes that have related purpose.

For refactoring reasons, it's bad to use `..` in include paths. Instead, the project root directory should be added as include search path to allow top-down paths in includes.

Example:

~~~
game_engine
  - graphic
    - texture.hpp
    - texture.cpp
    - image.hpp
    - image.cpp
  - audio
    - sound_stream.hpp
    - sound_stream.cpp
    - music.hpp
    - music.cpp
  - util
    - asset_manager.hpp
    - asset_manager.cpp
    - templates.hpp
    - constants.hpp
~~~

**asset_manager.hpp**

```c++
#pragma once

// bad: '..' in include paths
#include <../audio/sound_stream.hpp>
#include <../graphic/texture.hpp>

// good: top-down paths, easier to modify code
#include <game_engine/audio/sound_stream.hpp>
#include <game_engine/graphic/texture.hpp>
```

With some IDEs and build systems, to make it work you might need to add project root directory to compiler search paths.

#### Question: Why templates have no corresponding source file?

Since templates must be fully defined before they are used, complete template definitions are put into headers. There is nothing to put into source file.

The same applies for constants - these usually have internal linkage or are implicitly inline.
