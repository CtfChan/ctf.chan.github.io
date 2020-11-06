---
title: "Cpp Blog: Conccurency"
tags: [cpp]
style: fill
color:
description: Doing things at the same time
---

# Back to Basics: Concurrency
[![](http://img.youtube.com/vi/F6Ipn7gCOsY/0.jpg)](http://www.youtube.com/watch?v=F6Ipn7gCOsY "Concurrency")

These are my notes from Arthur O'Dwyer's talk from CppCon 2020 titled "Back to Basics: Concurrency".

## Thread-safe static initialization
Since C++11, the initialization of static variables is guaranteed to be thread-safe. After the first thread's arrival the other threads will block and wait for the first to succeed or fail with an exception. This technique can be useful for singleton classes

```cpp
inline auto& SingletonFoo::getInstance() {
    static SingletonFoo instance;
    return instance;
}
```
## Initializing Members with `once_flag`
For non-static members initialization, we can ensure thread safety using the `once_flag`. This flag has a free function `std::call_once` that you can use to initialize a member variable.

The example below illustrates how you could use the `once_flag` to initialization the `conn_` variable by passing a lambda to `std::call_once`.

```cpp
class Logger {
    std::once_flag once_;
    std::optional<NetworkConnection> conn_;

    NetworkConnection& getConn() {
        std::call_once(once_, []{
            conn_ = NetworkConnection(defaultHost);
        });
        return conn_;
    }
};
```

## `shared_mutex` reader/writer lock
There are two ways to lock a `shared_mutex`. Using `unique_lock` as in `void set(...)` below will acquire an exclusive/writer lock. Using `shared_lock` as in `int get(...)` below you will acquire a reader lock meaning.

```cpp
class ThreadSafeConfig {
    std::map<std::string, int> settings_;
    mutable std::shared_mutex rw_;

    void set(const std::string& name, int value) {
        std::unique_lock<std::shared_mutex> lk(rw_);
        settings_.insert_or_assign(name, value);
    }

    int get(const std::string& name) const {
        std::shared_lock<std::shared_mutex> lk(rw_);
        return settings_.at(name);
    }
};
```

## C++20 `counting_semaphore`
Semaphore is thread safe counter; O'Dwyer likens it to a bag of chips. `.acquire()` removes a chip from the bag. If there are no chips available, it will block until a chip is available. `release()` returns a chip to the bag.

```cpp
class AnonymousTokenPool {
    std::counting_semaphore<256> sem_{100}; // 256 = max count, 100 = initial counter value
    void getToken() {
        sem_.acquire();
    }

    void returnToken() {
        sem_.release();
    }
};
```

A more RAII based approach to the token class above is the wrap the semaphores in a `unique_ptr` that calls `.release()` in its destructor.
```cpp
using Sem = std::counting_semaphore<256>;
struct SemReleaser {
    bool operator()(Sem *s) const { s->release(); }
};

class AnonymousTokenPool {
    Sem sem_{100};
    using Token = std::unique_ptr<Sem, SemReleaser>;
    Token borrowToken() {
        sem_.acquire();
        return Token(&sem);
    }
};
```

## C++20 `std::latch`
Latch is a threadsafe counter that starts positive and counts down to zero. It is gate that waits for everyone to arrive at this point, then unblocks everyone simultaneously. It supports three main functions:
- `latch.wait()` -> block until counter reaches 0
- `latch.count_down()` -> decrements the counter, if counter reaches zero, we unblock all waiters
- `latch.arrive_and_wait()` -> decrements and begins waiting

Here is an example of using a latch to synchronize thread outputs.
```cpp
std::latch myLatch(1);
std::thread threadB = std::thread([&](){
    myLatch.wait();
    std::cout << "hello B" << std::endl
});
std::cout << "hello A" << std::endl
myLatch.arrive();
threadB.join();
std::cout << "hello A again" << std::endl
```

## C++20 `std::barrier`
Barrier is a resettable latch. Once the the counter reaches 0, it unblocks all threads waiting for it and atomically refreshes the counter. `std::barrier` exposes a `.wait()`, `.count_down()` and `.arrive_and_wait()` function as with latches.

In the code snippet below, you should see A and B both say that they are setting up, green flag then A and B is running.
```cpp
std::barriar b(2, []{ puts("green flag, go!"); } );
std::thread threadB = std::thread([&](){
    std::cout << "B is setting up" << std::endl
    b.arrive_and_wait();
    std::cout << "B is running" << std::endl
});
std::cout << "A is setting up" << std::endl
b.arrive_and_wait();
std::cout << "A is running" << std::endl
threadB.join();
```





