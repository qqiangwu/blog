C++拒绝了static if的提案，这一度让我觉得很奇怪。现在，我来对比一下[N3329](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3329.pdf)和[N3613](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2013/n3613.pdf)，来看一下为什么拒绝static if，以及，如何使用C++14来实现static if的功能。

为什么拒绝？实际上，static if本质上实现了三种语义，语义和功能上的混杂，使得代码更加难以理解。

# Conditional Compilation
## 语句选择
eg.
```C++
static if (sizeof(T)==8)
    gun(32);
else
    gun();

static if (sizeof(T)==8) {
    long n = x;
}

static if (sizeof(T)!=8) {
    tx n = x;
}
```
显然，这段代码理解时很困难，阅读代码时必须时刻记得当前处于哪个branch中，哪些declaration被采用。并且，static if 不引入作用域，容易使用代码更乱

```C++
template <class It, class T>
void uninitialized_fill(It b, It e, const T& x) {
    static if (std::is_same<typename std::iterator_traits<It>::iterator_category,
        std::random_access_iterator_tag>::value) {
        assert(b <= e);
}

static if (std::has_trivial_copy_constructor<T>::value) {
	std::fill(b, e, x);
} else {
	// Conservative implementation
	for (; b != e; ++b) new(&*b) T(x);
}
```
这段代码是IDE不友好的，许多错误只有在编译时才能被检测出来。从用户角度来说，实现功能时，用这段代码问题不大，除了阅读起来过于复杂。

## Class Scope
```C++
template <unsigned long n>
struct factorial {
    static if (n <= 1) {
        enum : unsigned long { value = 1 };
    } else {
        enum : unsigned long { value = factorial<n - 1>::value * n };
    }
};
```
看起来没有什么问题，但实际上，用constexpr是一个更好的选择。

```C++
template<typename T, typename A>
class dynarray {
    T* start_;
    T* finish_;
    static if (!is_empty<A>::value) {
        A alloc_;
    }
};
```
出现这样的代码时，每一次我们访问dynarray时，都需要写额外的代码来访问`alloc_`，显然，这会使得代码更加凌乱。一种好的方法时，将所有逻辑封装在一个类中。

# Static If and the Preprocessor
static if与已有的宏功能冲突太大，且合作起来并不是很好。

# Constraining Templates
```C++
template<typename I, typename T>
I uninitialized_fill(I first, I last, const T& value) if (std::is_convertible<T, typename iterator_traits<I>::value_type>::value)
{/* ... */}
```
首先，这个功能很有用(template constrains)，但是，使用if实现过于复杂，且重载时很麻烦，因为，我们需要写这样的代码：
```C++
template<typename I>
void advance(I& i, int n) if (is_input_iterator<I>::value && !is_bidirectional_iterator<I>::value)
{ ... }

template<typename I>
void advance(I& i, int n) if (is_bidirectional_iterator<I>::value && !is_random_access_iterator<I>::value)
{ ... }

template<typename I>
void advance(I& i, int n) if (is_random_access_iterator<I>::value)
{ ... }
```

目前，`enabled_if`某种程序上实现了类似的功能，但是，使用Concept-lite是一个更好的选择。

# 其他
+ 它与Concept等概念冲突
+ 这使用IDE更加难以开发
+ 使用static if的代码更加难以阅读

# 替代方案
## 语句选择
对于根据元信息进行语句选择的代码，使用静态分发机制或者Policy对象封装机制是一个很好的选择，这样的代码相对而言一个功能块中逻辑较为完整，不会导致过于零散的代码。
对于Class Scope中的选择语句，使用Policy对象是一个更好的选择。
对于Factorial<10>::value，这样的代码用constexpr来实现更加合适。所有静态计算都应该使用constexpr来实现！这才是它最主要的用处。

有一类特殊的用法是，根据类模板参数判定是否定义一个成员函数。如
```C+
template <class T>
class A {
    static if (is_some_thing<T>::value) {
        void foo();
    }
}
```
我们有两种选择，一是实现重载两个完全不同的类。毕竟，这样的代码更加清晰：它们是两个不同的东西！但有时，差异可能只有一点点，我们可以将公共的成员放到一个基类中去。或者，我们可以使用`require`语句，以及`std::enable_if`。

## 模板Constraints
使用Concept-lite

# 总结
开始在D语言中看到static if时，还觉得它很厉害。但看它论文后才知道，在C++中，它并不是一个好东西。不得不说，BS的眼光还真是敏锐！

这也告诉我，写代码时，要使用最恰当的语言特性，来表达自己的意图。如，使用constexpr来实现静态计算。同时，要提高代码的可读性，一个实体只做一件事情，不要用很多很花哨的技巧！