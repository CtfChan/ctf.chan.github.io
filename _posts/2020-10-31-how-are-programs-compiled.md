---
title: "Cpp Blog: How are executables made?"
tags: [cpp]
style: fill
color:
description: The mysteries of the compiler
---

The process of creating an executable can be broken down into a compiling and a linker stage.

The stages of compiling are:
- pre-processing
- linguistic analysis
- assembling
- optimization
- code emission

The stages of linking are:
- relocation
- reference resolving

## Compiling
This input to the compiler is a translation unit; this is a text file containing source code (e.g. `SOMETHING.cpp`). The compiler output will be a collection of binary object files, one for each translational unit.

### Pre-processing
As suggusted from its name, this stage deals with preprocessor macros. This includes turning`#define` statements into constants, converting macro definitions into code where needed, conditionally excluding code based on `#if`, `#elif`, and `#endif` directives.

Here are some examples of code that the preprocessor would deal with.
```cpp
# define SOME_CONSTANT 1
int i = SOME_CONSTANT + 1;

#ifdef test
int someFunction() {
    return 0;
}
#endif
```

### Linguistic Analysis
This is also known as the "complainer stage". This stage is further broken down into three stages:

1. Lexical Analysis
This means breaking down the source code into non-divisible tokens. This means breaking down your source code into keywords (e.g. `int`, `double`, `float`), separators (e.g. `;`), operators (e.g. `+`, `-`), identifiers (e.g. a variable `x`), literals (e.g. `4`) etc.

2. Parsing/Syntax Analysis
This state concatenates the extracted tokens inthe chains of tokens and verifies if the ordering makes sense to the programming language. Some examples of C syntax rules include: separating statements with semicolon, ensuring conditional if statements are enclosed with curly braces.

3. Semantic Analysis
Semantic analysis involves discovering whether the syntactically correct statements actually make sense. For instance, suppose the compiler saw the line: `++x`. If `x` were an `int`, the behaviour would be defined. However, if `x` were a `float` the behaviour would be undefined.

### Assembling
Here, the compiler will turn the code into CPU instructions. suppose you had a file `function.c`, on x86 platforms you can view it's assembly output in Intel or AT&T format using:
```consolve
$ gcc -S -masm=att function.c -o function.s
$ gcc -S -masm=intel function.c -o function.s
```

### Optimization
In this stage, the compiler tries to minimize its use of registers and delete any uneeded code (e.g. unused variables).

### Code Emission
At this stage, the assembly instructions will be converted into binary values corresponding to machine instructions (also known as opcodes) and written to their specific object files.

## Linking
### Relocation
At this point, you have a collection of object files that need to be combined into one single program. The relocation stage moves the zero-based address ranges of the object file code into the address ranges of resultant program memory map.

### Resolving References
When translational units reference each other, they do not know where in the final program these variables or functions will be located. With all of the required code pasted into the final program map, the linker can now replace the dummy addresses use for external references in translational units with the concrete address of the variables/functions.

## Sources
- C and C++ Compiling by Milan Stevanovic
