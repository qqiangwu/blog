# 通用理念
1. 使用不可移植的方式，实现可移植的语义。标准库大量提供了这样的例子，如offsetof，以在不扩大语言核心的基础上，提供更加强大的功能。
2. C std命名规则 
+ `__name`及`_Name`，是预留给系统使用的
+ C++：`_name`这样的名字只能在非全局空间使用
+ locale中fmtval的例子：名字隐藏。如果不在标准头文件及实现中使用fmtval，用户可以使用fmtval，Linker如果在用户代码中找不到它的定义，会自动扫描标准库代码，如果找到，则不扫描

# 实现
## assert.h
1. 为什么要使用assertion
2. 如何正确地使用assertion：assertion不能取代错误处理，错误恢复有时是很重要的！仅仅将assertion用在invariant和pre/post，而不要将其做为检测错误的方法
3. assert.h可以多次被包含，其行为取决于NDEBUG是否被定义，当然，最好在make中定义NDEBUG，使得assertion的行为一致
4. Benign redefinition与benigh undefinition
5. 改变一个宏的定义时，需要先undef。如，在assert.h中，需要先undef assert
6. 其他实现细节：`_Assert(__FILE__ ":" _STR(__LINE__) " " # test)`
+ `__FILE__`是一个字符串，可以安全使用
+ #：string-creation operator
+ 字符串的自动拼接
+ `__LINE__`是一个常量，如果直接使用`#__LINE__`，会得到错误结果，所以，要先将它作为宏实参传入，将其变成一个integer constant，当然，此时，不能使用#，否则，值不会被替换。得到integer constant之后，再使用一个宏，从而将其转换成字符串，省略任意一个，都会得到字符串"__LINE__"

## ctype.h
1. 对字符进行分类是一个很常见的工作
2. 使用ctype，而不是手动写代码判定，因为，不同字符编码，其判定规则是不同的，手写的代码一般针对于特定字符编码
3. 在头文件中同时定义宏与函数，使用宏内联提高效率，提供函数以便取地址
4. Translate Table技术
5. 对函数名加()防止宏扩展
6. ctype函数可以接受EOF及所有unsigned char可以表示的字符。传入一个可能有符号的char，如char(-2)，其结果是未定义的。一个安全的写法应该是isalpha((unsigned char)ch)，其中ch为char类型

## float.h
1. 浮点运算是复杂的，有些机器甚至不提供浮点运算。
2. 浮点运算有自身的内在不足
+ overflow：结果未定义
+ underflow：结果未定义
+ signifcance loss
+ 不同机器即使使用同一表示，其行为可能也会有差异，如round mode
3. C标准无法严格定义浮点数，因此，它只是描述了一个浮点representation模型，并用宏来描述了它的性质，如最大浮点数，最小浮点数，最小精度之类以及行为。实际上，即使给出这些，也很难写出可移植的代码
4. 总之，涉及浮点运算时，要格外小心
5. 技巧：使用union来观察representation

## limits.h
1. 和float.h相似，用于描述类型的特性
2. 里面的宏可以要预处理中使用，以进行类型selection以及运行环境检测
3. 其他，标准规定int可以有三种合法的编码，我们所熟悉的是二补码。其他两种形式会有两个0，正负0。因此，不要对int的表示有任何假设！