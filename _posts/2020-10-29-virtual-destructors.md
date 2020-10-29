---
title: "Cpp Blog: Virtual Destructors"
tags: [cpp]
style: fill
color:
description: Why are destructors virtual?
---

As a general rule of thumb you should always make the destructor of your base class virtual. Why is that?

Consider the following piece of code which defines a `Base` and a `Derived` class.

```cpp
class Base {
public:
    Base() {
        cout << "Base Constructor" << endl;
    }

    ~Base() {
        cout << "Base Destructor" << endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        cout << "Derived Constructor" << endl;
    }

    ~Derived() {
        cout << "Derived Destructor" << endl;
    }
};

int main()
{
    Base* b = new Derived();
    delete b;
}
```

This piece of code will print:
```
Base Constructor
Derived Constructor
Base Destructor
```

Because the variable `b` was of type `Base*`, we only called the destructor of `Base`. If the `Derived` class were to dynamically allocate memory in its constructor, our call to `delete b` would not cleanup all heap allocations as desired.

To correct this behaviour we need to declare the destructor of `Base` as `virtual`.


```cpp
class Base {
public:
    Base() {
        cout << "Base Constructor" << endl;
    }

    virtual ~Base() {
        cout << "Base Destructor" << endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        cout << "Derived Constructor" << endl;
    }

    ~Derived() {
        cout << "Derived Destructor" << endl;
    }
};

int main()
{
    Base* b = new Derived();
    delete b;
}
```

This piece of code prints:
```
Base Constructor
Derived Constructor
Derived Destructor
Base Destructor
```

This is the desired behaviour that we want. So in general when dealing with polymorphic types in C++, make sure your destructors are virtual.



Sources:
- https://stackoverflow.com/questions/461203/when-to-use-virtual-destructors
- http://www.gotw.ca/publications/mill18.htm
- https://www.geeksforgeeks.org/virtual-destructor/