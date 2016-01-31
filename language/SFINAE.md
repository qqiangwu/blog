SFINAE是由标准所定义的，具有明确行为的一个语言特性。不过C++也真是作孽！

看这段代码：

```C++
template <class Iter>
Iter mean(Iter first, Iter last)     // #1
{
    using Value_type = typename Iter::value_type;
}

template <class T>
T* mean(T*, T*);            // #2

void f(std::vector<int>& v, int* p, int n)
{
    auto x = mean(v.begin(), v.end());  // call #1
    auto y = mean(p, p + n);            // pick both and call #2 for best matching
}
```
当一个模板调用被实例化时，它会先找到可用的模板声明（注意，只考虑签名），然后将其加入到重载集中。上面的例子中，对于y的调用来说，两个模板都符合，故出现二义错误。如果将`#1`改为：

```C++
template <class Iter>
typename Iter::value_type mean(Iter first, Iter last);
```

那么，在用此模板声明进行匹配时，匹配会失败，帮不会加入重载集。此时，只会考虑第二个模板

之后，编译器会在重载集中选出最佳匹配，然后实例化它。如果出错（即模板定义中出现了错误），则直接报错，而不是尝试其他的匹配。

# 作用
+ 实现`has_foo`函数