# What
So what's Instagram? Basically, it's a social community. You can share photos with your friends. When you post a new photo, all your friends will see it.

# The People
There are only 3 engineers in the team, which means self-hosting is not acceptible. Their architecture is composed of plenty of services. Most of the services are external services or open-source projects.

# The Architecture
I will briefly talk about the architecture in several aspects.

## The Load Balance Layer
All traffic is routed by ELB and transports to 3 Nginx. These nginx will furthur forward traffic.

## The Persistence Layer
There are huge amount of data in Instagram, so the persistence layer is of great importance.  

+ Amazon S3 for storage
+ PostgreSQL for structured data. There are 12 instances and all instances have two replicas for high availability
+ Redis for session storage and main/activity feed since redis is fast for heavy reading

Especially I'd like to address how Redis is used since Redis is of high popularity recently.

### Session storage
Sessions are important and disturbing too. In order to make all application server stateless, we generally move sessons to the storage layer. Redis is a great candidate since it's very fast!

### Feed
If you've ever use any kind of social network applications, you'll notice that you have a timeline in your main page. All new events from your friends will appear in the timeline in real time. 

In order to access your timeline quickly, put them into Redis is a great choice!

See also my post about Twitter timeline!

## The Business Layer
Instagram is comprised of lots of components. In order to respond quickly, these components are linked via asynchronous scheduling.

Gearman is the core of the scheduling system. There are 200 workers consuming the Gearman queue.

When one post a new photo, a FanOut task will be put into Gearman. When fanout is done, a Notification(notify all subscribers of the new photo) task will be put into Gearman.

Employing Gearman extensively makes the website respond very quickly!

## Misc
There are also some other aspects which are important for a production but I won't talk more about them since they are quite easy to understand and implement.

+ Deployment: Fabric
+ Monitoring and reporting: 
    + Pingdom
    + Sentry
    + PagerDuty
+ Metrics: Munin