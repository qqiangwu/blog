# Why Monitoring?
For example, you are running a distributed system comprised of dozen of nodes and hundreds services. Then how do you know status of your nodes and services? How do you know the performance, the liveness, the workload? 

Yeah, you need to monitor your system, find fact and keep improving your system or take actions when something goes wrong.

# Monitoring What?
## Health of your nodes and services and get notified when they are down
To keep your system work, replication and failover are not enough. You may have to repair unfunctional nodes and fix bugs in services that crashed unexpectedly. 

When your nodes or services are down, you must be notified. 

I'd like to take a detour to talk about notifications. If your nodes are down, who notifies you? Yeah, we need a external monitor in this case. But if your services are down or are in bad states, who reports it? I've seen lots of applications built this logic into the applications themselves. I don't think this is a good idea since you might duplicate the alerting code hundreds of times if you have hundreds of services. So, using an external monitor is still a bettere choice. But how does the monitor know that your services are in bad states? Keep reading!

## Metrics of your nodes
Knowing status of your nodes is of great importance. For example, you've deployed 3 services in node X, and node X get overloaded. Via node metrics, you can soon get aware that node X are lack of cpu, and you may decide to move a service into another node and just change node X into a more powerful one.

Also, you can graph metrics of your nodes in a period and know your nodes' workload patterns.

## Metrics of your services
Apart from knowing that your services are live, it's more important to know other service-specific metrics, eg. login times in a day in a web app.

Generally, we can build in metrics counting functions into your services since your services know better what to count, just like what [spring-boot-actuator](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-enabling) does. Still, I don't think this is a good idea. In one hand, it's tedious, you have to duplicate metric recording code several times. In the other hand, it's not extensible. Say, you've recorded 100 metrics in your service X. What if you want to record one more metrics? You have no choice but to modify your code!

So, how can we observe the life of your services? How do we know what our services are doing? Yeah, that's it! Log! Log! Log!

Log traces our system, we can know almost everything about our system via log. There are two aspect of application of log:

+ Alerting: when there are ERRORs in log, the monitor detects it and reports it!
+ Metrics: since log records what our services have done, we can simply filter out specific log record and count them!

# Tools
+ [Health Check](https://www.nagios.org/)
+ Node Metrics
    + [Nagios](https://www.nagios.org/)
    + [Ganglia](http://ganglia.sourceforge.net/)
+ Services Metrics
    + [Logging](https://www.elastic.co/products/logstash)
    + [Metric Filter](https://github.com/logstash-plugins/logstash-filter-metrics)