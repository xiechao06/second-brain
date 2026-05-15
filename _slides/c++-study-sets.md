---
marp: true
title: "c++-study-sets"
theme: gödel
size: 16:9
---

## What is the const reference to temporary rvalue?

---

In the following snippet:

```c++
std::string get_name() {
  return "John Doe";
}
const std::string& name = get_name();
```

`name` is a const reference to a temporary rvalue returned by get_name(). The temporary object created by get_name() will be destroyed at the end of the full expression, but since name is a const reference, it extends the lifetime of the temporary object until name goes out of scope. This allows us to safely use name without worrying about dangling references.

<!-- here is some comments -->

---

The previous code snippet is roughly equal to:

```c++
// The temporary object returned by get_name() is stored in a local variable temp
std::string temp = get_name();
const std::string& name = temp;
```

---

## How does vector's push_back provide strong exception guarantees?

---

If there isn't enough capacity for the new element pushed back. The vector will allocate a new memory block, and `move` the existing elements to it, since `move` operation is the most efficient. However, if an exception occurs during `move`, vector will not come back to the state before `push_back`, since `move` isn't a recoverable operation. To provide **strong exception guarantees**, vector will `move` the existing elements if the move constructor is marked as `noexcept`, otherwise it will `copy` the existing elements, since `copy` is a recoverable operation.

Check [[what-are-exception-guarantees]] for more details.

---

## What is noexcept specifier and noexcept operator?

---

It indicate a function won’t throw exceptions, like:

```c++
void foo() noexcept {}
```

or a function won’t throw exceptions on certain condition:

```c++
template <typename T> T bar() noexcept<std::is_nothrow_constructible_v<T>> {
 return T{};
}
```

Here `foo` won’t throw any exceptions if T’s constructor `T()` won’t throw.

---

`noexcept` could also work as an operator to check if a function is declared to not throw any exception at compile time:

```c++
std::cout << noexcept(bar<int>()) << std::endl; // output 1
```

---

## What are exception guarantees in C++?

---

The C++ language provides 4 levels of exception guarantees:

* **No exception guarantees**. If an exception occurs, there's no guarantee if the program is left in valid state.

* **Basic exception guarantees**. If an exception occurs, the program is left in a consistent and stable state.

* **Strong exception guarantees**. If an exception occurs, the program will be left in state before the operation started.

* **No-throw exception guarantees**. All operations will complete successfully.

[what-are-exception-guarantees]: what-are-exception-guarantees "what are exception guarantees?"

---

## Explain keyword `decltype` in C++

---

`decltype` preserves the exact type properties of an expression, including const-volatile qualifiers and reference qualifiers. For example:

```c++
// x has no cv-qualifiers and is not a reference
int x{5};
// auto deduces type as int
auto y{x}; 
// decltype deduces type as int
decltype(x) z; 
// decltype deduces type as int& (reference to x), `(x)` is an lvalue expression
decltype((x)) w = x; 
```

and another example:

```c++
const int &x{10};
decltype(x) y{x}; // y is const int&
decltype((x)) z{x}; // z is const int& (reference to x),
```

---

## Why variables that have both *static storage* and *dynamic initialization* are considered as problematic?

---

Variables with static storage are initialized before `main()`, and variables with dynamic initialization are initialized at runtime. This can lead to the **static initialization order fiasco**, where the order of initialization of static variables across different translation units is undefined. If one static variable depends on another static variable that has not yet been initialized, it can lead to undefined behavior. This is why such variables are considered problematic.

---

## how to avoid variables with static storage to have dynamic initialization?

---

Use keyword `constinit` to ensure that a variable with static storage initialized at compile time, like:

```c++
int foo() { return 42; }
constinit int x = foo(); // error: initializer is not a constant expression
constexpr int bar() { return 42; };
constinit int y = bar(); // ok
```

---

## What are immediate functions?

---

Immediate functions must be evaluated at compile time. They are declared with the `consteval` specifier, and only non-member functions or function templates, class constructors could be immediate functions. For example:

---

```c++
consteval int add(int a, int b) {
  return a + b;
} 

struct point {
  consteval point(int x, int y) : x(x), y(y) {}
  int x, y;
};

add(1, 2); // ok
point p{3, 4}; // ok
int a{1}, b{2};
add(a, b); // error: arguments are not constant expressions
point q{a, b}; // error: arguments are not constant expressions
```

---

## What are `constexpr` functions?

---

Functions that are evaluated at compile time if all their arguments are constant expressions. For example:

```c++
constexpr add(int a, int b) {
  return a + b;
}

add(1, 2); // ok, evaluated at compile time
int a{1}, b{2};
add(a, b); // ok, evaluated at runtime
```

---

## How to make a `constexpr` function aware of if it's evaluated at compile time or runtime?

---

Use `std::is_constant_evaluated()` to check if a `constexpr` function is evaluated at compile time or runtime, like:

```c++
constexpr double power(double base, int exp) {
  if (std::is_constant_evaluated()) {
    // evaluated at compile time, use a simple loop
    double result = 1.0;
    for (int i = 0; i < exp; ++i) {
      result *= base;
    }
    return result;
```

---

```c++
  } else {
    // evaluated at runtime, use std::pow for better performance
    return std::pow(base, exp);
  }
}
```

---

## Could virtual functions be evaluated at compile time?

---

In **C++20**, yes, as long as the virtual function is declared as `constexpr` and the invocation is a constant expression. For example:

```c++
struct Base {
  virtual constexpr int foo() const { return 42; }
};
struct Derived : Base {
  virtual constexpr int foo() const override { return 24; }
};
constexpr int call_foo(const Base& b) {
  return b.foo();
}
constexpr Derived d;
constexpr int result = call_foo(d); // ok, evaluated at compile time, returns 24
```

---

## Why `static_cast` to downcast is dangerous?

---

Use `static_cast` to downcast is syntactically correct, but it doesn't perform any runtime checks to ensure that the object being cast is actually of the target type. When downcast, user usually mean to get the target type, if it is not, it leads to undefined errors. For example:

```c++
class Base {
  virtual void foo() {}
}
class Derived : public Base {
  virtual void bar() {}
};

Base b;
Derived d;

Derived* pd1 = static_cast<Derived*>(&b); // ok, but pd1 is a dangling pointer, dereferencing it leads to undefined behavior
pd1->bar(); // undefined behavior
```

---

## why use static_cast to upcast is safe?

---

Because a *derived class* object is also a *base class* object, but not vice versa. So when upcasting, the object being cast is always of the target type. But you seldomly see people use `static_cast` to upcast, since the implicit conversion from derived class pointer/reference to base class pointer/reference is already provided by the language.

---

## what is `const_cast` and when to use it?

---

`const_cast` is used to add or remove `const` qualifier from a variable. It can also be used to add or remove `volatile` qualifier. It is used to cope with legacy code that call a function that doesn't accept `const` parameters, but the parameters are actually not modified by the function. Besides, `const_cast` only works on the syntax level, it doesn't generate any machine instructions

---

## What is `reinterpret_cast`?

---

It tells c++ just interpret the bits as a certain type. That's a type of unsafe cast, it doesn't generate machine instructions too.

---

## What is expression's value category tree

---

In c++, every expression has a *value category* beside *type*, like:

```txt
         expression
        /          \
    glvalue        rvalue
    /     \       /     \
lvalue   xvalue  xvalue  prvalue
```

---

## Explain `move` semantics

---

`move` semantics enable moving resources from one object to another, instead of copying them. This is particularly useful for classes that manage resources such as dynamic memory, file handles, or network connections.
The only movable value category is **xvalue**, which stands for "expiring value". An expression of category xvalue represents an object that is about to be moved from, you use `std::move` (std::static_cast<T&&> actually) to convert a *lvalue* to *xvalue*, for example:

```cpp
std::vector<int> v1{1, 2, 3};
// v1 is an lvalue, but std::move(v1) is an xvalue
// which indicates that v1 can be moved from
std::vector<int> v2 = std::move(v1);
```

---

## Explain the difference between rvalue reference and forward reference

---

An rvalue reference is a reference that can bind to an rvalue, it is declared with `&&`, like:

```c++
void foo(int&& x) { 
  // x is an rvalue reference, it can bind to an rvalue
}
```

---
A forward reference is a reference that can bind to both lvalues and rvalues, it is declared with `&&` in a template context, like:

```c++
template <typename T>
void bar(T&& x) {
  // x is a forward reference, it can bind to both lvalues and rvalues
}

int a = 42;
bar(a); // x is an lvalue reference to int
bar(42); // x is an rvalue reference to int
```

---

## what is difference between `std::move` and `std::forward`?

---

std::move is used to indicate that an object can be moved from, it unconditionally casts its argument to an rvalue reference, while std::forward is used in a template context to preserve the value category of its argument, it conditionally casts its argument to an rvalue reference if it is an rvalue, otherwise it returns the argument as is. For example:

```c++
template <typename T>
void baz(T&& x) {
  // std::forward preserves the value category of x
  qux(std::forward<T>(x));
}
```

In this example, if `baz` is called with an lvalue, `std::forward<T>(x)` will return an lvalue reference, and if `baz` is called with an rvalue, `std::forward<T>(x)` will return an rvalue reference. This allows `qux` to properly handle both *lvalue* and *rvalue* passed to `baz`.

---

## What is perfect forwarding?

---

```tikz
\begin{document}
\begin{tikzpicture}[scale=2]
  \draw (0,0) rectangle (3,2);
  \node at (1.5,1) {\Large Hello!};
\end{tikzpicture}
\end{document}
```
