# SpringBoot集成Jedis框架-实现Redis调用

### Redis原理介绍

#### 1、Redis 性能为什么那么快？
##### Redis Epoll原理
Redis是一个单线程却性能非常好的内存数据库，主要用来作为缓存系统。Redis采用网络IO多路复用技术，保证了多个连接的时系统依然有吞吐量表现。

**Redis选择I/O多路复用的原因？**
首先，Redis是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是由于读写操作等待用户输入或者输出都是阻塞的，所以I/O操作往往不能直接返回，这会导致某一个文件的I/O阻塞导致整个进程无法对其它客户端提供服务，而I/O多路复用就是为了解决这个问题。

Redis的IO模型主要基于Epoll实现的，不过它还提供了Select和Kqueue的实现，默认采用Epoll模型
Epoll模型属于诸多IO多路服用模型中的一种，但是相比其他IO多路复用模型技术（Select、Poll等）

**Epoll有诸多优点：**
> - Epoll没有最大并发限制，上线是系统最大文件的数目，具体数目可以 cat /proc/sys/fs/file-max 察看；
> - 效率提升，Epoll最大的优点就是它只管“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于Select和Poll；
> - 内存拷贝，Epoll在这点上使用了“共享内存”，这个内存拷贝也省略了；

**Epoll与Select/Poll的区别：**
> - Select、Poll、Epoll都是IO多路复用机制。I/O多路复用就是一种机制，可以监视多个描述符，一旦某个描述符就绪，能够通知程序进行相应的操作；
> - Select的本质是采用32个整数的32位，即32*32=1024来标识，fd的值为1～1024。当fd的值超过1024限制时，就必须修改FD_SETSIZE的大小。这个时候可以标识32*max值范围的fd；
> - poll和select不同，通过一个pollfd数组向内核传递关注的事件，故没有描述符个数的限制，pollfd中的events字段和revents分别用于标示关注的事件和发生的事件，故pollfd数组只需要被初始化一次；
> - Epoll还是Poll的一种优化，返回后不需要对所有的fd进行遍历，在内核中维持了fd的列表。Select和Poll是将内核列表维护在用户态，然后传递到内核态中。与Poll/Select不同，Epoll不再是一个单独的系统调用，而是由epoll_create/epoll_ctl/epoll_wait三个系统调用组成，Epoll在Linux2.6以后内核才支持；

