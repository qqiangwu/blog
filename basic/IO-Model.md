# Why IO Model?
Processing and IO are the basic element of computation. In order to speed our computation, we should take care of these two aspects. 

For fast processing, we need to invent new algorithms, new computation models. But this is not the whole story. IO Model also matters in our program.

In this post, I will briefly cover the IO models provided by Unix and illustate what an IO model is and why it matters. After that, I will especially talk about general IO models that apply to every platform.

# Unix IO Models
Unix provides 5 IO models, divided to two categories: synchronous and asynchrous. The terminology is quite confusing, but it's no harm to your understanding.

When we perform an read operation, what do we do? We do the following things:

+ Tell OS that we are going to read from a fd and wait until the fd is ready.
+ Perform a read on that fd.

Write is somehow the opposite, but the principle applies.

The 5 models differ in these two steps:

+ Do we need to block our processing waiting for the fd?
+ When performing the read, do we block our processing.

Synchronous IO models are blocking when we are perform the ready, but differ in the first step, either blocking or non-blocking. Asynchronous IO models are non-blocking in both the two steps.

## Synchronous IO models
There are 4 synchronous IO models.

+ Blocking IO
+ Non-blocking IO
+ IO multiplexing
+ Signal driven IO

They differ in the first step:

+ Blocking IO is the most clumsy mechanism. They will block until the fd is ready. e.g. C stdio and Java IO.
+ Non-blocking IO is just a little better than the first one. You can check whether a fd is ready. If not, continue to do something else and check it later. If yes, perform the read.
+ IO multiplexing: Non-blocking is somehow useful because we can check whether the fd is ready and avoid waiting. But what if we have tens of fds? In this case, checking them one by one is stupid. IO multiplexing help us with this check. We tell IO multiplexing to help us check our interested fds, when some are ready, perform reads on them. But in this case, we also block our processing since the control flow is transfered to the IO multiplexing mechanism.
+ Signal driven IO: We just tell OS that we are going to read a fd and contine our processing. When the fd is ready, OS send us a signal. It seems that this is the most efficient way.

## Asynchronous IO model
Asynchronous IO seems to be the most efficient way to perform IO operations. Think of nodejs, its high thoughput is due to async IO model.

So, what's async IO? To put it simply, when you want to read from a fd, you just prepare a buffer and tell OS that you want to read and continue to do other things. OS will take care of all the waiting and reading steps. When the read is done, your buffer is filled with data and OS notify you that your operation is done, typically via a callback.

There are several async IO frameworks, such as nodejs, Java AIO, boost.asio. In boost.asio, when the underlying OS doesn't support async IO model, boost.asio will use IO multiplexing, but it provides the client an synchronous IO abstraction.

# Realworld IO models
So, why I talk about IO models? In reality, say, we are writing a web server for high throughput, IO is most crucial part. If we use blocking IO, to handle a request, we have to block our thread when we are reading from a database. If we are handling tens of requests, we have tens of thread blocked. That's a nightmare since thread creation, conext switching are both expensive operations. If the number of requests is few, the operation is quite compact and efficient. But if we have thousands of connections, bang!

In this section, I will describle 3 typical IO models. These IO models are general concepts and apply to almost every platform.

+ Sync API, Sync IO: Yeah, it is the blocking IO I've mentioned above. It's quite efficient when we just have a few of fds. It's compact and easy to program. We just initiate an IO operation, and wait, and execute the next statement.
+ Async API, Async IO: Think of nodejs, we initiate an IO operation and provide a callback, when the IO operation is done, our callback is called. Async IO is pleasant, but async api is not so pleasant.
+ Sync API, Async IO: This is the most awesome IO form. It's implemented via a lightweight thread mechanism called fiber(or coroutine?). You just write your IO operations as normal, in a step by step form, and the fiber framework helps your with the rest. When you initiate a read, the current fiber will suspend and another fiber will execute. When the IO operation is done, your original fiber continues. I won't talk about the detail. If you're interested, try [vibed](vibed.org).