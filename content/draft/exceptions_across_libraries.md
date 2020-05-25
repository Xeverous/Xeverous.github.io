---
layout: article
---

# Throwing and catching exceptions across shared library boundaries

No, there is no simpler title for this article.

> What is this even about? Is there some problem with it?

???

## prerequisities

You must understand these terms:

- **static** vs **dynamic** (shared) library
- how exceptions work (basically the stack unwinding process; knowing or understanding how it is done on machine code level is not required)

## what you need to know

???

### in high-level languages

??

### in compiled languages

???

## GCC

From the manual on options `-shared-libgcc` and `-static-libgcc`:

On systems that provide libgcc as a shared library, these options force the use of either the shared or static version, respectively. If no shared version of libgcc was built when the compiler was configured, these options have no effect.

There are several situations in which an application should use the shared libgcc instead of the static version. The most common of these is when the application wishes to throw and catch exceptions across different shared libraries. In that case, each of the libraries as well as the application itself should use the shared libgcc.

Therefore, the G++ driver automatically adds -shared-libgcc whenever you build a shared library or a main executable, because C++ programs typically use exceptions, so this is the right thing to do.

If, instead, you use the GCC driver to create shared libraries, you may find that they are not always linked with the shared libgcc. If GCC finds, at its configuration time, that you have a non-GNU linker or a GNU linker that does not support option --eh-frame-hdr, it links the shared version of libgcc into shared libraries by default. Otherwise, it takes advantage of the linker and optimizes away the linking with the shared version of libgcc, linking with the static version of libgcc by default. This allows exceptions to propagate through such shared libraries, without incurring relocation costs at library load time.

However, if a library or main executable is supposed to throw or catch exceptions, you must link it using the G++ driver, or using the option -shared-libgcc, such that it is linked with the shared libgcc.

## Clang

??

## MSVC

??
