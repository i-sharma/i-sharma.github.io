---
layout: post
title: >
  Move semantics: Design and Rules of thumb
tags: [hobbies]
---

# Move semantics: Design and Rules of thumb

This article is the second part of a three-part series on move semantics in modern C++. Please find [Part1](2024-07-24-move-semantics-part1.md) and [Part3](2024-07-24-move-semantics-part3.md).

> Note: In the coming sections, when I say copy constructor, I'll be referring to copy constructor and copy assignment operator collectively. If I want to specifically refer to the copy assignment operator, I'll say copy assignment operator.

We concluded in Part1, that we want a way to know whether the copy constructor is being called with parameter as an actual object or with a temporary.

We need to get familiarized with a few more concepts, and then we'll look the solution. I promise!

# Lvalues and Rvalues

An expression is a sequence of variables, literals, function calls, operators that evaluates to a *single value*. These are all valid expressions:

~~~cpp
5;                  // expr1: 5 is a literal
a;                  // expr2: a is a variable of type int
a + 4.6 * func();   // expr3: where func() returns int
x.get();            // expr4: where X is a class and get is a method with signature X& X::get();
~~~

All expressions have two properties: type and value category.

We are all familiar with types, additionally expressions can be categorised broadly into two categories based on value: lvalue and rvalue. 

- **lvalues** are expressions that refer to a memory location. Compiler allows you to take address of such expressions using `&` operator. Since they refer to a memory location, you can assign a value to such expressions.
These are some examples of lvalues:

~~~cpp
a = 6;
x.get() = 10;
~~~

- **rvalues** are expressions that are not lvalues. They do not refer to an identifiable memory location, thus you can't take the address of such an expression using `&` operator. Neither can you assign a value to such an expression.
Examples:

~~~cpp
5;                    // literal
a + 1;                // temporary  
X();                  // temporary object 
a + 4 * func() = 6;   // Error: Cant assign value to an rvalue
~~~

> Lifetime of rvalues end after the statement evaluation is done. 
> *Gotcha*: except when bound by rvalue references. We'll come back to it in a minute.

# Lvalue references and Rvalue references

An `lvalue reference` is another name or an alias, if you will, of an existing identifiable object, i.e., an lvalue. The process of giving the name or attaching an lvalue reference to an lvalue is called *binding*.

~~~cpp
int x = 5;
int &y = x;     // y is an lvalue reference of x. Type of y is int&
int &p;         // Error, lvalue reference must be initialized
int &q = 6;     // Error, can't bind an lvalue ref to an rvalue.
~~~

#### Refresher: lvalue reference vs pointer

~~~cpp
int x = 5;
int &y = x;
int *ptr = &x;
~~~

`x` is the name of a memory location, the bytes of which are set to `0x5`. `y` is another name of the same memory location. `ptr` is a new variable, it occupies a different memory location. The type of `ptr` is `int*`, the value stored in this new memory location is `&x`, which evaluates to a byte value. Read the comments in snippet below for more clarity.

~~~cpp
std::printf("%p\n", &x);    // addr of x: say 0x7ffe00bc8e4c
std::printf("%p\n", &y);    // (addr of x)==(addr of y). Prints 0x7ffe00bc8e4c
std::printf("%p\n", &ptr);  // addr of ptr, a different value, say 0x7ffe00bc8e38
std::printf("%p\n", ptr);   // value stored in memory location of ptr, addr of x, prints 0x7ffe00bc8e4c
std::printf("%p\n", *ptr);  // dereference operator - goes to the memory stored in ptr, 
                            // i.e. 0x7ffe00bc8e4c and prints the value stored there = 0x5
~~~


#### Interesting caveat 

Pre C++11, lvalue reference couln't bind to an rvalue. How would a copy constructor get a reference to an rvalue, then? If the copy construct looked like `DynamicArray(DynamicArray& rhs);`, it wouldn't be possible. However, the designers allowed const lvalue references to bind to rvalues. Since you couldn't assign a const lvalue reference to another value, it was safe to do so. That's the reason copy constructor is `DynamicArray(const DynamicArray& rhs)`.

#### rvalue reference

An `rvalue reference` is a name given to a temporary. **It can only bind to an rvalue, not an lvalue.**

~~~cpp
int x = 10;       
int&& x = 5;      // syntax, type of x is int&&, but the value category is lvalue. What? Wait a minute. 
X&& obj = X();    // obj is the name of this temporary object
int&& y = x;      // Error, can't bind an rvalue reference to an lvalue.
~~~

Let's discuss a few properties of lvalue ref vs rvalue ref:

1. **Type**

Type of lvalue is `T&`, whereas that of an rvalue is `T&&` where `T` is the type of referent.

2. **Value category**

An lvalue reference is an lvalue. That's alright, but surprisingly, an rvalue reference is also an lvalue! Why? Because it refers to a memory location and its address is well defined. Please remember this. 

3. **Lifetime**

The lifetime of an lvalue reference is independent of the value it's referring. As a result, it can so happen that an lvalue reference earlier valid is now dangling, i.e., the referent has died but the reference is alive.

If an rvalue was not attached to an rvalue reference, its lifetime would be limited to the statement where it is present. But, an rvalue reference extends the lifetime of an rvalue to match the lifetime of an rvalue reference. Clean, right?

1. **Reseat not allowed**

Once attached to a value, both lvalue references and rvalue references can not be made to point to another object. Interestingly, modifying an rvalue reference is allowed. Though, it probably doesn't mean what you think it does. 

~~~cpp
X&& x = X(5);
x = X(10);
~~~

In the first statement, a temporary is constructed and then rvalue reference is attached. This temporary **is not destructed**. In the second statement, another temporary is constructed, its value copied into the mem referred by x (using copy assignment operator) and then this temporary **is destructed**.

#### Quiz time

~~~cpp 
class X {
public:
    explicit X() {
        std::cout << "constr " << std::endl;
    }

    X(X const &rhs) {
        std::cout << "copy constr " << std::endl;
    }

    X &operator=(X const &rhs) {
        std::cout << "copy asgn " << std::endl;
        return *this;
    }

    X(X &&rhs) {
        std::cout << "move constr " << std::endl;
    }

    X &operator=(X &&rhs) noexcept {
        std::cout << "move asgn " << std::endl;
        return *this;
    }

    ~X() {
        std::cout << "destr " << std::endl;
    }
};

class Xwrapper {
    X x;
public:
    Xwrapper(): x() {}

    X&& GetRvalueRef(){
        return static_cast<X&&>(x);
    }
};



int main() {
    X x = X();
    X y = x;
    x = y;
    Xwrapper w;
    X z = w.GetRvalueRef();
    z = w.GetRvalueRef();
}
~~~

Predict the output.

<details>
    <summary>Answer</summary>
    constr <br>
    copy constr <br>
    copy asgn <br>
    constr <br>
    move constr <br>
    move asgn <br>
    destr <br>
    destr <br>
    destr <br>
    destr <br>
</details>



# Take me to move semantics please

Now coming to the interesting part:
Why is rvalue reference needed? It looks like in the most recent snippet above, `X x = X();` would also achieve the same result. 

But remember the problem we started with? We want to distinguish if a function (the copy constructor) can be made to work differently with identifiable objects versus with temporaries as arguments.

Now we know these values are lvalues and rvalues, respectively. So what we essentially want is to overload the copy constructor such that one can bind to lvalues only, and other can bind to rvalues only. We now know that the types of parameters will be lvalue reference and rvalue references respectively.

> rvalues prefer rvalue references over const lvalue references if given a choice during function overload resolution.
 
The modified class `DynamicArray` from part 1 with move constructor and move assignment operator:
~~~cpp
class DynamicArray{
public:
    size_t n{};
    void *ptr{};

    explicit DynamicArray(size_t n) : n(n), ptr(malloc(n)) {
    }

    DynamicArray(const DynamicArray& rhs) : n(rhs.n), ptr(malloc(rhs.n)) { // copy constructor
        std::memcpy(this->ptr, rhs.ptr, n);
    }

    DynamicArray& operator=(const DynamicArray& rhs) { // copy assignment
        if (this != &rhs) {
            free(ptr); // Clean up existing resource
            n = rhs.n;
            ptr = malloc(n);
            std::memcpy(ptr, rhs.ptr, n);
        }
        return *this;
    }

    DynamicArray(DynamicArray&& rhs)  noexcept { // move constructor
        std::swap(n, rhs.n);
        std::swap(ptr, rhs.ptr);
    }

    DynamicArray& operator=(DynamicArray&& rhs)  noexcept { // move assignment
        if (this != &rhs) {
            // cleanup required: free ptr, discussed later
            std::swap(n, rhs.n);
            std::swap(ptr, rhs.ptr);
        }
        return *this;
    }

    virtual ~DynamicArray() {
        free(ptr);
    }
};
~~~

A **rule of thumb** is to use move constructor and move assignment operator with [*noexcept*](https://stackoverflow.com/questions/10787766/when-should-i-really-use-noexcept). Make sure they don't throw exceptions. If you drop noexcept, certain STL containers will not use the move constructor even when it is defined. Example, std::vector's resize() determines relocation policy based on whether a type is move constructible.

Sometimes we want to construct an rvalue reference from an lvalue and call the move constructor of a new object. It's like giving up ownership of the old object. We have already seen one example of such a case in the Xwrapper example above. `static_cast<X&&>` does just that. The standard library provides a utility function to do this: `std::move`. 

Let's look at the implementation of `std::move` 

~~~cpp
template<typename T>
    _GLIBCXX_NODISCARD
    constexpr typename std::remove_reference<T>::type&&
    move(T&& param) noexcept
    { return static_cast<typename std::remove_reference<T>::type&&>(param); }
~~~
 
`_GLIBCXX_NODISCARD` is a macro for `[[__nodiscard__]]`, which instructs compiler to warn against unused return value. `constexpr` hints the compiler to evaluate this function at compile time. `std::remove_reference<T>::type` will, well, remove *ampersand* from type `T` if `T` was deduced as `X&` during template type deduction, can happen in case of universal reference is called with an lvalue. This is discussed next. Phew, C++ is not for the faint hearted.

One special concept here is universal reference. Notice `T&&` as the type of param. To an innocent beholder, it looks like it will resolve to an rvalue reference. But, that's not the case: `T&&` is something called a universal reference. It can act as either lvalue reference or rvalue reference depending on the context in which it is called.

Let's see what this function evaluates to after ignoring unimportant parts and template type deduction:

~~~cpp
// case 1: called with lvalue
X x = X(); 
// called as std::move(x)

template<typename X&>
X&& move(X& param) { return static_cast<X&&>(param); }
~~~

~~~cpp
// case 2: called with rvalue
// called as std::move(X())

template<typename X>
X&& move(X&& param) { return static_cast<X&&>(param); // param was an lvalue, converting to rvalue ref again
~~~

**An aside on universal references:**

When I first read about universal references at an airport, I was unreasonably angry. I wrote down a rant, one month before writing this article. It took me a month to calm down. The passage below is without changes, as I had written then, barring censorship. 

```
first cpp blog

to be clear, the following view matters because fresh eyes see what institutionalised eyes can't

why is cpp not simple, it's the opposite: too many if elses, and in order to "know cpp" you are required to know all if-elses
and these things keep changing
eg: static initialization fiasco

case in point:
this big ___ article on why the ___ does && have two meanings: https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers
excerpts from the article: 
	“&&” in a type declaration sometimes means rvalue reference, but sometimes it means either rvalue reference or lvalue reference.
>> this fills me with rage
	The details of when “&&” indicates a universal reference (i.e., when “&&” in source code might actually mean “&”) are tricky, 
>> no ___!
	This information is useful only if you are able to distinguish lvalues from rvalues.  A precise definition for these terms is difficult to develop (the C++11 standard generally specifies whether an expression is an lvalue or an rvalue on a case-by-case basis),
>> you've got to be kidding me
	Remember that “&&” indicates a universal reference only where type deduction takes place.  Where there’s no type deduction, there’s no universal reference.  In such cases, “&&” in type declarations always means rvalue reference. 
	the form of the reference declaration must be “T&&” in order for the reference to be universal. That’s an important caveat.
	Here, T is a template parameter, and push_back takes a T&&, yet the parameter is not a universal reference!  How can that be?
>> ~~~exception par exception par exception~~~ he's clarifying the rule and is warning against common pitfall 
>> well actually the push back example is pretty mind blowing

-------------------------------

reference collapsing
"The truth is that a universal reference is really just an rvalue reference in a reference-collapsing context."
```

Now we are near the end. One more caveat:
Now we know that the rhs in move constructor can be an lvalue, the lifetime of which can exceed the calling statement. So, if you swap the resources in move constructor, the desctructor of this->resource can get deferred to an unspecified time in the future. That means, if your destructor has side effects, you should cleanup your resources before exiting the move constructor. 

Too many things to take care of, right! 

Well that's cpp. 

[Performace tests](https://www.modernescpp.com/index.php/copy-versus-move-semantic-a-few-numbers/)

If you're interested, have a look at perfect forwarding in [Part3](2024-07-24-move-semantics-part3.md).


# References

[1] [learncpp](https://www.learncpp.com/) <br>
[2] [Thomas Becker's amazing article](http://thbecker.net/articles/rvalue_references/section_01.html) <br>
[3] [Scott Meyers' article that made me rage](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers) <br>
[4] [Performance tests](https://www.modernescpp.com/index.php/copy-versus-move-semantic-a-few-numbers/) <br>
