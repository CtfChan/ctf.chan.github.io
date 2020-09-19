---
title: "Cpp Blog: Return Value Optimization & Named Return Value Optimization"
tags: [cpp]
style:
color:
description: Letting the compiler do the work.
---

# Return Value Optimization (NRVO) & Named Return Value Optimization (NRVO)
Cope samples borrowed from Shahar Mike's Web Spot.

## What is this?
NVO and NRVO are two types of return value optmization that the C++ compiler performs for us.

Consider a call:
```cpp
T some_variable = functionName();
```
The idea is for the compiler to use the memory space for `some_variable` outside of `functionName()`'s scope to directly initialize the object return from the function.

Without RVO or NRVO, the compiler would be creating duplicate copies of the same object. To illustrate this point, consider this definition of `SomeClass` which just couts the type of constructor/operator/destructor being called on it.
```cpp
class SomeClass
{
public:
    SomeClass()
    {
        std::cout << "Constructor called" << std::endl;
    }

    ~SomeClass()
    {
        std::cout << "Destructor called" << std::endl;
    }

    SomeClass(const SomeClass &s)
    {
        std::cout << "Copy constructor called" << std::endl;
    }

    SomeClass(SomeClass &&s)
    {
        std::cout << "Move constructor called" << std::endl;
    }

    SomeClass &operator=(const SomeClass &s)
    {
        std::cout << "Copy assignment called" << std::endl;
        return *this;
    }

    SomeClass &operator=(SomeClass &&s)
    {
        std::cout << "Move assignment called" << std::endl;
        return *this;
    }
};
```

Suppose we have some factory function for `SomeClass` that our code calls on:
```cpp
SomeClass createSomeClassObject() {
    return SomeClass();
}

int main() {
    auto obj = createSomeClassObject();
}
```

We will run the aforementioned code twice. Once with the `-fno-elide-constructors` argument which disables RVO and once with RVO enabled.
```console
$ clang++ -std=c++11 -fno-elide-constructors main.cpp && ./a.out
Constructor called
Move construct called
Destructor called
Move constructor called
Destructor called
Destructor called

$ clang++ -std=c++11 main.cpp && ./a.out
Constructor called
Destructor called
```

With RVO we can see that `SomeClass` is only constructed once. On the other hand, the compiler creates 3 instances of `SomeClass`. The first instance is created when we call `return SomeClass` in `createSomeClassObject()`. The second instance is created for the returned object inside `main()`. The third instance is created for the named object `obj`.

NRVO operates very similar to RVO, except the return object is named within the function.
```cpp
// RVO
SomeClass createSomeClassObject() {
    return SomeClass();
}

// NRVO
SomeClass createSomeClassObject() {
    auto some_class = SomeClass()
    return some_class;
}
```

## When RVO doesn't happen?
- Function returns an instance that is determined at runtime.
- Function returns a parameter to the function or a global variable.
- Function returns by `std::move()`
- Using `operator=` on an existing object.
- Return member variables.


## Sources
- [Return Value Optimization \| Shahar Mike's Web Spot](https://shaharmike.com/cpp/rvo/)
- [Return value optimizations - Fluent C++](https://www.fluentcpp.com/2016/11/28/return-value-optimizations/)



