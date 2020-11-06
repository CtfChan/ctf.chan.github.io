---
title: "Cpp Blog: Templates"
tags: [cpp]
style: fill
color:
description: Great talk on templates
---

[![](http://img.youtube.com/vi/VNJ4wiuxJM4/0.jpg)](http://www.youtube.com/watch?v=VNJ4wiuxJM4 "")

# Back to Basics: Templates
These are my notes from Andreas Fertig 's talk from CppCon 2020 titled "Back to Basics: Templates".


## Introduction
Generic programming involves writing algorithms in terms of types that are to be specified later. Templates are commonly used for generic programming tasks in C++.

In C++ there are 3 types of templates
- function templates
- class templates
- variable templates

Templates can take in 3 types of parameters
- type parameter (e.g. `int`, `char` or even class)
- non-type parameter (e.g. typically values like `3`)
- template-template parameter (required when we pass a template as a parameter to a template)


Here is a simple example of a simple function template for `Size`. Our template takes `T` a type parameter and `N` a non-type parameter. This function takes in a raw buffer and returns its size. Our template function will instantiate a new function for different combinations of input types. In this case, two instantiations will be made (one for each N).
```cpp
template <typname T, size_t N>
constexpr auto Size(const T (&)[N]) {
  return N;
}

char buffer[16]{};
static_assert(Size(buffer) == 16);
char buffer2[32]{};
static_assert(Size(buffer2) == 32);
```

## Template Specialization
Template specialization means providing a concrete implementation for some argument combination of a function template. This technique is often used for templates that use integer types and floating point types since different techniques are required to check the equality of the two.

The `equal` function below illustrates a template specialization for `double`.
```cpp
template<typename T>
bool equal(const T& a, const T& b) {
  return a==b;
}

template<>
bool equal(const double&a, const double& b) {
  return std::abs(a-b) < 0.00001;
}

static_asssert(equal(1, 2) == false); // instantiate first
static_asssert(equal(1.0, 2.0) == false); // instantiate second
```

## Class Templates
Similar to functions, classes can also be implemented as templates. Methods for our call templates can be implemented inside or outside of the class declaration.


In the class `Array` below, we define every function outside inside of the class with the exception of `T* data();`. In order to define a class method outside of the class, we need to explicitly state that it's a template as shown below.
```cpp
template<typename T, size_t SIZE>
struct Array {
  T* data();

  const T* data() const {
    return std::addressof (mData[0]);
  }

  constexpr size_t size() const {
    return SIZE;
  }

  T* begin() {
    return data();
  }

  T* end() {
    return data() + size();
  }

  T& operator[](size_t idx) {
    return mData[idx];
  }

  T mData[SIZE];
};

// defining method outside of class
template<typename T, size_t SIZE>
T* array<T, SIZE>::data() {
  return std::addressof(mData[0]);
}
```

## Method templates
Methods inside a class can themselves be templates as well. In the example below, we overload the `operator=` of the class `Foo` that allows us to cast other types `U` to `T`.
```cpp
template<typename T>
class Foo {
private:
    T mX;
public:
    Foo(const T& x) : mX{x} {}

    template<typename U>
    Foo<T>& operator=(const U& u) {
        mX = static_cast<T>(u);
        return *this;
    }
};

Foo<int> fi{3};
fi = 2.5;
```

## Inheritance in Class Templates
Similar to vanilla classes, template classes can inherit from one another. In order to call methods from the base class in the derived class, we can use the `this` pointer or make the name known using `Base<T>::func`.

In the example below, the class `Bar` calls on it's base class `Foo` using the aforementioned techniques.
```cpp
template<typename T>
class Foo {
public:
    void Func() {}
};

template<typename T>
class Bar : public Foo<T> {
public:
    void BarFunc() {
        // Func(); THIS WON'T WORK
        this->Func();
        Foo<T>::Func();
    }
}

Bar<int> b{};
b.BarFunc();
```

## Alias Templates
Alias templates allow you to create synonyms for templates and partial specialization of templates

In the example below, we see that `CharArray` is an alias template for `std::array` using char and of size `N`.
```cpp
#include <array>

template<size_t N>
using CharArray = std::array<char, N>;

CharArray<25> arr;
```

## `std::span`
`std::span` is a helpful addition to C++20 that helps us reduce code bloat. `std::span` is a class template that is used to store a contiguous sequence of objects as well as its length.


In the first version of `Send` defined below, everytime we call `Send` with a different `N` we instantiate a new version of `Send` increasing your binary size. Using `std::span` as a "wrapper" around `std::array`, our compiler will only instantiate one version of `Send`, thus reducing code bloat.

```cpp
// BEFORE
template<size_t N>
bool Send(const std::array<char, N>& data) {
    return write(data.data(), data.size());
}

// AFTER
bool Send(const span<char>& data) {
    return write(data.data(), data.size());
}

// can call both
std::array<char, 1'024> buffer();
Send(buffer);
```


constexpr if
- compile time conditional statements
- only the branches the yield true are preserved
```cpp
template <typname T>
auto getValue(T t) {
    if constexpr(std::is_pointer_v<T>) {
        assert(nullptr != t);
        return *t;
    } else {
        return t;
    }
}

int i = 4;
int *ip = &i;
auto iv = getValue(i);
auto ipv = getValue(ip);
auto itv = getValue(43);
```


----

# part 2

Variadic Templates
- variable number of arguments into templates
- using three dots after typename


```cpp
template <typename T, typename... Ts>
constexpr auto min(T& a, T&b, const Ts&... ts) {
    const auto m = a < b ? a : b;
    if constexpr(sizeof...(ts) > 0 ) {
        return min(m, ts...);
    }
    return m;
}

static_assert( min(3,2,3,4,5) == 2);
static_assert( min(3,2) == 2);

```

`typename... Ts` generates a type template parameter pack. `ts...` unpacks the argument




Folded Expressions
- used to unpack a parameter pack using an operation
- all ops of the same type

unary
- right fold (pack op ...)
- left fold (... op pack)

binary
- right fold (pack op ... op init)
- left fold (init op ... op pack)

```cpp
template<typename T, typename... Ts>
void Print(const T& targ, const Ts&... args)
{
  std::cout << targ;
  aut coutSpaceAndArg = [](const auto& arg) {
    std::cout << ' ' << arg;
  };
  (..., coutSpaceAndArg(args));
}
Print("hello", "C++", 20);
```


```cpp
buildCSVLine
```
- uses left fold


Variable templates
- making variables templates
```cpp


```


SFINAE
- we enable with `std::enable_if_t`
-


tag dispatch as alternative to SINFINAE
- we use empty class tag



C++ replaces most SFINAE concepts
- using requires


template template parameters
- passing templates to another template
- need to use class instead of typename

