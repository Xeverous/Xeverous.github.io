---
layout: article
---

As you may know, there are multiple programming languages. But what are the differences?

#### Syntax

The first programming languages were designed to be easy to read and somehow to mimic human language. A good example od this is Pascal, which uses words `begin`, `end` all over the place. In many later languages, this was found to be too verbose and actually reduce readability — many words have been replaced by symbols, mostly various types of brackets. Some languages resigned from these and have different mechanism for controlling blocks of code  Python relies on indentiation level.

Many languages borrowed C style and use symbols (usually `{` and `}`) to wrap blocks of code.

TODO paste some examples of the same code

#### Typing

**Statically typed languages** — these are the languages which specify clearly what's the thing they are working on — a number is a number (stored in memory as binary value), and you can't e.g. ask how many decimal digits it has since it's not a character sequence (where each character would be stored as a separate number)

**Dynamically typed languages** — these are the languages which do not have strict feeling of data types, instead they perform conversions on the go. You assigned number `5` to `x`? Ok, but when you add `"3"` (not `3`) it becomes a character sequence `"53"`. If you then try to divide it like `x / 2` it suddenly becomes a number back again.

Alternative names are **strong** and **loose** typing.

C and C++ are examples of very typical statically typed languages. There is more to specify and rules regarding data types are more strict.

#### Execution method

**Compiled** — source code (text files) is transformed (compiled) into machine code (executable files) and then it can be run on the target machine. Each processor architecture has different machine instruction set so developing software to different types of processors requires compilations with different settings.

**Virtualized** — source code is compiled, but not to machine instructions but some intermediate form — usually named bytecode. Then the bytecode can be understood by a virtual machine which executes real machine instructions. Typical languages using virtual machine are Java (running on JVM) and C# (running on .NET). The same bytecode can be run on different machines, but it requires virtual machine support — not all platforms are supported.

**Interpreted** — code is transformed on the go to machine instructions. You don't have to do anything with the code to make it runable — interpreter loads it directly as text. As with virtualized languages, it requires interpreter support on the given platform. Typical interpreted languages are Python, Ruby, JavaScript.

Interpreters and virtual machines therefore must be written in a compiled language — otherwise we would need an interpreter or virtual machine to run itself which forms an infinite loop. And in fact, most of these are written in C or C++.

Compiled languages yield the best performance, because machine instructions are executed directly — there are no layers in between. Modern virtual machines implement JIT (just in time) compilation where possible, which transforms bytecode into machine instructions which yields better performance but not as good as direct compilation.

Interpreted languages are the slowest of these, but the performance is not their primary design — their purpose is to make running and writing short programs easy with minimal configuration.

Interpreted languages are usually dynamically typed while others statically. Static typing improves performance as the generated machine instructions are more specialized.

#### Memory management

**Manual** — mostly in C and assembly languages. It's programmer own responsibility to allocate and deallocate memory properly. Nothing is done automatically except stack–allocated objects.

**Deterministic** — an advanced set of abstractions over manual management to ease and automatize acquisition and release of resources, RAII idiom. The backbone of memory management mechanisms in C++ and Rust. Zero–overhead or low–overhead depending on the situation and chosen method.

**Garbage collection (non–deterministic)** — no real management from the programming language, instead a side process is running (GC — garbage collector) in the background which cleans up and releases unused resources. The most popular type of memory management, used by practically all virtualized and interpreted languages.

Garbage collected languages have very simple management from the programmer point of view, but complex algorithms are run in the background which are hard to predict or decide when some resource will be released. This solution simplifies languages from the code side (just write `new` when you want a new object) but imposes a performance loss and causes the program to consume more memory than the equivalent program which would clean up by itself.

Programs using manual or deterministic memory management incorrectly may crash or leak memory (acquire more but never release) but the same problem may happen in garbage collector implementation. Additionally, not every resource can be garbage collected which may sometimes cause problems in languages which were designed to work primarily with GC.

Over time, garbage collection algorithms have undergone multiple changes, and the GC is running better and better. GC languages have also gained features which simplify problematic situations where resource can not be GC'ed.

C++ started with basic abstraction (`new`, `delete` keywords) over C `malloc()`, `free()` and later has gained significant abstractions to further ease acquisition and release of resources (RAII idiom is the big thing here). Language offers various smart pointers and other zero–overhead wrappers which decide at compile time how resources are manipulated. `new` and `delete` are no longer used, unless someone writes own resource management abstractions.