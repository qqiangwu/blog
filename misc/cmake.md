C++虽然是一个跨平台的语言，但由于第个平台的构建工具均有不同，因此，其跨平台工作难度也就大了许多。于是，许多跨平台的构建工具也随之出现。比较著名的有：

1. qmake
2. cmake
3. premake

当然，还有boost弄的那一堆东西，就不提了。

其中，premake是基于lua语言的，从理念上看，我认为它比较优秀。但是，现在许多C++开源项目都是基于cmake的，同时，Biicode也是基于cmake的，因此，我决定学习它。

# cmake
我们可以将cmake看作一门指令式语言。它有变量，有控制语句。cmake的工作对象是CMakeLists.txt，我们可以将它看作cmake这门语言的源代码。一个文件中可以引用另外一个文件。

考虑到cmake文件实际上是描述了一个工程，因此，它在某种意义上更像是一门说明式语言，我们最终的目标是定义一个编译目标，为了定义这个目标，我们需要确定它所使用的源文件，以及编译时使用的选项，以及连接的库，考虑到工程的复杂性，它还可能包含子工程。它的主要部分：

1. 工程名：`project("project-name")`
2. 版本说明：`cmake_minimum_required(VERSION 2.8)`
3. 编译目标生成：`add_executable/add_library`
4. 编译目标设置：
    + 全局设置：`include_directories/add_definitions`
    + 目标设置：`target_compile_definitions/target_compile_options/target_link_libraries/target_include_directories/target_include_directories`
    
对于目标设置，通常我们还需要指定PRIVATE及PUBLIC，来确定它是否是可传递的，即，包含它的新工程是否也使用这些性质。

我们所有的目标，都是生成executable以及library等。

# 值与变量
## 变量的值
cmake中，所有值都是string。其基本数据类型可以看作有atom与list，但atom实际上也只是元素数为1的list。如：

1. atom: "value"
    + bool: "ON"/"OFF"
2. list: "value1;value2;value3"

简单值如v可以不需要加""，但复杂值如"hello world"则必须加""。其中，有一类特殊的atom，其值为ON/OFF，可以将其当作bool值直接在if中使用。

## 变量名
实际上，变量名也只是一个string，我们可以通过${}来获取其对应的值。可以理解为cmake维护了一张符号表。如：

1. `set(MYVAR /var/run/bootstamp)`
2. `set(MYVAR "/var/run/bootstamp")`
3. `set(MYVAR "element1")`
4. `set(${MYVAR}_name "hello")`
5. `set(MYVAR2 element1 "${MYVAR}_name" element3 ${${MY_VAR}_name}})`
6. `set(myvar ${a}-${b})`

在值内部，可以发生变量值替换。说起来cmake更像是一个宏替换语言，因此，如果`message(${not_exist})`会报错

## 变量操作
基本上所有函数都支持list，我们可以直接使用list literals，或者使用多个值，由函数来它们聚合成list，如：
1. `set(var a b c)`
2. `set(var "a;b;c")`

1. 赋值：`set(var_name value1 value2 value3 ...)`

## List操作
1. 删除：
    + `list(REMOVE_ITEM MYLIST elem1 elem3)`
    + `list(REMOVE_ITEM BII_LIB_SRC “src/nat/UDPwin32.cpp”)`
2. 添加：
    + `list(APPEND MYLIST elem5 elem6)`
3. 包含：
```
    list(FIND MYLIST_VAR "src/google.cpp" HasGoogleFile)
    IF(HasGoogleFile EQUAL -1)
        message(FATAL_ERROR "I could not find google.cpp")
    ENDIF()
```

## 预定义变量
1. 目录相关
    - `CMAKE_CURRENT_SOURCE_DIR`: source directory currently being processed
    - `CMAKE_CURRENT_BINARY_DIR`: binary directory currently being processed
    - `CMAKE_RUNTIME_OUTPUT_DIRECTORY`: directory being used as IDE run
2. 系统描述
    - `APPLE`: True if running on Mac OS X
    - `UNIX`: True for UNIX and UNIX like operating systems (i.e.: APPLE and CYGWIN)
    - `MSVC`: True when using Microsoft Visual C
    - `WIN32`: True on windows systems, including win64
    - `MINGW`: True when generating MinGW Makefiles
    - `CMAKE_SYSTEM_NAME`: Name of system cmake is being run on

## 环境变量
通用`$ENV{var}`获取环境变量
    
# 控制语句
+ 条件  
```
set(flag ON)

if(flag)
    message("flag is on")
else()
    message("flag is off")
endif()
```
+ 迭代  
```
set(mylist a b c d e f)

foreach(i ${mylist})
    message(${i})
endforeach()
```

# 文件操作
用于指定文件，因为每个编译目标都需要定义其对应的源文件。

1. 通用操作：file 如 `file(GLOB_RECURSE HEADER_FILES ./**/*.hpp)`
2. 使得操作：`aux_source_directory(dirname varname)` 将dirname下所有文件添加入varname中
    
# 编译目标生成  
1. `add_executable(exename files ...)`
2. `add_library(libname files ...)`

# 编译目标选项设定
## 全局属性
对所有编译目标起作用  
1. `include_directories`
2. `add_definitions`
    
## 目标特定属性
1. `target_compile_definitions`
2. `target_compile_options`
3. `target_link_libraries`
4. `target_include_directories`

## 添加子项目
注意，在子项目中不需要指定project  
1. `add_subdirectory(dirname)`
2. 使用include引入另外一个file或者module，这样，可以使用其中的cmake代码

## 测试相关
1. 通过`enable_testing()`，可以编译test targets。
2. `add_test(testcasename exename arg1 arg2 ...)`：执行exename arg1 arg2 ... 其中，exename必须已经由`add_executable`定义过。`add_test`相当于是为其定义了一个快捷方式

## 预定义option
由作者定义的option，可以理解为输入吧，或者command line之类的东西。如:
```
option( CPP-NETLIB_BUILD_SHARED_LIBS "Build cpp-netlib as shared libraries." OFF )
option( CPP-NETLIB_BUILD_TESTS "Build the cpp-netlib project tests." ON)
option( CPP-NETLIB_BUILD_EXAMPLES "Build the cpp-netlib project examples." ON)
```

# `find_package`
用于添加依赖，不过实际上做得真心不怎么样，最起码不具有通用性

# Output
1. `message("hello world MYVAR=${MYVAR}")`
2. `message(SEND_ERROR "Should not have got here")`
3. `message(FATAL_ERROR "Something bad happened")`

# 结语
就先看这些了，需要用到更复杂功能的时候再研究