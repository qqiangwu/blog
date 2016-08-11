# 概览
IO是Java中的最重要的一个部分. 其中, java.io是所有编程者都应该掌握的IO方式. 在Java 1.4中, NIO被引入, 它引进了一种新的相对于流模型的新的IO模型, 以为非阻塞IO提供支持. 在Java 7中, NIO2又在NIO的基础上, 引入了对异步IO的支持. 在这篇文章我, 我将对这几种IO方式进行一个比较系统的说明及总结, 同时, 分析每一种IO模型的适用范围.

# 流IO
流IO是一种最为简洁的IO方式, 几乎所有的编程语言在其标准库中都提供了对流IO的支持, 比如C的FILE, C++的iostream. 同样, 在Java中, 流IO是最为基础也是最为广泛使用的IO方式, 一般来说, 大家对这种方式都比较熟悉了, 总的来说, Java提供Byte输入流/Byte输出流/Char输入流/Char输出流, 同时, 又提供了一系列的Decorator类来在基础流的功能之上, 添加新的功能, 如Buffering.

用下面的四张图可以很好地概括整个流IO的相关类.

+ ![输入流](http://7xqdwr.com1.z0.glb.clouddn.com/java-io-class-hierarchy-diagram-inputStream.gif)
+ ![输出流](http://7xqdwr.com1.z0.glb.clouddn.com/java.io.OutputStream.gif)
+ ![字符输入流](http://7xqdwr.com1.z0.glb.clouddn.com/java-io-class-hierarchy-diagram.gif)
+ ![字符输出流](http://7xqdwr.com1.z0.glb.clouddn.com/java.io.Writer.gif)

# NIO
一般来说, 流IO表现得很好, 对于大部分的IO场景, 它都能适应. 但是, 由于它的阻塞性, 每一个流的读写都需要占用一个线程. 这意味着, 流IO的可伸缩性很差. 因此, 引入非阻塞IO就再正常不过了. 实际上, NIO就是IO Multiplexing在Java中的实现. IO Multiplexing在系统级语言如C/C++中应用了很长时间. 使用IO Multiplexing, IO的伸缩性大大提高, 使用单个线程, 就可以处理大量的IO对象.

在介绍NIO的非阻塞IO之前, 先大致了解一下NIO提供的IO模型. NIO的概念概念有三个, Buffers/Channels/Selectors. 其中, Channels是输入/输出的管道, 所有的读写操作都需要通过它来完成. Channel读写的粒度是Block, 而不是像流IO一样, 提供一个字节流或者字符流的抽象. 这个Block的抽象即Buffer. 所有的读操作会由Channel将数据读入Buffer, 然后用户来处理Buffer, 所有的写操作需要先将数据填到Buffer中, 再由Channel来消费Buffer中的数据. NIO的第三个核心概念是Selector, 它是一个事件监控器, 我们将它注册我们所感兴趣的IO事件, 并且对其进行Polling, 来确定事件是否发生, 发生则做相应的IO操作. 其中, Selector所监控的对象是Channel, 我们在Selector上声明我们关心哪一个Channel的什么事件, Selector会监控这些Channels, 并在事件发生时通知我们.

现在, 考虑三个问题:

+ **为什么要引入Channel, 直接扩展已有的Stream类不行吗?**: 流的抽象已经很完备了, 添加更多的特性与概念只会将流的概念进一步复杂化, API更加难以使用, 这是一种很不好的API设计方式. 因此, NIO引入了一套新的抽象. Do one thing, and do it well.
+ **为什么引入Buffer? 直接用byte数组可以吗?**: 实际上肯定是可以的, 但Buffer类提供了更加方便的操作. 同时, Buffer提供了很多性能上的优化.
+ **为什么引入Buffer? 直接读写byte不行吗?**: 如果直接操作byte, 性能会很低, 实际上还是需要buffering来提供性能, 与其加一层buffering抽象, 不如直接给用户提供Buffer. 最重要的是, 基于Buffer的IO操作, 某些情况下可以直接映射成系统调用, 性能极高!

NIO支持阻塞与非阻塞两种模式. 阻塞模式下, 实际上与流IO差不多, 非阻塞模式下, Channels与Selector配合, 才是它最大的威力所在.

我们可以大体将Channel分成两类, 一种是支持SelectableChannel(除了FileChannel以外都是, 一般是网络相关的操作.), 另一种与non-SelectableChannel(即FileChannel). 前者可以与Selector一起使用, 提供强伸缩性的IO.

## IO VS NIO
考虑IO与NIO的区别. 除了在概念模型的差别, IO与NIO在性能上也会有很大差异. 我们从三个方面来考虑性能问题:

+ 可伸缩性: 流IO的在IO对象数较少及大规模IO的情况下, 表现得很好, 但是当需要处理成百上千的IO对象时, 它的性能会Drop得很快. 相反, NIO在非阻塞模式下(阻塞模式下应该与流IO具有相同的特点, 这是阻塞IO的共性), 即使用Selector, 它可以处理大量的非活跃连接, 是实现C10K的关键技术.
+ GC: 许多号称高性能的服务器实现, 都以Zero Allocation作为一个重要的功能点. 理想情况上, 如果没有GC的开销, 服务器可以将所有时间花在有效地工作上, 并且保持一个可靠的延迟. 然后GC是不可避免的, Zero Allocation也只能是尽力而为. 而相比较而言, NIO只需要申请一个Buffer, 可以反复使用, 而字符流在这方便表现的就比较差了, 如readLine()这类接口, 需要分配大量临时的String对象.
+ API抽象层次: 相对而言, 基于Buffer的NIO抽象层次比流IO在低一些. 特别的, 系统调用级别的IO, 都是基于Buffer的. 当使用DirectBuffer时, 某些平台下, OS可以直接将数据复杂到DirectBuffer中, 避免了流IO中, OS将数据复制到OS Buffer后, 又需要向JVM Heap复制地过程. Zero Copy与Zero Allocation都是高性能服务器的重点技术. 特别的, 在使用Channel时, 需要使用DirectBuffer, 因为Channel内部使用的是DirectBuffer. 如果使用HeapBuffer, 则读写时, Channel会申请一个临时的DirectBuffer, 造成性能开销.

## Memory Mapping
前面提到, FileChannel不支持非阻塞模式. 那么, 它是不是用处不大呢? 毕竟, NIO与IO相比最大的优势是非阻塞.

NIO中, FileChannel都一些属于自己的特性. 即, Memory Mapping. Memory Mapping是一个比较觉见, 在此不加多说. 无论是在顺序读写, 还是随机读写中, Memory Mapping都能够提供不弱于BufferedInputStream或者RandomAccessFile的性能.

特别强调的是, Memory Mapping可以Map的容量仅与虚拟内存大小有关, 与物理内存大小及JVM堆大小都没有关系. 因此, 在64位平台下, Memory Mapping可以工作得非常好.

# NIO2
聊过非阻塞IO后, 再来看看异步IO. IO方面的概念很多, 阻塞性与异步性是其关键概念. 简单而言, 凡是需要由应用程序将数据读写到应用程序内存中的IO, 都是同步IO, 比如上面的流IO与NIO. 相对的, 凡是由OS来完成读写的, 就是异步IO. 这个说法有些迷惑. 举例而言, 在NIO中, 当应用程序检测到某个Channel有可读数据时, 必须显示发起一个read请求. 而在异步IO中, 应用程序仅仅需要告诉OS, 我需要什么数据, 并提供给OS一个Buffer和一个回调. OS会自己检测Channel的可读性, 但其发起其可读, 会自动将数据复制到Buffer中, 并通知应用程序任务完成. 异步IO的典型实现是NodeJS及Boost.ASIO. 显然, 由于将任务进一步下发到了OS, 应用程序的可伸缩性及性能会大大增强. 并且, 比起非阻塞的NIO, 异步IO编程更加容易一些, 性能也基本上总是优于它的.

NIO2最大的改进是引入了四个异步Channel, 用于支持异步读写. 同时, 它还增加了对文件系统和文件属性的支持, 提供了WatchService/FileVisitor这些高级功能. 
