
---
title: "Cpp Blog: Exceptions"
tags: [cpp]
style:
color:
description: Exceptional exceptions
---

This post contains my notes from Klaus Iglberger's talk on Exceptions from CppCon 2020.

[Back to Basics: Exceptions - Klaus Iglberger - CppCon 2020 - YouTube](https://www.youtube.com/watch?v=0ojB8c0xUd8&ab_channel=CppCon)

## Intro to Exceptions
Exceptions is a widely debated topic in the C++ community. Some companies use it and some companies completely ban the use of exceptions in their code bases. There are four main reasons why people are sometimes against using exceptions:
1. When an error occurs during an exception, it will incur a significant penalty.
2. Exceptions make it hard to reason about functions since we typically expect functions to have one set of output(s).
3. Exceptions rely of dynamic memory. On some embedded platforms this might not be available or desirable.
4. Exceptions increase the size of your binary file.


## Mechanics of an Exception
An exception is usually characterized by three keywords `throw`, `try` and `catch`. Below is a snippet of code that demonstrates the use of exceptions in C++ code. Your `main()` will try to call `someFunction()`, if the error conditions occurs then `someFunction()` will throw an exception which we will catch.

```cpp
void someFunction() {
    std::string s = "blah blah";
    if ( /* some error condition */) {
        throw std::runtime_error("you messed up...");
    }
    // ...
}

int main() {
    try {
        // ...
        someFunction();
    } catch( std::exception const& ex )  {
        /* handle the exception */
    }
}
```

In the case of an exception, a stack unwinding will occur in which objects on the stack will be destroyed in reverse order. So in this case, when `std::runtime_error` is thrown, we will destroy `s` then any stack initializations that happen before `someFunction()` in the `try` block.

If an exception is not caught, no stack unwinding will occur and `std::terminate` will be called. This could potentially lead to leaked resources.

# When to use exceptions
There are three scenarios in which you would want to use an exception:
1. For errors that occur very rarely.
2. For errors that cannot be dealt with locally. For instance, if a file is not found in a function or a key is not in the map.
3. For operators and constructors. These functions do not have any other mechanisms for error feedback.

# When not to use exceptions
1. For errors that are expected to fail frequently.
2. For functions that are expected to fail sometimes (e.g converting string to int). Use `std::optional` or `boost::expected` in these cases.
3. If you require guaranteed response times.
4. Things that should never happen (e.g. dereference nullptrs or out-of-range access). Use `assert` in these scenarios.

# Tips on using exceptions
It is best practice to build on the standard `std::execption` when using exceptions. You should try to throw by rvalue to efficiency reasons. Also, it is best to catch by const references to avoid unnecessary copies.


# Exception Safety Guarantees
Iglberger identifies 4 levels of exception safety guarantees.
1. exception unsafe
    - Mo guarantees with respect to invariants and resources
2. basic exception safety gurantees
    - invariants are preserved
    - no resources are leaked
3. strong exception guarantees
    - invariants are preserved
    - no resources are leaked
    - no state change
    - not always possible
4. no-throw guarantee
    - the operation cannot fail
    - expressed in code with noexcept

Striving for a no-throw guarantee is always the best option. Below is an example of a class that follows this guarantee.

```cpp
class Widget {
private:
    int i = 0;
    std::string s;
    std::unique_ptr<Resource> ptr;

public:
    // copy constructor
    Widget( Widget const& w) :
        i { w.i },
        s { w.s }
    {
        if (w.ptr)
            ptr = std::make_unqiue<Resource> ( *w.ptr );
    }

    // Copy assignment operator
    Widget& operator=(Widget const& w) {
        if (this === &w)
            return *this;

        Widget tmp(w); // temp-move idiom
        *this = std::move(tmp);

        return *this;
    }

    Widget& operator=( Widget&& w) noexcept;
}
```








