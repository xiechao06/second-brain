# FMT (version 12.2.0) lib essentials

`fmt` provide python's f-string capabilities to c++.

[official site](https://fmt.dev)

## Basic usage

```c++
#include <fmt/base.h>

fmt::println("{} + {} equals to {}", 1, 2, 3);
fmt::println("{a} + {b} equals to {sum}", 
  fmt::arg("a", 1), fmt::arg("b", 2), fmt::arg("sum" 3));
```

## Use `format_to` to build string

```c++
fmt::memory_buffer buf;
fmt::format_to(std::back_inserter(buf), "Hello, ");
fmt::format_to(std::back_inserter(buf), "world");

fmt::println("{}", fmt::to_string(buf)); // Hello, world
```

```c++
fmt::memory_buffer buf;
auto fmt_res{fmt::format_to_n(std::back_inserter(buf), 5, "Hello, {}!", "world")};
// output the total size
fmt::println("{}", fmt_res.size); // > 13
// continue from iterator after the output range 
fmt::format_to(end_it.out, ", 42"); 

fmt::println("{}", fmt::to_string(buf)); // > Hello, 42
```

## Create string

```c++
auto s{fmt::format("The answer is {}.", 42)};
```

## Format User-Defined Types

### Use `format_as`

```c++
#include <fmt/format.h>
namespace kevin_namespacy {

enum class film {
  house_of_cards, american_beauty, se7en = 7
};

auto format_as(film f) { 
  using T = std::underlying_type_t<film>; // aka int
  return static_cast<T>(f); 
}
}

struct animal {
  std::string sounds; 
};

auto format_as(animal a) {
  return a.sound;
}

int main() {
  fmt::println("{}", kevin_namespacy::film::se7en); // > 7
  fmt::println("{:'>8}", animal{"meow"}); // > ''''meow
}
```

But you can't define your own format specifiers in this way.

### Specialize template `std::formatter`

```c++
#include <fmt/base.h>
#include <fmt/format.h>

enum class color { red, green, blue };

// inherits fmt::formatter
template <> struct fmt::formatter<color> : fmt::formatter<string_view> {
  auto format(color c, format_context &ctx) const {
    string_view name = "unknown";
    switch (c) {
    case color::red:
      name = "red";
      break;
    case color::green:
      name = "green";
      break;
    case color::blue:
      name = "blue";
      break;
    }
    // let parent handle context
    return formatter<string_view>::format(name, ctx);
  }
};

int main(int argc, const char **argv) {
  fmt::println("{:>10}", color::red);
  return 0;
}
```

the following are an example of user-defined format specifier:

```c++
#include <fmt/base.h>
#include <fmt/format.h>

struct point {
  double x;
  double y;
};

template <> struct fmt::formatter<point> {
  char presentation = 'd'; // default presentation

  constexpr auto parse(format_parse_context &ctx) {
    auto it = ctx.begin(), end = ctx.end();
    if (it != end && (*it == 'd' || *it == 'p')) {
      presentation = *it;
      ++it;
    }
    if (it != end && *it != '}') {
      throw format_error("invalid format");
    }
    return it;
  }

  template <typename FormatContext>
  auto format(const point &p, FormatContext &ctx) const {
    if (presentation == 'd') {
      return fmt::format_to(ctx.out(), "({}, {})", p.x, p.y);
    }
    if (presentation == 'p') {
      return fmt::format_to(ctx.out(), "x={}; y={}", p.x, p.y);
    }
    // fallback for unsupported presentation
    throw format_error("unsupported format");
  }
};

int main(int argc, const char **argv) {
  point p{3.0, 4.0};
  fmt::print("Default format: {}\n", p);
  fmt::print("Pretty format: {:p}\n", p);
}
```

## With the support of `std::ostream`

```c++
#include <fmt/ostream.h>

struct date {
  int year, month, day;
  friend std::ostream& operator<<(std::ostream &os, const date&d) {
    return os << d.year << '-' << d.month << '-' << d.day;
  }
};

template <> struct fmt::formatter<date> : ostream_formatter {};

fmt::println("The date is {}", date{2012, 12, 9});
```

## Terminal colors and text styles

```c++
fmt::print(fmt::emphasis::bold | fg(fmt::color::red),
           "Elapsed time: {0:.2f} seconds", 1.23);
```

or:

```c++
fmt::print("Elapsed time: {0:.2f} seconds",
           fmt::styled(1.23, fmt::fg(fmt::color::green) |
                             fmt::bg(fmt::color::blue)));
```

## Utilities

### Format address

```c++
auto x{1};
fmt::println("{}", fmt::ptr(&x);
```

### Format integers with ',' as thousands separator

```c++
fmt::println("{}", fmt::group_digits(12345));
```

## Ranges & Tuple

Ranges & tuple could be formatted if `<fmt/ranges.h>` is included. Fmt also provide a join utility like:

```c++
fmt::print("{}", fmt::join({1, 2, 3}, ":")); // 1:2:3
```

## Date & Time

```c++
#include <fmt/chrono.h>

int main() {
  auto now {std::chrono::system_clock::now()};
  fmt::println("The date is {:%Y-%m-%d %H:%M:%S}.",
               std::chrono::floor<std::chrono::seconds>(now));
}
```

More chrono format syntax is [here](https://fmt.dev/12.0/syntax/#chrono-format-specifications).

## STL types

`fmt/std.h` provides formatters for:

* std::atomic
* std::atomic_flag
* std::bitset
* std::error_code
* std::exception
* std::filesystem::path
* std::monostate
* std::optional
* std::source_location
* `std::thread::id`
* std::variant

# # `vformat` and why?

`fmt` perform format parse at compile time with `consteval` expression. So user could define functions with compile-time checkes on `fmt::format_string`.

```c++
template <typename... T>
void log(const char* file, int line,
         fmt::format_string<T...> fmt, T&&... args) {
        // ...
}
```

If you implement `log` using `fmt::print` like: 

```c++
template <typename... T>
void log(const char* file, int line,
         fmt::format_string<T...> fmt, T&&... args) {
    // fmt::print is a template method, so it will expand
    fmt::print("{}: {}: {}", file, line, fmt::vformat(fmt, fmt::make_format_args(args...)));
}
```

Your code will bloat by macro expansion. Here's what type-erased
format comes to rescue:

```c++
// vlog is not a template function, T is erased
void vlog(const char *file, int line, fmt::string_view fmt,
          fmt::format_args args) {
  // vformat don't do compile-time check
  fmt::print("{}: {}: {}", file, line, fmt::vformat(fmt, args));
}

template <typename... T>
void log(const char* file, int line,
         fmt::format_string<T...> fmt, T&&... args) {
    // 1. vlog is not a template function and will not expand
    // 2. make_format_args captures lvalues, so, you shouldn't do
    //    perfect forwarding here.
    vlog(file, line, fmt, fmt::make_format_args(args));
}
```

