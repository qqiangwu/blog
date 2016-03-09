# 静态内容服务
动态内容与静态内容是Web中两大类内容, 由于静态内容与动态内存的IO模型及缓存模型有着显著的差异, 因此, 它们通常会使用不同的方式加以处理.

客户端之后是ContentDelivery层, 用于服务静态资源, 提供资源缓存等功能. 一般的, 它由以下三层组成(注意, 所有负载均衡器全部被省略了):

+ CDN
+ 缓存服务器
+ 静态服务器

对于静态资源, 如js/image/css等, 当与文件指纹相配合时, 它们都是可以进行强缓存的. 而对于html页面, 我们不能对其进行强缓存(即使用`Cache-Control: max-age=3600`). 否则, 我们在服务器端改变html/js/css等资源时, 客户端将无法在不进行强制刷新的情况下得知此更新, 这是因为ContentDelivery层通常有很多中间服务器(如CDN/缓存服务器/负载均衡服务器). 于是, 有以下规则:

+ 对于静态资源js/css/images等, 使用强缓存. 使得它们可以被CDN/浏览器等缓存
+ 不对html使用强缓存, 但可以使用协商缓存(Last-Modifited/ETag).
+ 在html中引用静态资源时, 使用带有文件指纹后缀的文件名(一般是md5), 当静态资源被修改时, 更改引用

# CDN
CDN本质上是一个缓存服务器, 但它在进行DNS解析时, 会自动返回离用户最近的服务器的IP. 如果此服务器已经缓存了所需要的内容, 则直接返回, 否则, 会将请求转发到ContentDelivery层的下一层(缓存服务器或者静态服务器).

CDN使用Http协议中的缓存相关协议进行缓存. 一般的, 我们只对可以强缓存的内容使用CDN. 这样, 一旦有静态资源改变了, 已有的CDN缓存不会影响内容的更新. 否则, 比如, 我们主页是index.html. 如果其被CDN缓存了, 而我们对其进行了修改, 客户端访问时, 仍然会得到CDN里的旧的内容.

# 缓存服务器
缓存是降低延迟的终极武器. 在整个Web栈的每一层, 全部都用到了缓存技术. 在ContentDelivery层, 有两种终极技术, 一是页面静态化, 二是缓存服务器. 两者各有优劣, 但都被大量应用于内容服务的网站中. 比如, 页面静态化被大量应用于新闻站点, 京东也曾经用过此类技术. Wikipedia没有使用页面静态化(虽然页面静态化很适用于这种场合), 而是使用了缓存服务器.

最具盛名的缓存服务器是Varnish. 而实际上, 万金油Nginx也能干(可以将Nginx理解为一个静态资源服务平台, 提供了缓存/服务/压缩/代理/负载均衡, 等多种多样的功能)

缓存服务器可以缓存多种内容, 如静态资源(js/css), 页面(由动态服务器生成的页面, Wikipedia就是这么干的).

# 静态资源服务器
Web服务器通常会提供两类资源, 一种是静态资源, 以文件的形式存在；另一种是动态资源, 通常会需要访问数据库以及进行额外的运算后生成的内容. 而Web应用服务器Tomcat/Jetty等, 主要用于提供动态资源, 在静态资源上表现不佳. 对于静态资源, 使用Nginx这种异步IO模型的服务器效率更加高效. 据Nginx官方站点说, 世界上60%的网站都使用了Nginx.

一般的, 当我们写Web应用时, 其他的静态资源会有一个发布的过程, 我们在对静态资源处理(合并,压缩,加文件指纹)后, 会将其发布到静态资源服务器中.

另一方面, 静态资源服务器通常还与页面静态化配合使用. 比如, 当网站后台编辑一条新闻完成后, 会触发一条新闻页面生成事件, 进行新闻相关页面的生成或者更新, 生成完成后, 得到的页面会被上传到静态服务器上. 生成缩略图等其他资源生成与更新的问题方法类似. 值得一提的是, 假设我们有3台静态服务器, 由一个负载均衡服务器为入口. 根据CAP理论, 我们需要在A与C之间作为平衡. 要么, 我们在生成页面时, 同时将内容上传到3台服务器上, 都上传成功才算成功；要么, 我们需要损失一定的可用性, 如, 内容已经上传到A上, 但还没有同步到B/C, 此时, 负载均衡器可能会将请求导到B/C上, 但此时, B/C上还没有最新生成的静态内容.

相比较而言, 使用缓存服务器可能是一种更好的选择. 我们只需要在新闻编辑后, 向所有缓存服务器发送一条信息, invalidate掉这条新闻相关页面缓存即可.

# 参考资料
+ [构建高性能Web站点](https://read.douban.com/ebook/1410608/)
+ [NGINX ADMIN GUIDE AND TUTORIAL](https://www.nginx.com/resources/admin-guide/)
+ [静态页面总结](http://wuqq.me/blogs/architecture/StaticPages.md)
+ [Wikipedia Architecture](http://perspectives.mvdirona.com/2008/12/wikipedia-architecture/)
+ [Varnish Cache](https://www.varnish-cache.org/)