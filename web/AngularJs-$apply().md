# 回调
由于JS是Turn-based执行的。当程序中有回调发生，相当于发生一个新的Turn，其执行完毕后，AngularJS并不知道，此时，我们需要用：

```JS
$scope.$apply(callback()); 
```

以通知AngularJS，让其检查Model的变化。同时，它会检查异常，并在AngularJS内部处理它们。更进一步，实际上，进行模型检查的主要是$digest()方法，但是，请不要使用它，直接使用$apply()。

事实上，AngularJS不允许你在$apply()中再次调用$apply()，可以通过检查$scope.$$phase来判断是否已经$apply()过。

# AngularJS 提供的封装
AngularJS提供了`$timeout`，这并不是画蛇添足。`$timeout`的大致实现可以看作这样:

```JS
$timeout = function(callback, delay) {  
    setTimeout(function(){   
        callback();   
        $rootScope.apply();  
    }, delay);  
};
```

也即，所有可能会对数据生成改变的地方，都必须显式调用`$apply()`

AngularJS对如下服务提供了封装：

+ ng-* Event Listeners
+ $http
+ $timeout

# 结论
1. 尽可能使用AngularJS服务
2. Native代码中，所有回调函数，都需要在函数调用后使用$apply()

# 参考
1. <http://jimhoskins.com/2012/12/17/angularjs-and-apply.html>