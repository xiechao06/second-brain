---
marp: true
theme: gödel
size: 16:9
---

# C++ Study Sets

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
  // std::forward preserves the value category of x, 
  // it is also called perfect forwarding
  qux(std::forward<T>(x));
}
```

In this example, if `baz` is called with an lvalue, `std::forward<T>(x)` will return an lvalue reference, and if `baz` is called with an rvalue, `std::forward<T>(x)` will return an rvalue reference. This allows `qux` to properly handle both *lvalue* and *rvalue* passed to `baz`.

---

## Why should we prefer `make_unique` over `unique_ptr` constructor?

---

`make_unique` is an *atomic* way to create a `unique_ptr`, it ensures that the object is created and the `unique_ptr` is initialized in a single step, which prevents potential memory leaks if an exception is thrown during object creation. For example:

```c++
foo(std::unique_ptr<foo>(new foo), std::unique_ptr<bar>(new bar));
```

if `new foo` and `new bar` are executed before create 2 `unique_ptr`s, and `new bar` throws an exception, then memory leaks.

---

However, in

```c++
foo(std::make_unique<foo>(), std::make_unique<bar>());
```

The newly created **foo will always be freed**. Though since c++17, compilers tightening the evaluation order, `std::make_unique` is clearer and a safer bet.

---

## How to make a `unique_ptr` of an array of certain type?

---

Use `std::make_unique<T[]>(size)`

---

## How to make a `unique_ptr` of an object of certain type?

---

```c++
std::make_unique<foo>(foo{1, 2, 3}); // provide a foo object
std::make_unique<foo>(1, 2, 3); // provide parameters of foo's constructor
```

---

## What is an **aggregate**?

---

An **aggregate** is an array or class that could be initialized with [aggregate initialization](https://en.cppreference.com/cpp/language/aggregate_initialization). It is qualified by:

* No user-defined constructor
* All non-static members are public
* No virtual methods
* Won't inherit using `protected`, `private` or `virtual` keywords

An **aggregate** emphasis more on how to initialize an object, it could contain
**non-aggregate** members.

---

## What is a **trivial type**?

---

A **trivial type** is a type whose life time operations: *construct*, *copy*,
*moving* and "destruct" involves no runtime logic, it emphasis more on *usage*
aspect. It is qualified by:

* have default constructor, copy/move constructor, copy/move assignment operators and destructor (rule of 5) generated by compiler or explicitly set to `= default`.

* No virtual features

* All members are **trivial**

---

## Why **trivial type** is important?

---

* **Performance optimization**. *trivial type* is safe to **bit-copy**, so `std::vector` could just use `memcpy` but not copy constructors when reallocating space. And compilers will make use of this feature to do optimizations like `simd auto-vectorization` or `elide function call for operator=`.
* **Trivial buffers**, writing to binary file, network serialization and data transferring to GPU is much easier for trivial types.

So you should use *trivial type* whenever possible, its simple and easier to reason.

---

## Show me an example of a trivial type that is not an `aggregate`

---

Most **trivial types** you seen are **aggregates**, this is an exception:

```c++
class fixed_point {
  int raw_bits;
  explicit fixed_point(int raw_bits): raw_bits(raw_bits) {}
public:
  fixed_point() = default; // default constructor
  static FixedPoint from_float(float f) {
      return FixedPoint(static_cast<int>(f * 256.0f)); // Scaling factor
  }
  float to_float() const {
      return static_cast<float>(raw_bits) / 256.0f;
  }
};
```

---

## When should we use a *virtual destructor*?

---

**virtual destructor** must be used to ensure the destructors be called when an object is deleted through its base class pointer. In the following example:

```c++
class base {
  ~base() { std::cout << "base destructor" << std::endl; }
};
class derived: public base {
  ~derived() { std::cout << "derived destructor" << std::endl }
}

base *pb = new derived();
delete pb; // "base destructor"
```

A `derived` pointer is casted to `base` pointer statically, or, a **static binding** is performed, so delete `pb` will invoke `base`'s destructor.

---

But in the following example:

```c++
class base {
  virtual ~base() { std::cout << "base destructor" << std::endl; }
};
class derived: public base {
  virtual ~derived() { std::cout << "derived destructor" << std::endl }
}

base *pb = new derived();
delete pb; // "derived destructor", "base destructor"
```

Since destructor is declared `virtual`, compiler will perform `dynamic binding`, and call destructors from the most derived one.

---

## What is **standard layout**?

---
A type with memory layout compatible with C memory layout rule.

---

## Explain `std::make_unique_for_overwrite`?

---

like `make_unique`, `make_unique_for_overwrite` constructs an array of  objects and wrap them in `unique_ptr` but **without zero-initializing them** if the objects are **trivial**. In other word, `make_unique_for_overwrite` behaves differently than `make_unique` only
when the array of objects are **trivial**.

---

## Is **malloc** faster than **calloc**?

---

For small chunk of memory, **malloc** is usually faster than **calloc** for 1.5 times, but for large chunk, they are nearly the same, since new memory pages are zero-initialized before allocation.

---

## What are **shared circular references** and why it mean make memory leak?

---

An object has a direct or indirect shared reference to another object, which also has a direct or indirect shared reference to the first object.

Since the 2 shared references always hang there, neither of the 2 will ever be freed.

---

## Explain `weak_ptr`

---

`weak_ptr` stands for **weak pointer**, it is a *smart pointer* that doesn't own, i.e., don't have a **shared reference** to a dynamically allocated object.

---

## What is the point of `weak_ptr` compared to naked pointer?

---

`weak_ptr` works with `shared_ptr` to detect the dangling pointer:

```c++
auto sp{std::make_shared<int>(42)};
std::weak_ptr wp{sp};

std::cout << wp.expired() << std::endl; // 0
if (auto locked = wp.locked()) {
  std::cout << *locked << std::endl; // 42
}
```

---

But naked pointer won't:

```c++
int *raw;
{
  raw = std::make_shared<int>(42).get();
}

*raw; // dangling pointer
```

---

## Explain `enable_shared_from_this`

---

This is a helper class to create a `shared_ptr` or `weak_ptr` to `this`.

```c++
struct student;
struct teacher : public std::enable_shared_from_this<teacher> {
  std::string name;
  teacher(const char *name) : name(name) {}
  void take_student(student *s);
};
struct student {
  void take_teacher(std::shared_ptr<teacher> t) { this->teacher = t; }
  std::shared_ptr<teacher> teacher;
};
void teacher::take_student(student *s) { s->take_teacher(shared_from_this()); }
```

---

```c++
  auto teacher{std::make_shared<teacher>("Mr. Smith")};
  student alice;
  alice.take_teacher(t);
  std::cout << "Alice's teacher is " << alice.teacher->name << std::endl;
  student bob;
  bob.take_teacher(t);
  std::cout << "Bob's teacher is " << bob.teacher->name << std::endl;
```

In this example, `teacher` create a `weak_ptr` in each instance which points to `this`, So `student`s could have shared reference to it.

---

in `std::make_shared<teacher>("Mr. Smith")`, `enable_share_from_this` create a
`shared_ptr` and `weak_ptr` (which is accessible through `weak_from_this`) on it. When you invoke `share_from_this`, you actually get `weak_ptr::lock`.

```c++
auto t = std::make_shared<teacher>("Mr. Smith");
std::cout << t->weak_from_this().use_count() << std::endl; // output 1
std::cout << t->share_from_this().use_count() << std::endl; // output 2
```

---

Note, you must use `make_shared` to use `enable_share_from_this`.

```c++
  auto t = teacher("Mr. Smith");
  std::cout << t.weak_from_this().use_count() << std::endl; // 0
```

---

## How to compare signed and unsigned integers safely?

---

Use `cmp_equal`, `cmp_less` etc.

---

## Explain three-way comparison operator

---

Three-way comparison operator `<=>` is a clean way define operators '<', '<=', '>', '>=', but not operators '==' and '!='.

---

## What happened behind the scenes in `auto win{ create_app_window() }`?

### or why `move` constructor is not needed here

---

In the following code sample:

```c++
window create_app_window() {
  window local_win{...};
  return local_win;
}

auto win{ create_app_window() }
```

An optimize tech called **NRVO**, or **named return value optimization** is used, `local_win` will be not be regarded as **lvalue** but a **prvalue**(pure rvalue), so the return value will be constructed in `win` directly, and no `move` and `copy` will be invoked here.

---

## When will **NRVO** or **RVO** happen?

---

```c++
struct foo {
  int x;
};

foo func1(bool condition) {
  if (condition) {
    return foo{42};
  }
  return foo{24};
}

foo func2(bool condition) { return foo{condition ? 42 : 24}; }
```

In `func2`, you could construct object in caller directly and **RVO** takes effect.

---

## Explain `concept`

---

`concept` is the logical combination of `type_trait`s or other `concept`s, and as the constraint to template argument. For example:

```c++
template <typename T>
concept numerical = std::is_integral_v<T> || std::is_floating_point_v<T>;

template <numerical T>
struct numerical_wrapper {
  T value;
}

template <numerical T>
numerical_wrapper<T> wrap(T value) { return {value}; }

template <numerical T>
T unwrap(numerical_wrapper<T> wrapper) { return wrapper.value; }

```

---

## How to define `concept` with **require expression**?

---

Should support `operator +`.

```c++

template <typename T>
concept addable =  requires (T a, T b) { { a + b } -> std::same_as<T>; }

template <addable T>
T add(T a, T b) { return a + b; }

add(1, 2); // ok
add("foo", "bar"); // error

```

---

Define multiple requirements:

```c++
template <typename T> concept container = requires(T a) {
  typename T::value_type;
  typename T::iterator;
  { a.begin() } -> std::input_iterator;
  { a.end() } -> std::input_iterator;
};

template <container T> void print_container(const T& c) {
  for (const auto& item : c) {
    std::cout << item << " ";
  }
  std::cout << std::endl;
}
print_container(std::vector<int>{4, 5, 6}); // ok
int arr[]{1, 2, 3};
print_container(arr); // error
```

---

## Why **require-expression** is a **prvalue** of type `bool`?

The require-expression ```requires (T a, T b) { {a + b} => std::same_as<T> }```

1. takes no address
2. temporary
3. pure, computed at compile time

---

## Explain abbreviated template functions

---

A simplified syntax for defining function template.

```c++
// could use concept `floating_point`
auto add(std::floating_point auto a, std::floating_point auto b) {
  return a + b;
}

// could create specialization for `double`
template <> auto add(double a, double b) {
  return std::round((a + b) * 100) / 100.0;
}
```

---

## What is `range` and what is `view`?

---

`range` is a sequence of elements that can be iterated with an iterator and an and sentinel and `view` is a **range adaptor**.

---

## How to save a range into a container?

---

Use `ranges::to`.

---

## What is member pointer and member function pointer?

---


---

## What is **non-virtual interface idiom**?

---

A fundamentally and widely-used pattern that use non-virtual public methods as interfaces and private/protected virtual methods as implementation to provides **runtime polymorphism**.

---

```c++
struct control {
  // public interface
  void draw() {
    erase_background();
    paint();
  }
private:
  virtual void erase_background() = 0;
  virtual void paint() = 0;
};
```

---

```c++
class button: public control {
  virtual void erase_background() override {
    // do erase background for button
  }
  virtual void pain() override {
    // paint button
  }
};
class text: public control {
  virtual void erase_background() override {
    // do erase background for text
  }
  virtual void pain() override {
    // paint text
  }
};
```

---

```c++
std::vector<std::unique_ptr<control>> controls {
  std::make_unique<button>(),
  std::make_unique<text>(),
};
```

---

## What is **curiously recurring template pattern**?

---

A pattern that uses template to provide **compile-time polymorphism**. It defines a base template class that requires derived classes have certain non-virtual interfaces.

```c++
// as common ancestor
class controlbase {
public:
  virtual void draw() = 0;
  virtual ~controlbase() = default;
};

template <typename T> struct control : public controlbase {
  T *derived() { return static_cast<T *>(this); }
  void draw() {
    // requires child have methods erase_background and paint
    derived()->erase_background();
    derived()->paint();
  }
};
```

---

```c++
struct button : public control<button> {
  void erase_background() { std::cout << "button::erase_background\n"; }
  void paint() { std::cout << "button::paint\n"; }
};
```

---

## How to use `concept`s to implement duck typing?

---

```c++
// define a duck type
template <typename T>
concept has_hello_method = requires(T a) {
  { a.hello() } -> std::same_as<void>;
};

// require a duck type
template <has_hello_method T> void call_hello(T &obj) { obj.hello(); }
```

---

## How to put unrelated duck types in vector?

---

The basic idea is wrap them in a template classes which share a same common ancestor.

```c++
// 2 types with same methods
class button {
public:
  void erase_background() {
    std::cout << "Erasing background of button" << std::endl;
  }
  void paint() { std::cout << "Painting button" << std::endl; }
};

class checkbox {
public:
  void erase_background() {
    std::cout << "Erasing background of checkbox" << std::endl;
  }
  void paint() { std::cout << "Painting checkbox" << std::endl; }
};
```

---

A concept for duck type:

```c++
template <typename T>
concept drawable = requires(T a) {
  { a.erase_background() } -> std::same_as<void>;
  { a.paint() } -> std::same_as<void>;
};
```

---

```c++
struct control {
  template <typename T>
  control(T &&obj)
      : ctrl(std::make_shared<control_wrapper<T>>(std::forward<T>(obj))) {}
  void draw() { ctrl->draw(); }

private:
  struct control_concept {
    virtual ~control_concept() = default;
    virtual void draw() = 0;
  };

  template <drawable T> struct control_wrapper : public control_concept {
    // T && is universal reference, both binds to lvalue and rvalue
    control_wrapper(T &&obj) : obj(obj) {}
    void draw() override {
      obj.erase_background();
      obj.paint();
    }

  private:
    T &obj;
  };

private:
  std::shared_ptr<control_concept> ctrl;
};
```

---

Usage:

```c++
  checkbox cb;
  std::vector<control> v{cb, button{}};
  for (auto &ctrl : v) {
    ctrl.draw();
  }
```

---

## How to implement a thread-safe singleton?

---

```c++
template <typename T> class singleton_base {
protected:
  singleton_base() = default;
  ~singleton_base() = default;

public:
  singleton_base(const singleton_base &) = delete;
  singleton_base &operator=(const singleton_base &) = delete;

  static T &instance() {
    // thread-safe guaranteed by compiler
    static T instance;
    return instance;
  }
};
```

This is an example of singleton using `curiously recurring template pattern`.

---