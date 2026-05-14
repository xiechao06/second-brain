# C & CPP Tricks

## How to create an auto variable from a string literal?

A string literal is interpreted as `const char*`.

```c++
auto s {"hello"}; // type of s is const char*
```

However it could be converted to string by standard user-defined literal `s`.

```c++
using namespace std::string_literals;

auto s {"hello"s}; // type of s is std::string
```

The same technique could be applied to CTAD (class template arguments deduction) as well.

```c++
using namespace std::string_literals;

std::vector str_v {"hello"s, "world"s}; // type of str_v is std::vector<std::string>
```

## What is the difference between `vector<int> v(5)` and `vector<int> v{5}`

`vector<int> v{5}` or `vector v{5}` invokes vector's constructor

```c++
constexpr vector( std::initializer_list<T> init, 
                  const Allocator& alloc = Allocator());
```

means *initialize vector with an initializer list with single element is 5*.

Whilst `vector<int> v(5)` invokes vector's constructor

```c++
constexpr explicit vector( size_type count,
                           const Allocator& alloc = Allocator() );

```

meas *create a vector of 5 elements*.

You may wonder why `vector v{5}` invokes the constructor with `initializer_list` as argument, that is because:

if there's a constructor with initializer_list as argument, it will be invoked in *uniform **initialization**.
