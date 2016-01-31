前段时间了解了int与uint的区别，于是，`std::size_t(-1)`这句代码到底是不是可移动的就一起挂在了我的心上。现在解决下这个问题。

除了`reinterpret_cast`之外的cast，作用的是value，而不是representation，所以，它们都是可移植的！
要区分值和bit。bit是实现定义的，而位操作是依赖于bit的，故对值使用位操作，其行为需要额外注意。
例子：

1. unsigned long x = ~0u; 不一定得到全1
2. unsigned x = ~0; 不一定得到全1
3. -1 其值不一定全1
4. 0 其值一定全0
5. `std::size_t(-1)`到底做了些什么：`std::size_t`的表示未定义，但其行为定义了，具有`2^n-modulo`，也即，`size_t(-1) === 2^n - 1`，也即`std::numeric_limit::max()`，