---
layout: article
---

Everything than can be later referred to has an **identifier**. It's just the name of the entity.

Valid names can consist of:

- lowercase latin letters (a - z)
- uppercase latin letters (A - Z)
- underscore symbol (_)
- digits (0 - 9), with the exception that the name can not start with a digit

Actually there are more possible characters (also from Unicode) but REALLY NO ONE uses them and tools, including compilers many not support them.

Example valid identifiers:

```c++
func
x1
x2
y
foo_bar
a_b_c_d_e_f
XABCDX
WeIrD_nAmE
```

Example invalid identifiers:

```c++
if  // (this is a language keyword)
1x  // this is a number, not an identifier
x@y // symbol @ is reserved
```

Additionally, the C++ standard reserves following names (you should not use them):

- `_X*` — anything beginning with an underscore followed by an uppercase letter
- `__*` — anything beginning with 2 underscores
- anything containing 2 consecutive underscores
- anything in the namespace `std`
- anything in the namespace `posix`
- [keywords](https://en.cppreference.com/w/cpp/keyword)

More about namespaces later. For now, simply remember to not start names with `_` character.

___

There are two standard library identifiers you have already seen: `std` and `cout`.

## Name styles

There have been established few core naming styles, these conventions are broadly used in the entire IT, not just programming.

```
lowercase_name_style (sometimes referred to as snake case)
UPPERCASE_NAME_STYLE
PascalCaseNameStyle
camelCaseNameStyle (in camelCase first letter is lowercase)
```

It's bad to mix these styles — any of the following names are against convention:

```
Account_Manager
Accountmanager
PrintHTTPDocument
print_HTTP_document
```

correct names:

```
account_manager
AccountManager
accountManager
print_http_document
PrintHttpDocument
printHttpDocument
```

C++ (following C convention) standard library uses `UPPERCASE_NAME_STYLE` for macros and `snake_case_style` for almost everything else. All tutorial content on this website uses relevant styles — just simply follow them.
