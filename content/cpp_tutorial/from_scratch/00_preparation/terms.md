---
layout: article
---

It's not hard to learn math operations if you don't know the difference between *digit* and *number* terms but it is frustrating to encounter totally unusual problem which you can't understand (e.g. "sum of digits in number"). This page is to prevent such situations and give you basic understanding what actually is happening.

**Compiler** - the most important thing here. It is a program which transforms the source code (text) to an intermediate form which is then used to generate machine code (binary executable files or libraries). The process of generating machine code and merging multiple libraries and compiled source is done by the **linker**.

There are 3 major C++ compilers:

- MSVC (Microsoft Visual C++ compiler) — installed with Visual Studio, works only on Windows
- GCC (GNU compiler collection) — de facto standard compiler for most GNU/Linux distributions
- Clang — a newer compiler based on LLVM project, an alternative to both above

There are more compilers but none are as popular as these and most of the others are propertiary and do not support newer C and C++ standards.

GCC and Clang are also available on Windows through various ports. The most popular is MinGW (Minimalist GNU for Windows) which comes with ported compilers, linkers, GNU make and other build tools.

All of mentioned above compilers come together with other necessary tools. All of these tools are command–line programs with no graphic interface so it's recommended to use an IDE which automates build tasks.

It's recommended to use the latest versions of compilers:

- GCC (and it's ports) >= 7.2
- Clang >= 6.0
- MSVC - from Visual Studio 2017 with updates if available

**Debugger** — a program to ... debug other programs. Launches executables in sort of sandbox and allows to "play the god mode" — look up and change any variable and any time, step through the program execution and more. Many debuggers allow to map results to the source code.

**IDE** (integrated development environment) — A program with multiple features to ease the development of writing programs. Typical features are:

- smart editor with rich autocomplete and syntax highlighting
- refactoring and code generation tools
- build system integration (you click build button instead of calling compiler, linker etc directly)
- debugger integration (as above)

Most popular free C++ IDEs:

- Visual Studio
- Eclipse
- Qt Creator
- Code::Blocks
- KDE

It's recommended to use one of these. All except Visual Studio work by default with GCC (or it's ports) and are multiplatform. Visual Studio comes with Microsoft's own compiler and while it can be made to work with different compiler or on a different OS it's better to use other IDEs there.

**Toolchain** — a set of tools used to build a project. The minimal toolchain is build script (usually done by IDEs) + compiler + linker.

**Libraries** are just code bases (compiled or not) with multiple functions (and classes and templates and other...) than can be reused. This tutorial uses various parts of C++ **standard library** (which comes with the compiler). Additional codebases which you can download from the internet will be **external libraries**.

**Dynamic link library** (.dll files on Windows, .so on Unix systems) — compiled code in the form which allows to reuse it by calling from executables. The content of dynamically linked libraries is mostly machine code — just like with executable files, the core difference is that libraries do not have entry point (there is no start) — they are just reusable pieces of compiled code. Dynamic link libraries are loaded at runtime, usually at the start of the application that wants to use these. The same library can be used by multiple executables.

**Static link library** (.lib, .a and more) — compiled code but not ready to execute. Static libraries are merged with other compiled code to form executables. These files don't exist on the user side as their contents are inside other binary files. After merging static library(ies) with other compiled code (which may have start point) executable or dynamic link library is produced.

Depending on the used toolchain, it is possible to transform dynamic and static link libraries from one form to another. Multiple toolchains may use additional, different intermediate files and have more or less possible transformations.

## build process

typical possible build processes:

TODO add illustrations

source code => object code => static link library

source code => object code => dynamic link library

source code => object code => executable

static link library + object code => executable

