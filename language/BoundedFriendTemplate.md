有这样一种奇怪的语法：

```C++
template <class T>
struct ostream_iterator {
    friend bool operator==<>(const ostream_iterator<T>& x, const ostream_iterator<T>& y);
};

template <class T>
bool operator==(const ostream_iterator<T>& x, const ostream_iterator<T>& y) { /* ... */}
```

类模板中友元函数上的<>不能省略。

类模板的友元可以是普通函数和其他类。也可以是不依赖于类模板类型参数的函数模板及类模板，此时，它将友元关系授于所有友元模板生成的实体。前两种情况下，友元声明可以直接当实体声明用。还有第三种友元，即，友元是模板，但这个模板依赖于类模板的类型参数，即（bounded），此时，我们应该如何写这个友元声明呢？可能的写法可能有这些：

```C++
// way 1
template <class T>
struct ostream_iterator {
    template <> friend bool operator==(const ostream_iterator<T>& x, const ostream_iterator<T>& y);
};

// way 2
template <class T>
struct ostream_iterator {
    friend bool operator==(const ostream_iterator<T>& x, const ostream_iterator<T>& y);
};

// way 3
template <class T>
struct ostream_iterator {
    friend bool operator==<>(const ostream_iterator<T>& x, const ostream_iterator<T>& y);
};

// way 4
template <class T>
struct ostream_iterator {
    friend bool operator==<>(const ostream_iterator& x, const ostream_iterator& y);
};

// way 5
template <class T>
struct ostream_iterator {
    friend bool operator==<T>(const ostream_iterator& x, const ostream_iterator& y);
};
```

way1是错误的，因为类友元不是能一个模板函数的特化。
way2会导致模板实例化(用int)后去寻找一个非模板函数`bool operator==(const ostream_iterator<int>&, const ostream_iterator<int>&)`。
way3/way4/way5是相同的。在一个模板类中，`Myt`与`Myt<T>`相同。我们用`<>`或者`<T>`来表明这个友元依赖于类模板的类型参数。需要注意的是，Bounded友元必须前置声明其模板形式。

实际上，这样写好像也可以：
```C++
template <class T>
struct ostream_iterator {
    friend bool operator==(const ostream_iterator& x, const ostream_iterator& y)
    {}
};
```
即，直接在类中进行定义。