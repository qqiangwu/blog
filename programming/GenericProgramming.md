# 概述
定义抽象的concepts，本根据concepts来编写算法与数据结构，是GP的本质。concept/modeling/refinement是GP的三大要素。

1. 某个class model了某个concept，或称concept的model
2. 操作某个concept的算法

# Concept
由一组Requirements定义。在现有程序语言之中并没有对应物！

### Requirement
1. valid expressions
2. associated types

### 基本Concept
+ Assignable
+ DefaultConstructilbe
+ EqualityComparable
+ LessThanComparable

Model以上所有Concept的类型称为Regular Type

## Refinement
举例来说，BidirectionalIterator是ForwardIterator的refinement。形式化来说，如果Concept B提供Concept A的所有功能，并且有其他功能，则B是A的refinement。

## Modeling
描述了type 与 concept之间的关系。

## 与OOP的比较
+ Inheritance描述了两个type之间的关系
+ Modeling描述了一个type与一组types之间的关系
+ Refinement描述了两组types之间的关系

## 如何为Concept添加associated types
我们需要为一个类添加额外信息或者行为(Policy/Strategies)。解决方法是加一层间接，即使用traits类。C++中，可以通过类模板，来attach信息，或者retrieve类的内置信息。

## Concept函数重载
对了对Hierarchy中的Concept进行函数重载，我们必须提供一种方式。然而，由于语言本身并不直接支持Concept，因此，我们不得不为concept提供额外的tag（即，以C++的类型系统来表示Concept），函数来进行分发。典型的例子是`iterator_tag`。

# Iterator
将其看作组件，或者Concepts的Hierarchy，而非Concpet，它本身包含6个Concept

1. InputIterator：InputIterator一次只支持一个用于遍历的实例，且是单次只读的，单次遍历的。可以将InputIterator理解为input stream。
2. OutputIterator：单次只写，一个值只能写一次，类似Output stream。如何实现只写（使用Proxy class）
3. ForwardIterator：只能前向遍历、可多次读写、可存在多个实例
4. BidirectionlIterator
5. RandomAccessIterator

InputIterator与OutputIterator均不对应C++内存模型，故其行为极其特殊。故其没有指针与引用的概念。也没有Equality的概念。

特别的：
+ Mutable iteator
+ Const iterator
实际上，可以看这也看作Concept，但它会使得问题更加复杂。所以忽略之。不过在实际Iterator时，需要同时定义两个版本。

# Function objects (Concept)
使用函数对象时，尽量声明为const member function，以提高适用面。

Concepts:  

+ Generater: f();
+ UnaryFunction <- Predicate: f(x);
+ BinaryFunction <- BinaryPredicate: f(x, y);

通过，我们也可以为Functor提供相关类型，这可以通过继承或者声明成员类型完成。然而，由于这些类型用得不是那么广泛，故没有traits类。只有AdaptiveFunctionObject可以与Adaptors一起使用。

### Function objects adaptors

# Containers
值得一提的是block类模板，它定义了一个Aggregate Type，以支持{}赋值，支持Collapse。

# 其他
## 基本Concept
+ StrictWeaklyComparable：LessThanComparable的精化，提供了等价关系(Total ordering是最特殊的)