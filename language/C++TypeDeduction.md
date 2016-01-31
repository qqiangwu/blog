C++中的type deduction太多了，来总结一下吧：

# 类模板推导
+ 一般模板：`T`直接为传入的表达式的类型
+ 特化模板：严格匹配`T`顶层的内容，剩下的部分进行Pattern-match，以得到`T`

# auto-related deduction
```C++
template <class T>
void f(ParamType param);

f(expr);
```
分三类情况
## Non-URef Reference/Pointer Parameters
即，非universal reference时的有引用或者指针的类型推荐。
规则：
+ 如果expr是引用类型，去掉引用
+ Pattern-match expr与ParamType

eg.
+ `f(T& param)`中，T可以是`int`或者`const int`，取决于expr的类型
+ `f(const T& param)`，T是`int`，无论expr的类型是什么

+ **指针同理**
+ **auto在类型推导时的相当于T**

## Universal Reference
```C++
template <class T>
void f(T&& param);

f(expr);
```
规则：

+ expr为左值时，`T&&`为`E&`，其中，T为`T&`
+ expr为右值时，`T&&`为`E&&`，其中，T为`T&&`

## By Value
```C++
template <class T>
void f(T param);

f(expr);
```
规则：

+ 去掉expr的引用
+ 去掉expr的cv
+ 剩下的就是T

总结来说，顶层的cv会被去掉。

**特例**：
如果expr是函数或者是数组的话，推导规则如下：

+ 如果是引用，则推导出函数或者数组类型
+ 否则，退化为指针

**特例**：
如果使用`{1，2，3}`，推导规则复杂一些

+ 如果是函数，则推导失败
+ 如果是`auto x = { ... }`，则为`std::initailizer_list`
+ 如果是`auto x{10}`，则括号中只能使用一个元素，用其类型进行auto推导

# Lambda captured var deduction
+ By Ref: 使用模板推导规则
+ By Value: 使用模板推导规则且cv保留
+ Init capture：使用auto规则

注意：lambda默认为`operator() const`，当然，可以这样：`[c] () mutable {}`  
注意：不要使用typeid来观察类型，因为其结果很多时候是错误的。

# decltype
+ decltype(name)：使用name的声明类型
+ decltype(expr)：如果expr是左值，则添加&
+ decltype((name))：将name转换成expr

# 函数返回类型推导
+ auto：使用模板推导规则，而不是auto变量推导规则，即，不支持`{}`
+ decltype(auto)：使用decltype规则

结论：

+ 不需要引用的时候，使用auto
+ 可能需要引用的时候，使用decltype(auto)