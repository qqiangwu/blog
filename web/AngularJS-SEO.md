# A NodeJs + AngularJS SEO Friendly Single-page Web Application
As we know, single-page web applications have come into fashion since the appearance of Web 2.0, but their inability to search engines make us quite unhappy. 

There are, of course, some workaround to deal with this, and I'm about to present what I've done in my recent work. 

You could have heared that the Google robot can execute js in web pages it crawled, thus you may think that it's unnecessary to do SEO, but I doubt it. Actually, most of search engines cannot execute js in pages, so you should not rely on that.

## General Idea
The general idea to deal with the problem is some sort of prerendering. That is, the server responds to search engines with prerendered pages. There are two well-known solutions in AngularJS.

## Solution 1: Hashbang
Some search engines are able to deal with a special kind of url which contains hashbangs:`#!`. They will convert `site/x/#!/b` to `site/x/?_escaped_fragment_/b`. Therefore your web server will know that a search engine is asking for a page. 

[This article](http://www.yearofmoo.com/2012/11/angularjs-and-seo.html) talks about this solution in detail.

The **hashbang** scheme is nice, but applies only to specific search engines(As far as I know, only Google and Bing support it). If you live in countries where Google is banned then you have to resort to other solutions.

## Solution 2: 
The second solution is what I used in my implementation. In AngularJS, there is something called *html mode*, in which case your website can neatly eliminates hashbangs. Which means, you can use `mysite.net/main` instead of `mysite.net/#!/main`. But this requires extra effort in your backend. Your backend must redirect `mysite.net/main` to `mysite.net`, or you will receive a 404 since `mysite.net/main` is just a state of `mysite.net`, you can safely view it as an alias to `mysite.net/#!/main`. 

Why *html mode* matters? Because search engines will view `mysite.net/#xxxx` as `mysite.net`, they will simply discard address after `#`. In your single-page application, there may be a lot of views such as `mysite.net/#/home`, `mysite.net/#/blog`, and so on. Search engines will view them all as `mysite.net`.

### Backend
All above done, we can deal with SEO. All web states are prefixed with `/ui/`, when server see these pages, it check the `User-Agent` header(I doubt it is a suitable way to recognize whether the request comes from a search engine).  
 
1. If the request comes from a browser, simply return `index.html`. 
2. If the reqeset comes from a search engine, render it using **PhantomJS**, and return the result page.

### Code
#### Frontend  
```Javascript
app.config(['$locationProvider',
    function($locationProvider){
        $locationProvider.html5Mode(true);
    }
]);

app.config(['$routeProvider', 
    function($routeProvider){
        $routeProvider
            .when('/ui/main', {
                templateUrl: 'tpl/main.html',
                controller: 'MainCtrl'
            })
            .when('/ui/self', {
                templateUrl: 'tpl/self.html'
            })
            .when('/ui/view/:dir/:path', {
                templateUrl: 'tpl/view.html',
                controller: 'ViewCtrl'
            })
            .otherwise({
                    redirectTo: '/ui/main'
            });
    }
]);
```
#### Backend web server
```Nodejs
var express = require('express');
var app = express();
var port = process.env.PORT || 8080;
var site = 'http://qqiangwu.herokuapp.com';

var getContent = function(uri, callback) {
    var content = '';
    var err = '';
    var url = site + uri;
    
    console.log('SEO - prerender: ', url);
    
    var phexe = __dirname + '/node_modules/phantomjs/bin/phantomjs';
    var phantom = require('child_process').spawn('node', [phexe, 'phantom-server.js', url]);
    
    phantom.stdout.setEncoding('utf8');
    phantom.stdout.on('data', function(data) {
        content += data.toString();
    });
    phantom.stderr.on('data', function(data) {
        err += data;
    });
    
    phantom.on('exit', function(code) {
        if (code !== 0) {
            console.log('We have an error:', err);
            console.log('Content read:', content);
        }
        
        callback(content, code !== 0);
    });
};

var seoDetector = new RegExp('Baiduspider|Googlebot|BingBot|Slurp!', 'i');

var responsePage = function (req, res) {
    var url = req.originalUrl;
    var isRobot = seoDetector.test(req.get('User-Agent'));
    
    console.log('Serve %s [SEO: %s]', req.url, isRobot);
    
    if (isRobot) {
        getContent(url, function(content, err){
            if (err) {
                res.status(500).end();
            }
            else {
                res.send(content);
            }
        });
    }
    else {
        res.sendFile(__dirname + '/web/index.html');
    }
};

app.use(function(req, res, next){
    console.log('%s %s', req.method, req.url);
    
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'GET');
    res.header('Access-Control-Allow-Headers', 'Content-Type');
  
    next();
});

app.get('/', responsePage);
app.get('/ui/*', responsePage);
app.get('*', express.static(__dirname + '/web'));

app.listen(port);
```
#### Backend prerendering server
```Nodejs
var page = require('webpage').create();
var system = require('system');

var lastReceived = new Date().getTime();
var requestCount = 0;
var responseCount = 0;
var requestIds = [];
var startTime = new Date().getTime();

page.onResourceReceived = function (response) {
    if(requestIds.indexOf(response.id) !== -1) {
        lastReceived = new Date().getTime();
        responseCount++;
        requestIds[requestIds.indexOf(response.id)] = null;
    }
};

page.onResourceRequested = function (request) {
    if (requestIds.indexOf(request.id) === -1) {
        requestIds.push(request.id);
        requestCount++;
    }
};

page.open(system.args[1], function () {});

var checkComplete = function () {
  if((new Date().getTime() - lastReceived > 300 && requestCount === responseCount) || new Date().getTime() - startTime > 5000)  {
    clearInterval(checkCompleteInterval);
    console.log(page.content);
    phantom.exit();
  }
}

var checkCompleteInterval = setInterval(checkComplete, 1);
```