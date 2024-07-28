---
layout: post
title: >
  Move semantics: Perfect Forwarding
tags: [cpp]
---

# Move semantics: Perfect Forwarding


This article is the third part of a three-part series on move semantics in modern C++. Please find [Part1](2024-07-24-move-semantics-part1.md) and [Part2](2024-07-24-move-semantics-part2.md).

As a result of all the concepts we have covered so far, we can have a look at the solution of one interesting problem for free.

Suppose you have a class `X`, which accepts two arguements in its constructor. You want to create a utility function to validate the arguments and construct a new object of type `X` with the given arguments if validation is succesful.

How's this:

~~~cpp
template<typename Arg1, typename Arg2>
std::unique_ptr<X> validateArgsAndMakeObj(Arg1&& a, Arg2&& b) {
    if (validateArgs(a, b)){
        return std::make_unique<X>(a, b);
    }
    return nullptr;
}
~~~

If you carefully notice, both a and b will be passed as lvalue references to `std::make_unique` even if either of them was an rvalue reference. That's not what we want. We could overload the function `validateArgsAndMakeObj` to accept arguments of all combination of lvalue and rvalue references, like we did in copy vs move constructor, but that's impractical here as the number of combinations increase exponentially with the number of arguments. Also the argument list can be variadic.

What do we do? We use STL's `std::forward`.

~~~cpp
template<typename Arg1, typename Arg2>
std::unique_ptr<X> validateArgsAndMakeObj(Arg1&& a, Arg2&& b) {
    if (validateArgs(a, b)){
        return std::make_unique<X>(std::forward<Arg1>(a), std::forward<Arg2>(b));
    }
    return nullptr;
}
~~~

`std::forward` preserves the lvalue referenceness and rvalue referenceness of the arguments.

Let's have a look at the implementation of `std::forward`.

~~~cpp
template<typename T>
    _GLIBCXX_NODISCARD
    constexpr T&&
    forward(typename std::remove_reference<T>::type& param) noexcept
    { return static_cast<T&&>(param); }
~~~

As before, `_GLIBCXX_NODISCARD` is a macro for `[[__nodiscard__]]`, which instructs compiler to warn against unused return value. `constexpr` hints the compiler to evaluate this function at compile time. `std::remove_reference<T>::type` will, well, remove any *ampersand* from type `T`.

Let's see what this function evaluates to after ignoring unimportant parts and template type deduction:

~~~cpp
// case 1: called with lvalue
template<typename Arg>
Arg& forward(Arg& param) { return static_cast<Arg&>(param); }
~~~

~~~cpp
// case 2: called with rvalue
template<typename Arg>
Arg&& forward(Arg&& param) { return static_cast<Arg&&>(param); }
~~~

Pretty neat trick, right?

That concludes our discussion on rvalue references, move semantics and perfect forwarding. Thanks for reading!