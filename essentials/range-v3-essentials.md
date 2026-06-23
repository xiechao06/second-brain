# Range-v3 essentials

[range-v3](https://ericniebler.github.io/range-v3/index.html) provides cross-platform range interfaces for C++14/17/20. It is the basis of the C++20 `<ranges>` library.

```cpp
#include <range/v3/all.hpp>
using namespace ranges;
```

---

## Views (Lazy, composable, non-mutating)

## `views::addressof`

Take `std::addressof` of each lvalue reference in the source range.

```c++
std::vector v{1, 2, 3};
// output: 1, 2, 3
for (auto ptr: v | views::addressof) {
    fmt::print("{}\n", *ptr);
}
```

## `views::adjacent_filter`

For each adjacent pair, drop the second if binary predicate returns `false`.

```c++
std::vector v{1, 1, 2, 3, 4, 4};
// -> {1, 1, 2, 3, 4, 4}  (std::not_equal_to)
auto rng = v | views::adjacent_filter(std::not_equal_to{});
```

## `views::adjacent_remove_if`

For each adjacent pair, drop the first if binary predicate returns `true`.

```c++
std::vector v{1, 1, 2, 3, 3, 4};
// -> {1, 2, 3, 4}  (drop first of equal pair)
auto rng = v | views::adjacent_remove_if(std::equal_to{});
```

## `views::all`

Wrap a container/range into a view.

```c++
std::vector v{1, 2, 3};
auto rng = views::all(v);  // same elements: 1, 2, 3
```

## `any_view<T>`

Type-erased range with value type `T`.

```c++
std::vector v{1, 2, 3};
any_view<int> rng = v | views::transform([](int i){return i*2;});
// -> 2, 4, 6
```

## `views::c_str`

View a `\0`-terminated C string as a range of `char`.

```c++
auto rng = views::c_str("hello");
// -> 'h', 'e', 'l', 'l', 'o', '\0'
```

## `views::cache1`

Cache the most recent element (useful after `filter` + `transform`). Single-pass.

```c++
std::vector v{1, 2, 3, 4};
auto rng = v | views::filter([](int i){return i%2==0;}) | views::cache1;
// -> 2, 4 (cached so transform recomputed only once)
```

## `views::cartesian_product`

Enumerate the n-ary cartesian product of N ranges.

```c++
std::vector a{1, 2}, b{'a', 'b'};
// -> {1,'a'}, {1,'b'}, {2,'a'}, {2,'b'}
for (auto [x, y] : views::cartesian_product(a, b)) {}
```

## `views::chunk`

Split a range into contiguous subranges of size N.

```c++
std::vector v{1, 2, 3, 4, 5};
// -> {1,2}, {3,4}, {5}
for (auto chunk : v | views::chunk(2)) { fmt::print("{}\n", chunk); }
```

## `views::common`

Convert to a common range (same type for `begin` and `end`).

```c++
std::vector v{1, 2, 3, 4, 5};
auto rng = v | views::take(3) | views::common;
// needed for passing to std:: algorithms
std::sort(rng.begin(), rng.end());
```

## `views::concat`

Concatenate N source ranges.

```c++
std::vector a{1, 2}, b{3, 4}, c{5, 6};
// -> 1, 2, 3, 4, 5, 6
auto rng = views::concat(a, b, c);
```

## `views::const_`

Present a `const` view of a source range.

```c++
std::vector v{1, 2, 3};
auto rng = v | views::const_;  // elements are const int&
```

## `views::counted`

Create a range from iterator `it` and count `n`.

```c++
std::vector v{10, 20, 30, 40, 50};
// -> 20, 30, 40
auto rng = views::counted(v.begin()+1, 3);
```

## `views::cycle`

Endlessly repeat the source range.

```c++
std::vector v{1, 2, 3};
// -> 1, 2, 3, 1, 2, 3, 1, 2, 3, 1
auto rng = v | views::cycle | views::take(10);
```

## `views::delimit`

End at first occurrence of a value.

```c++
std::vector v{1, 2, 0, 3, 4};
// -> 1, 2
auto rng = views::delimit(v, 0);
```

## `views::drop`

Skip the first N elements.

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 4, 5
auto rng = v | views::drop(3);
```

## `views::drop_last`

Drop the last N elements (needs `sized_range`).

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 1, 2, 3
auto rng = v | views::drop_last(2);
```

## `views::drop_exactly`

Drop exactly N elements (source must have ≥ N).

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 4, 5  (sized_range)
auto rng = v | views::drop_exactly(3);
```

## `views::drop_while`

Drop elements from the front while predicate is true.

```c++
std::vector v{-3, -1, 0, 2, 4};
// -> 0, 2, 4
auto rng = v | views::drop_while([](int i){return i<0;});
```

## `views::empty<T>`

Create an empty range of value type `T`.

```c++
// -> (no elements)
auto rng = views::empty<int>;
fmt::print("{}\n", distance(rng));  // 0
```

## `views::enumerate`

Pair each element with its index.

```c++
std::vector v{"a", "b", "c"};
// -> {0,"a"}, {1,"b"}, {2,"c"}
for (auto [idx, val] : v | views::enumerate)
    fmt::print("{}: {}\n", idx, val);
```

## `views::filter`

Keep elements that satisfy the predicate.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
// -> 2, 4, 6
auto rng = v | views::filter([](int i){return i%2==0;});
```

## `views::for_each`

Flat-map: apply a function returning a range, flatten.

```c++
// -> 1, 2,2, 3,3,3
auto rng = views::ints(1, 4) | views::for_each([](int i){
    return views::repeat_n(i, i);
});
```

## `views::generate`

Infinite range from a nullary function.

```c++
int x = 0;
// -> 0, 1, 2, 3, 4
auto rng = views::generate([&](){return x++;}) | views::take(5);
```

## `views::generate_n`

Generate N elements from a nullary function.

```c++
// -> 7, 7, 7, 7, 7
auto rng = views::generate_n([&](){return rand()%10;}, 5);
```

## `views::chunk_by`

Group contiguous elements by a binary predicate.

```c++
std::vector v{1, 2, 5, 6, 2, 3, 4};
// -> {1,2}, {5,6}, {2,3,4}  (groups where a < b)
auto rng = v | views::chunk_by(std::less{});
```

## `views::indirect`

Dereference each element (e.g. range of pointers/iterators).

```c++
int a=1, b=2, c=3;
std::vector ptrs{&a, &b, &c};
// -> 1, 2, 3
for (int val: ptrs | views::indirect)
    fmt::print("{}\n", val);
```

## `views::intersperse`

Insert a value between adjacent elements.

```c++
std::vector v{1, 2, 3};
// -> 1, 0, 2, 0, 3
auto rng = v | views::intersperse(0);
```

## `views::ints`

Monotonically increasing `int`s. `[from,to)` half-open.

```c++
// -> 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
for (int i : views::ints(0, 10)) {}
// views::ints() -> 0, 1, 2, 3, ... (infinite)
```

## `views::iota`

Generalized `ints` for any incrementable type.

```c++
// -> 0.0, 0.5, 1.0, 1.5, ..., 9.5
auto rng = views::iota(0.0, 10.0);
```

## `views::join`

Flatten a range of ranges. Optionally with a delimiter.

```c++
std::vector<std::string> words{"ab", "cd", "ef"};
// -> 'a', 'b', 'c', 'd', 'e', 'f'
auto rng = words | views::join;
```

## `views::keys`

Extract first element from a range of `pair`s.

```c++
std::map<int, std::string> m{{1,"a"},{2,"b"},{3,"c"}};
// -> 1, 2, 3
for (int k : m | views::keys) {}
```

## `views::linear_distribute`

Distribute N values linearly in `[from, to]`.

```c++
// -> 0.0, 0.25, 0.5, 0.75, 1.0
auto rng = views::linear_distribute(0.0, 1.0, 5);
```

## `views::move`

Cast each element to an rvalue reference.

```c++
std::vector<std::string> v{"hello", "world"};
// moves strings into v2
std::vector<std::string> v2 = v | views::move | to<std::vector>();
```

## `views::partial_sum`

Running partial sum with a binary function.

```c++
std::vector v{1, 2, 3, 4};
// -> 1, 3, 6, 10
auto rng = v | views::partial_sum(std::plus{});
```

## `views::remove`

Filter out elements equal to a value.

```c++
std::vector v{1, 0, 2, 0, 3};
// -> 1, 2, 3
auto rng = v | views::remove(0);
```

## `views::remove_if`

Filter out elements satisfying the predicate.

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 2, 4
auto rng = v | views::remove_if([](int i){return i%2==1;});
```

## `views::repeat`

Repeat a value infinitely.

```c++
// -> 42, 42, 42, 42, 42
auto rng = views::repeat(42) | views::take(5);
```

## `views::repeat_n`

Repeat a value N times.

```c++
// -> hi, hi, hi
auto rng = views::repeat_n("hi", 3);
```

## `views::replace`

Replace elements equal to `old` with `new`.

```c++
std::vector v{1, 0, 2, 0, 3};
// -> 1, 99, 2, 99, 3
auto rng = v | views::replace(0, 99);
```

## `views::replace_if`

Replace elements satisfying a predicate.

```c++
std::vector v{-3, -2, 0, 1, 2};
// -> 0, 0, 0, 1, 2
auto rng = v | views::replace_if([](int i){return i<0;}, 0);
```

## `views::reverse`

Traverse the source range in reverse.

```c++
std::vector v{1, 2, 3};
// -> 3, 2, 1
auto rng = v | views::reverse;
```

## `views::sample`

Random sample from a range.

```c++
std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
std::mt19937 gen{std::random_device{}()};
// -> 3 random elements from v
auto rng = v | views::sample(3, gen);
```

## `views::single`

Create a range with exactly one element.

```c++
// -> 42
auto rng = views::single(42);
```

## `views::slice`

Sub-range with lower/upper bounds. Supports `end-2` syntax.

```c++
std::vector v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
// -> 2, 3, 4, 5, 6, 7  (index 2 to end-2)
auto rng = v | views::slice(2, end-2);
```

## `views::sliding`

Sliding window of size N over the range.

```c++
std::vector v{1, 2, 3, 4};
// -> {1,2}, {2,3}, {3,4}
for (auto win : v | views::sliding(2))
    fmt::print("{}\n", win);
```

## `views::split`

Split by a delimiter value/range. Delimiters excluded.

```c++
std::string s = "hello world foo";
// -> "hello", "world", "foo"
for (auto word : s | views::split(' '))
    fmt::print("{}\n", word | views::c_str);
```

## `views::split_when`

Split by a predicate or function boundary detector.

```c++
std::vector v{1, 2, 0, 3, 0, 4, 5};
// -> {1,2}, {3}, {4,5}
auto rng = v | views::split_when([](int i){return i==0;});
```

## `views::stride`

Take every Nth element.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
// -> 1, 3, 5
auto rng = v | views::stride(2);
```

## `views::tail`

Drop the first element (range must have ≥1 elements).

```c++
std::vector v{1, 2, 3, 4};
// -> 2, 3, 4
auto rng = v | views::tail;
```

## `views::take`

Take the first N elements.

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 1, 2, 3
auto rng = v | views::take(3);
```

## `views::take_exactly`

Take exactly N elements (source must have ≥ N).

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 1, 2, 3  (sized_range)
auto rng = v | views::take_exactly(3);
```

## `views::take_last`

Take the last N elements (needs `sized_range`).

```c++
std::vector v{1, 2, 3, 4, 5};
// -> 4, 5
auto rng = v | views::take_last(2);
```

## `views::take_while`

Take elements from front while predicate is true.

```c++
std::vector v{1, 3, 5, 2, 4, 6};
// -> 1, 3, 5
auto rng = v | views::take_while([](int i){return i%2==1;});
```

## `views::tokenize`

Regex token iterator over a range.

```c++
std::string s = "hello 42 world";
// -> "hello", "42", "world"
auto rng = s | views::tokenize(std::regex{"\\w+"});
```

## `views::transform`

Apply a unary function to each element.

```c++
std::vector v{1, 2, 3};
// -> 1, 4, 9
auto rng = v | views::transform([](int i){return i*i;});
```

## `views::trim`

Trim elements from both ends matching predicate (bidirectional).

```c++
std::vector v{0, 0, 1, 2, 3, 0, 0};
// -> 1, 2, 3
auto rng = v | views::trim([](int i){return i==0;});
```

## `views::unbounded`

Infinite range starting from an iterator.

```c++
std::vector v{1, 2, 3};
auto it = v.begin();
// elements from it to infinity (careful!)
auto rng = views::unbounded(it) | views::take(3);
```

## `views::unique`

Filter out consecutive equal elements, keeping the first.

```c++
std::vector v{1, 1, 2, 2, 2, 3, 1, 1};
// -> 1, 2, 3, 1
auto rng = v | views::unique;
```

## `views::values`

Extract second element from a range of `pair`s.

```c++
std::map<int, std::string> m{{1,"a"},{2,"b"},{3,"c"}};
// -> "a", "b", "c"
for (auto& val : m | views::values) {}
```

## `views::zip`

Zip N ranges into a range of tuples.

```c++
std::vector a{1, 2, 3}, b{"a","b","c"};
// -> {1,"a"}, {2,"b"}, {3,"c"}
for (auto [x, s] : views::zip(a, b))
fmt::print("{}: {}\n", x, s);
```

## `views::zip_with`

Zip N ranges with an N-ary function.

```c++
std::vector a{1, 2, 3}, b{10, 20, 30};
// -> 11, 22, 33
auto rng = views::zip_with(std::plus{}, a, b);
```

---

## Algorithms (Iterator- and range-based, eager)

## `adjacent_find`

Find first occurrence of two adjacent equal elements.

```c++
std::vector v{1, 2, 2, 3, 4};
auto it = adjacent_find(v);
// *it == 2  (first of the pair)
```

## `adjacent_remove_if`

Remove elements where adjacent pair satisfies binary predicate.

```c++
std::vector v{1, 2, 2, 3, 3, 4};
auto it = adjacent_remove_if(v, std::equal_to{});
v.erase(it, v.end());  // -> {1, 2, 3, 4}
```

## `all_of`

Check if all elements satisfy a predicate.

```c++
std::vector v{2, 4, 6, 8};
bool r = all_of(v, [](int i){return i%2==0;});
// r == true
```

## `any_of`

Check if any element satisfies a predicate.

```c++
std::vector v{1, 3, 5, 6, 7};
bool r = any_of(v, [](int i){return i%2==0;});
// r == true
```

## `binary_search`

Check if a value exists in a sorted range.

```c++
std::vector v{1, 3, 5, 7, 9};
bool r = binary_search(v, 5);
// r == true
```

## `contains`

Check if a range contains a value (C++23 style).

```c++
std::vector v{10, 20, 30, 40};
bool r = contains(v, 30);
// r == true
```

## `copy`

Copy elements from source to output.

```c++
std::vector v{1, 2, 3};
std::vector<int> out(3);
copy(v, out.begin());  // out == {1, 2, 3}
```

## `copy_backward`

Copy elements backward.

```c++
std::vector v{1, 2, 3};
std::vector<int> out(3);
copy_backward(v, out.end());  // out == {1, 2, 3}
```

## `copy_if`

Copy elements satisfying a predicate.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
std::vector<int> out;
copy_if(v, ranges::back_inserter(out), [](int i){return i%2==0;});
// out == {2, 4, 6}
```

## `copy_n`

Copy exactly N elements.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector<int> out(3);
copy_n(v.begin(), 3, out.begin());  // out == {1, 2, 3}
```

## `count`

Count elements equal to a value.

```c++
std::vector v{1, 2, 2, 3, 2, 4};
auto n = count(v, 2);
// n == 3
```

## `count_if`

Count elements satisfying a predicate.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
auto n = count_if(v, [](int i){return i%2==0;});
// n == 3
```

## `ends_with`

Check if a range ends with another range.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector suffix{4, 5};
bool r = ends_with(v, suffix);  // r == true
```

## `equal`

Check if two ranges are equal.

```c++
std::vector a{1, 2, 3}, b{1, 2, 3};
bool r = equal(a, b);
// r == true
```

## `equal_range`

Find subrange of equivalent elements in a sorted range.

```c++
std::vector v{1, 2, 2, 2, 3, 4};
auto [lo, hi] = equal_range(v, 2);
// lo points to first 2, hi points after last 2
```

## `fill`

Assign a value to all elements in a range.

```c++
std::vector<int> v(5);
fill(v, 42);
// v == {42, 42, 42, 42, 42}
```

## `fill_n`

Assign a value to N elements.

```c++
std::vector<int> v{1, 2, 3, 4, 5};
fill_n(v.begin(), 3, 0);
// v == {0, 0, 0, 4, 5}
```

## `find`

Find the first element equal to a value.

```c++
std::vector v{10, 20, 30, 40};
auto it = find(v, 30);
// *it == 30
```

## `find_if`

Find the first element satisfying a predicate.

```c++
std::vector v{1, 3, 5, 6, 7};
auto it = find_if(v, [](int i){return i%2==0;});
// *it == 6
```

## `find_if_not`

Find the first element not satisfying a predicate.

```c++
std::vector v{2, 4, 6, 7, 8};
auto it = find_if_not(v, [](int i){return i%2==0;});
// *it == 7
```

## `find_end`

Find last occurrence of a subsequence.

```c++
std::vector v{1, 2, 3, 1, 2, 3, 4};
std::vector sub{1, 2};
auto r = find_end(v, sub);  // points to {1,2} at index 3
```

## `find_first_of`

Find first element from one range that is in another.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector targets{6, 3, 9};
auto it = find_first_of(v, targets);
// *it == 3
```

## `fold_left`

Left fold (C++23 style).

```c++
std::vector v{1, 2, 3, 4};
auto r = fold_left(v, 0, std::plus{});
// r == 10  (0+1+2+3+4)
```

## `fold_left_first`

Left fold using the first element as initial value.

```c++
std::vector v{1, 2, 3, 4};
auto r = fold_left_first(v, std::plus{});
// r == 10  (1+2+3+4)
```

## `fold_right`

Right fold.

```c++
std::vector v{1, 2, 3, 4};
auto r = fold_right(v, 0, std::plus{});
// r == 10  (1+(2+(3+(4+0))))
```

## `fold_right_last`

Right fold using the last element as initial value.

```c++
std::vector v{1, 2, 3, 4};
auto r = fold_right_last(v, std::plus{});
// r == 10  (1+(2+(3+4)))
```

## `for_each`

Apply a function to each element.

```c++
std::vector v{1, 2, 3};
// prints: 2, 4, 6
for_each(v, [](int& i){ i *= 2; });
```

## `for_each_n`

Apply a function to the first N elements.

```c++
std::vector v{1, 2, 3, 4, 5};
for_each_n(v, 3, [](int& i){ i = 0; });
// v == {0, 0, 0, 4, 5}
```

## `generate`

Fill a range from a nullary function.

```c++
std::vector<int> v(5);
int x = 0;
generate(v, [&](){ return x++; });
// v == {0, 1, 2, 3, 4}
```

## `generate_n`

Fill N elements from a nullary function.

```c++
int x = 0;
std::vector<int> v(5);
generate_n(v.begin(), 5, [&](){ return x+=2; });
// v == {2, 4, 6, 8, 10}
```

## `includes`

Check if sorted range includes another sorted range.

```c++
std::vector a{1, 2, 3, 4, 5}, b{2, 4};
bool r = includes(a, b);
// r == true
```

## `inplace_merge`

Merge two consecutive sorted ranges in-place.

```c++
std::vector v{1, 3, 5, 2, 4, 6};
inplace_merge(v, v.begin()+3);
// v == {1, 2, 3, 4, 5, 6}
```

## `is_heap`

Check if range is a heap.

```c++
std::vector v{9, 5, 7, 1, 3};
bool r = is_heap(v);  // r == true (max-heap)
```

## `is_heap_until`

Find first element not in heap order.

```c++
std::vector v{9, 5, 7, 1, 10};
auto it = is_heap_until(v);  // it points to 10
```

## `is_partitioned`

Check if range is partitioned by predicate.

```c++
std::vector v{2, 4, 6, 1, 3, 5};
bool r = is_partitioned(v, [](int i){return i%2==0;});
// r == true (evens before odds)
```

## `is_permutation`

Check if one range is a permutation of another.

```c++
std::vector a{1, 2, 3, 4}, b{4, 2, 1, 3};
bool r = is_permutation(a, b);  // r == true
```

## `is_sorted`

Check if range is sorted.

```c++
std::vector v{1, 2, 3, 4, 5};
bool r = is_sorted(v);  // r == true
```

## `is_sorted_until`

Find first element out of sorted order.

```c++
std::vector v{1, 2, 3, 0, 4};
auto it = is_sorted_until(v);  // it points to 0
```

## `lexicographical_compare`

Lexicographically compare two ranges.

```c++
std::vector a{1, 2, 3}, b{1, 2, 4};
bool r = lexicographical_compare(a, b);  // r == true
```

## `lower_bound`

Find first position where value can be inserted (sorted).

```c++
std::vector v{1, 2, 4, 5};
auto it = lower_bound(v, 3);
// it points before 4; insert 3 there to get {1,2,3,4,5}
```

## `make_heap`

Turn a range into a heap.

```c++
std::vector v{3, 1, 4, 1, 5, 9};
make_heap(v);  // v is now a max-heap, v[0] == 9
```

## `max`

Return the maximum of two values or a range.

```c++
std::vector v{3, 7, 2, 9, 1};
auto m = max(v);  // m == 9
// or
auto m2 = max(5, 8);  // m2 == 8
```

## `max_element`

Find the largest element.

```c++
std::vector v{3, 7, 2, 9, 1, 5};
auto it = max_element(v);
// *it == 9
```

## `merge`

Merge two sorted ranges into output.

```c++
std::vector a{1, 3, 5}, b{2, 4, 6};
std::vector<int> out(6);
merge(a, b, out.begin());  // out == {1,2,3,4,5,6}
```

## `min`

Return the minimum of two values or a range.

```c++
std::vector v{3, 7, 2, 9, 1, 5};
auto m = min(v);  // m == 1
```

## `min_element`

Find the smallest element.

```c++
std::vector v{3, 7, 2, 9, 1, 5};
auto it = min_element(v);
// *it == 1
```

## `minmax`

Return both min and max.

```c++
std::vector v{3, 7, 2, 9, 1, 5};
auto [lo, hi] = minmax(v);
// lo == 1, hi == 9
```

## `minmax_element`

Find both min and max elements.

```c++
std::vector v{3, 7, 2, 9, 1, 5};
auto [it_lo, it_hi] = minmax_element(v);
// *it_lo == 1, *it_hi == 9
```

## `mismatch`

Find first position where two ranges differ.

```c++
std::vector a{1, 2, 3, 4}, b{1, 2, 9, 4};
auto [it1, it2] = mismatch(a, b);
// *it1 == 3, *it2 == 9
```

## `move`

Move elements from source to output.

```c++
std::vector<std::string> v{"a", "b", "c"};
std::vector<std::string> out(3);
move(v, out.begin());  // out == {"a","b","c"}
```

## `move_backward`

Move elements backward.

```c++
std::vector<std::string> v{"a", "b", "c"};
std::vector<std::string> out(3);
move_backward(v, out.end());  // out == {"a","b","c"}
```

## `next_permutation`

Generate next lexicographical permutation.

```c++
std::vector v{1, 2, 3};
do { fmt::print("{}\n", v); } while (next_permutation(v));
// 123, 132, 213, 231, 312, 321
```

## `none_of`

Check if no element satisfies the predicate.

```c++
std::vector v{1, 3, 5, 7};
bool r = none_of(v, [](int i){return i%2==0;});
// r == true
```

## `nth_element`

Partially sort: nth element is in sorted position.

```c++
std::vector v{5, 2, 8, 1, 9, 3};
nth_element(v, v.begin()+2);
// v[2] == 3 (the 3rd smallest)
```

## `partial_sort`

Sort the first N elements.

```c++
std::vector v{5, 2, 8, 1, 9, 3};
partial_sort(v, v.begin()+3);
// v[0..2] are {1, 2, 3}
```

## `partial_sort_copy`

Copy the top N sorted elements.

```c++
std::vector v{5, 2, 8, 1, 9, 3};
std::vector<int> out(3);
partial_sort_copy(v, out);  // out == {1, 2, 3}
```

## `partition`

Reorder so predicate-true elements come first.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
auto it = partition(v, [](int i){return i%2==0;});
// evens before odds: e.g. {2,4,6, 1,3,5}
```

## `partition_copy`

Copy partition-true and false to separate outputs.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
partition_copy(v, ranges::back_inserter(evens), ranges::back_inserter(odds),
[](int i){return i%2==0;});
// evens == {2,4,6}, odds == {1,3,5}
```

## `partition_point`

Find the partition point in a partitioned range.

```c++
std::vector v{2, 4, 6, 1, 3, 5};
auto it = partition_point(v, [](int i){return i%2==0;});
// *it == 1
```

## `pop_heap`

Pop the largest element from heap.

```c++
std::vector v{9, 5, 7, 1, 3};
pop_heap(v);  // v[0..3] remains heap, v[4] == 9
v.pop_back();  // remove the popped value
```

## `prev_permutation`

Generate previous lexicographical permutation.

```c++
std::vector v{3, 2, 1};
do { fmt::print("{}\n", v); } while (prev_permutation(v));
// 321, 312, 231, 213, 132, 123
```

## `push_heap`

Push a new element onto heap (last element).

```c++
std::vector v{9, 5, 7, 1, 3};
v.push_back(10);
push_heap(v);  // v[0] == 10
```

## `remove`

Remove elements equal to a value (logical, returns new end).

```c++
std::vector v{1, 2, 3, 2, 4, 2, 5};
auto it = remove(v, 2);
v.erase(it, v.end());  // v == {1, 3, 4, 5}
```

## `remove_copy`

Copy elements not equal to a value.

```c++
std::vector v{1, 2, 3, 2, 4};
std::vector<int> out;
remove_copy(v, ranges::back_inserter(out), 2);  // out == {1, 3, 4}
```

## `remove_copy_if`

Copy elements not satisfying predicate.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector<int> out;
remove_copy_if(v, ranges::back_inserter(out), [](int i){return i<3;});
// out == {3, 4, 5}
```

## `remove_if`

Remove elements satisfying predicate.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
auto it = remove_if(v, [](int i){return i%2==1;});
v.erase(it, v.end());  // v == {2, 4, 6}
```

## `replace`

Replace elements equal to `old` with `new`.

```c++
std::vector v{1, 0, 2, 0, 3};
replace(v, 0, 99);
// v == {1, 99, 2, 99, 3}
```

## `replace_copy`

Copy with replacement of `old` → `new`.

```c++
std::vector v{1, 0, 2, 0, 3};
std::vector<int> out(5);
replace_copy(v, out.begin(), 0, 99);  // out == {1,99,2,99,3}
```

## `replace_copy_if`

Copy with conditional replacement.

```c++
std::vector v{-3, -1, 0, 2, 4};
std::vector<int> out(5);
replace_copy_if(v, out.begin(), [](int i){return i<0;}, 0);
// out == {0, 0, 0, 2, 4}
```

## `replace_if`

Replace elements satisfying predicate.

```c++
std::vector v{-3, -1, 0, 2, 4};
replace_if(v, [](int i){return i<0;}, 0);
// v == {0, 0, 0, 2, 4}
```

## `reverse`

Reverse the range in-place.

```c++
std::vector v{1, 2, 3, 4, 5};
reverse(v);
// v == {5, 4, 3, 2, 1}
```

## `reverse_copy`

Copy in reverse order.

```c++
std::vector v{1, 2, 3};
std::vector<int> out(3);
reverse_copy(v, out.begin());  // out == {3, 2, 1}
```

## `rotate`

Left-rotate the range around `middle`.

```c++
std::vector v{1, 2, 3, 4, 5};
rotate(v, v.begin()+2);
// v == {3, 4, 5, 1, 2}
```

## `rotate_copy`

Copy with rotation.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector<int> out(5);
rotate_copy(v, v.begin()+2, out.begin());  // out == {3,4,5,1,2}
```

## `sample`

Randomly sample N elements.

```c++
std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
std::vector<int> out(3);
std::mt19937 gen{std::random_device{}()};
sample(v, out.begin(), 3, gen);  // 3 random elements from v
```

## `search`

Search for a subsequence.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
std::vector pattern{3, 4, 5};
auto r = search(v, pattern);  // r is subrange {3,4,5}
```

## `search_n`

Search for N consecutive occurrences of a value.

```c++
std::vector v{1, 2, 2, 2, 3, 4};
auto r = search_n(v, 3, 2);  // finds {2,2,2} starting at index 1
```

## `set_difference`

Elements in first sorted range but not in second.

```c++
std::vector a{1, 2, 3, 4, 5}, b{3, 5};
std::vector<int> out;
set_difference(a, b, ranges::back_inserter(out));  // out == {1, 2, 4}
```

## `set_intersection`

Intersection of two sorted ranges.

```c++
std::vector a{1, 2, 3, 4, 5}, b{3, 4, 5, 6, 7};
std::vector<int> out;
set_intersection(a, b, ranges::back_inserter(out));  // out == {3, 4, 5}
```

## `set_symmetric_difference`

Symmetric difference of two sorted ranges.

```c++
std::vector a{1, 2, 3, 4}, b{3, 4, 5, 6};
std::vector<int> out;
set_symmetric_difference(a, b, ranges::back_inserter(out));
// out == {1, 2, 5, 6}
```

## `set_union`

Union of two sorted ranges.

```c++
std::vector a{1, 2, 3}, b{3, 4, 5};
std::vector<int> out;
set_union(a, b, ranges::back_inserter(out));  // out == {1, 2, 3, 4, 5}
```

## `shuffle`

Randomly shuffle the range.

```c++
std::vector v{1, 2, 3, 4, 5};
shuffle(v, std::mt19937{std::random_device{}()});
// v is now randomly shuffled
```

## `sort`

Sort the range (unstable).

```c++
std::vector v{5, 2, 8, 1, 9, 3};
sort(v);
// v == {1, 2, 3, 5, 8, 9}
```

## `sort_heap`

Turn a heap into a sorted range.

```c++
std::vector v{9, 5, 7, 1, 3};
sort_heap(v);  // v == {1, 3, 5, 7, 9}
```

## `stable_partition`

Partition while preserving relative order.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
stable_partition(v, [](int i){return i%2==0;});
// v == {2, 4, 6, 1, 3, 5}  (relative order kept)
```

## `stable_sort`

Sort the range (stable).

```c++
std::vector v{5, 2, 8, 1, 9, 3};
stable_sort(v);
// v == {1, 2, 3, 5, 8, 9}
```

## `starts_with`

Check if a range starts with another range.

```c++
std::vector v{1, 2, 3, 4, 5};
std::vector prefix{1, 2};
bool r = starts_with(v, prefix);  // r == true
```

## `swap_ranges`

Swap elements from two ranges.

```c++
std::vector a{1, 2, 3}, b{4, 5, 6};
swap_ranges(a, b);
// a == {4, 5, 6}, b == {1, 2, 3}
```

## `transform` (unary)

Apply function to source, write to output.

```c++
std::vector v{1, 2, 3};
std::vector<int> out(3);
transform(v, out.begin(), [](int i){return i*i;});
// out == {1, 4, 9}
```

## `transform` (binary)

Apply binary function to two sources.

```c++
std::vector a{1, 2, 3}, b{10, 20, 30};
std::vector<int> out(3);
transform(a, b, out.begin(), std::plus{});
// out == {11, 22, 33}
```

## `unique`

Remove consecutive duplicates (returns new end).

```c++
std::vector v{1, 1, 2, 2, 3, 1, 1};
auto it = unique(v);
v.erase(it, v.end());  // v == {1, 2, 3, 1}
```

## `unique_copy`

Copy without consecutive duplicates.

```c++
std::vector v{1, 1, 2, 2, 3, 1};
std::vector<int> out;
unique_copy(v, ranges::back_inserter(out));  // out == {1, 2, 3, 1}
```

## `unstable_remove_if`

O(1) per-element removal (unordered result, bidirectional).

```c++
std::vector v{1, 2, 3, 4, 5, 6};
auto it = unstable_remove_if(v, [](int i){return i%2==1;});
v.erase(it, v.end());  // v == {6, 2, 4} (order not preserved)
```

## `upper_bound`

Find first position where value goes after (sorted).

```c++
std::vector v{1, 2, 2, 2, 3, 4};
auto it = upper_bound(v, 2);  // points to 3
```

---

## Numeric Algorithms

## `accumulate`

Left fold over a range (like `std::accumulate`).

```c++
std::vector v{1, 2, 3, 4};
auto sum = accumulate(v, 0);  // sum == 10
auto prod = accumulate(v, 1, std::multiplies{});  // prod == 24
```

## `adjacent_difference`

Compute differences between adjacent elements.

```c++
std::vector v{1, 3, 6, 10};
std::vector<int> out(4);
adjacent_difference(v, out.begin());  // out == {1, 2, 3, 4}
```

## `inner_product`

Inner product of two ranges.

```c++
std::vector a{1, 2, 3}, b{4, 5, 6};
auto r = inner_product(a, b, 0);  // r == 32 (1*4+2*5+3*6)
```

## `iota`

Fill a range with incrementing values (like `std::iota`).

```c++
std::vector<int> v(5);
iota(v, 10);  // v == {10, 11, 12, 13, 14}
```

## `partial_sum`

Compute partial sums (like `std::partial_sum`).

```c++
std::vector v{1, 2, 3, 4};
std::vector<int> out(4);
partial_sum(v, out.begin());  // out == {1, 3, 6, 10}
```

---

## Actions (Eager, mutating, composable — operate on containers in-place)

## `actions::drop`

Remove the first N elements.

```c++
std::vector v{1, 2, 3, 4, 5};
v = std::move(v) | actions::drop(2);
// v == {3, 4, 5}
```

## `actions::drop_while`

Remove first elements satisfying predicate.

```c++
std::vector v{-3, -1, 0, 2, 4};
v = std::move(v) | actions::drop_while([](int i){return i<0;});
// v == {0, 2, 4}
```

## `actions::erase`

Erase a sub-range or all elements after a position.

```c++
std::vector v{1, 2, 3, 4, 5};
auto sub = v | views::drop(2) | views::take(2);
v |= actions::erase(sub);
// v == {1, 2, 5}
```

## `actions::insert`

Insert a range at a position.

```c++
std::vector v{1, 5};
std::vector more{2, 3, 4};
v |= actions::insert(v.begin()+1, more);
// v == {1, 2, 3, 4, 5}
```

## `actions::join`

Flatten a range of ranges into a container.

```c++
std::vector<std::string> words{"ab", "cd"};
std::string flat = words | actions::join;  // "abcd"
```

## `actions::push_back`

Append elements to the tail.

```c++
std::vector v{1, 2};
std::vector more{3, 4, 5};
v |= actions::push_back(more);
// v == {1, 2, 3, 4, 5}
```

## `actions::push_front`

Prepend elements to the head.

```c++
std::deque<int> d{3, 4, 5};
std::vector more{1, 2};
d |= actions::push_front(more);
// d == {1, 2, 3, 4, 5}
```

## `actions::remove`

Remove all elements equal to a value.

```c++
std::vector v{1, 0, 2, 0, 3};
v = std::move(v) | actions::remove(0);
// v == {1, 2, 3}
```

## `actions::remove_if`

Remove all elements satisfying predicate.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
v = std::move(v) | actions::remove_if([](int i){return i%2==1;});
// v == {2, 4, 6}
```

## `actions::reverse`

Reverse the container in-place.

```c++
std::vector v{1, 2, 3, 4, 5};
v = std::move(v) | actions::reverse;
// v == {5, 4, 3, 2, 1}
```

## `actions::shuffle`

Shuffle the container in-place.

```c++
std::vector v{1, 2, 3, 4, 5};
v = std::move(v) | actions::shuffle(std::mt19937{});
// v is now randomly shuffled
```

## `actions::slice`

Keep only the specified sub-range, discard the rest.

```c++
std::vector v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
v = std::move(v) | actions::slice(2, end-2);
// v == {2, 3, 4, 5, 6, 7}
```

## `actions::sort`

Sort the container in-place (unstable).

```c++
std::vector v{5, 2, 8, 1, 9, 3};
v = std::move(v) | actions::sort;
// v == {1, 2, 3, 5, 8, 9}
```

## `actions::split`

Split into subranges by a delimiter.

```c++
std::string s = "a,b,c";
auto parts = s | actions::split(',');
```

## `actions::stable_sort`

Stable sort the container in-place.

```c++
std::vector v{5, 2, 8, 1, 9, 3};
v = std::move(v) | actions::stable_sort;
// v == {1, 2, 3, 5, 8, 9}
```

## `actions::stride`

Keep every Nth element, remove others.

```c++
std::vector v{1, 2, 3, 4, 5, 6};
v = std::move(v) | actions::stride(2);
// v == {1, 3, 5}
```

## `actions::take`

Keep the first N elements, remove the rest.

```c++
std::vector v{1, 2, 3, 4, 5};
v = std::move(v) | actions::take(3);
// v == {1, 2, 3}
```

## `actions::take_while`

Keep leading elements satisfying predicate.

```c++
std::vector v{1, 3, 5, 2, 4};
v = std::move(v) | actions::take_while([](int i){return i%2==1;});
// v == {1, 3, 5}
```

## `actions::transform`

Replace each element with result of a unary function.

```c++
std::vector v{1, 2, 3};
v = std::move(v) | actions::transform([](int i){return i*i;});
// v == {1, 4, 9}
```

## `actions::unique`

Remove adjacent duplicates (sorted → all duplicates).

```c++
std::vector v{1, 1, 2, 2, 3, 1};
v |= actions::sort | actions::unique;
// v == {1, 2, 3}
```

## `actions::unstable_remove_if`

Fast unordered removal by predicate (bidirectional).

```c++
std::vector v{1, 2, 3, 4, 5, 6};
v = std::move(v) | actions::unstable_remove_if([](int i){return i%2==1;});
// v == {6, 2, 4} (order not preserved)
```

---

## Pipe Syntax Overview

```cpp
// Views (lazy): range | view_adaptor | view_adaptor | ...
auto rng = vec | views::filter(pred) | views::transform(fn);

// Actions (eager, by-value chain): container | action | action
auto result = std::move(vec) | actions::sort | actions::unique;

// Actions (in-place): container |= action | action
vec |= actions::sort | actions::unique;

// Algorithms (free functions): algorithm(range, args...)
sort(vec);
auto it = find_if(vec, pred);
```
