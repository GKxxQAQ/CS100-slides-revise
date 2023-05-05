---
marp: true
math: mathjax
---

# CS100 Lecture 22

---

## Contents

Standard Template Library (STL)

- Overview
- Sequence containers and iterators
- Algorithms and function objects (aka "functors")
- More on iterators
- Associative containers

---

# Overview of STL

---

## **S**tandard **T**emplate **L**ibrary

Added into C++ in 1994.

- Containers
- Iterators
- Algorithms
- Function objects
- Some other adapters, like container adapters and iterator adapters
- Allocators

---

## [Containers](https://en.cppreference.com/w/cpp/container)

- Sequence containers
  - `vector`, `list`, `deque`, `array` (since C++11), `forward_list` (since C++11)
- Associative containers
  - `set`, `map`, `multiset`, `multimap` (often implemented with *binary search trees*)
- Unordered associative containers (since C++11)
  - `unordered_set`, `unordered_map`, `unordered_multiset`, `unordered_multimap` (implemented with *hash tables*)
- Container adapters: provide a different interface for sequential containers, but they are not containers themselves.
  - `stack`, `queue`, `priority_queue`
  - (since C++23) `flat_set`, `flat_map`, `flat_multiset`, `flat_multimap`

---

## [Iterators](https://en.cppreference.com/w/cpp/iterator)

### Without iterators:

- Traverse an array
  ```cpp
  for (int i = 0; i != sizeof(a) / sizeof(a[0]); ++i)
    do_something(a[i]);
  ```
- Traverse a `vector`
  ```cpp
  for (std::size_t i = 0; i != v.size(); ++i)
    do_something(v[i]);
  ```
- Traverse a linked-list?
  ```cpp
  for (ListNode *p = l.head(); p; p = p->next)
    do_something(p->data);
  ```

---

## [Iterators](https://en.cppreference.com/w/cpp/iterator)

A generalization of pointers, used to access elements in different containers **in a uniform manner**.

### With iterators:

The following works no matter whether `c` is an array, a `std::string`, or any container.

```cpp
for (auto it = std::begin(c); it != std::end(c); ++it)
  do_something(*it);
```

**Equivalent way: range-based for loops**

```cpp
for (auto &x : c) do_something(x);
```

---

## [Algorithms](https://en.cppreference.com/w/cpp/algorithm)

The algorithms library defines functions for a variety of purposes:
- searching, sorting, counting, manipulating, ...

Examples:

```cpp
// assign every element in `a` with the value `x`.
std::fill(a.begin(), a.end(), x);
// sort the elements in `b` in ascending order.
std::sort(b.begin(), b.end());
// find the first element in `b` that is equal to `x`.
auto pos = std::find(b.begin(), b.end(), x);
// reverse the elements in `c`.
std::reverse(c.begin(), c.end());
```

---

## [Algorithms](https://en.cppreference.com/w/cpp/algorithm)

Example: Map every number in `data` to its rank. (“离散化”)

```cpp
auto remap(const std::vector<int> &data) {
  auto tmp = data;
  std::sort(tmp.begin(), tmp.end()); // sort
  auto pos = std::unique(tmp.begin(), tmp.end()); // drop duplicates
  auto ret = data;
  for (auto &x : ret)
    x = std::lower_bound(tmp.begin(), pos, x) - tmp.begin(); // binary search
  return ret;
}
```

---

## Function objects

Things that look like "functions": *Callable*
- functions, and also function pointers
- objects of a class type that has an overloaded `operator()` (the function-call operator)
- lambda expressions

More in later lectures ...

---

# Sequence containers and iterators

Note: `string` is not treated as a container but behaves much like one.

---

## Sequence containers

- `std::vector<T>`: dynamic contiguous array (we are quite familiar with)

<a align="center">
  <img src="img/vector.png", width=400>
</a>

- `std::deque<T>`: **D**ouble-**E**nded **Que**ue (often pronounced as "deck")
  - `std::deque<T>` supports fast insertion and deletion **at both its beginning and its end**. (`push_front`, `pop_front`, `push_back`, `pop_back`)

<a align="center">
  <img src="img/deque.png", width=400>
</a>

- `std::array<T, N>`: same as `T[N]`, it is a **container**
  - It will never decay to `T *`.
  - Container interfaces are provided: `.at(i)`, `.front()`, `.back()`, `.size()`, ..., as well as iterators.

---

## Sequence containers

- `std::list<T>`: doubly-linked list
  - `std::list<T>` supports fast insertion and deletion **anywhere in the container**,
  - but fast random access is not supported (i.e. no `operator[]`).
  - Bidirectional traversal is supported.

<a align="center">
  <img src="img/list.png", width=400>
</a>

- `std::forward_list<T>`: singly-linked list
  - Intended to save time and space (compared to `std::list`).
  - Only forward traversal is supported.

<a align="center">
  <img src="img/forward_list.png", width=400>
</a>

---

## Interfaces

STL containers have consistent interfaces. See [here](https://en.cppreference.com/w/cpp/container#Member_function_table) for a full list.

Element access:

- `c.at(i)`, `c[i]`: access the element indexed `i`. `at` performs bounds checking, and throws `std::out_of_range` if `i` exceeds the valid range.
- `c.front()`, `c.back()`: access the front/back element.

---

## Interfaces

Size and capacity: `c.size()` and `c.empty()` are what we already know.

- `c.resize(n)`, `c.resize(n, x)`: adjust the container to be with exactly `n` elements. If `n > c.size()`, `n - c.size()` elements will be appended.
  - `c.resize(n)`: Appended elements are **value-initialized**.
  - `c.resize(n, x)`: Appended elements are copies of `x`.
- `c.capacity()`, `c.reserve(n)`, `c.shrink_to_fit()`: only for `string` and `vector`.
  - `c.capacity()` returns the capacity (number of elements that *can* be stored in the current storage)
  - `c.reserve(n)`: reserves space for at least `n` elements.
  - `c.shrink_to_fit()`: requests to remove the unused capacity, so that `c.capacity() == c.size()`.

---

## Interfaces

Modifiers:

- `c.push_back(x)`, `c.emplace_back(args...)`, `c.pop_back()`: insert/delete elements at the end of the container.
- `c.push_front(x)`, `c.emplace_front(args...)`, `c.pop_front()`: insert/delete elements at the beginning of the container.
- `c.clear()` removes all the elements in `c`.

---

## Interfaces

Modifiers:

- `c.insert(...)`, `c.emplace(...)`, `c.erase(...)`: insert/delete elements at a specified location.
  - **Warning**: For containers that need to maintain contiguous storage (`string`, `vector`, `deque`), insertion and deletion somewhere in the middle can be **very slow** ($O(n)$).
  - These functions have a lot of overloads. Remember a few common ones, and STFW (Search The Friendly Web) when you need to use them.

---

## Iterators

