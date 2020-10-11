---
title: "Cpp Blog: Atomics, Locks and Tasks"
tags: [cpp]
style:
color:
description: Concurrency basics in C++
---

#  Cpp Blog: Atomics, Locks and Tasks

These are my notes from CppCon from  Rainer Grimm's talk titled "Atomics, Locks and Tasks".

[![Atomics, Locks, and Tasks](http://img.youtube.com/vi/o0i2fc0Keo8/0.jpg)](http://www.youtube.com/watch?v=o0i2fc0Keo8 "")



## Definitions
A **data race** occurs when two threads access shared state concurrently and at least one thread tries to modify the shared state. This could lead to undefined behaviour.

A **critical section** is a shared state which can only be accessed concurrently by at most one thread.

A **deadlock** is a state in which one thread is blocked forever because it is waiting for the release of a resource it can never acquire.



## Mutex
`std::mutex` is a conccurrency primitive that ensures only one thread can enter a critical section at once. It supports several locking related methods such as `lock()`, `unlock()` and `try_lock()`.

Typically `std::mutex` is wrapped with `std::unique_lock` or `std::lock_guard` which are RAII (resource acquisition is initialization) wrappers. Both wrappers will call `lock()` on a given mutex on construction and call `unlock()` on the given mutex on destruction. A `unique_lock` offers a richer set of functionalities over a `lock_guard`. For instance, `unique_lock` allows the mutex to be repeatedly locked and unlocked.

```cpp
#include <mutex>

std::mutex m;

// using lock_guard
{
    std::lock_guard<std::mutex> lock(m1);
    // critical section
}



// using unique_lock
{
    std::unique_lock<std::mutex> lock(m1);
    // critical section
}
```

## Atomics
Atomic variables allows threads to concurrently access and update variables without data races. These synchronizations are typicaly achieved with hardware. `std::atomics` are typically used for primitive types such as `int`, `bool`, `float`, etc.

Here is an example of multiple threads concurrently updating an atomic variable
```cpp
auto constexpr num = 100;
std::vector<std::thead> vec(num);
std::atomic<int> i;

// create num threads that each update i 10 times
for (auto &t : vec) {
    t = std::thread([&i]{
        for (int n = 0; n < 10; ++n) ++i;
    })
}

// join our threads
for (auto &t : vec)
    t.join();
```


## Acquiring Multiple Locks at the Same Time
Suppose Jim and Kate were sitting at the dinner table and there was only one pair of chopsticks. If each of them have a chopstick, one of them will have to yield their chopsticks otherwise no one gets to eat. It is best if one person acquires both sticks at the same time.

Similarly, threads should acquire mutexes of interest at the same time otherwise we could have a deadlock. `std::lock` accepts an arbitruary number of mutexes that it will attempt to lock at the same time.

```cpp
std::unique_lock<std::mutex>(mutex1, std::defer_lock);
std::unique_lock<std::mutex>(mutex2, std::defer_lock);
std::lock(mutex1, mutex2); // can lock arbitrary of unique locks in an atomic fashion
```

In C++17, we can use `std::scoped_lock` which serves the exact function as `std::lock` except you do not need to declare the `unique_locks` before the function call.

```cpp
std::scoped_lock<std::mutex> scoped_lock(mutex1, mutex2);
```

## Scoped Thread and `jthread`
When we spawn an `std::thread` we must always remember to call `.join()` or `.detach()` on it. Otherwise, our program will throw an error. Usually we will write RAII wrappers around our threads to automatically do this for us. Here is an example:

```cpp
class ScopedThread {
private:
    std::thread t_;
public:
    explicit ScopedThread(std::thread t) : t_(std::move(t)) {
        if (t.joinable() == false)
            throw std::logic_error("No thread");
    }

    ~ScopedThread() {
        t.join();
    }

    // delete copy and copy assignment operator
    // we don't wanna end up with multiple instances of
    // ScopedThread calling join() on the same thread
    ScopedThread(ScopedThread&) = delete;
    ScopedThread& operator=(ScopedThread const &) = delete;
}
```
## Tasks
![Visualization of Tasks from Presentation](/assets/tasks.png)

Grimm describes tasks in C++ as a data channel where you have a sender with an `std::promise` and a receiver using `std::future`.

## Tasks vs Threads
Here is a small code example comparing threads with tasks in C++.

```cpp
int result;
std::thread t([&]{ result = 3 + 4; });
t.join();
std::cout << res << std:::endl;

// std::async is a promise
auto fut = std::async([]{ return 3 + 4; });
std::cout << fut.get() << std:::endl;
```

Grimm shared a wonderful table in his presentation outlining the difference between tasks and threads in C++.
| Characteristic            | Thread                              | Task                                 |
|---------------------------|-------------------------------------|--------------------------------------|
| header                    | `<thread>`                            | `<future>`                             |
| participants              | creator and child thread            | promise and future                   |
| communication             | shared variable                     | communication channel                |
| synchronization           | `join` call waits                   | `get` call blocks                    |
| exception in child thread | child and creator thread terminates | return value of the get call         |
| kind of communication     | values                              | values, notifications and exceptions |



## Parallel STL
Since C++17 the standard library has offered many parallelized versions of common algorithms such as `sort`, `count`, `transform` etc. The STL offers three execution policies for these algorithms:
1. `std::execution::seq` = do sequentially in one thread
2. `std::execution::par` = parallel execution
3. `std::execution::par_unseq` = parallel and vectorized (uses SIMD for even greater speedup)

Note that (3) might not necessarily offer any speedup depending on your hardware and what version of compiler you're using.


## Condition Variables
Condition variables (cv) enable you to synchronize threads. You can have a sender thread send notifications using `cv.notify_one()` or `cv.notify_all()`. You can have you receiver thread wait for notifications using
`cv.wait(lock, ...)`, `cv..wait_for(lock, relTime, ...)` or `cv.wait_until(lock, absTime, ...)`. The `...` is used for a boolean condition to tackle the lost wakeup and spurious wakeup problem. The lost wakeup happens when the sender thread notifies the receiver thread before it begins waiting so the wakeup is lost forever. The spurious wakeup happens when a receiving thread wakes up, only to find that the condition it was waiting for is not satisfied yet.

Here is a simple use case for condition variables
```cpp
std::condition_variable cv;
// sender
{
  std::lock_guard<std::mutex> lck(mut); // use lock guard here
  read = true;
}
cv.notify_one();

// receiver
{
  std::unique_lock<std::mutex> lck(mut);
  cv.wait(lck, []{ return ready;}); // ready is a predicate
  // do the work
}
```

Typically promises and futures are preferred over conditional variables. However promises only enable monodirectional thread synchronization where as conditional variables enable bidirectional thread synchronization.