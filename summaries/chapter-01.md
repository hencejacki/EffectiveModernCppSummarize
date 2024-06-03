# Deducing Types

> + Three rules for template type deduction
> + Rules for `auto` type deduction
> + Usage of `decltype`
> + Methods for Viewing deducted type

## Three rules for template type deduction

1. Parameter passed by reference or a pointer

The template we use as follows:

~~~c++
template<typename T>
void foo(T& param);
~~~

+ Ignore the reference part of the parameter

For example:

~~~c++
int x = 10;
const int cx = x;
const int& rx = x;

foo(x);     // T: int, param: int&
foo(cx);    // T: const int, param: const int&
foo(rx);    // T: const int, param: const int&
~~~

2. Parameter passed by universal reference

The template we use as follows:

~~~c++
template<typename T>
void foo(T&& param);
~~~

+ If the expr is an lvalue, both the template parameter `T` and the
function parameter `param` are deduced to be lvalue reference.

+ If the expr is an rvalue, same as the fist rule.

For example:

~~~c++
int x = 10;
const int cx = x;
const int& rx = x;

foo(x);     // T: int&, param: int&
foo(cx);    // T: const int&, param: const int&
foo(rx);    // T: const int&, param: const int&
foo(20);    // T: int, param: int&&
~~~

3. Parameter passed by neither a reference nor a pointer

The template we use as follows:

~~~c++
template<typename T>
void foo(T param);
~~~

+ Ignore the reference part

+ Ignore the `const` part and `volatile` part

For example:

~~~c++
int x = 10;
const int cx = x;
const int& rx = x;

foo(x);     // T: int, param: int
foo(cx);    // T: int, param: int
foo(rx);    // T: int, param: int
~~~

Both function argument and array argument are treated as a pointer, unless they
are used to initialize references.

For example:

~~~c++
// function argument
template<typename T>
void foo(T param);

template<typename T>
void bar(T& param);

void func(int);

foo(func);      // param's type is deduced to void (*)(int)
bar(func);      // param's type is deduced to void (&)(int)

// array argument
template<typename T>
void foo(T param);

template<typename T>
void bar(T &param);

const char name[] = "hello";

foo(name);      // T: const char*, param: const char*
bar(name);      // T: const char [6], param: const char (&)[6]
~~~

## Rules for `auto` type deduction

In most cases, `auto` type deduction is the same as the template deduction, 
the difference between them is that the `auto` depends on the initialized value
to deduce type and if the value decorated with `auto` is initialized by braced
initializer, then, `auto` type deduction will assumes it represents a `std::initializer_list`.

For example:

~~~c++
auto x = {1, 2, 3};     // x's type is std::initializer_list<int>

template<typename T>
void foo(T param);

template<typename T>
void bar(std::initializer_list<T> param);

foo(x);     // error!
bar(x);     // correct!
~~~

When the `auto` is used in a parameter type and a returned value, the template
deduction used, not auto type deduction.

For example:

~~~c++
auto foo()
{
    return {1, 2, 3};       // error!
}

auto bar = [](const auto& v){...}
bar({1, 2, 3});     // error!
~~~

## Usage of `decltype`

`decltype` always report reference type for lvalue other than names.

For example, `x` is a name, but `(x)` is a expression:

~~~c++
decltype(auto) foo()
{
    int x = 0;
    return x;       // decltype(x) is int
}

decltype(auto) bar()
{
    int x = 0;
    return (x);     // decltype((x)) is int&
}
~~~

This rule may cause some unexpected behavior, because the returned value
use template deduction. For example:

~~~c++
// c++11 style
template<typename Container, typename Index>
auto accessValue(Container&& c, Index& i)
-> decltype(std::forward<Container>(c)[i])
{
    return std::forward<Container>(c)[i];
}

template<typename Container, typename Index>
decltype(auto) accessValue(Container&& c, Index& i)
{
    return std::forward<Container>(c)[i];
}

std::vector<int> arr;

accessValue(arr, 5) = 9;       // error! because the returned value is lvalue
~~~

## Methods for Viewing deducted type

1. IDE editor

2. Runtime output

`std::type_info::name` mandates that the type be treated as if it has been
passed to a template function as a by-value parameter, there may be cause
error deduction in some situation.

You can use the thirty library to get a correct result, e.g. Boost.

3. Compiler complain
