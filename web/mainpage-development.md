开发自己的个人主页时遇到的问题。

1. 服务器的选择：懒得租，先用着github.io吧。
2. 留言问题：使用第三方工具。
3. 博客列表：只能先用脚本预处理一遍了。
4. 搜索引擎：为了能让搜索引擎使用，必须添加sitemap，特别是对这种动态站点。

# 遇到的问题
1. Google在分析网页的时候会执行js，这是相当nice的，但在我第一次看到Google的分析结果时，我发现了这样一条错误：  
     `XMLHttpRequest cannot load http://qqiangwu.github.io/conf.json. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://webcache.googleusercontent.com' is therefore not allowed access.  `  
解决：
	+ 这是由Web应用的安全模型所决定的。浏览器只允许页面运行与自己来源相同的script。
	+ 同样，这也适应于XMLHttpRequests。总之就是XMLHttpRequest不能跨域请求。
    + 有许多Hack尝试去解决这些问题，比较jsonp什么的，但都不是很完美。
    + Cross-Origin Resource Sharing (CORS)：这是一个新的标准，目前，许多浏览器已经实现。它的作法是，由服务器在Response中添加如下Headers:  
    ```Python
    def myview(_request):
        response = HttpResponse(json.dumps({"key": "value", "key2": "value"}))
        response["Access-Control-Allow-Origin"] = "*"
        response["Access-Control-Allow-Methods"] = "POST, GET, OPTIONS"
        response["Access-Control-Max-Age"] = "1000"
        response["Access-Control-Allow-Headers"] = "*"
        return response
    ```
2. 由于使用的Github的服务器，因此，得看看它支不支持这CORS。不过在网上看了一样，好像没有看到。除非完全改用github v3 api。
3. 看到个crossdomain.xml的东西，不知道能用否。
