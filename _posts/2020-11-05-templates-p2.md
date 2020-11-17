---
title: "Cpp Blog: Templates Part 2"
tags: [cpp]
style:
color:
description: Great talk on templates
---

# Back to Basics: Templates Part 2

[![](http://img.youtube.com/vi/0dtjDTEE0hQ/0.jpg)](http://www.youtube.com/watch?v=0dtjDTEE0hQ "")

These are my notes from the second part of Andreas Fertig 's talk from CppCon 2020 titled "Back to Basics: Templates".

## Variadic Templates
Variadic templates allow you to pass a variable number of arguments to a template. It is denoted using three dots `...`.

In the example below, `min` continuously expands the items in the initializer list while finding the min until there are two items left.
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

## Folded Expressions
Folded expressions are used to unpack a parameter pack using an operations. There are two four types of folded expressions listed below. The op denotes some operation of that is of the same type.

unary
- right fold (pack op ...)
- left fold (... op pack)

binary
- right fold (pack op ... op init)
- left fold (init op ... op pack)

In the example below, we use a unary left fold to unpack our parameter pack into `std::cout`.

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


## Variable Templates
Using variable templates we can define constants such as `pi` or `true_type`. The example below illustrates the power of variable templates. Using the `integral_constant` struct we can define a `true_type` and a `false_type`. We then define two templates `is_pointer` that will derive from `false_type` or `true_type` depending on its input. In doing so, we can have a compile time check for the type of a variable.

```cpp
template<class T, T v>
struct integral_constant {
    static constexpr T value = v;
};

using true_type = integral_constant<bool, true>;
using false_type = integral_constant<bool, false>;

// derive from integral_constant types above
template<class T>
struct is_pointer : false_type{};

template<class T>
struct is_pointer<T*> : true_type {};

static_assert(is_pointer<int*>::value);
static_assert(not is_pointer<int>::value);

```

## SFINAE
Substitution failure is not and error (SFINAE) is a technique that gives the compiler hints as to how to compiler our code if its initial subsitution fails.

In this example our compiler decides on the function to instantiate not on the raw type of T, but rather on the result of `std::is_floating_point_v`.
```cpp
template <typename T>
std::enable_if_t<not std::is_floating_point_v<T>, bool> equal(const T& a, const T& b) {
    return a == b;
}


std::enable_if_t<std::is_floating_point_v<T>, bool> equal(const T& a, const T& b) {
    return std::abs(a - b) < 0.0001;
}

equal(2, 1);
equal(2.0, 1.0);
```

## Tag Dispatch
Tag dispatch is an alternative to SFINAE. In this technique, we define empty class tags and pass them to our functions as additional parameters. In doing so, we can overload functions with identical parameters.

In the example below, we define two empty structs `notFloatingPoint` and `floatingPoint` which we pass to the `equal` function to distinguish between the two.
```cpp
namespace internal {
    struct notFloatingPoint {};
    struct floatingPoint {};

    template <typename T>
    bool equal(const T& a, const T& b, notFloatingPoint)  {
        return a == b;
    }

    template <typename T>
    bool equal(const T& a, const T& b, notFloatingPoint)  {
        return std::abs(a - b) < 0.0001;
    }
} // namespace internal

template<typename T>
bool equal(const T& a, const T& b) {
    using namespace internal;

    if constexpr (std::is_floating_point_v<T>) {
        return equal(a, b, floatingPoint{});
    } else {
        return equal(a, b, notFloatingPoint{});
    }
}
```

## C++20 Concepts
Using Concepts in C++20, we can use the `requires` keyword to replace SFINAE.


```cpp
template<typename T>
requires(not std::is_floating_point_v<T>) bool equal(const T& a, const T& b) {
    return a == b;
}

template<typename T>
requires(std::is_floating_point_v<T>) bool equal(const T& a, const T& b) {
    return std::abs(a - b) < 0.0001;
}
```


## Template Template Parameter
You can also pass templates to a template. In the example below we use `template<class, class>` to speciy that this is a template template parameters and then we proceed defining the template as usual.

```cpp
template<
    template<class, class>
    class Container,
    class T,
    class Allocataor = std::allocator<T>>
void Fun(const Container<T, Allocator>& c) {
    for (auto& e : c) {
        printf("%d\n", e);
    }
}

std::vector<int> v {2, 3, 3};
Fun(v);

std::list<char> li {'a', 'b'};
Fun(li);
```
