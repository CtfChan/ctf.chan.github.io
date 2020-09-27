---
title: "Cpp Blog: Move Semantics Part 2"
tags: [cpp]
style:
color:
description: We like to move it move it (part 2).
---

[![Move Semantics Part 2](http://img.youtube.com/vi/pIzaZbKUw2s/0.jpg)](http://www.youtube.com/watch?v=pIzaZbKUw2s "")

## Forwarding References
A forwarding reference is like a shapeshifter. It is an lvalue if initialized by an lvalue and rvalue if initialized by an rvalue. They typically appear in the form of `T&&` or `auto&&` and are used when type deduction is involved.
```cpp
template <typename T>
void f(T&& x);

auto&& var2 = var1;
```

## Reference Collapsing
The rules below are used by the compiler to collapse lvalue and rvalue references adjacent to one another.
`& &` -> `&`
`&& &` -> `&`
`& &&` -> `&`
`&& &&` -> `&&`


## Perfect forwarding
`std::forward` will considtionally cast its input into an rvalue reference. Given an lvalue, it will cast the input to lvalue. Given an rvalue, it will cast the input to an rvalue

```cpp
template <typename T>
T&& forward(std::remove_reference_t<T>& t) noexcept {
    return static_cast<T&&>(t);
}
```

Suppose I pass an lvalue, so we replace `T` with `SomeClass&`. By reference collapsing `SomeClass& &&` become `SomeClass&`. Remove reference will strip lvalue or value references from the `SomeClass` type so `std::remove_reference_t<SomeClass&>& t` evaluates to `SomeClass& t`. The resulting forward function instantiated is.
```cpp
template < >
SomeClass& forward(SomeClass& t) noexcept {
    return static_cast<SomeClass&>(t);
}
```

Suppose I pass an rvalue, so we replace `T` with `SomeClass&&`. Similar to the logic above, `SomeClass&& &&` become `SomeClass&&` and `std::remove_reference_t<SomeClass&&>& t` evaluates to `SomeClass& t`.The resulting forward function instantiated is.
```cpp
template < >
SomeClass&& forward(SomeClass& t) noexcept {
    return static_cast<SomeClass&&>(t);
}
```

## Perfect Forwarding
With the help of perfect forwarding we can forward the type of an argument into a function into another function being called. To illustrate the power of perfect forwarding we use the example of the `make_unique` function. A naive approach might be similar to the ones below.
```cpp
// the argument `arg` is always copied into function and passed as lvalue to T
template<typename T, typename Arg>
unique_ptr<T> make_unique(Arg arg) {
    return unique_ptr<T>(new T(arg) );
}

// `arg` is passed as lvalue to T, but what if I want an rvalue to be passed to T :(
template<typename T, typename Arg>
unique_ptr<T> make_unique(Arg&& arg) {
    return unique_ptr<T>(new T(arg) );
}
```

With `std::forward` we can forward the type of `args` into `T`'s constructor. So `T` can now be initialized using an rvalue or lvalue without the need for expensive copies.
```cpp
template<typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T>(new T(std::forward<Args>(args)... ) );
}
```


## Move
The machanics of `std::move` is very similar to `std::foward` in that it uses `std::remove_reference_t` to gernerate the correct reference type.
```cpp
template <typename T>
std::remove_reference_t<T>&& move(T&& t) noexcept {
   return static_cast<std::remove_reference_t<T>&&>(t);
}
```



