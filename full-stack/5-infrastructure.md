# 基础设施
分布式系统固然有许多优点, 然而, 这些优点并不是免费的, 相比Monolith, 分布式系统的缺点也不少. 如果没有适当的基础设施的支持, 分布式系统将成为维护的噩梦.

考虑你有一个由10个应用组成的分布式系统, 每个应用都有着3-5相实例. 如果你有5台物理机, 如何分布这些应用的实例? 如果你有10台机子, 如何分布这些应用的实例? 如果你需要更新某一个应用, 你如何不保证更新期间, 此应用的服务不会中断? 运行过程中, 如果某个应用的某个实例挂掉了, 你应该如何知道? 你如何处理这些应用的日志? 你如何知道某个应用已经达到负载的极点, 需要添加新的实例以分流? 当你增加一个新的应用, 如何将其加入已有的应用联邦? 显然, 我们需要一个智能的应用部署/协调/监控/管理平台. 事实上, 这就是目前所有PaaS平台所解决的问题, 也是所有云平台的核心技术之一.

如果你使用GAE/Heroku等PaaS平台, 它们可能帮助你解决了以上部分问题. 然而很多时候, 我们可能更加喜欢租借IaaS平台, 然而自己手动来处理这些事情. 因此它更加便宜.

# 原始解决方案
在我的上一个项目中(小牛财经), 由于生产环境中物理机只有三台, 应用的数量也不多(前台+后台+行情获取+k线计算+行情信息服务), 只有五个. 因此, 我手动地将其分布在三台机器上, 并搭建了Nginx来为同一应用的不同实例做负载均衡. 我使用了ansible来进行应用的部署, 使用了进程管理器supervisor来启动和管理实例.

嗯, 虽然并不完美, 但是它的确可以work.

# 基于容器的解决方案
仔细思考一下原始解决方案, 可以发现, 它实际上是存在问题的. 一台物理机上部署了多个实例, 这些实例都是以进程的形式存在于物理机上的, 如果某个进程占用了大量资源(比如出现了死循环), 那么其他所有的进程也就同时无法正常工作了. 其原因是, 我们缺少资源隔离. 传统的资源隔离方案是虚拟机. 然而虚拟机用在部署单个应用上, 又显然太重量级了. 一个经典的解决方案是使用control group来进行资源限制与隔离. 而更加全面的解决方案是使用容器(eg. Docker).

事实上, Docker火起来之后, 基于Docker的各类PaaS平台层出不穷, 比如, 类Heroku的Dokku, Google出品的Kubernetes, Twitter出品的Mesos, 以及Yelp开源的号称产品级的PaaS平台PaaSTA.

这些PaaS平台都提供了自动化部署, 服务注册与发现, Rolling Update, 负载均衡, 监控等功能, 极大地提高了生产力.

然而, 注意注意的是, 没有免费的午餐. 对于小型项目, 或者传统Monolith项目, 使用这些东西只会引入额外的复杂度. 而控制复杂度则又是我们的终极目标.

# 参考资料
+ [Ansible](https://www.ansible.com/)
+ [Supervisor](supervisord.org)
+ [Docker实践精粹Yelp | 构建一个Production-Ready的PaaS平台](http://toutiao.com/news/6252312599757586945/)
+ [Heroku](http://heroku.com/)
+ [Dokku](http://dokku.viewdocs.io/dokku/)
+ [Kubernetes](kubernetes.io)