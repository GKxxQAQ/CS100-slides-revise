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

## Motivation: function template

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

## Motivation: function template

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

## Template argument deduction

To instantiate a function template, every template argument must be known, but not every template argument has to be specified.

When possible, the compiler will deduce the missing template arguments from the function arguments.

```cpp
template <typename To, typename From>
To my_special_cast(From f);
double d = 3.14;
int i = my_special_cast<int>(d);   // calls my_special_cast<int, double>(double)
char c = my_special_cast<char>(d); // calls my_special_cast<char, double>(double)
```

---

## Template argument deduction

Suppose we have the following function

```cpp
template <typename P>
void fun(P p);
```

and a call `fun(a)`, where the type of **the expression `a`** is `A`.

- The type of **the expression `a`** is never a reference: Whenever a reference is used, we are actually using the object it is bound to.
- For example:
  
  ```cpp
  const int &cr = 42;
  fun(cr); // A is considered to be const int, not const int &
  ```

---

## Template argument deduction

Suppose we have the following function

```cpp
template <typename P>
void fun(P p);
```

and a call `fun(a)`, where the type of **the expression `a`** is `A`.

- If `A` is `const`-qualified, the top-level `const` is ignored:
  
  ```cpp
  const int ci = 42; const int &cr = ci;
  fun(ci); // A = const int, P = int
  fun(cr); // A = const int, P = int
  ```

---

## Template argument deduction

Suppose we have the following function

```cpp
template <typename P>
void fun(P p);
```

and a call `fun(a)`, where the type of **the expression `a`** is `A`.

- If `A` is an array or a function type, the **decay** to the corresponding pointer types takes place:
  
  ```cpp
  int a[10]; int foo(double, double);
  fun(a); // A = int[10], P = int *
  fun(foo); // A = int(double, double), P = int(*)(double, double)
  ```

---

## Template argument deduction

Suppose we have the following function

```cpp
template <typename P>
void fun(P p);
```

and a call `fun(a)`, where the type of **the expression `a`** is `A`.

- If `A` is `const`-qualified, the top-level `const` is ignored;
- otherwise `A` is an array or a function type, the **decay** to the corresponding pointer types takes place.

**This is exactly the same thing as in `auto p = a;`!**

---

## Template argument deduction

Let `T` be the type parameter of the template.

- The deduction for the parameter `T x` with argument `a` is the same as that in `auto x = a;`.
- The deduction for the parameter `T &x` with argument `a` is the same as that in `auto &x = a;`. `a` must be an lvalue expression.
- The deduction for the parameter `const T x` with the argument `a` is the same as that in `const auto x = a;`.

The deduction rule that `auto` uses is **totally the same** as the rule used here!

---

## Template argument deduction

Deduction forms can be nested:

```cpp
template <typename T>
void fun(const std::vector<T> &vec);

std::vector<int> vi;
std::vector<std::string> vs;
std::vector<std::vector<int>> vvi;

fun(vi); // T = int
fun(vs); // T = std::string
fun(vvi); // T = std::vector<int>
```

Exercise: Write a function `map(func, vec)`, where `func` is any unary function and `vec` is any vector. Returns a vector obtained from `vec` by replacing every element `x` with `func(x)`.

---

## Template argument deduction

Exercise: Write a function `map(func, vec)`, where `func` is any unary function and `vec` is any vector. Returns a vector obtained from `vec` by replacing every element `x` with `func(x)`.

```cpp
template <typename Func, typename T>
auto map(Func func, const std::vector<T> &vec) {
  std::vector<T> result;
  for (const auto &x : vec)
    result.push_back(func(x));
  return result;
}
```

Usage:

```cpp
std::vector v{1, 2, 3, 4};
auto v2 = map([](int x) { return x * 2; }, v); // v2 == {2, 4, 6, 8}
```

---

## Forwarding reference

Also known as "universal reference" (万能引用).

Suppose we have the following function

```cpp
template <typename T>
void fun(T &&x);
```

A call `fun(a)` happens for an expression `a`.

- If `a` is an rvalue expression of type `E`, `T` is deduced to be `E` so that the type of `x` is `E &&`, an rvalue reference.
- If `a` is an lvalue expression of type `E`, **`T` will be deduced to `E &`**.
  - The type of `x` is `T & &&`??? (We know that there are no "references to references" in C++.)

---

### Reference collapsing

If a "reference to reference" is formed through type aliases or templates, the **reference collapsing** rule applies:

- `& &`, `& &&` and `&& &` collapse to `&`;
- `&& &&` collapses to `&&`.

```cpp
using lref = int &;
using rref = int &&;
int n;
 
lref&  r1 = n; // type of r1 is int&
lref&& r2 = n; // type of r2 is int&
rref&  r3 = n; // type of r3 is int&
rref&& r4 = 1; // type of r4 is int&&
```

---

## Forwarding reference

Suppose we have the following function

```cpp
template <typename T>
void fun(T &&x);
```

A call `fun(a)` happens for an expression `a`.

- If `a` is an rvalue expression of type `E`, `T` is deduced to be `E` so that the type of `x` is `E &&`, an rvalue reference.
- If `a` is an lvalue expression of type `E`, **`T` will be deduced to `E &`** and the reference collapsing rule applies, so that the type of `x` is `E &`.

---

## Forwarding reference

Suppose we have the following function

```cpp
template <typename T>
void fun(T &&x);
```

A call `fun(a)` happens for an expression `a`.

- If `a` is an rvalue expression of type `E`, `T` is `E` and `x` is of type `E &&`.
- If `a` is an lvalue expression of type `E`, `T` is `E &` and `x` is of type `E &`.

As a result, `x` is always a reference depending on the value category of `a`! Such reference is called a **universal reference** or **forwarding reference**.

The same thing happens for `auto &&x = a;`!

---

## Perfect forwarding

A forwarding reference can be used for **perfect forwarding**:

- The value category is not changed.
- If there is a `const` qualification, it is not lost.

Example: Suppose we design a `std::make_unique` for one argument:

- `makeUnique<Type>(arg)` forwards the argument `arg` to the constructor of `Type`.
- The value category must not be changed: `makeUnique<std::string>(s1 + s2)` must move `s1 + s2` instead of copying it.
- The `const`-qualification must not be changed: No `const` is added if `arg` is non-`const`, and `const` should be preserved if `arg` is `const`.

---

## Perfect forwarding

- The value category must not be changed: `makeUnique<std::string>(s1 + s2)` must move `s1 + s2` instead of copying it.
- The `const`-qualification must not be changed: No `const` is added if `arg` is non-`const`, and `const` should be preserved if `arg` is `const`.

The following does not meet our requirements: An rvalue will become an lvalue, and `const` is added.

```cpp
template <typename T, typename U>
auto makeUnique(const U &arg) {
  return std::unique_ptr<T>(new T(arg));
}
```

---

## Perfect forwarding

- The value category must not be changed: `makeUnique<std::string>(s1 + s2)` must move `s1 + s2` instead of copying it.
- The `const`-qualification must not be changed: No `const` is added if `arg` is non-`const`, and `const` should be preserved if `arg` is `const`.

We need to do this:

```cpp
template <typename T, typename U>
auto makeUnique(U &&arg) {
  // For an lvalue argument, U=E& and arg is E&.
  // For an rvalue argument, U=E  and arg is E&&.
  if (/* U is an lvalue reference type */)
    return std::unique_ptr<T>(new T(arg));
  else
    return std::unique_ptr<T>(new T(std::move(arg)));
}
```

---

## Perfect forwarding

`std::forward<T>(arg)`: defined in `<utility>`.

- If `T` is an lvalue reference type, returns an lvalue reference to `arg`.
- Otherwise, returns an rvalue reference to `std::move(arg)`.

```cpp
template <typename T, typename U>
auto makeUnique(U &&arg) {
  return std::unique_ptr<T>(new T(std::forward<U>(arg))).
}
```

How can we allow multiple arguments? Discussed later.

