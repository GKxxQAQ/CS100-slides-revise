---
marp: true
math: mathjax
---

# CS100 Lecture 23

Templates

---

## Contents

- Function templates
- Class templates
- Template specialization
- Non-type template parameters
- Variadic template

---

## Define a function template

```cpp
int compare(int a, int b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
int compare(double a, double b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
int compare(const std::string &a, const std::string &b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
```

---

## Define a function template

```cpp
template <typename T>
int compare(const T &a, const T &b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
```

- Type parameter: `T`
- A template is a **guideline** for the compiler:
  - When `compare(42, 40)` is called, `T` is deduced to be `int`, and the compiler generates a version of the function for `int`.
  - When `compare(s1, s2)` is called, `T` is deduced to be `std::string`, ....
  - The generation of a function based on a function template is called the **instantiation** (实例化) of that function template.

---

## Type parameter

```cpp
template <typename T>
int compare(const T &a, const T &b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
```

- The keyword `typename` indicates that `T` is a **type**.
- Here `typename` can be replaced with `class`. **They are totally equivalent here.** Using `class` doesn't mean that `T` should be a class type.

---

## Type parameter

```cpp
template <typename T>
int compare(const T &a, const T &b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
```

- Why do we use `const T &`? Why do we use `b < a` instead of `a > b`?

---

## Type parameter

```cpp
template <typename T>
int compare(const T &a, const T &b) {
  if (a < b) return -1;
  if (b < a) return 1;
  return 0;
}
```

- Why do we use `const T &`? Why do we use `b < a` instead of `a > b`?
  - Because we are dealing with an unknown type!
  - `T` may not be copyable, or copying `T` may be costly.
  - `T` may only support `operator<` but does not support `operator>`.

**\* Template programs should try to minimize the number of requirements placed on the argument types.**

---

## Type deduction

```cpp
template <typename T, class U>
void fun(const T &, const U &);
```

We may let the compiler deduced the argument types, or write the types explicitly.

```cpp
fun(42, 3.14); // T = int, U = double
std::string s; std::vector<int> v;
fun(s, v); // T = std::string, U = std::vector<int>
fun<int, int>(42, 3.14); // T = U = int, and 3.14 is converted to 3
```

---

## Type deduction

Sometimes the compiler is not provided with enough information to deduce the type.

```cpp
template <typename T, typename U>
void foo(const U &x);
foo(42); // Error! What is T?
foo<std::string>(42); // OK, T = std::string, U = int
```

---

## Type deduction