---
marp: true
math: mathjax
---

# CS100 Lecture 18

---

## Contents

- Rvalue References
- Move Operations
  - Move Constructor
  - Move Assignment Operator
  - The Rule of Five
- `std::move`
- Automatic Move and Copy Elision

---

## Motivation: copy is slow

```cpp
std::string a = some_value(), b = some_other_value();
std::string s;
s = a;
s = a + b;
```

Consider the two assignments: `s = a` and `s = a + b`.

How is `s = a + b` evaluated?

---

## Motivation: copy is slow

```cpp
s = a + b;
```

1. Evaluate `a + b` and store the result in a temporary object, say `tmp`.
2. Perform the assignment `s = tmp`.
3. The temporary object `tmp` is no longer needed. Destroy it by calling its destructor.

Can we make this faster?

---

## Motivation: copy is slow

```cpp
s = a + b;
```

1. Evaluate `a + b` and store the result in a temporary object, say `tmp`.
2. Perform the assignment `s = tmp`.
3. The temporary object `tmp` is no longer needed. Destroy it by calling its destructor.

Can we make this faster?

- The assignment `s = tmp` is done by **copying** the contents of `tmp`.
- But `tmp` is about to die! Why can't we just *steal* the contents from it?

---

## Motivation: copy is slow

Let's look at the other assignment:

```cpp
s = a;
```

- **Copy** is needed here, because `a` lives long. It is not destroyed immediately after this statement is executed.
- You cannot just "steal" the contents from `a`. The contents of `a` must be preserved.

---

## Distinguish between the different kinds of assignments

<div style="display: grid; grid-template-columns: 1fr 1fr;">
  <div>

```cpp
s = a;
```
  </div>
  <div>

```cpp
s = a + b;
```
  </div>
</div>

What is the key difference between them?

- `s = a` is an assignment from an **lvalue**,
- while `s = a + b` is an assignment from an **rvalue**.

If we only have the copy assignment operator, there is no way we can distinguish them.

**\* Define two different assignment operators, one accepting an lvalue and the other accepting an rvalue?**

---

## Rvalue References

A kind of reference that is bound to **rvalues**:

```cpp
int &r = 42;        // Error: lvalue reference cannot be bound to rvalue
int &&rr = 42;      // Correct: `rr` is an rvalue reference
const int &cr = 42; // Also correct:
                    // lvalue reference-to-const can be bound to rvalue
const int &&crr = 42; // Correct, but useless
                      // rvalue reference-to-const is seldom used.

int i = 42;
int &&rr2 = i;           // Error: rvalue reference cannot be bound to lvalue
int &r2 = i * 42;        // Error: lvalue reference cannot be bound to rvalue
const int &cr2 = i * 42; // Correct
int &&rr3 = i * 42;      // Correct
```

- Lvalue references can only be bound to lvalues.
- Rvalue references can only be bound to rvalues.

---

## Overload Resolution

```cpp
void fun(const std::string &);
void fun(std::string &&);
```

- `fun(s1 + s2)` matches `fun(std::string &&)`, because `s1 + s2` is an rvalue.
- `fun(s)` matches `fun(const std::string &)`, because `s` is an lvalue.
- Note that if `fun(std::string &&)` does not exist, `fun(s1 + s2)` also matches `fun(const std::string &)`.

---

# Move Operations

---

## Overview

The **move constructor** and the **move assignment operator**.

```cpp
struct Widget {
  Widget(Widget &&) noexcept;
  Widget &operator=(Widget &&) noexcept;
  // Compared to the copy constructor and the copy assignment operator:
  Widget(const Widget &);
  Widget &operator=(const Widget &);
};
```

- Parameter type is **rvalue reference**, instead of lvalue reference-to-`const`.
- **`noexcept` is necessary!** (Will be covered in later lectures)

---

## The Move Constructor

Take the `Dynarray` as an example.

```cpp
class Dynarray {
  int *m_storage;
  std::size_t m_length;
 public:
  Dynarray(const Dynarray &other) // copy constructor
    : m_storage(new int[other.m_length]), m_length(other.m_length) {
    for (std::size_t i = 0; i != m_length; ++i)
      m_storage[i] = other.m_storage[i];
  }
  Dynarray(Dynarray &&other) noexcept // move constructor
    : m_storage(other.m_storage), m_length(other.m_length) {
    other.m_storage = nullptr;
    other.m_length = 0;
  }
};
```

---

## The Move Constructor

```cpp
class Dynarray {
  int *m_storage;
  std::size_t m_length;
 public:
  Dynarray(Dynarray &&other) noexcept // move constructor
    : m_storage(other.m_storage), m_length(other.m_length) {


  }
};
```

1. *Steal* the resources of `other`, instead of making a copy.

---

## The Move Constructor

```cpp
class Dynarray {
  int *m_storage;
  std::size_t m_length;
 public:
  Dynarray(Dynarray &&other) noexcept // move constructor
    : m_storage(other.m_storage), m_length(other.m_length) {
    other.m_storage = nullptr;
    other.m_length = 0;
  }
};
```

1. *Steal* the resources of `other`, instead of making a copy.
2. Make sure `other` is in a valid state, so that it can be safely destroyed.

**\* Take ownership of `other`'s resources!**

---

## The Move Assignment Operator

**Take ownership of `other`'s resources!**

```cpp
class Dynarray {
 public:
  Dynarray &operator=(Dynarray &&other) noexcept {

      
      m_storage = other.m_storage; m_length = other.m_length;


    return *this;
  }
};
```

1. *Steal* the resources from `other`.

---

## The Move Assignment Operator

```cpp
class Dynarray {
 public:
  Dynarray &operator=(Dynarray &&other) noexcept {

      
      m_storage = other.m_storage; m_length = other.m_length;
      other.m_storage = nullptr; other.m_length = 0;

    return *this;
  }
};
```

1. *Steal* the resources from `other`.
2. Make sure `other` is in a valid state, so that it can be safely destroyed.

Are we done?

---

## The Move Assignment Operator

```cpp
class Dynarray {
 public:
  Dynarray &operator=(Dynarray &&other) noexcept {

      delete[] m_storage;
      m_storage = other.m_storage; m_length = other.m_length;
      other.m_storage = nullptr; other.m_length = 0;

    return *this;
  }
};
```

0. **Avoid memory leaks!**
1. *Steal* the resources from `other`.
2. Make sure `other` is in a valid state, so that it can be safely destroyed.

Are we done?

---

## The Move Assignment Operator

```cpp
class Dynarray {
 public:
  Dynarray &operator=(Dynarray &&other) noexcept {
    if (this != &other) {
      delete[] m_storage;
      m_storage = other.m_storage; m_length = other.m_length;
      other.m_storage = nullptr; other.m_length = 0;
    }
    return *this;
  }
};
```

0. **Avoid memory leaks!**
1. *Steal* the resources from `other`.
2. Make sure `other` is in a valid state, so that it can be safely destroyed.

**\* Self-assignment safe!**

---

## Lvalues are Copied; Rvalues are Moved
