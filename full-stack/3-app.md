# 应用服务器
经过千回百转, 客户端的请求终于来到了我们的应用. 应用服务器的任务很简单, 访问数据库, 取出数据, 根据数据生成动态的页面(eg. JSP), 并返回. 然而, 在完成任务的前提上, 我们还需要实现一些其他的质量属性.

# 高性能
说到性能, 提供性能的最简单方式就是: 缓存!缓存!缓存!

对于应用层, 缓存可以分成两类, 进程内缓存及分布式缓存. 分布式缓存是一个比较通用的解决方案, 推荐使用Redis. 对于进程内缓存, 优点是性能非常高, 但是缺点以及限制也是一堆, 需要了解自己数据的特点后才谨慎使用.

J2E定义了缓存相关的Annotation, Spring有一个使用AOP实现的缓存实现.

还有一些其他的提高性能的手段, 但相对缓存来说, 但不是那么的直观. 比如, 一般来说, 使用Go/nodejs这种异步IO模型的语言/框架, 就比使用传统CGI进程模型的应用要快.

# 高可用
保障应用高可用最直观的手段就是冗余, 即添加更多的副本. 然而, 对于有状态的应用与无状态的应用, 其添加副本的手段是不同的. 一般的, 12Factor应用以及CloudNative应用的实践告诉我们, 保证应用无状态, 从而可以水平扩展.

# 高可伸缩性
如高可用一节中所说, 为了保证水平扩展性, 我们只需要让应用无状态即可.

然而, 有时, 问题可能更加复杂, 比如, 在写一个行情获取应用的时候, 我们使用了第三方服务来获取实时行情. 由于此第三方服务只允许一个IP接入, 因此, 我们只无法对行情获取应用进行水平伸缩. 当然, 我们还是可以通过冗余的手段来实现高可用. 但是此时实现高可用也稍微复杂了一些.

# 高可扩展性
扩展一个应用可以从两方面入手. 一方面是应用内添加功能以扩展应用, 另一方面是添加新的应用, 与已有应用形成应用联邦来协作完成功能. 从本质上, 两者是类似的, 因此, 我将在之后的章节中重点介绍应用联邦的相关思想与技术.

简单来说, 添加新功能时, 修改已有代码是下之下也. 比如, 我们有一个新闻应用. 最开始的需求是发布完新闻后, 将其存入数据库. Yes, 这个需求很简单.

```Java
void saveNews(final News n) {
    mNewsDao.save(n);
}
```

之后, 我们又需要将新闻推送到手机客户端:

```Java
void saveNews(final News n) {
    mNewsDao.save(n);
    mPushService.push(n);
}
```

再之后, 我们又需要为新闻生成静态页面:

```Java
void saveNews(final News n) {
    mNewsDao.save(n);
    mPushService.push(n);
    mGenerationService.generate(n);
}
```

上面的例子比较简单, 然而在真实环境下, 情况可能坏的多. 对扩展开放, 对修改封闭的原则告诉我们, 这样是不好的! 同时, 它也违反了`Do one thing and do it well`的原则, 我们无法一句话说明这个函数到底在做什么. 上述代码过于耦合. 我们需要解耦合, 而解耦的终极方法是消息. 我们可以在新添新的新闻时发布一条消息`news:add`. 但我们需要添加新的功能时, 只需要写一个任务, 并订阅`news:add`.

```Java
@Publish(topic = "news:add")
News saveNews(final News n) {
    mNewsDao.save(n);
}

@Subscribe(topic = "news:add")
void onMessage(final News n) {
    mPushService.push(n);
}
```

# 高安全性
安全性问题是一个老生常谈的话题, 一般的, 复杂的安全性问题我们应该交给专门的人来做, 比如nginx扩展模型, 云防火墙等. 一条通用的原则是: 对所有输入消毒, 对所有输出转码.

对于JavaEE环境, 常见的错误主要有以下三类:

+ XSS: 注意消毒即可
+ SQL注入: 使用ORM框架的话, 不用在意这个, 框架会帮你处理的
+ CSRF: 这个需要额外处理一下, SpringSecurity中有个模块可以处理它. 可以考虑使用.

# 参考资料
+ [构建高性能Web站点](https://read.douban.com/ebook/1410608/)
+ [大型网站技术架构](http://book.douban.com/subject/25723064/)
+ [Patterns of enterprise application architecture](http://www.martinfowler.com/books/eaa.html)
+ [Spring Cache](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html)
+ [The Twelve Factors](http://12factor.net/)
+ [Migrating to Cloud Native Application Architectures](http://pivotal.io/platform/migrating-to-cloud-native-application-architectures-ebook)
+ [淘宝技术这十年](http://book.douban.com/subject/24335672/)
+ [参考项目源代码](https://github.com/qqiangwu/reins-ssh)
