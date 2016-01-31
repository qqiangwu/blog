开发自己的个人主页时遇到的问题。

1. 服务器的选择：懒得租，先用着github.io吧。
2. 留言问题：使用第三方工具。
3. 博客列表：只能先用脚本预处理一遍了。
4. 搜索引擎：
    + 为了能让搜索引擎使用，必须添加sitemap，特别是对这种动态站点。
    + robots.txt的用途：我还没有用，不过它用途好像很大的样子。
5. SEO
    + 使用`#!`，这是搜索引擎的一种约定，它会将其替换为`?_escaped_fragment_=`，同时，请求头里面也可以识别请求是由谁发的。这让服务器有了hack的机会。
    + GoogleBot会忽略`#`后面的东西，除了`#!`，当然，百度都会忽略。
    + `<meta name="fragment" content="!">`
    + 算了，不想了，把时间花在这上面划不来！
6. Google Analytics for SPA: angulartics
7. Add Travis CI to avoid unnecessary deployment by Heroku

# 遇到的问题
1. Google在分析网页的时候会执行js，这是相当nice的，但在我第一次看到Google的分析结果时，我发现了这样一条错误：  
     `XMLHttpRequest cannot load http://qqiangwu.github.io/conf.json. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://webcache.googleusercontent.com' is therefore not allowed access.  `  
解决：
    + 这是由Web应用的安全模型所决定的。浏览器只允许页面运行与自己来源相同的script。
    + 同样，这也适应于XMLHttpRequests。总之就是XMLHttpRequest不能跨域请求。
    + 有许多Hack尝试去解决这些问题，比较jsonp什么的，但都不是很完美。
    + Cross-Origin Resource Sharing (CORS)：这是一个新的标准，目前，许多浏览器已经实现。它的作法是，由服务器在Response中添加如下Headers:        
    ```Python
    def myview(request):  
        response = HttpResponse(json.dumps({"key": "value", "key2": "value"}))  
        response["Access-Control-Allow-Origin"] = "*"  
        response["Access-Control-Allow-Methods"] = "POST, GET, OPTIONS"  
        response["Access-Control-Max-Age"] = "1000"  
        response["Access-Control-Allow-Headers"] = "*"  
        return response  
    ```
2. 由于使用的Github的服务器，因此，得看看它支不支持这CORS。不过在网上看了一样，好像没有看到。除非完全改用github v3 api。
3. 看到个crossdomain.xml的东西，不知道能用否。(已经试过，不能用)
4. 花了一个下午，将markdown集成到了notepad++中。
5. 尝试向网页中添加jiaThis分享功能，但总是出现`Uncaught Syntax error, unrecognized expression: [object HTMLDivElement]`，于是无奈放弃。
6. [2015/1/26]发现Google Webmaster，但是Googlebot在render时无法获取api.github的资源，于是又调回到原来的直接访问。但这样一来google cache又不work了。真是！
7. 发现Googlebot无法识别`#!`。唉。再看看吧。

# Heroku
还是决定用一个服务器了。于是又是一堆新问题了。

1. heroku不支持git submodule。=> 移除submodule。
2. 使用NodeJS + Express
3. SEO: 使用PhantomJS来根据User-Agent动态生成页面。