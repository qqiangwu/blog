最近决定自己实现一遍C++的标准库。这里，记录一些其中遇到的问题。

## 类型分发
在实现泛型算法时，要根据迭代器的类型要选择相应的算法实现，开始，我是这样实现的：
```C++
template <class Tag, class Iterator>
inline void foo(Iterator iter) = delete;

template <class InputIter>
inline void foo<input_iterator_tag>(InputIter iter) { /* ... */ }
```
但是问题就出现在这里，C++中，函数模板不支持这样的特化，即，函数模板无法重载。解决方法倒是很简单，要什么用类添加一层间接，要么用函数参数分发机制。
```C++
template <class InputIter>
inline void foo(InputIter iter, input_iterator_tag);

template <class RandIter>
inline void foo(RandIter iter, random_access_iterator_tag);
```

需要注意的是，函数模板不支持偏特化，但支持特化：

```C++
namespace test {
    template <class T> 
    constexpr const char* name();
        
    template <>
        constexpr const char* name<int>() { return "int"; }
    
    template <>
        constexpr const char* name<char>() { return "char"; }
}
```

## 类与非成员函数
注意，对于一个用户自定义类来说，与它处于同一命名空间的非成员函数仍然可以看作类的一部分，这在函数调用时可以体现。即，对于一个未用名空间限制的函数调用，它同时也会在参数所在的名空间内找相应的函数，这会导致某种冲突，如：

```C++
namespace mstd {
    template <class RandIter>
    void sort_heap(RandIter first, RandIter last)
    {
        for (; first != last; --last) {
            mstd::pop_heap(first, last);
        }
    }
}

std::vector<int> v;
sort_heap(v.begin(), v.end());
```
当我不使用`mstd`来限定`pop_heap`时，函数重载解析也会将`std::pop_heap`考虑进去，因为，它与`vector<int>::iterator`同在一个名空间内，查找时会被找到。

这也启示着，实现泛型算法时，为避免参数类型的类定义可能"包含"这个非成员函数，于是会产生二义错误，即，除非刻意，否则对于泛型算法，一律使用全限定名来调用函数！另一方面，标准库在实现`std::swap`时，实际上使用的是ADL机制。也就是说，使用swap的算法不应该显示用`std::`限定，否则会绑定，这样，即使用户定义的新类型有一个swap，它也不会被考虑，无论这个新类型在什么名空间里。

## noexcept
新的异常规则的实现，意味着我们实现函数时，要考虑其是否为noexcept，像inline和const一样思考这个问题！

## 类多参数模板中的冲突问题
今天在实现`hash_table`时，遇到了这样一个问题：
```C++
size_type bucket_num_(const key_type& key, size_type n) const
{
    return hash_(key) % n;
}

size_type bucket_num_(const_reference obj, size_type n) const
{
    return bucket_num_(key_(obj), n);
}
```
当我使用`hash_table<int, int>`，会出现冲突，其中`using const_reference = const value_type&`。这也启示着，当一个类模板有多个类型参数时，对不同类型参数重载要格外小心。

## 引用不能用统一初始化语法初始化？
可以，我用的编译器有Bug而已。

## 类模板的模板构造函数有什么影响来着？
+ 首先要明白复制构造函数与移动构造函数的定义，它们都必须是非模板。
+ 其次要明白，默认构造函数只有在无用户定义构造函数的情况下才被生成。
以上两者是正交的。

另外：

+ 如果提供了模板构造函数，则默认构造函数不被生成
+ 由于默认复制构造函数是`const`，因为，在有模板的情况下，由于模板通常是非`const`的，故其优先级高。

## `std::rel_ops`
关于这个东西，有以下几点想说的：

1. 可以使用它来为类提供全方位的比较支持。使用方法是，在使用点`using std::rel_ops`
2. 由于`using`了整个名空间，它会改变代码的重载解析空间，可能会产生意想不到的问题
3. 使用`boost::operator`是一个更好的选择
4. 其实我认为的最佳方法是ADL。我尝试这样的代码：  
```C++
namespace mstd {
    class MyT {};

    using std::rel_ops;
}
```
结果证明我想多了。`std::rel_ops`并不会成为`Myt`的关联名空间，从而也就不会进其中查找。但是呢，如果`Myt`有基类，那么，其基类的名空间，会成为关联名空间。于是乎，我们可以这么做：
```C++
#include <utility>

namespace std {
    namespace rel_ops {
        struct comparable {};
    }
}

namespace mstd {
    struct Myt : private comparable {};
}
```
但是，这样仍然有问题，因为标准不允许向`std`内添加内容，且，如果使用了内联名空间，它就失效了。

另外，我觉得自己有必要复习下`C++`中的名字查找与重载的规则了。

好吧，刚看了下标准。尼玛简直不是人看的东西！不过得出以下结论：

+ 对于ADL，如果A是关联名空间，其直接子内联名空间也是关联名空间
+ 对于ADL，关联名空间中的using directives会被忽略。
+ 对于非限定查找，nested classes的定义中句容查找规则等同于类中成员的声明查找规则。对于定义查找，其查找空间可以为整个类。

## {} 与 explicit构造器
explicit 构造器无法使用`{}`

## 类中名字查找的一些Tips
1. 类及其内置类，为经过两次查找，首先解析所有声明（如成员函数、成员typedef），此时，不考虑定义，名字解析由声明点向上查找。
2. 模板定义中名字的查找有些特殊，它的解析的上下文时实例化点，故声明只需要出现在实例化点之前即可。
```C++
namespace mstd {
    struct Container {
        struct iterator;

        iterator begin()
        {
        return iterator{...};
        }
    };

    struct Container::iterator {
    };
}
```

说得清楚一些，这里，依赖地实际上仍然是ADL。上面的`iterator`使用ADL法则，做会在实例化点的上下文的mstd中寻找定义。

## 名字查找的一个Bug
```C++
namespace mstd {
struct my_iterator : public forward_iterator<T> {
reference operator*() const;
}
}
```
这里，出现了一个名字查找的Bug。这段代码的前面，不幸包含了metafunction`template <> reference {}`，而`forward_iterator`中也有一个`reference`类型typedef。此时，解析成了前者，这是为什么？

## C++中的函数变量
在看`std::function`实现的时候，看到这样一段代码：

```C++
  /// Retrieve the result type for a function type.
  template<typename _Res, typename... _ArgTypes> 
    struct _Weak_result_type_impl<_Res(_ArgTypes...)>
    {
      typedef _Res result_type;
    };

  /// Retrieve the result type for a function reference.
  template<typename _Res, typename... _ArgTypes> 
    struct _Weak_result_type_impl<_Res(&)(_ArgTypes...)>
    {
      typedef _Res result_type;
    };

  /// Retrieve the result type for a function pointer.
  template<typename _Res, typename... _ArgTypes> 
    struct _Weak_result_type_impl<_Res(*)(_ArgTypes...)>
    {
      typedef _Res result_type;
    };
```
后来得出结论，函数声明实际上可以看到声明一个函数变量。即，`void(int)`它是一个函数类型。与`void(&)(int)`和`void(*)(int)`是正交的。我试了以下这段代码：
```
int main()
{
    using Fn = void(int);

    Fn t;
}
```
这段代码的作用相当于一个函数声明`void t(int)`。

另一方面，函数类型与数组类型有着类似的性质。如：
```C++
using Fty = void(int);

void run_fun(Fty fn)
{
    // Now the type of fn is void(*)(int)
    fn(10);
}
```
这也就意味着，当作为函数参数时，函数类型会退化成指针类型。这和数组相似。

再看以下例子：
```C++
void f();

using FunT = void();

int main()
{
    auto pf = f;// pf is FunT*

    static_cast<FunT>(pf);// error
    reinterpret_cast<FunT>(pf);  // error
    static_cast<FunT&>(pf); // ok
    static_cast<FunT>(f);  // error; f degenerates to FunT*
    static_cast<FunT&>(f);// ok
}
```

## 迭代器解引用问题
以往，在实现一个算法时，一个迭代器可能会多次解引用，于是，我会使用这样的写法`auto& x = *iter`，而实际上，这样的写法可能是错误的，因为，`*iter`也许会返回一个右值。此时，我们可以这样：`auto&& x = *iter`。

## `return make_tuple` 与 `return {}`
每次从函数中返回pair的时候我都要停下来想想使用`{}`到底行不行，今天要把这个问题弄清楚。看这段代码

```C++
class Employee {
public:
    Name name() const;
    Address address() const;
    Date hireDate() const;
};

Employee findByID(unsigned eid);

std::tuple<Name, Address, Date> employeeInfo(unsigned employeeID)
{
    Employee e(findByID(employeeID));
    return std::make_tuple(e.name(), e.address(), e.hireDate());
}
```
其中，函数的return语句不能使用`{}`，因为，tuple的构造函数要么是explicit的，要么是模板构造器，两者都不能使用`{}`。（其中，`void f(T x)`传入`{}`时，不会进行类型推导）。注意，仅仅是tuple这是样！

```C++
template< class... UTypes >
explicit constexpr tuple( UTypes&&... args );
```