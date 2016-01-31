# Why Services
I don't want to address why services are important in our architecture. There are tons of blogs, posts, and articles talking about this.

When you are writing an application, everything outside your application's core business can be viewed as services. If you've ever used PaaS platforms such as [Heroku](http://heroku.com/) and heared the golden rule of [12Factors](12factor.net), you will agree. I hight recommended you visit these sites first.

I will briefly talk about what services means for both service providers and services clients.

## Service Providers
Say, you are a DBA, and are running a MySQL for your company's website. What do you want? For example, the old machine is broken, and you're going to move your database to another machine. Or, you want change your master-master configuration to master-slave configuration. You just want your clients are totally unware of all these nasty tasks.

Also, if you are writing a rest service as a component in your company's system, in order to make your service visible to other components, you might have to register the service itself to a center server. But what if your boss want your services usable via a domain name?

In addition, what if you want to use other services?

## Service Clients
If you are in a big company, and you are writing a new product, you might have to use your company's core business functionalities. How do you achieve this? If the core functionalities are messed in a monolith, then that would be a disaster. If the core functionalities are accessible via REST services, that's great! But how do you reference those services? Via IP? Via service discovery? Using IP is not reliable, perhaps you decide to use some service discovery framework. Now you have to wire some wired template code in your code. And your application is bound to this configuration. What if you are going to use another service discovery framework or even drop it for lack of resources? Although these cases are rare, they do exist!

How industrial platforms do this? For example, Heroku, uses env variables to manage dependencies. This proved to be a huge success!

# How
So how to provide and consume services both efficiently and hassle-free? Here are some of my thoughts.

## Service Providers
As a service provider, I don't want to impose any boilerplace in my clean code, for example, I don't want to register myself to a service manager, I don't want to cope with fails inside my service and try to start myself if I crashed, and I don't want to change my database connections when an instance of the database dies. Also, I don't want to send emails to the administrator when there is a fatal error in execution. 

**I just want to write my own code!**

What I means is that I don't want to duplicate these boilerplates in every component. Even these boilerplates are highly encapusalted(they can make your executable undesirable large).

All those boilerplates can be delegated to the supporting platform! 

I just declare my code as a service, the replication level(eg. Service A should have 3 replicas), dependencies in a manifest file. The supporting platform should run my code, wire dependencies via `ENV`, launch n replicas according to the manifest. All the load balancing and failover should be done by the platform.

## Service Clients
As a client, I declare all my dependencies in a manifest, and the platform will wire dependencies via `ENV`, all my requests to services are automatically load balanced by the platform. When one instance of the service I'm using is crashed, the platform will route my requests to another instance. All these thing are totally transparent to my code! When my executable is down, the platform will automatically restart it and the platform can monitor my executable via logs.

# Implementation
Yeah, the above description is awesome, but does the supporting platform exist? Yeah, they exists! They are just what the industries does these years. There are also lots of open source supporting platforms emerged:

+ [Heroku](heroku.com)
+ [Google App Engine](https://appengine.google.com/)
+ [Deis](https://github.com/deis/deis)
+ [Kubernetes](kubernetes.io)