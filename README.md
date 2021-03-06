# multi_container

Container that is capable of storing and iterating over any number of other containers. Another name would be `tied_container`, that's why I included an alias for that. It also features a `multi_iterator` class that is used for iterating over it, and works much like a `zip_iterator`. One important note is that `multi_container` owns the containers it stores, and copies the containers given to it in the constructor.

# Examples

We'll start by declaring some containers to use in the following examples.

```cpp
std::vector<int> vi { 0, 1, -1, 2, -2 };
std::vector<float> vf {11.0f, 12.0f, 13.0f, 14.0f, 15.0f};
std::array<long long, 5> all = { 1, 2, 3, 4, 5 };
```

Now, we declare our `multi_container`. Luckily we have type deduction from the constructor, so there is no need to specify all container types manually:

```cpp
mvg::multi_container m(vi, vf, all);
```

***Structured bindings***

```cpp
for(auto[a, b, c] : m)
{
  //use a, b and c
}
```

This does require some explanation, because you can probably see something with this: There is no `auto&`, just `auto`. Still, `a`, `b` and `c` are references to the elements. This is because under the hood, the `multi_iterator` used for iterating over the `multi_container` uses a wrapper around `std::tuple` for references to return when dereferencing. So even when copying the result of the dereference, you keep an iterator. This does have the downside that you can't explicitely add `const`, e.g

```cpp
for(auto const[a, b, c] : m)
{

}
```

Will not give const references to `a`, `b` and `c`. Using structured bindings on a `const multi_container`, will still give const references though.

***STL Algorithms***

`multi_container` is fully compatible with most/all STL algorithms. For example:
```cpp

mvg::multi_container m(vi, vf, all); //See the containers above

auto it = std::find(m.begin(), m.end(), std::make_tuple(1, 2.0f, 5ll));
if (it == m.end())
{
    std::cout << "Element not found";
}

m.erase(std::remove_if(m.begin(), m.end(),
            [](auto elem)
            {
                return std::get<0>(elem) <= 3;
            }), m.end());
```

These are just 2 examples of standard algorithms that works smoothly with `multi_container`. There is one special case, and that is when you start using an algorithm that compares elements, like `std::sort`. As there is no good way of ordering when comparing with `operator<`, `operator>`, or any other relational comparison, `multi_container` uses the result of the comparison of the first 2 elements. This has an interesting side effect, that when sorting a `multi_container`, the first container will be sorted, and all others will be sorted ***in the order of the first one***. Example:

```cpp
mvg::multi_container m(vi, vf, all);
std::sort(m.begin(), m.end());
```
Since `vi`, the first container, looks like this: `{ 0, 1, -1, 2, -2 }`, It will be sorted to: `{-2, -1, 0, 1, 2}`. Containers `vf` and `all` will be sorted in the same order. A way to visualize this is that `vi` specifies the indices of the elements in the sorted vector.
`


***Other features***

Below you can find a complete list of all member types and methods.

# Member types

| member type              | definition                                                                 |
|--------------------------|----------------------------------------------------------------------------|
| `iterator `              | `multi_iterator<typename detail::underlying_iterator<Ts>::type ...>`       |
| `const_iterator`         | `multi_iterator<typename detail::underlying_const_iterator<Ts>::type ...>` |
| `reverse_iterator `      | `std::reverse_iterator<iterator>`                                          |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator> `                                   |
| `value_type`             | `detail::tuple_wrapper<Ts...> `                                            |
| `reference`              | `detail::tuple_wrapper<std::add_lvalue_reference_t<Ts> ...>`               |
| `pointer`                | `std::add_pointer_t<value_type>`                                           |
| `size_type`              | `std::size_t`                                                              |
| `difference_type`        | `std::ptrdiff_t `                                                          |

# Methods

Very simply said, `multi_container` supports nearly all operations `std::vector` supports, except a few. Also, if any of the containers is for example an `std::array`, calling `push_back` on the `multi_container` will trigger a compiler error. Below is a list of all operations allowed on `multi_container`, assuming the containers it holds support those operations too.

- ***Iterators***
  - `iterator begin()` returns an iterator to the beginning of the sequence
  - `iterator end()` returns an iterator to the element past the end of the sequence
  - `const_iterator begin() const` is a `const` version of `begin()`
  - `const_iterator end() const` is a `const` version of `end()`
  - `const_iterator cbegin() const` returns a constant iterator to the beginning of the sequence
  - `const_iterator cend() const` returns a constant iterator to the element past the end of the sequence
  - `reverse_iterator rbegin()` returns a reverse iterator to the beginning of the sequence
  - `reverse_iterator rend()` returns a reverse iterator to the element past the end of the sequence
  - `const_reverse_iterator crbegin() const` returns a const reverse iterator to the beginning of the sequence
  - `const_reverse_iterator crend() const` returns a const reverse iterator to the element past the end of the sequence
- ***Access to underlying containers***
  - `std::tuple<Ts...>& data()` access the underlying tuple storing the containers
  - `std::tuple<Ts...> const& data() const` access the underlying tuple storing the containers
  - `template<class C> C& get_container()` returns the container with specified type
  - `template<class C> C const& get_container() const` returns the container with specified type
  - `templace<size_t I> tuple_element_t<I, tuple<Ts...>>& get_container()` returns the container at specified index
  - `templace<size_t I> tuple_element_t<I, tuple<Ts...>> const& get_container() const` returns the container at specified index
- ***Element access***
  - `detail::tuple_wrapper<Ts& ...> operator[](std::size_t index)` returns a tuple containing references to the elements of all containers at the specified index. Does not perform bounds checking.
  - `detail::tuple_wrapper<Ts const& ...> operator[](std::size_t index) const` returns a tuple containing references to the elements of all containers at the specified index. Does not perform bounds checking.
  - `detail::tuple_wrapper<Ts& ...> at(std::size_t index)` returns a tuple containing references to the elements of all containers at the specified index. Throws an instance of `std::out_of_range` when `index >= size()`.
  - `detail::tuple_wrapper<Ts const& ...> at(std::size_t index) const` returns a tuple containing const references to the elements of all containers at the specified index. Throws an instance of `std::out_of_range` when `index >= size()`.
  - `detail::tuple_wrapper<Ts& ...> front()` returns a tuple containing references to the first element of every container. Same as calling `*(m.begin())`.
  - `detail::tuple_wrapper<Ts const& ...> front() const` returns a tuple containing const references to the first element of every container. Same as calling `*(m.begin())`.
  - `detail::tuple_wrapper<Ts& ...> back()` returns a tuple containing references to the last element of every container. Same as calling `*(m.end() - 1)`.
  - `detail::tuple_wrapper<Ts const& ...> back() const` returns a tuple containing references to the last element of every container. Same as calling `*(m.end() - 1)`.
- ***Capacity***
  - `bool empty() const` returns `true` if all containers stored are empty
  - `size_type size() const` returns the size of the ***smallest container***.
- ***Modifiers***
  - `void clear()` clears all stored containers
  - `template<class... Elems> void push_back(std::tuple<Elems...> const& elems)` Appends elements at the end of every container
  - `template<class... Elems> iterator insert(iterator pos, std::tuple<Elems...> const& elems` Inserts elements before `pos`
  - `template<class... Elems> iterator insert(const_iterator pos, std::tuple<Elems...> const& elems)` Inserts elements before `pos`
  - `template<class InputIt> iterator insert(iterator pos, InputIt first, InputIt last)` Inserts elements in the range `[first, last[`. `InputIt` must be a `multi_iterator` matching this `multi_container`. The behavior is undefined when `[first, last[` is not a valid range.
  - `template<class... Elems> iterator insert(iterator pos, size_type count, std::tuple<Elems...> const& elems)` Inserts `count` copies of `elems` before `pos`
  - `iterator erase(iterator pos)` Erases element at `pos`.
  - `iterator erase(const_iterator pos` Erases element at `pos`.
  - `iterator erase(iterator first, iterator last)` Erases elements in the range `[first, last[`. The behavior in undefined when `[first, last[` is not a valid range.
  - `iterator erase(const_iterator first, const_iterator last)` Erases elements in the range `[first, last[`. The behavior in undefined when `[first, last[` is not a valid range.
  - `void pop_back()` removes the last element from the container






