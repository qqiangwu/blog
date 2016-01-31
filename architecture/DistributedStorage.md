# 绪论
10年前,Google发表了三篇跨时代的论文,拉开了分布式系统在工业界大规模应用的序幕.之后,随着Web2.0的发展,分布式系统在全世界范围内占据了统治地位,传统的依赖单机强大计算能力及昂贵的企业级存储设备的企业信息系统应用,都逐渐向分布式系统过渡.那么,分布式系统到底是什么,为什么它能够有这么强大的能力?

分布式系统的概念早在上个世界60年代就已经出现,对它的研究也从而断绝过,然而,它一直是一个学术概念,并没有在世界范围内得到大规模的应用.然而,随着Web的发展,数据量已经突破了单机的处理及存储极限,仅仅依靠Scale up已经无法当满足现实的需要,于是,计算的并行与数据的分发成为了解决问题最为直观与简单的手段.本文将尝试对典型分布式系统进行解释与分析,了解分布式系统设计与实现中的相关问题,加一步加深对分布式系统的理解.

# 分布式存储系统概述
## 分布式系统
当单机的处理及存储能力无法满足人们的要求的时候,使用多台机器来处理和存储数据是一个十分直观的选择.正如CPU由单核变成多核以及RAID的出现与应用.于是,**Scalability** 成为了分布式系统最为直观也最为本质的物质.同时,当系统将单机变成多机时,系统的故障率将大大增加,**Fault Tolerance** 也成为了分布式系统不可忽视的特性.至于 **Performance**/**Consistency** 等等特性,我认为可以作为两种基本属性的衍生特性. **Scalability** 带来了 **Performance** 上的垂直提升；同时,数据被分布后,在故障存在的情况下,其 **Consistency** 成为了一个自然的要加以考虑的结果.

以下各节,将从 **Scalabiltiy** 与 **Fault Tolerance** 两个角度来对分布式系统进行分析, 并使用抽象/虚拟化,模块,分层等概念来剖析系统. 我将以 **可伸缩性** 与 **容错性** 来表示 **Scalabiltiy** 与 **Fault Tolerance** (我认为可扩展性批的是Extensibility).

## 分布式系统的分类
在考虑如何解决可伸缩性与容错性时,我们尝试将问题 **特化**. 一般地,分布式系统可以被分成有状态的与无状态的.无状态的通常为计算节点,有状态的通常为存储节点.典型的例子是,在一个网站中,网站应用通常是无状态的,所以有状态被放在了持久化层,如分布式存储与分布式数据库.

对于无状态的计算节点,我们可以通过简单地加节点来实现可伸缩性.同时,只要系统中任意一台机器还在工作,系统就还能够正常运作.这样的容错能力是近乎完美的!

而对于有状态的存储节点,事情就复杂了许多,分布式系统中的绝大部分工作都是在研究此类系统,如Paxos,Gossip,Chord,Broadcast.因此,本文中的重点将是有状态的存储系统.

当然, 事情会不问题绝对的, 很多系统是Hybric的, 它是计算节点, 但它是有状态的, 这并不是没有道理的, **想要极限性能, 必须要减一层抽象**. 这不是好的学术实践, 但它是合理的工程实践!

## 存储系统
存储系统为我们的系统提供了持久化的能力,通常它是一个系统中最复杂的部分,它与系统的性能直接相关.

我们可以将一个存储系统分成三 **层**:
+ 存储介质层: 提供数据持久化的能力, 如磁盘  
+ 存储引擎层: 构建于介质层之上, 提供原始存储能力, 典型的存储引擎如B树, Hash表, LSM
+ 存储模型层: 构建于引擎层之上, 提供对象模型, 供应用层使用, 如文件模型, kv模型, 关系模型

我们只考虑存储引擎层与存储模型层, 因为这才是与我们的软件相关的. 于是显然, 状态管理被集中在了存储引擎层. 模型层通常只是加一些缓存, 来提高模型的效率, 如Bigtable. 当然, 也有有状态的模型层, 如GFS. 相比而言, GFS的错误恢复就比Bigtable复杂了一些.

## 分布式存储系统
分布式存储系统是分布式系统中最难的部分, 按照分层与模块的思考, 我们对其功能进行 **正交分解** (实际上,很多时候,分层还是模块,并不是那么直观,我简单地认为,分层是水平功能分割,模块是垂直功能分割,总之,它们将是将系统分解,并实现 **单一职责原则** 的基本手段):  
+ 分布式成员协议: 提供成员管理, 失败检测, 成员加入, 成员退出的支持
+ 存储协议: 数据分布, 数据存取的支持

以下, 我将对典型的分布式存储系统进行学习, 并按照 **可伸缩性与容错性**, **存储引擎与存储模型**, **分布式成员协议与存储协议** 三个层面来进行剖析.

# 典型系统
## GFS
GFS是一个特化的系统, 它是一个典型的分布式存储系统, 但是它不是一个通用的分布式文件系统, 而是根据Google内部业务需求而特别定制的一个实现. 这就使得它对于Google内部的应用有着非常高的效率. 这在某种程序上告诉我们, **使用一个特化的, 适合自己场景的系统**. 与之相对的, 淘宝TFS, Facebook Haystack均对少量小文件做了优化, 这样的分布式文件系统同样也工作得非常好! 对于分布式存储系统这样一个复杂的领域, 问题的解决本身就已经十分复杂, 因此, **不要尝试做一个通用的系统**. 如果GFS在一开始就像NFS那样, 实现一套完全兼容于inode的文件系统接口, 那么, 其实现复杂度及效率都将是出乎我们的想象.

### 分布式成员协议
GFS使用了一个中心化的节点(即Master)来管理所有数据存储节点. 要知道, 在10年前, P2P技术已经发展得相当完善了, 为什么GFS还要使用中心化节点呢? 要知道, 中心节点意味着瓶颈的存在, 意味着可伸缩性差. 既然要做一个分布式系统, 为什么不使用P2P技术, 来完全避免中心节点的存在呢? 一个很奇怪的现象是, 当今所有成功的分布式存储系统, 全部都是有中心节点的! 相反, 使用完全P2P技术的分布式存储系统, 其成功程序远远落后其他系统. 典型的, 本文所介绍的系统, 除了Dynamo, 其他所有系统分部都是有中心节点的存在的. 这是为什么?

或者, 用 **Worse is better** 以及 **KISS** 这两条原则可以解释这一现象吧. 一般, 中心化节点用于管理整集群以及应用的元数据. 我将分别从这两人方面来进行分析.

+ 集群管理: 中心节点用于管理整个集群, 当集群内有成员加入/退出/失败时, 都能够进行处理, 这可以简单地使用心跳机制来完成. 如果使用去中心化方案, 要完成相同的任务, 我们需要Gossip, 需要Failure Detector, 需要考虑同步问题, 需要考虑一致性问题. 显然, 其实现难度大了许多倍. 然而, 它真的带来了想像中的好处了吗?
+ 应用元数据管理: GFS中, 应用的元数据库是各Chunk的信息, 以及文件命名空间. 在GFS中, 一个Chunk不足64个字节, 一个文件名字信息也不足64个字节. 一个普通的内存为16GB的服务器, 可以存储TB级数据, 如果再对数据进行压缩处理, 其存储量还提升. 显然, 中心化节点已经能够在很大程度上满足我们的需求了.

通过上面的分析,我们可以看出,中心化的节点已经足够我们的使用,**不要追求完美**.

接着,我们再对成员协议进行说明. GFS中有两类节点:  
+ 中心节点NameServer, 负责维护集群成员及应用元数据
+ 存储节点ChunkServer, 负责提供存储引擎功能.注意,这里,存储节点并没有提供复杂的存储模型,存储模型是由NameServer与Client库一起提供的.

ChunkServer会周期性地向NameServer发送心跳以及自己所维护的应用状态,NameServer根据心跳信息维护集群成员数据,根据应用状态信息维护应用元数据.

### 存储引擎与存储模型
GFS的存储引擎是一个简单的线性Chunk引擎,ChunkServer提供Chunk存储,每个Chunk由一个唯一的ID来标识.NameServer维护着整个系统的ChunkID名空间. 实际上,ChunkID名空间的维护是可选的,甚至可以不维护的,它可以由其他数据导出. 具体原因之后会详细说明.

GFS的存储模型由NameServer各Client共同提供. 它们实现了一个文件模型,或者说Blob模型.NameServer维护着文件名空间,以及文件名到到其所包含的Chunk之间的映射关系.

以上的元信息都是需要持久化保存的. NameServer还维护着一类信息,即ChunkID到ChunkServer的映射.而此信息不是持久化的.试想,我们可以在NameServer中完全去掉ChunkID名空间以及ChunkID到ChunkServer的映射,这不会影响系统的正确性,但会影响到系统的效率.对于每一个ChunkID的操作,我们可以轮询所有的ChunkServer.因为,ChunkServer才是Chunk的真正所有者,它才知道一个Chunk的存在与否以及具体位置.NameServer中保存的数据只是一个View.即,**Dual writes are bad**, **每个数据只有一个所有者,其他的全部是缓存或者View**,这样可以使得一致性问题的处理变得更加简单与自然.

从某种意义上来说, 存储引擎与存储模型的分离, 即是所谓的 **机制与策略的分离**. 在GFS中, 这个原则可能体现得不是那么明显, 因为存储模型层也需要复杂的逻辑来维护自己的状态. 而在Bigtable中, 此原则则体现的更加明显, 所有状态全部存储在存储引擎中, 存储模型从本质上来说无状态, 这使得其容错的实现变得相当简洁. 从另一个角度来看, GFS与TFS的存储引擎层几乎完全一样,它们提供了存储的机制, 而其存储模型层则则分别对大文件及小文件做了优化.

#### 存储引擎实现
现在考虑存储引擎的具体实现. 所有ChunkServer保存着Chunk,它是存储引擎处理的基础单位,每当ChunkServer启动时,它会扫描所有的Chunk,并将Chunk信息报告给NameServer,以供NameServer构建ChunkID名空间以及ChunkID到ChunkServer的映射关系.

考虑Chunk的增删改查操作.
+ 增: NameServer根据ChunkServer的负载情况, 选择一台,创建一个新的Chunk
+ 删: NameServer通知ChunkServer删除Chunk
+ 改: 修改操作是存储引擎中最为复杂的操作. 分布式系统中有一个非常重要的原则: **每个数据只能有一个写者**. 当然,这不是教条,而是一个经验.有很多方法可以实现多个写者,然而其效果,并没有我们想象中的那么好.在GFS中,一个Chunk会有多个副本,我们到底写哪个?显然,根据已有的经验(MySQL主从备份),我们可以很自然地选出一个主Chunk,在其上进行所有的写操作.此时,我们需要一个互斥锁来保证资源的独享性.分布式环境中,我们通常使用Lease来实现锁功能.GFS中,Lease的功能由NameServer提供.从某种意义上来说,NameServer的职责过于复杂.这些类似成员管理/分布式锁的功能,可以使用单独的分布式基础组件来完成.这一点在Bigtable中有着较为清晰的体现.值得一提的是,整个GFS的设计都是以简单为上的,而在写操作中,GFS使用了许多复杂的技巧,来保证写操作的一致性以及效率,比如控制流与数据注的分离,比如数据流式推送.当然,这些过于细节的东西在此我不加详述.
+ 查: Client向ChunkServer发起读请求, 多个Chunk副本可以使用负载均衡的形式进行读操作,以实现读操作的性能

#### 存储模型实现
GFS提供了文件系统的存储模型, 由NameServer与Client共同实现. 文件的底层存储由Chunk存储引擎提供.

NameServer维护了文件命名空间及文件名到其所包含的Chunk之间的映射. GFS的使用者通过GFS Client来对文件模型进行操作. 考虑文件系统增删改查操作的实现方式:  
+ 创建文件: GFS Client发起文件创建请求, NameServer在文件命名空间中创建一个新的文件(默认分配一个Chunk), 并调用存储引擎创建一个新的Chunk
+ 删除文件: GFS Client发起文件删除请求, NameServer在文件命名空间中将此文件重命名为一个隐藏文件, 一定时间后, 此文件中的所有Chunk会被垃圾回收, 同时通知存储引擎层删除所有Chunk
+ 修改文件: 考虑到性能及应用场景, GFS **针对常用操作进行了优化**. GFS总体上来讲追求一个简单即美的实现方式, 但在细节上, 它做了许多优化, 特别是在文件修改操作上. 可以说, 文件修改操作, 是整个系统的重点所在. Hadoop及TFS, 在最初版本实现文件修改操作时, 都没有使用GFS的实现方式, 可见其复杂性. 其工作方式如下:
    + GFS Client向NameServer请求文件特定Chunk(如果是随机写, 则请求相应的Chunk, 如果是追加操作, 则请求最后一个Chunk)
    + GFS Client向主Chunk发起来写请求. 我们仅仅考虑追加操作, 因为它是GFS所特别优化过的. 在追加操作上, GFS支持原子追加, 并通过两种特别的技术对追加操作进行了优化
        + 流式数据同步
        + 数据流与控制流分离
    + GFS使用数据流与控制流分离的手段, 当需要进行写操作时, 它会选择距离GFS Client最近的拥有目标Chunk的ChunkServer写入数据, 这台ChunkServer会同步地将数据流式地推送到另外的ChunkServer, 依此类推, 当所有的数据推送都完成后, GFS Client会有Master Chunk所在的ChunkServer发送控制命令, 此ChunkServer会通知另外两台ChunkServer进行提交操作, 同时自己也进行提交, 全部提交完成后, 响应客户端.
+ 读取文件: 出于效率考虑, GFS **限制了其使用范围(特化)**, 它主要用于顺序读取操作, 在些情况下, GFS Client可以有效地利用缓存的文件元信息, 进行顺序读写, 而不需要缓存chunk数据. 读取操作会在GFS Client端进行负载均衡. 其工作方式是, GFS Client向NameServer请求某个文件的元信息, NameServer返回其所有Chunk以及Chunk所在ChunkServer, GFS Client会缓存这些信息, 然后直接与ChunkServer进行读取操作

### 可伸缩性与容错性
GFS中有两类节点, 我们分别考虑其可伸缩性与容错性.

#### NameServer
+ 可伸缩性: 根据前面的分析可是, NameServer是系统的一个瓶颈, 它不支持伸缩. 然而, 即使是单Master结构, 它也足支持PB级的数据了
+ 容错性: 显然, NameServer是系统的一个单节, 对于此类单点, 我们使用Commit Log + Snapshot的方式来进行容错, 即所谓的 **Log Updates**. 我们可以有两种方式可以进行容错
    + 快速恢复: 这主NameServer进程挂掉时, 进程管理器会发现, 并立即对它进行重启, NameServer采用了精心设计过的数据结构, 可以在数秒内由Snapshot及log文件恢复
    + 主从切换: 当主NameServer所有服务器挂掉后, 外部的监视器会发现此故障, 并通知从NameServer切换为主服务器, 同时, 将NameServer的VIP绑定到新的主NameServer上. 注意, 这里, 我们需要VIP来 **加一层抽象**, 因此, 对于GFS Client及ChunkServer来看, 它们只需要记住一台服务器的地址, 主从服务器的切换对于它们来说是透明的.

#### ChunkServer
+ 可伸缩性: 可伸缩性问题体现在存储引擎层上, 对于存储模型层, 加减机器的问题是透明的. 对于存储引擎层, NameServer充当一个协调者. 当需要加机器时, 只需要在机器在启动ChunkServer进程即可, 此进程会自动向NameServer发送心跳, NameServer将其加入当前机器列表. 减机器时, 简单移除即可, NameServer也会自动检测到长时间不发送心跳的ChunkServer.
+ 容错性: 理论上, ChunkServer也可以使用通用的容错方式, 即Commit Log + Snapshot, 这也是传统SQL数据库所采用的方式. 然而, 此类方式的成本太高, 且开销太大(因为每次复制都需要复制一台机器所有的Chunk数据). 因此, 我们使用备份Chunk的方式. 容错机器也是在存储引擎层完成的. 每个Chunk会默认维护N个副本, 当充当协调者的NameServer检测到某个Chunk的副本数少于N时, 会启动复制任务, 选择源ChunkServer与目标ChunkServer, 对Chunk进行复制.

## Dynamo
Key-value存储是一种很常见的存储, 常常用于缓存系统. 可以将其看作分布表格存储的一种特例, 同属于半结构化存储. 相对于非结构化存储, 半结构化存储使用起来更加方便. 常见的key-value存储有Dynamo/Tair/Redis/Memcache

相比于GFS/Tair, 及至于Bigtable, Dynamo具有极高的复杂度. 可以是, 它是一个学术性非常强的系统, 它是这里我所有介绍系统中唯一一个没有使用中心节点的系统. 为了追求极端的分布式, 它摈弃了传统的Master节点架构, 使用P2P技术来维护成员列表. 为了解决一致性问题, 它引入了向量时钟, 数据回传, Merkle树, 冲突处理, NWR等等十分复杂的P2P技术. 同时, 它对上层应用也添加了额外的影响.

总的来说, Dynamo违反了诸如 **Worse is better**/**One Writer Principle** 等原则, 极度追求分布式, 导致Dynamo成为一个比较失败的系统.

### 分布式成员协议
为了追求完全的分布性, Dynamo使用P2P协议, 所有节点逻辑上完全等价. 每一个节点都维护了整个集群的信息. 整个集群使用基于Gossip的成员协议以及错误检查. 相比于基于Master的成员协议, 基于Gossip的成员协议实现起来更加复杂, 对网络压力也大了许多.

### 存储引擎与存储模型
#### 存储引擎实现
Dynamo的存储引擎很简单, 只是一个简单的kv存储. 任何提供kv能力的存储系统都能负担起这一责任. 比如: Berkerly DB, MySQL InnoDB. 这是典型的 **分离机制与策略** 的方法, 它使得整个系统非常的灵活, 结构也格外的清晰.

#### 存储模型实现
Dynamo的存储模型是一个比较典型的DHT实现. 它使用了添加了虚拟节点 **加一层抽象** 的一致性Hash方法.

在存储模型中, 提供存储能力的基本单位是虚拟节点, 每个虚拟节点被分配了唯一的token, 这些token被均匀地分布在整个Hash空间中. 现在, 考虑DHT中最重要的一个问题, 如何查到特定key所有的虚拟节点?

现代DHT主要协议为Chord等, 然而Dynamo的DHT出现的较早, 因此并没有使用Chord, 但是其思想是一致的. 所有的key在Hash后被映射到Hash空间中的某个值. 当需要查找某个key时, 对其进行Hash操作, 得到x, 并在Hash空间找到token值大于x且位置上最接近的token对应的虚拟节点. 同时, 为了容错, 副本存N份, 即从token开始的存储节点向后共N个节点都存储了此key的信息. 由于成员协议层已经维护了所有的成员, 存储模型层很容易维护整个虚拟节点空间, 因此可以在常量时间内完成key的寻址. 在实际上, 由于一致性/容错性等原因, 寻址过程还要再复杂一些, 不, 是复杂很多, 这些内容将放在下面的章节中说明.

解决了寻址问题后, 我们来考虑存储模型的读写问题. 读写问题是整个存储模型的中最为复杂的部分, 因为它需要考虑一致性与读写效率的问题. 这里, 一致性问题有很多复杂的含义, 比如, 数据的分布, 数据的复杂, 存储系统需要在这些情况下保证系统的一致性. 系统设计中有一条很常用的原则 **唯一写者原则**. 然而, Dynamo的实现正好是这一模式的反模式. 为了追求读写的速度, 它放弃了这一原则, 引入了NWR机制. 其中, N表示副本数, W表示对此副本进行写操作时最少需要写的副本数, R表示读操作时最少需要读的副本数. 如果追求速度, 可以将W/R都设置为1, 然而, 在这种情况下, 系统的一致性将受到极大的挑战. 如果为了追求高一致性, 需要将W/R都设置为大于N/2的值.

下面, 描述读写的实现:  
+ 读:
    + 客户端向任意一个存储节点发送读取key的请求, 获取key所有副本所在虚拟节点
    + 客户端从返回的节点中随意选择一个作为操作的协调者, 并向其发送读请求
    + 协调者在其他副本所在节点中选择R个副本, 并发送读请求, 同时, 它也读取本地数据, 当有R-1个副本返回值, 协调者可以返回客户端(算上自己总共有R个)
        + 如果R个副本完成一致, 则随意返回一个
        + 否则, 根据冲突处理规则合并多个副本的结果, 并发起读取修复操作以尽力恢复一致性
+ 写:
    + 客户端向任意一个存储节点获取key所在的所有的节点
    + 从所有返回的节点中选择一个协调者, 并通过它完成写操作
    + 协调者向其他备选节点发送写操作, 同时, 进行本地写
    + 当有W-1个副本完成写操作时, 回复客户端写成功(此时, 本地写也应该已经成功了). 同时, 继续完成所有完成的写操作(**延迟隐藏**).

### 可伸缩性与容错性
由于使用了Gossip, 从分布式成员协议层来看, Dynamo的伸缩比较简单. 然而, 存储模型层为了维护系统不变式, 需要进行比较复杂的数据迁移操作. 在此不加缀述. 下面, 考虑系统中最复杂的部分: 容错. 分布式系统中,

#### 一致性
由于违反了 **唯一写者原则**, Dynamo需要比较复杂的机制来解决数据冲突的问题. Dynamo为系统中每个对象维护了一个向量时钟, 以尝试解决冲突. 当向量时钟间发生冲突时, Dynamo要么依赖客户端解决冲突, 要么使用在分布式系统中并不可靠的(Last write wins)的方法. 即, Dynamo只保证最终一致.

#### 异常处理策略
Dynamo将异常分为临时异常与永久异常. 对于不同类型的异常, 它使用了不同的手段加以处理, 这种 **特殊情况特殊处理** 而不是使用通用手段的方法, 使得异常处理更加高效, 然而这也带来了实现上的复杂性.

+ 数据回传: 当某个节点暂时不可用, 所有对其的写操作会被发送到其他的特定节点. 如果指定时间内此节点恢复, 则数据会被回传
+ Merkle树同步: 当某个节点失效超过一定时限后, 系统认为其已经永久失效了, 此时, 系统需要从其他节点将数据复杂到新的节点, 以维护副本数. 显然, 这样的复制量是非常大的. Dynamo使用有Merkle树来进行复制操作, 以尽量减少需要复制的数据量.

## Bigtable
Bigtable是一个典型的半结构化分布式存储系统. 相比较于前面提到的两个系统, 我认为其实现更加精巧与简洁, 可以说是架构典范. Google之后的Megastore就是基于Bigtable做了进一步抽象的. 在结构上, Bigtable使用中心化节点的方式, 在层次上, Bigtable的层概念更加分明, 在实现上, Bigtable用了更加复杂的数据结构与机制. 可以说, Bigtable同时具有了学术与工业的价值.

与GFS不同的是, Google写Bigtable时, 分布式基础组件已经比较完善, 因为, Bigtable的实现更为优美.

### 分布式成员协议
Bigtable由三部分组成:  
+ Client: 客户端程序
+ Master: 主节点, 管理子表
+ Tablet Server: 存储节点, 提供子表的存储

显然, 分布式成员协议的关注点在Master与Tablet Server上. Bigtable使用Chubby来管理成员协议以及主节点的选举(**Separation of Concerns**), 这大大简化了Master的设计. Master不需要自己再实现与Tablet Server的心跳逻辑, 也不需要实现主从Master的心跳逻辑. 它只需要通过Chubby, 就可以知道当前有哪些Tablet Server, 以及当前自己是否为主节点.

### 存储引擎与存储模型
#### 存储引擎实现
可以看到, 有了Chubby的存储, 分布式成员协议变得很简洁. 同样, 有了GFS, Bigtable的存储引擎也变得十分简洁, 所以数据持久化在GFS中. 由于存储引擎层的优秀性质, Bigtable本身的容错及可伸缩机制变得相对比较简单.

#### 存储模型实现
存储模型部分可以说是Bigtable同最为复杂的一部分了. 然而, 其概念上是非常简洁的, 只不过为了高性能, 它使用了LSM这种数据结构, 以及Commit Log, 这也就使得其在实现细节上非常复杂. 然而, 我并不准备深究其实现细节, 而不从宏观角度从Bigtable的存储模型进行说明.

相比较GFS, Bigtable的存储模型层是无状态的, 当然, 说是无状态的可能并不恰当, 或者可以称之为soft state. 它介于有状态与无状态之间, 状态的存在可以帮助其快速地进行执行请求, 但它并不依赖这些状态, 因为这些状态都是 **缓存** 或者说 **索引**. 缓存本身可以说已经成为一种通用的技术手段了, 在某种意义上, 它实现了 **延迟隐藏** 的效果.

下面, 正式对Bigtable存储模型的实现加以说明.

Bigtable存储的对象是表, 为了实现分布式, 这些表每分隔成子表, 子表是系统管理的基本单位. 子表存储于GFS之中. 在考虑如何对子表进行增删改查操作之前, 我们首先需要考虑如何对一个子表进行寻址. Bigtable中, 子表被组织成三层:  
+ 第一层: root table, 仅有一个, 用户维护二级子表的信息
+ 第二层: meta table, 元数据表, 维护了其他所有用户表的信息
+ 第三层: user table, 具体负责存储的子表

其中, Chubby维护了root表的位置信息, 而root表本身存储于GFS中, Master可以载入root表, 进而载入二级元数据表, 从而对整个子表空间进行管理. 寻址时, Client可以对元信息进行缓存和预取, 从而提高效率.

考虑增删改查的操作:  
+ 增: 当需要创建一个Table时, Client与Master通信, Master会创建子表(在GFS中), 并更新元数据二级甚至一级元数据表, 将由子表分配给某个子表服务器管理.
+ 删: 删除操作类型创建操作, Client向Master发起请求, Master更新元数据表, 并回收子表服务器对于Tablet的占有权, 之后再通知GFS删除子表内容
+ 改: 同样, Client首先需要需要子表路由表, 路由表可以是缓存的路由表, 也可以是向Master请求得到的路由表. Client查找路由表, 找到子表所属服务器, 对此服务器发起写请求. 子表服务器维护两类数据, Commit Log以及Memtable. 所有的操作由Commit Log进行持久化以及错误恢复, Memtable可以看作一个缓存, 它缓存着一定时间内的写操作. Memtable存在于内容中, 当其大小到了一定程度后, 会写入GFS, 同时创建一个新的Memtable, 用于接收写请求, 并将旧的Memtable写入GFS. 具体实现细节可以参考Bigtable论文, 或者LSM的实现. 值得一提的是, Bigtable使用Group Commit技术, 达到了 **延时隐藏** 的效果.
+ 查: 与写操作类型, Client在定位到子表服务器后, 向此子表服务器发起读请求. 子的服务器存储的全部信息包括两类, 存储于GFS中的所有SSTable, 以及存储于内容中的缓存表Memtable, 子表服务器会综合查找两类数据, 来找到需要读取的项. 需要说明的是, SSTable在内容中有索引, 以提供其查找速度.

特别的, Tablet有两类特殊的操作, 合并与分裂. 当Tablet经过一系列删除操作后, 其内容过少, Bigtable需要将小Tablet合并. 相对的, 相Tablet持续写入内容, Bigtable需要对其进行分裂操作. 合并操作将Master控制, 分裂操作由Master和Tablet Server协作完成.

另外, Tablet还会进行另外一类操作, 即, Compaction. Compaction作用于SSTable, 以减少存储容量. 由于这不是存储系统的关键所在, 且其行为比较复杂, 因此不加多说.

### 可伸缩性与容错性
Bigtable的可伸缩性定存储引擎层与存储模型层共同决定. 在GFS章节中, 我们了解了GFS的可伸缩性. 对于Bigtable的存储模型层, Tablet Server仅仅充当一个索引层, 故其不会对系统的可伸缩性产生任何不良影响. 另一方面, 系统的所有元数据即三层表存储在GFS中, 因此Master也不会成为系统的瓶颈所在. 于是, 我们可以看到, 限制系统的扩展的关键在于三层表. Bigtable中规定子表的大小在100~200MB, 假设其元信息大小为1KB, 则一级元数据表的存储量为200MB*(200MB/1KB) = 39TB, 二级元数据表能够支持的数据量为39TB * (200MB / 1KB) = 4992PB, 显然, 这个数量级已经能够满足我们的需要了.

再考虑容错性的问题.

与所有中心化节点类型, Master使用主从备份的方式工作, 并通过Chubby选取主节点. 当主Master挂掉后, Chubby会指定新的一个Master服务器为主节点, 并向外提供服务.

下面考虑Tablet Server的容错机制. 由于存储引擎层的强大, Tablet Server的容错十分简洁. Master在检测到某台Tablet Server失效后, 会将其上的Tablet分配到其他的Tablet Server上去, 新的Tablet Server会载入SSTable的索引, 并根据Commit Log重构Memtable. 这两样工作完成后, 恢复工作即告完成. 实际上, Bigtable所做的事情比这要复杂一点点, 然而其本质确实如此. 需要说明的是, 每个Tablet在同一时间内仅仅被一台Tablet Server所占有, 即 **单一写者原则**. 这使得写操作的管理变得十分简单, 却也带来了另外一个缺点: 当Tablet Server失效后, 其在的Tablet将在一段时间内无法提供服务.

## Spanner
终于到了分布式数据库了. 数据库是性质最为良好的存储系统, 传统的关系型数据库提供了ACID模型,关系代数及事务支持, 它使得我们应用的编写变得十分简单. 我们可以完成不同在意持久化层的各种细节. 可以说, 关系型数据库是企业级信息系统时代的标准配置. 然而, 当进行分布式时代, 传统的SQL数据库就变得不那么好用的, ACID及事务的支持在分布式场景下变得十分复杂, 以致很多人放弃了ACID及事务的支持, 转向NoSQL技术, 并提出了BASE来取代单机场景下的ACID模型. 然而, 实践表明, 我们仍然需要SQL, 需要事务, 需要关系代数模型. 然而, 要在分布式系统中拥有这一切, 我们必须要解决分布式事务的问题. 需要说明的是, 并不是分布式事务无法实现, 而是其延迟过高, 以致于在实践环境下根本不可用.

Spanner是Google的全球级分布式数据库, 它号称扩展性可以达到全球级, 支持数据百个数据中心, 数百万台机器, 上万亿条记录.

### 分布式成员协议
Spanner由以下组件组成:  
+ Universe Master: 监控Universe中的Zone
    + Universe表示一个Spanner系统.
    + Universe中包含多个Zone, 每个Zone属于一个数据中心, 但是一个数据中心内会有多个Zone
+ Placement Driver: 提供跨Zone数据迁移
+ Zone
    + Zone Master
    + Location Proxy: 提供位置服务, 客户端需要通过它知道数据由哪一台Spanner Server服务.
    + Spanner Server: 提供存储服务.

类似Bigtable, 我们可以使用分布式基础组件来实现成员协议.

### 存储引擎与存储模型
#### 存储引擎实现
Spanner构建于Google新一代文件系统Colossus之上. 相比于GFS, Colossus在实时性上有了较大的改进, 并且提供小文件支持. 需要注意的是, 由于Spanner是全球级的分布式数据库, 数据的分布变得至关重要. 与Bigtable不同, Spanner部署了多个Colossus系统, 每个数据中心内都有一个, 这样, 即使某个副本失效, 其他副本仍然可以提供读服务. 那么, 这些副本是如何同步的? 这将在下面讲解.

#### 存储模型实现
作为一个全球级分布式数据库, Spanner最主要的挑战就是分布式事务. 因此, 在这一节中, 我将忽略其他细节, 主要讲解分布式事务的实现.

解决一个问题最好的方法是什么? 答案是 **解决一个问题最好的方法是不解决它**. 基于这一原则, Spanner定义了特殊的数据模型, 使得约大部分事务都会在单机内完成, 无法在单机内完成的, 则特别处理, 使用更加复杂的技术, 同时也更加低效的技术去处理. 这种 **特殊问题分类处理** 的方法, 较好地解决了分布式事务的问题.

##### 单机事务
Spanner引入了目录的概念, 数据库中表是层次化的. 举例来说, User表下有一个目录表Albums, 则, 对于User表中某一项, 比如User(1), 其下所有的Albums数据, 构成一个目录. 这类似于对User做Sharding. Spanner尽力将一个目录中所有数据放置到一个台机器上, 因此, 只要目录不太大, 同一个目录内的操作都可以在一台机器上完成. 这就是单机事务的解决方案.

##### 多机事务
当事务涉及到多个目录时, Spanner会使用多机事务. 有一些新的概念, 将在下面的章节中介绍, 这里, 先换一个理解方式. Spanner中, 同一个目录会有多个副本, 这些副本位于不同的Zone, 暂时称之为一个副本组. 当进行单机事务时, 在一个副本组内就可以完成了, 如果要进行分布式事务, 需要在多个副本组内进行. 每一个副本组有一个主副本, 负责写操作. 多个副本组中选出一个副本组的主副本作为协调者, 另外的副本本主副本作为参与者, 从而发起两人阶段提交协议, 以完成分布式事务.

##### 事务ID的生成
为了实现并发控制, 数据库需要给每个事务分配全局唯一的ID. 在单机环境下, 这很容易做到, 然而, 要分布式场景下, 这成了一个很大的问题. Google Percolator中使用了一个外部系统专门用于生成全局唯一ID, 这个外部系统是一个单点. Spanner使用了一种史无前例的方法: 全球时钟同步机制TrueTime. 简单来说, 可以理解为, TrueTime提供了高效的时间服务, 解决了分布式场景下时钟不可靠的问题. 当然, 这种方法也有其缺点, 在此不加多说.

### 可伸缩性与容错性
Spanner的可伸缩实现与前面的系统类似. 这里主要介绍其容错性.

对于Universe Master/Zone Master/Placement Driver/Location Proxy这类节点, 都可以用主从热备的方式实现高可用. 对于Spanner Server, 我们使用备份子表的方式来实现. 一个子表是一个表的一部分, 其中可能包含多个目录. 每个子表会在多个Zone中有一个副本. 这些副本合起来称为一个副本组. 为了实现可伸缩性, Spanner没有采用GFS那样的中心选主方式, 而是让一个副本组使用Paxos协议, 自己选择主副本. 因此, 副本组也称作Paxos组. 虽然Paxos协议相对低效, 但是副本组的主副本可以在当选主副本后不断续约, 从而避免重复的Paxos操作.

# 系统原则分析
在介绍了四个典型的分布式存储系统后, 我将对其进行总结, 提取其中所用到的系统设计的原则与方法. 考虑到 **KISS** 等原则过于宽泛, 我将使用更加细致且具体的原则, 如: **单一职责原则**, **单一写者原则**, **Separation of concerns**, **Worse is better**.

+ 分层/功能正交分解: 根据系统分层及功能正交分解的原则, 我们可以将上面提到的系统分解成存储引擎层及存储模型层, 分布式成员协议及存储协议. 这样的系统职责分明, 方便研究者从不同角度观察与理解系统.
+ 单一职责原则: 在将系统功能正交分解之后, 理想情况下系统的每一部分都符合单一职责原则, 如:
    + GFS中ChunkServer只负责提供Chunk接口, 对于GFS所提供的文件系统的概念, 它是一无所知的
    + GFS中NameServer的职责则有些混乱, 它既作为存储引擎层的协调者, 又作为存储模型层的主要组件, 又提供了分布式锁的功能
    + Bigtable中, 功能分解得极为细致, Chubby提供分布式成员协议以及分布式锁的支持, Master提供子表的管理功能, Tablet Server提供子表的存储及操作服务
+ Separation of concerns: 这条原则与单一职责原则类似, 然而, 它更加强调分离. 单一职责原则是从个体角度上考虑的, 而Separation of concerns是从系统角度上考虑的. 有设计系统时, 我们需要列出所有的Concerns, 然后分别加以解决. 实际上, 所有设计良好的系统都反映了此规则
    + GFS: NameServer做协议, ChunkServer做存储, Client做Proxy
    + Bigtable: Chubby提供分布式基础服务, Master提供全局管理功能, Tablet Server提供存储功能
    + Spanner: Placement Driver专门做Zone迁移, Location Proxy专门做目录映射
+ 使用一个特化的, 适合自己场景的系统: 当解决一个特定问题时, 我们拥有的条件越多, 限制越少, 需求越少, 这使得我们可以集中解决特定问题. 如GFS.
+ 不要尝试做一个通用的系统: 这是对上一条原则从另一个方面的阐释. 或者说: Generic Envy. 大而全的系统没有重点, 要考虑的东西太多, 限制太多, 淘宝在做TFS及Oceanbase时, 均使用了定制的内存分配组件以及网络组件, 而没有使用网上通用的库, 因为那些库为了追求功能, 而牺牲了特定场景下的效率. GFS也是从设计之初就以大文件支持作为首先任务, TFS及Haystack均以少量小文件支持作为首要任务, 这使用两者要存储模型上差别较大.
+ Worse is better: 作为一个分布式系统, 中心节点的存在通常都是令人置疑的, 因为它是可能存在的单点. 然而, 几乎所有成功的分布式存储, 都使用了中心化的架构, 反观Dynamo, 使用了P2P技术, 却使得系统变得极端复杂.
+ 不要追求完美: 这是对上一条原则的另一个阐释, 或者说, 是一个强调, 即: Perfect Envy. 人都有一颗追求完美的心, 然而, 需要承认的是, 系统上没有完美的系统, 技术并不能解决一切需求!
+ Dual writes are bad, 每个数据只有一个所有者, 其他的全部是缓存或者View: 当数据只有一个所有者, 我们需要对此一个所有者做高可用. 其他View的数据即使丢失, 也可以很快恢复, 这简化了系统的设计与维护.
    + GFS中NameServer维护了Chunk到ChunkServer的映射, 而这些数据本质上是由ChunkServer自己决定的. 因此, 将其作为缓存, 在写Commit Log时, 不对其进行持久化. 这样, 可以加快系统的响应速度.
+ 机制与策略的分离: 这个原则类型与单一职责原则, 以及Separation of concerns. 然而, 后两者更加强调功能的原子性. 这一原则则更加强调功能的层次性. 机制层与策略层的分离, 使得我们可以对两层都进行更加灵活的开发.
    + Dynamo中, 存储引擎与存储模型的分离, 使得我们可以更换存在引擎: 如Berkerly DB, MySQL InnoDB等
    + Bigtable中, 存储引擎与存储模型的分离, 使得元信息也可以存储于存储引擎层, 极大地简化了系统的设计与扩展
+ 单一写者问题: 一个数据, 只有有一个写者, 多个写者会引起冲突, 进而引起不一致
    + GFS: Chunk副本组中, 只有主Chunk才可以接收写请求
    + Dynamo: 违反了此原则, 造成系统极端的复杂性
    + Bigtable: 每个Tablet, 只有一个Tablet Server才有服务
    + Spanner: 写操作只能由所有Paxos组的主副本才能处理
+ 针对常用操作进行了优化: 根据80-20原则, 优化是要有目的性, 针对常用操作/关键操作的优化才是有必要的. GFS中对写操作进行了特别的处理, 提供了高效的写性能.
+ Log Updates: 所有系统都使用了此技术作为其容错技术的基础.
+ 加一层抽象: 加一层抽象是计算机科学中最为重要的一种方法. 在上面各中心化分布式存储系统中, Master节点均绑定了VIP. 将Master主从切换时, VIP也进行切换, 从而, 外部系统所看到的只有一个VIP, 屏蔽了Master失效转移的细节.
+ 延迟隐藏: 延迟无法被避免, 那就隐藏它. 我们可以使用诸如缓存/组提交/并发等等技术来隐藏延迟.
    + GFS/Bigtable都有客户端缓存机制
    + Bigtable提供了组提交机制
+ 特殊情况分类处理: 当处理某一问题时, 可以将问题分成几种情况考虑, 不同的情况使用不同的方法, 如fast-path, slow-path.
    + Spanner: 事务集中在同一目录上, 使用单机事务. 否则, 启用更加低率的多机事务
    + Dynamo: 针对临时异常与永久异常, 使用不同的错误处理机制
+ 解决一个问题最好的方法是不解决它: 要解决一个问题, 我们就要花费代价, 因此, 解决一个问题的最好方法就是不解决它, 这样我们就不需要花费太大的代价. 在分布式存储中, 分布式事务是最主要的问题. Spanner使用了特殊的数据模型, 使得事务大多数集中在一台机器上, 从而避免了分布式事务.
+ 将容错作为一个功能: 出错是分布式系统的一个固有属性, 机器数量的增多, 网络故障, 硬件故障, 这些问题都会造成系统出错, 因此容错是分布式系统必不可少的特性. 本文中所提供的所有分布式系统, 都在容错上面花费了巨大功夫.