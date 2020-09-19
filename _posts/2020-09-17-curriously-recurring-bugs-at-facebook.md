---
title: "Cpp Blog:  Curiously Recurring C++ Bugs at Facebook"
tags: [cpp]
style:
color:
description: Summary from a CppCon 2017 talk by Louis Brandy
---

[![Curriously Recurring Bugs at Facebook](http://img.youtube.com/vi/lkgszkPnV8g/0.jpg)](http://www.youtube.com/watch?v=lkgszkPnV8g "")


This is my summary of a CppCon talk by Louis Brandy on the "Curiously Recurring C++ Bugs at Facebook".

## Bug 1: `std::vector::operator[]`, `array[]`, `*ptr`
It is quite common for us to check the index of an element outside of a vector's range.

GCC and Clang both have a `-fsanitize=address` option helps you check for errors like these. Typically, you can build your unit tests with this option.

## Bug 2: `std::map::operator[]`
In C++ square bracket operator will actually create an element if the element you are trying to find does not exist. If the value in the map is an `int`, it will be initialized to 0.
```cpp
map<string, int> mp;
mp["hey"] = 12;
std::cout << mp["hye"]; // THIS PRINTS: 0
```

Brandy does not have many suggestions for this bug other than maybe using the `at()` operator in `std::map` which will throw if the key is not in the map.

## Bug 3: Smuggling a reference to a temporary through a function
A common appearance of smuggling references occurs when people create a function that tries to get a value from a map using a given key that returns a default value if not found.
```cpp
// INCORRECT
auto &v = get_or_return_default(my_map, key, "123");
```

This code is incorrect. If `key` is not in `my_map`, `v` is a reference to a temporary that has now been destroyed.

Brandy's method of dealing with this bug is using the `-fsanitize-address-use-after-scope` flag to identify these issues.

## Bug 4: Volatile
Some programmers mistakenly believe that volatile will make their code threadsafe and thus use misuse the keyword.

To address this, Brandy added a lint rule for all usecases of `volatile`.

## Bug 5: `std::shared_ptr`
Some programmers mistakenly believe that `std::shared_ptr` is threadsafe. Only the control block of `std::shared_ptr` is threadsafe, not the actual pointer.

Brandy mitigates these issues by using thread sanitizers and using atomic shared pointer implementations.

Another common issue is when programmers try to dereference a shared pointer immediately after it is returned from a function.
```cpp
auto& ref = *returns_a_shared_ptr();
ref.do_something(); // whoops, ref might be empty
```
After you dereference the shared pointer above, the shared pointer is gone so you will lose the protection count of a shared pointer. Luckily, sanitizers can catch this problem too.

## Bug 6: Not locking
Sometimes when programmers declare a `unique_lock` they will forget to add a variable name to it. As such, the lock is never initialized and the code is not threadsafe.
```cpp
unique_lock<mutex> g(m_mutex); // works fine
unique_lock<mutex> (m_mutex); // oops I forgot the g
```

Brandy's method of recommended way of mitigating this is using the `-Wshadow` to find this. He saids that Facebook has programmed their linters to find this.

## Conclusions
The lesson from this talk is that tools are your best weapon. Enabling certain compiler flags for address and thread santizers can help you prevent many unwanted bugs.





