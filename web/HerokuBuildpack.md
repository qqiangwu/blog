Heroku 简直太好用了！用来以为它只支持固定语言，原来这些都是可以通过buildpack来定制的啊！

Heroku定义了一些默认的buildpack，其中有一个detect脚本，用于来检查app的framework。这些默认的buildpack会依次执行。也就是说，用户可以定义自己的！

1. 覆盖默认的buildpack：heroku buildpack:set https://github.com/heroku/heroku-buildpack-ruby
2. 也可以在创建app时指定buildpack：heroku create myapp --buildpack https://github.com/heroku/heroku-buildpack-ruby
3. vibe-d buildpack: https://github.com/skirino/heroku-buildpack-vibe.d.git#cedar-14