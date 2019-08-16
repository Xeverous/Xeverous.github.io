---
layout: article
---

## naming convention problems

Many companies and well-known projects lived in isolation and imposed their own conventions. What's worse, some projects attempted to use conventions from other languages (mostly Java). At some point, one could say there was a misconception on the internet that all OOP languages have the same naming style. I have no idea who started all of this (but Microsoft is surely dirty in this regard). C++ Core Guidelines say that this inconsistency has caused enough suffering.

The original naming style in C++ follows the one of C which uses `snake_case`. **This also includes type and file names.**

Personally I am strongly against UsingCamelCaseInC++AndNotOnlyBecauseItIsHardToRead. It's straight in conflict with standard library code. Unlike Python or PHP, C++ standard library is consistent within itself so there is no "but previously it was different" argument. Well-known state-of-the-art libraries (eg all of boost libraries) stick to the standard library convention.

C++ uses `UppercaseCamelCase` names only for template parameters and concepts - this is a quite small part of code. `lowercaseCamelCase` is not used at all.

In some rare cases, certain names are enforced by the standard library (eg `const_iterator` type name or `size` function name). More correct naming enforcements may be implemented with concepts and are likely to come in the future.

## naming convention

- `file_name`
- `type_name`
- `function_name`
- `variable_name`
- `constant_name`
- `attribute_name`
- `keyword_name` (yes, C++ does have multi-word keywords with `_`)
- `MACRO_NAME`
- `TemplateParameterName`
- `ConceptName`

## file naming

C used `*.h` and `*.c` file names, and it sounds natural that C++ should have similar convention.

Thus, I recommend `*.hpp` and `*.cpp`. You may find some projects which use `*.hh` and `*.cc`.

You might see use of mixed `*.h` and `*.cpp` but I discourage this. There are projects that use both C and C++ code and having the same extension for headers can confuse both humans and some tools.

## member names

Some coding styles recommend to use different name styles for members. There is no strong convention for it in C++ or its standard library, but there are few common ones:

~~~
member_name (no special style)
member_name_
_member_name
m_member_name
~~~

Personally I am for no special style or the last one (it allows IDEs a very good autocomplete).

## reserved names

C++ reserves following name patterns for the implementation:

- `_X*` - anything beginning with an underscore followed by an uppercase letter
- `*__*` - anything containing 2 consecutive underscores

This makes sure that user code does not interfere with implementation, especially macros that can easily sabotage any other code.

Using reserved names that are not explicitly allowed by the implementation (like additional keywords or types, eg `__int128`) is undefined behaviour.

## discouraged resources

**One of the worst** coding styles you can find for C++ is Google's C++ Style Guide. It has numerous issues (including being in opposition of standard library idioms) and is mostly suited for writing legacy code.
