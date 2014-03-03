---
layout: post
title: Nginx 学习笔记
---

##nginx平台初探
####架构
nginx运行方式：

 + 后台以daemon的方式运行，后台进程包含一个master进程和多个worker进程
 + 手动关掉后台模式，让其在前台运行；
 + 过配置让nginx取消master进程，以单进程方式运行，但这大多是在调试的时候使用，在生产环境一定不要这样做；
* nginx默认是以多进程方式来工作的；
* 也支持多线程方式。
nginx启动后，会有一个master进程和多个worker进程。master进程主要用来管理worker进程，包含：

* 接收来自外界的信号
* 向各worker进程发送信号
* 监控worker进程的运行状态，当worker进程退出后（异常情况下），会自动重新启动新的worker进程

基本的网络事件都是放在worker进程中来处理的。多个worker进程之间是*对等独立*的，同等竞争来自客户端的请求，一个请求只可能实在一个worker进程中处理。
worker的数量一般会设置成cpu的核心数，这与nginx的进程模型以及事件处理模型是分不开的
![Image Title](https://github-camo.global.ssl.fastly.net/3856f1506eeb546785ac5ffd44a31dc4e3879dd3/687474703a2f2f74656e67696e652e74616f62616f2e6f72672f626f6f6b2f5f696d616765732f636861707465722d322d312e504e47)

可以通过kill向master进程发送信号来控制nginx，例如 
    
    kill -HUP pid
    
则告诉nginx从容地重启nginx。一般用这个信号来重启nignx或者重新加载配置，这样重启服务是不中断的。master接收到HUP信号后是这样的：

* 首先master重新加载配置文件
* 启动新的worker进程，这些worker就可以开始接收新的请求了
* 向老的worker进程发送信号，通知其可以光荣退休了，这些worker不在接受新的请求，在当前进程中所有未处理完的请求完成之后退出

nginx 0.8之后可以使用命令来管理，例如，

    ./nginx -s reload #重启nginx
    ./nginx -s stop #停止nginx运行
    
其中reload执行时，会启动一个新的nginx进程并解析到reload参数，则nginx会重新加载配置文件，向master进程发送信号，接下来的动作就和直接向nginx发送信号是一样的了。

nginx采用进程模型的好处：

* 首先，对于每个worker进程来说，独立的进程，不需要加锁，所以省掉了所带来的开销，同时在编程和问题查找时，也方便很多
* 其次，采用独立的进程，可使各个worker之间不会互相影响，一个进程退出后，其他进程还在工作，服务不会中断，master进程则很快启动新的worker进程。

nginx采用异步非阻塞的方式来处理事件。一个请求事件的完整过程是：

* 建立连接
* 接收数据
* 发送数据

具体到系统底层，就是读写事件，而当读写事件没有准备好的时候，必然不可操作，如果是阻塞调用，事件没有准备好就只能等事件准备好之后才能继续处理。阻塞调用会进入内核等待，cpu就会让出去给别的进程用。当网络事件特别多时，大多数进程都在等待，cpu空闲下来没人用，利用率很低，更别谈高并发了。那就要用非阻塞来实现，非阻塞就是事件没有准备好，马上返回[EAGAIN](http://blog.sina.com.cn/s/blog_48d5933f0100qnso.html)，告诉你，事件还没准备好呢，等会再过来吧。过一会儿再来检查一下事件，知道事件准备好了为止，在这期间，你就可以先去做其它事情，然后再来看看事件好了没。虽然不阻塞了，但需要不时的来检查事件的状态，可以做更多的事情了，但带来的开销也很大。所以才会有了异步非阻塞的事件处理机制，具体到系统调用就是select/poll/epoll/kqueue。他们提供了一种机制，让你可以同时监控多个事件，调用他们是**阻塞**的，但可以设置超时时间，在超时时间之内，如果有事件准备好了，就返回。这种机制正好解决了我们的两个问题，以epoll为例，当事件没准备好时，放到epoll里面，事件准备好了，就去读写，当读写返回EAGAIN时，将请求再次加入epoll里。只要有事件准备好了，就去处理。若所有事件都没有准备好，就在epoll里面等待。这样并发量就上来了，但是都是未完成的请求，线程只有一个，所以当前处理的请求只有一个，只是可以在大量的请求之间不断地切换。切换是因为异步事件没有准备好，而主动让给别的请求，这里的切换是没有任何代价的，可以理解为循环处理多个准备好的事件。与多线程相比，异步非阻塞不需要创建线程，每个请求占用的内存也很少，没有上下文切换，事件处理非常轻量级。
所以推荐设置worker数和cpu的核心数一致也就不难理解了，更多的worker数，只会导致进程来竞争cpu资源，从而带来不必要的上下文切换。而且，nginx为了根号的利用多核的特性，提供了cpu亲缘性的绑定选项，可以将某一个进程绑定在某一个核上，这样就不会因为进程的切换带来cache的失效。

伪代码来总结nginx的事件处理模型：

    while (true) {
        for t in run_tasks:
            t.handler();
        update_time(&now);
        timeout = ETERNITY;
        for t in wait_tasks: /* sorted already */
            if (t.time <= now) {
                t.timeout_handler();
            } else {
                timeout = t.time - now;
                break;
            }
        nevents = poll_function(events, timeout);
        for i in nevents:
            task t;
            if (events[i].type == READ) {
                t.handler = read_handler;
            } elif (events[i].type == WRITE) {
                t.handler = write_handler;
            }
            run_tasks_add(t);
    }
