---
title: "Cpp Blog: Header-only vs. Static vs. Dynamic Libraries"
style:
color:
description: Exploring the most common types of libraries in C++.
---

# Header-only vs. Static vs. Dynamic Libraries

Libraries are collections of reusable code for others to use. Static and dynamic libraries will expose header files that define the functionality of the library in addition to pre-compiled binaries that contain the implementation of the library. Header-only libraries define and implement their library's functionality in its header file(s).

## Header-only

Many popular C++ libraries are implemented as header-only. For instance, [Catch2](https://github.com/catchorg/Catch2) a popular unit testing framework and [nlohmann-json](https://github.com/nlohmann/json) a popular json manipulating framework.

Advantages:

- This is the only option when you are writing a library using templates.
- Clients of the library do not need to compile it beforehand. The header files just need to be added to an existing C++ project.

Disadvantages:

- Compilation times are much greater because the clients of the library will have to compile their code along with your library.
- Your compilation becomes more entangled. When changes are made to the header-only library, the client will have to recompile all of their code.
- Since you cannot create pre-compiled binaries as in static and dynamic libraries, you cannot hide any part of your implementation.

## Static & Dynamic

Static libraries will have a `.lib` (Windows) or a `.a` signature (Linux). These routines will be compiled and linked directly with the client's code.

Dynamic libraries will have a `.dll` (Windows) or a `.so` signature (Linux). These routines will be loaded with the client's application at runtime.

[Googletest](https://github.com/google/googletest) is an unit testing library that can be built as a static of dynamic library.

We compare the static vs. dynamic libraries in the table below

|                             | Static | Dynamic | Explanation                                                                                                              |
| --------------------------- | ------ | ------- | ------------------------------------------------------------------------------------------------------------------------ |
| Avoiding versioning issues  | v      |         | Static libraries are compiled with the project source code so versioning issues will be discovered at compile time.      |
| Reusability across projects |        | v       | Static libraries have to be recompiled for every project that that uses them. Dynamic libraries do not.                  |
| Ease of updating            |        | v       | All projects using a static library need to be recompiled ob library update. Dynamic ones only update the library files. |
| Speed                       | v      |         | Static libraries have a slight speed advantage given that they are compiled with the client code.                        |
| Code size                   |        | v       | Static libraries will incease the size of code in the project binaries.                                                  |

## Sources

- https://stackoverflow.com/questions/140061/when-to-use-dynamic-vs-static-libraries
- https://stackoverflow.com/questions/12671383/benefits-of-header-only-libraries#:~:text=5%20Answers&text=There%20are%20situations%20when%20a,the%20library%20might%20be%20used.
- https://www.learncpp.com/cpp-tutorial/a1-static-and-dynamic-libraries/
- https://medium.com/swlh/linux-basics-static-libraries-vs-dynamic-libraries-a7bcf8157779
