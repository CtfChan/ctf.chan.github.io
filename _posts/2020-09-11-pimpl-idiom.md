---
title: "Cpp Blog: Pimpl Idiom C++"
tags: [cpp, idiom]
style:
color:
description: The pointer to implmentation idiom.
---

# PIMPL

## What is PIMPL?

The PIMPL C++ idiom is typically used to reduced compile-time dependencies and construct C++ library interfaces with a stable ABI (application binary interface). This technique involves moving the implementation details of a class from it's object representation into a separate class accessed through an opaque pointer.

Below is a sample implementation of the PIMPL idiom in C++.

**Header File**

```cpp
class VisibleClass {
  VisibleClass();
  ~VisibleClass();
  // ...
private:
  class HiddenImpl; // things to hide
  unique_ptr<HiddenImpl> pimpl_; // opaque ptr
};
```

**Implementation File**

```cpp
class VisibleClass::HiddenImpl {
  // ...
};

VisibleClass::VisibleClass() : pimpl_(new HiddenImpl) {}
VisibleClass::~VisibleClass() = default;
```

In C++, deleting a pointer in which its type is not defined (only forward declared) will lead to undefined behaviour. To prevent this, `unique_ptr` will check if the definition of the type its pointer to is visible before calling delete in its destructor. If we were to not declare the destructor of `VisibleClass`, the compiler would generate an inlined version of it for us in the header file. Since the type of `HiddenImpl` is incomplete in the header file, your code will not compile. The solution is to define your own destructor so that by the time `unique_ptr` tries to deallocate the resources for `HiddenImpl` in Line 6 of the implementation file, `class VisibleClass::HiddenImpl;` will be fully defined.

It is recommended that you place all private nonvirtual members into the `HiddenImpl` class. In doing so, you eliminate the need to pass a back pointer to access members in the `VisibleClass` from `HiddenImpl`. Unfortunately you cannot hide virtual members in the member functions of `HiddenImpl`.

## When to use?

There are several reasons why you might want to use PIMPL in your code:

1. Decrease Compilation times
   Whenever the header file of class `X` changes, every client that uses `X` has to be recompiled. With PIMPL, only the implementation of `X` has to be recompiled when you makes changes to the underlying fields/methods.
2. Binary Compatibility
   Adding new fields/methods to `HiddenImpl` will not affect the binary interface your end user sees. So changes are less high risk.
3. Hiding Implementations/Data
   Because your source files will be compiled into binaries/libraries, when you share your headers and librar (.so or .a file) you will have hidden part of your implementation from the end user.

## Sources:

- [PImpl - cppreference.com](https://en.cppreference.com/w/cpp/language/pimpl)
- [PImpl Idiom in C++ with Examples - GeeksforGeeks](https://www.geeksforgeeks.org/pimpl-idiom-in-c-with-examples/)
- [c++ - Is the pImpl idiom really used in practice? - Stack Overflow](https://stackoverflow.com/questions/8972588/is-the-pimpl-idiom-really-used-in-practice)
- [How to implement the pimpl idiom by using unique_ptr - Fluent C++](https://www.fluentcpp.com/2017/09/22/make-pimpl-using-unique_ptr/)
- [GotW #100: Compilation Firewalls (Difficulty: 6/10) – Sutter’s Mill](https://herbsutter.com/gotw/_100/)
