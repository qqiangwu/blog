# 客户端
先从Web最前端的客户端开始讲起吧. 我会先列出优化方式, 再对其进行解释.

实际上, 客户端可以包括很多东西, 比如浏览器, 比如移动端, 这里所指的是浏览器.

当我们在浏览器中键入一个网址, 会发生些什么?

+ DNS解析: DNS方面还是有点东西的, 比如CDN接入, DNS负载均衡
+ TCP连接建立
+ HTTP/HTTPS协议请求与响应
+ 加载页面及资源
+ 解析页面及渲染

---

# 页面加载及呈现
## 优化措施
+ 将CSS放在`<head>`中
+ 内联关键CSS
+ 内联小图片(base64)
+ 将JS放在非首屏区, 通常是页面尾部
+ 合并多个CSS文件
+ 合并多个JS文件
+ 压缩JS/CSS
+ 使用Combo一次性加载多个资源
+ 合并多个ICON, 使用雪碧图(Sprite)
+ 为静态资源准备一个或者多个单独的域名
+ 优化图片
+ 延迟加载
+ 按需加载

## 优化说明
我们可以将浏览器功能其分成三个功能原子:  
+ 渲染引擎
+ 解析器
+ 加载器

当我们访问`example.com/`时, 加载器会将主页面的内部流式加载进来, 解析器会对其进行解析, 增量地构建DOM. 在解析的过程中, 解析器发现css/js/img时, 会通知加载器进行加载. 加载完css后, 会构建CSSDOM(非增量), 只有当DOM与CSSDOM都构建好后(DOM并不需要完全构建好), 才会将其交给渲染引擎去构建渲染树, 布局及呈现, 此时页面才会出来在屏幕上.

其中, 对于CSS, 由于渲染的必要条件是CSSDOM, 因此, 越早加载CSS, CSSDOM构建的也就越早. 这也是为什么我们总是将CSS放在`<head>`中的原因. 极限情况下, 我们甚至会将关键CSS内联, 以快速构建CSSDOM用于显示ATF内容(首屏内容).

对于JS, 当解析器构建DOM时, 如果发现JS, 就会等待其下载及运行完成才会继续执行, 同时, 等到CSSDOM构建完成时, JS才会执行, 因为JS会更改DOM及CSSDOM的内容, 因此只能等待以避免Race Condition. 因此, 最佳实践是将JS放在页面底部, 以避免其阻止ATF的渲染.

对于图片, 我们通过会对图片进化优化/压缩, 以减少其大小. 甚至内联一些小图片, 使用雪碧图. 以避免多余的网络来回.

需要注意的是, DOM的构建及页面的渲染都是增量式进行的, 因此, 快速渲染首屏页面是优化的目标. CSS会阻止渲染且是渲染的前提条件, 因此我们希望其快速加载完成. JS会阻止DOM树的构建, 因此我们将其放在最后, 这样, 在构建首屏内容的DOM时, 不会被JS阻塞住.

于是, 解析器及渲染引擎的目标就是: 尽早加载CSS, 避免在首屏中等JS. 但由于Http roundtrip是很费时的操作, 因为, 我们希望减少它, 于是常用的方法是合并多个JS为一个, 合并多个CSS为一个. 关键CSS/JS直接内联. ICON使用Sprite. 内联一些关键CSS及小图片, 可以避免多余的请求.

值得一提的是, 除了对解析器及渲染引擎进行优化, 加载器也有优化的点. 浏览器作为一个善意的客户端, 会限制对同一domain的并发数(如果不限制的话, 页面内有100张图片会发起100个并发请求, 基本就和DOS相差不远了). 一个优化是使用单独的静态资源网址, 如`img1.staticcontent.com`, `img2.staticcontent.com`. 同时, 这也是为什么要合并资源的一个原因.

再进一步讲, 我们希望将所有与首屏无关的内容全部 __延迟加载__ 或者 __按需加载__. 比如, 某段脚本用于处理对话框, 我们希望需要显示对话框时才加载它. 再比如, 京东首页的楼层效果, 只有滑动到对应楼层时, 楼层内容才会显示.

## 参考资源
+ [Critical rendering path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/?hl=en)
+ [静态网页资源的管理和优化](http://velocity.oreilly.com.cn/2010/index.php?func=session&name=%E9%9D%99%E6%80%81%E7%BD%91%E9%A1%B5%E8%B5%84%E6%BA%90%E7%9A%84%E7%AE%A1%E7%90%86%E5%92%8C%E4%BC%98%E5%8C%96)
+ [Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html)
+ [前端工程与性能优化](https://github.com/fouber/blog/issues/3)
+ [High Performance Web Sites](http://book.douban.com/subject/2084131/)
+ [Even Faster Web Sites](http://book.douban.com/subject/3686503/)
+ [High Performance Browser Networking](http://book.douban.com/subject/21866396/)

---

# HTTP
## 优化措施
+ Keepalive
+ gzip
+ Http缓存协商
+ Http缓存控制

## 优化说明
Http是整个Web的基础. 目前主流是Http/1.1. Http/2.0最近已经发布, 但是距离普及还是需要相当长的一段时间. 对于Http/1.1, 目前有许多良好的实践.

Http构建于Tcp之上, 每次通信, 都会打开一个Tcp连接, 而新建一个连接的开销是非常大的, 因此, Http/1.1默认使用长连接(Keepalive), 来复用tcp连接, 这样, 一次请求响应循环完成后, 底层tcp连接不会被关闭, 而是被下一次的请求响应循环所复用.

带宽是Http中一项非常重要的资源, 对于内容性网站(如新闻类网站), 静态内容传输会占据很大的带宽, 因此, 使用gzip压缩是一个非常常用的优化手段.

从所周知, 提供性能的一大利器就是缓存. Http/1.1中也包含了缓存相关的内容. 一种方法是为资源附加一些元信息(如最近修改时间, etag), 浏览器发送请求时, 会加上这些信息, 如果没有变化, 则返回304(Not Modified), 从而避免网络传输. 然而, 这仍然需要网络来回. 解决方法是使用缓存控制, 即`Cache-Control: max-age=3600`, 表明在1小时内, 内容不会发生变化, 需要此资源时, 浏览器会直接从缓存中获取. 很多静态内容通常都会把缓存时间设置到最大. 然而, 如果我们的静态内容发生了变化, 即如何更新? 实践表明: 不缓存html, 其他资源一律使用最大时间缓存, 当内容改变时, 更新html中的引用. 即, 为静态资源添加指纹: `<script src="/vendor-af8347ddf.js">`. 有一种旧式的方法`<script src="/vendor.js?v=20160901">`, 它存在着许多的问题, 不要用它!

## 参考资源
+ [浏览器缓存知识小结及应用](http://www.cnblogs.com/lyzg/p/5125934.html)
+ [图解Http](https://book.douban.com/subject/25863515/)
