# 失败的方法
可以使用`window.navigator.online`来进行测试。但实际上，不同浏览器的语义是不同的。但最起码在Chrome中，这个方法是不能完全起作用的。
> In Chrome and Safari, if the browser is not able to connect to a local area network (LAN) or a router, it is offline; all other conditions return true. So while you can assume that the browser is offline when it returns a false value, you cannot assume that a true value necessarily means that the browser can access the internet. You could be getting false positives, such as in cases where the computer is running a virtualization software that has virtual ethernet adapters that are always "connected."

# Fake AJAX Request
可以使用以下代码
```javascript
myApp.service('Internet', function($http){
  this.IsOk = function () {
    return $http({ method: 'HEAD', url: '/' + window.location.hostname + "/?rand=" + Math.floor((1 + Math.random()) * 0x10000) })
    .then(function(response) {
      var status = response.status;
      return status >= 200 && status < 300 || status === 304;
    });
  }
});
```
不过，无法作用于localhost，测试不方便

# Offline.js
一个现成的库