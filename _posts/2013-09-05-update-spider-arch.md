Date: 2013-09-11 14:48:43

# 爬虫 升级记

## 基础架构搭建
> 架构方面可能会转用 *Celery* 作为分布式的队列



#### Gearman
##### 安装依赖
    
    $ sudo apt-get -y install gcc autoconf bison flex\
         libtool make libboost-all-dev libcurl4-openssl-dev\
         curl libevent-dev memcached uuid-dev 
         
##### 安装Gearman 

    $ cd software
    $ wget 'https://launchpad.net/gearmand/1.2/1.1.9/+download/gearmand-1.1.9.tar.gz'
    $ tar zxvf gearmand-1.1.9.tar.gz
    $ cd gearmand-1.1.9
    $ ./configure && make
    $ sudo make install
    $ export PATH=$PATH:/usr/local/bin
    $ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    $ sudo mkdir -p /usr/local/var/log
    $ sudo gearmand -d -Lxxx.xxx.xxx.xxx -p4730 -f1000 -j4 -R -t100 --log-file='/usr/local/var/log/gearmand.log'
###### 手动编译python
> 版本2.7.4
> 
> 配置独立python环境
> 
> requirements.txt

##### 安装Sentry

    $ pip install sentry
    $ pip install supervisor
    
###### 准备sentry、sv相关的配置文件
* sentry默认的配置文件是放在家目录的    

        ~/.sentry/sentry.conf.py    
    启动的时候可以通过--config来指定配置文件(绝对路径)。
    具体配置文件见工程目录下:
        
        service/sentry.conf.py

* sv的配置文件需要执行命令

        $ echo_supervisord_conf > ./supervisord.conf 
    
    来生成默认的配置文件，编辑配置文件时，要有两个program分别为http server和upd server。
    具体配置文件见工程目录下:
    
        service/supervisord.conf

###### send log in celery to sentry
在celery中，默认是会将celery运行之前加载的logging配置全部重置的，
> By default any previously configured logging options will be reset, because the Celery programs “hijacks” the root logger.

所以要想把log信息发送给sentry就要将这个 *CELERYD_HIJACK_ROOT_LOGGER* 配置项设置为 __False__ 才行，具体的将sentry加入到工程里边的实现可以参考 [这篇文章](http://www.rkblog.rk.edu.pl/w/p/using-sentry-log-exceptions-and-logging-messages-django-projects/)。

### 清除celery中的任务
    $ celery amqp queue.purge <queue name>
    


### 全部安装包
* apt-get 式:

        $ source /path/to/bin/activity
        $ sudo apt-get -y install rabbitmq-server
    
* pip 式：
在测试环境全部调通程序之后，利用

        $ pip freeze
    命令来生成一份全部软件包的 *requirements.txt* 文件即可，这里只做流水记录：

        $ pip install virtualenv
        $ pip install celery>=3.0.23
        $ pip install eventlet
        $ pip install dnspython
        $ pip install librabbitmq  # an AMQP client implemented in C
        $ pip install requests
        $ pip install eventlet
        $ pip install dnspython
        $ pip install redis
        $ pip install gevent
        $ pip install pyrant
        $ pip install MySQL-python
        $ pip install 
* 在virtualenv下安装PIL下需要注意：

        $ sudo apt-get -y install libfreetype6-dev libjpeg8-dev libz-dev
        $ sudo ln -s /usr/lib/x86_64-linux-gnu/libfreetype.so /usr/lib/
        $ sudo ln -s /usr/lib/x86_64-linux-gnu/libz.so /usr/lib/
        $ sudo ln -s /usr/lib/x86_64-linux-gnu/libjpeg.so /usr/lib/ 

    
### 解决方案探索 2013-09-02

+ 模仿scrapy commands 中 parse.py来写worker，这作为request的入口
+ 用celery来作分布式队列
+ 明天进行初步的测试，写worker
+ 用downloadermw来呼叫  worker  获取   response 和 item
scrapy中webclient.py  不能处理各种特殊情况，只是一个pagegetter 不能处理302等，所以决定还是不放弃scrapy来作为workder端，它的downloader mw做的太好了
+ 明天测试的维度  之一是  在外部调用spider的效率如何，（程序设计的时候应保持一个spider在内存中，这样每次一个请求过来之后不用重新open spider，会比较有效率，但这只是猜想，明天还要看数据）。

+ 采用广度优先的遍历算法，cat url 的优先级要比 next page url 的优先级高
+ 设计的时候考虑加入监控指标：
    * 每个网站的每个分类的商品抓取量   多机之间用redis来存储计数器，不会造成同步冲突，incre为原子操作
    * 
---------------------------------------------------

# 笔记

    
    
### Celery

> You just learned how to call task using the tasks *delay* method, and this is often all you need, but sometimes you may want to pass the signature of a task invocation to another process or as an argument to another function, for this Celery uses something called *subtasks*.
> 
> A subtask wraps the arguments and execution options of a single task invocation in a way such that it can be passed to functions or even serialized and send across the wire.
>
>
>
> You can create a subtask for the *add* task using the arguments *(2, 2)*, and a countdown of 10 seconds like this:
> 
    >>> add.subtask((2,2), countdown=10)    
    tasks.add(2,2)
>   
> This is also a shortcut using star arguments:
>    
    >>> add.s(2, 2) 
    tasks.add(2, 2)
> *celery multi*     doesn't store infomation about workers so you need to use the same command line parameters when restarting. Also the same pidfile and logfile arguments must be used when stopping/killing.
> By default it will create pid and log files in the current directory, to protext against mulitple workers launching on top of each other you are encouraged to put htest in a dedicated directory:
>
    $ mkdir -p /var/run/celery
    $ mkdir -p /var/log/celery
    $ celery multi start w1 -A proj -l info --pidfile=/var/run/celery/%n.pid --logfile=/var/log/celery/%n.pid
 
> With the multi command you can start multiple workers, and there is a powerful commad line syntax to specify arguments for different workers too, e.g:
>
    $ celeryd multi start 10 -A proj -l info -Q:1-3 images,video -Q:4,5 data -Q default -L:4,5 debug

> For more examples see the *[celeryd_multi](http://docs.celeryproject.org/en/latest/reference/celery.bin.celeryd_multi.html#module-celery.bin.celeryd_multi)* module in the API reference.
> The --app argument specifies the Celery app instance to use, it must be in the form of moudle.path:celery, where the part before the colon is the name of the module, and the attribute name comes last. If a package name is specified instead it will automatically try to find a celery module in that package, and if the name is a module it will try to find a celery attribute in that module. This means that these are all equal:
>    
    $ celery --app=proj
    $ celery --app=proj.celery:
    $ celery --app=proj.celery:celery

> #### Calling Tasks
> You can call a task using the *delay()* method:
> 
    >>> add.delay(2, 2)
> This method is actually a star-argument shortcut to another method called *apply_async():
>
    >>> add.apply_async((2, 2))
> The latter enables you to sepcify execution options like the time to run (countdown), the queue it should be sent to and so on:
>
    >>> add.apply_async((2, 2), queue='lopri', countdown=10)
> In the above example the task will be sent to a queue named  *lopri* and the task will execute, at the earliest, 10 seconds after the message was sent.
> Applying the task directly will execute the task in the current process, so that no message is sent:
>
    >>> add(2, 2)
    4
> These three methods - *delay()*, *apply_async()*, and applying (*__call__*), represent the Celery calling API, which are also used for subtasks.
> Every task invocation will be given a unique identifier (an UUID), this is the task id.
> The *delay* and *apply_async* methods return an AsyncResult instance, which can be used to keep track of the tasks execution state. But for this you need to enable a *result backend* so that the state can be stored somewhere.
> Results are disabled by default because of the fact that there is no result backend that suits every application, so to choose one you need to consider the drawbacks of each individual backend. For many tasks keeping the return value isn't even very useful, so it's a sensible default to have. Also note taht result backeds are not used for monitoring tasks and workers, for that Celery uses dedicated event messages.
> If you have a result backend configured you can retrieve the return value of a task:
>
    >>> res = add.delay(2, 2)
    >>> res.get(timeout=1)
    4
> You can find the task's id by looking at the *id* attribute:
> 
    >>> res.id
    d6b3aea2-fb9b-4ebc-8da4-848818db9114
> You can also inspect the exception and traceback if the task raised an exception, in fact *result.get()* will propagate any errors by default:
>
    >>> res = add.delay(2)
    >>> res.get(timeout=1)
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/opt/devel/celery/celery/result.py", line 113, in get
        interval=interval)
    File "/opt/devel/celery/celery/backends/amqp.py", line 138, in wait_for
        raise self.exception_to_python(meta['result'])
    TypeError: add() takes exactly 2 arguments (1 given)
> If you don't wish for the errors to propagate then you can disable that by passing the *propagate* argument:
>
    >>> res.get(propagate=False)
    TypeError: add() takes exactly 2 arguments (1 given)
> In this case it will return the exception instance raised instead, and so to check whether the task succeeded to failed you will have to use the corresponding methods on the result instance:
>
    >>> res.failed()
    True
    >>> res.successful()
    False
> So how does it know if the task has failed or not? It can find out by looking at the tasks *stat*:
>
    >>> res.state
    'FAILURE'
> A task can only be in a single state, but it can progress through several states. The stages of a typical task can be:
>
    PENDING -> STARTED -> SUCCESS
> The stated state is a special state that is only recorded if the *CELERY_TRACK_STARTED* setting is enabled, or if the *@task(track_starte=True)* option is set for the task.
> The pending state is actually not a recorded state, but rather the default state for any task id that is unknown, which you can see from this example:
>
    >>> from proj.celery import celery
    >>> res = celery.AsyncResult('this-id-does-not-exist')
    >>> res.state
    'PENDING'
> If the task is retried the stages can become even more complex, e.g, for a task that is retried two times the stages would be:
>
    PENDING -> STARTED -> RETRY -> STARTED -> RETRY -> STARTED -> SUCCESS
> #### Canvas: Designing Workflows
> A subtask wraps the arguments and execution options of a single task invocation in a way such that it can be passed to funcions or even serialized and sent across the wire.
> You can create a subtask for the *add* task using the arguments *(2, 2)*, and a countdown of 10 seconds like this:
>
    >>> add.subtask((2, 2), countdown=10)
    tasks.add(2, 2)
> There is also a shortcut using star arguments:
    >>> add.s(2, 2)
    tasks.add(2, 2)
> #### And there's that calling API again...
> Subtask instances also supports the calling API, which means that they have the *delay* and *apply_async* methods.
> But there is a difference in that the subtask may already have an argument signature specified. The *add* task taskes tow arguments, so a subtask specifying two arguments would make a complete signature:
>
    >>> s1 = add.s(2, 2)
    >>> res = s1.delay()
    >>> res.get()
    4
> But, you can also make incomplete signatures to create what we call *partials*:
>
    # incomplete partial: add(?, 2)
    >>> s2 = add.s(2)
> s2 is now a partial subtask that needs another argument to be complete, and this can be resolved when calling the subtask:
>
    # resolves the partial: add(8, 2)
    >>> res = s2.delay(8)
    >>> res.get()
    10
> Here you added the argument 8, which was prepended to the existing argument 2 forming a complete signature of add(8, 2).
> Keyword arguments can also be added later, these are then merged with any existing keyword arguments, but with new arguments taksing precedence:
>
    >>> s3 = add.s(2, 2, debug=True)
    >>> s3.delay(debug=False) # debug is now False.
> As stated subtasks supports the calling API, which means that:
> + subtask.apply_sync(args=(), kwargs={}, **options)
> Calls the subtask with optional partial arguments and partial keyword arguments. Also supports partial execution options.
> + subtask.delay(*args, **kwargs)
> Star argument version of apply_async. Any arguments will be prepended to the arguments in the signature, and keyword arguments is merged with any existing keys.
> #### The Primitives
> ##### Groups
> A group calls a list of tasks in parallel, and it returns a special result instance that lets you inspect the results as a group, and retrieve the return values in order.
>
    >>> from celery import group
    >>> from proj.tasks import add
    >>> group(add.s(i, i) for i in xrange(10))().get()
    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
> Partial group
>
    >>> g = group(add.s(i) for i in xrange(10))
    >>> g(10).get()
    [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
> ##### Chains
> Tasks can be linked together so that after one task returns the other is called:
>
    >>> from celery import chain
    >>> from proj.tasks import add, mul
    # (4 + 4) * 8
    >>> chain(add.s(4, 4) | mul.s(8))().get()
    64
> or a partial chain:
>
    # (? + 4) * 8
    >>> g = chain(add.s(4) | mul.s(8))
    >>> g(4).get()
    64
> Chains can also be written like this:
>
    >>> (add.s(4, 4) | mul.s(8))().get()
> ##### Chords
> A chord is a group with a callback:
>
    >>> from celery import chord
    >>> from porj.tasks import add, xsum
    >>> chord((add.s(i, i) for i in xrange(10)), xsum.s())().get()
    90
> A group chained to another task will be automatically converted to a chord:
>
    >>> (group(add.s(i, i) for i in xrange(10)) | xsum.s())().get()
    90
> Since these primitives are all of the subtask type they can be combined almost however you want, e.g:
>
    >>> upload_document.s(file) | group(apply_filter.s() for filter in filters)

> ### Routing
> #### Basics
> ##### Automatic routing
> The simplest way to do routing is to use the *celery_create_missing_queues* setting(on by default).
> With this setting on, a named queue that is not already defined in *celey_queues* will be created automatically. This makes it easy to perform simple routing tasks.
> Say you have two servers, x, and y that handles regular tasks, and one server z, that only handles feed related tasks. You can use this configuration:
>   
   CELERY_ROUTES = {'feed.tasks.import_feed': {'queue': 'feeds'}}
> With this route enabled import feed tasks will be routed to the "feed" queue, while all other tasks will be routed to the default queue(named "celery" for historical reasons).
> Now you can start server z to only process the feeds queue like this:
>
    user@z:/$ celery worker -Q feeds
    
> You can specify as many queues as you want, so you can make this server process the default queue as well:
>
   user@z:/$ celery -Q feeds,celery
> ##### Changing the name of the default queue
> You can change the name of the default queue by using the following configuration:
>       
   from kombu import Exchange, Queue
   CELERY_DEFAULT_QUEUE = 'default'
   celery_queues = (
       Queue('default', Exchange('default'), routing_key='default'),
   )
> ##### How the queues are defined
> The point with this feature is to hide the complex AMQP protocol for users with only basic needs. However - you may still be interested in how these queues are declared.
> A queue named "video" will be created with the following settings:
>
   {'exchange': 'video', 
    'exchange_type': 'direct', 
    'routing_key': 'video'}
    
> The non-AMQP backends like ghttoq does not support exchanges, so they require the exchange to have the same as the queue. Using this design ensures it will work for them as well.
> #### Manual routing
> Say you have two servers, x, and y that handles regular tasks, and one server z, that only handles feed related tasks, you can use this configuration:
>   
    from kombu import Queue
    celery_default_queue = 'default'
    celery_queues = (
        Queue('default', routing_key='task.#'),
         Queue('feed_tasks', routing_key='feed.#'),
    )
    celery_default_exchange = 'tasks'
    celery_default_exchange_type = 'topic'
    celery_default_routing_key = 'task.default'
 
> *celery_queues* is a list of *Queue* instances. If you don't set the exchange or exchange type values for a key, these will be taken from the *celery_default_exchange* and *celery_default_exchange_type* settings.
> To route a task to the *feed_tasks* queue, you can add an entry in the *celery_routes* setting:
>
     celery_routes = {
         'feeds.tasks.import_feed': {
            'queue': 'feed_tasks',
             'routing_key': 'feed.import',
           },
     }
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
### Scrapy

> Item Loader
> 
> An Item Loader contains one input processor and one output processer for each item field. The input processor processes the extracted data as soon as it's reveived (through the *add_xpath()* or *add_value()*) and the result of the input processor is collected and kept inside the ItemLoader. After colecting all data, the *ItemLoader.load_item()* method is called to populate and get the populated *Item* Object. That's when the output processor is called with the data previously collected (and processed using the input processor). The result of the output processor is the final value that gets assigned to the item.
> 
> When use *add_value()* , the value to be collected is assigned directly, instead of being extracted from a XPath. However, the calue is strill passed through the input processors. In this case, since the value is not iterable it is converted to an iterable of a single element before passing it to the input processor, because input processor always reveive iteravles. 

>It's worth noticing that processors are just callable objects, which are called with the data to be parsed, and return a parsed value. So you can use any function as input or output processor. The only requirement is that they must accept one (and only one) positional argument, which will be an iterator.
>
>Bot input and output processors must reveive an iterator as their first argument. The output of those functions can be anything. The result of input processors will be appended to an iternal list (in the loader) containing the collected values (for that field). The result of the output processors is the value that will be finally assigned to the item.
>The values returned by input processors are collected internally (in list) ant the passed to output processors to pupulate the fields.

