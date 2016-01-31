# What's Unix computation model?
You must have heared of Unix philosophy which makes Unix a huge success.

So, how to write a **good** program? Unix tells us that:  

+ Do one thing, and do it well!
+ Use stdin as the sole input, and use stdout as the sole output.

These two principles are the building blocks of thousands of Unix/Linux applications which prove to be a great success. The Unix applications are connected up through pipes and they work seamlessly even without knowning each other!

eg. `awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -n 5`

So guess what the above commands does?

For simplicity and understanding, I call this Unix Computation Model, or UCM.

So, what's the magic? The principles above generally produce a pure-function-like application. You know, pure functions are great, so does an UCM application. We concentrate on our program unit, and all the complicated connections between program units are processed by Unix pipes, which makes UCM applications amazingly extensible and flexible.

Basically, `Do one thing and do it well` makes us concentrate on the pure application logic, `pipes` help us build a system out of little program units. Yeah, the connections between components are the main source of complexities. Generally, these connections are implicitly a directed graph.

So, can we apply UCM to other domains? Yes we can!

# Drawbacks of pipes
Before I proceeds, I'll talk about drawbacks of pipes first. 

Pipes connect our computation units, but they have their own drawbacks.

+ Connections are one-to-one which leads to a chain of computation processes rather than a directed graph.
+ Pipes are not usable in distributed systems.
+ Pipes are short-lived.
+ ...

The first one is the main downside of pipes. It limits the strength of our computation system consisting of computation units.

# UCM inside an application
Recently I'm writing a data integration application. What it does are as following:  

+ Get streamed data from a datasource, either by http pulling or tcp pushing. (I call this module Dataloader)
+ Save the data to the local file system. (I call this module Backup)
+ Save the data to mongodb. (I call this module Persist)
+ Serve the data to a computation application to do some computation. (I call this module Processor)

Yeah, each of the single task is very straightforward, but when they all mixed up, things become not very happy.

When the Dataloader get some data, how it let other modules to function. Initially, I use callbacks. So the Dataloader must retain references to other modules. Okay, it works. But soon, I was notified that another module consuming these data must be added. So what can I do? After that, I was further asked to remove the Backup since it consumes a lot of storage. Damn it! I have to modify my configuration code.

So, what's the issue? I've already partioned the whole system into submodules, and they `do one thing and do it well`. What I'm lack of is somewhat a pipe! A one-to-many pipe! Callbacks are not suitable in this case for its less flexible and extensible. Every time when a new module is added or an old module is removed, I have to update callback references.

Finally I decided to use messages. Whenever the Dataloader recieved data, it broadcast a `DATA.NEW` message. All other modules recieved `DATA.NEW` and begin to do their own work. This works! But I don't want to call it messages, I'd like to call it a pipe since pipes have more semantics than messages. When events are unordered, messages are all right, but most of the time, pipes are more appropriate since they remind us that we are using a transport channel. Also, using pipes makes our system architecture more clear and it can be graphically visualized as a graph. You can think of messages as an implementation of pipes.

When this model is applied, each component only need to think about what's my input, what's my output, how do I compute. Input and output are all described as pipes, computation becomes quite straightforward. 

# UCM inside web pages
I'm also writing a web page full of states and actions. There are lots of switches in the page, whenever a switch is changed, I have to:  

+ Update the page.
+ Update my localStorage.
+ Update my js objects.
+ Persist to backend.

This seems exactly the same as the above example. As usual, I use the callback way at first. But soon I'm exhausted since each switch has serveral callbacks and I have lots of switches which further lead to more callbacks. Finally I decided to refactor my code and employ pipes to manage component connections. BTW, I use `pubsub` as the implementation of pipes. 

It works amazingly good. All my components are decoupled and I concentrate on a single component without even knowning other components. What I need to do is just do some computation after I've received a message and produce another message when the computation is done.

# UCM inside concurrency
We all know that concurrency are powerful as well as nasty. Locks are bad. Is there a better way? I'm trying to apply UCM to concurrecy.

In this section, I will only consider threads as the underlying concurrency mechanism. 

To begin with, I must tell you that raw threads are evil, so never use it directly! Use other high level abstractions such as tasks, async, executors, schedulers, etc.

## Producer and consumer
If you've read any concurrency textbook, you must know this mode. You may read this post first: [Parallelism in one line](http://chriskiehl.com/article/parallelism-in-one-line/).

Producer and Consumer pattern has too many template code which is tedious. So can we apply UCM? 

Suppose that we are writing a web crawler. Think about what computation we need to do:  

+ Collect urls and distribute tasks
+ Crawl the url and emit new urls

Think of the first task. We simply write a task, which wait for the `URL.NEW` event and push url to be processed to a queue `URL.TODO`.

Think of the second task. We write a task, which reads from the queue `URL.TODO`, and crawling the url, downloading the content, parse it, detect new urls, and publish an `URL.NEW` event.

This seems not very straightforward as above, but it follows the same principle: we read from a pipe, perform computation, write the result to a pipe. What's the pipe are? We don't care, they are only transport channels which connect our components. So, we finally get a system consisting of decoupled components.

In practice, we only use a scheduler as a smart pipe. We simply push results to it. What really matters is that **NEVER EXPLICITLY REFER TO ANOTHER COMPONENT BY NAME**.

## Mapreduce
Mapreduce tasks can be also viewed as an UCM program. Mappers read from pipe A and produce results to pipe B. Reducers read from pipe B and write results to pipe C. Mappers and reducers don't know each others which achieved immense composibilty.

# UCM in distributed systems
A distributed system is somehow a larger concurrent system in a single machine. We can simply apply the same obervations. Each component reads from a pipe, performs a single task and push the result to another pipe. The problem is that what's the pipe in the distributed system? I deem that [Apache Kafka](kafka.apache.org), [Taobao Metaq](http://code.taobao.org/p/metaq/src/), etc. can be used as one. Also refer to the following articles:  

+ [The Log: What every software engineer should know about real-time data's unifying abstraction](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)
+ [Putting Apache Kafka To Use: A Practical Guide to Building a Stream Data Platform: Part1](http://www.confluent.io/blog/stream-data-platform-1/)
+ [Putting Apache Kafka To Use: A Practical Guide to Building a Stream Data Platform: Part2](http://www.confluent.io/blog/stream-data-platform-2/)

# Limitations of UCM
Note that UCM is not perfect. Think of the following scenario. You are writing a module and need to invoke another module and perform further operations on the result. In this case, pipes do not suffice since they are unidirectonal. 

You may argue that bidirectional pipes exist such as HTTP, TCP. Yeah, but the question is that bidirectional pipes need two endpoints. If module A want to call module B, it must knows the endpoint of B. That's less flexible. In addition, bidirectional pipes need to establish and maintain a connection which further compliates the issue. In the unidirectional case, A simply writes to a pipe identified by X, B simply reads from pipe X. They don't have to know about each other.

You may also argue that Service Discovery might help. Yeah, maybe, but it litters our code.

I've also devised some possible solutions, but I've never tried any one extensively and also, they are another story:  

+ Dependency Injection
+ Callbacks as argumens of asynchronous tasks
+ Interfaces as a pipe endpoint

# Reference
+ [Spring Integration](http://docs.spring.io/spring-integration/docs/4.2.0.RELEASE/reference/html/index.html)
+ [Messaging Patterns](http://www.enterpriseintegrationpatterns.com/patterns/messaging/)
+ [ESB](https://en.wikipedia.org/wiki/Enterprise_service_bus)