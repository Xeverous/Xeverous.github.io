---
layout: article
---

Before moving further, I will briefly describe the minimal program. I know that most readers are very eager to experiment and want to instantly fiddle with the code, even if they don't understand everything.

```c++
#include <iostream>
 
int main()
{
	std::cout << "hello, world";
	return 0;
}
```

## Include directive

The first line is telling the compiler about additional code which is needed to run this program. `#include` is a preprocessor directive — it tells the compiler to *include* the contents of the `<iostream>` header before this program. *iostream* means nothing more than *input/output stream* — it is the part of standard library used to input and output data. Of course more parts of the C++ standard library will be presented in further chapters.

## Main function

`main()` is a function. Similarly, like in math, functions are denoted with `()`. `int` is one of fundamental types (integer). The main function is the start of the program - later you will learn how to write own functions.

## Braces

`{` and `}` represent scope. Braces are an important part of C++ language (and many other derivatives of C language syntax) and will be used every time something needs to be specified how many code it covers.

## `std::cout`

`std::cout` is ***st**andar****d*** ***c**haracter **out**put* stream. It is *the thing* that is able to print characters on the console. How is it done depends on the operating system, screen type and other stuff. Not very important now. Just remember that after `<<` it can accept quite variety of things and it knows how to make them appear as text.

## Return statement

`return 0` is line ending the main function. Function has type `int` which means it's result is an integer value (here it's 0). This may sound strange now but will be clear after you read why functions exist and what's the purpose of them in programming.

The value `0` will not be printed on the console. It is an information for the caller of the function (here: operating system). `0` returned from main function indicates successful program execution.
