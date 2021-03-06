epoll
---

#### epoll是什么？
epoll是linux内核提供的一种高效的I/O多路复用机制，可以在一个进程内监听多个文件描述符的状态。
它高效的原因在于当进程监听的fd就绪时（可读或可写），进程会被主动唤醒，没有任何fd就绪时，进程
会睡眠，让出CPU。相比于select忙轮询所有监听的fd，在fd不是及其活跃的情况下，效率会更高。

#### epoll的原理
epoll提供epoll_create,epoll_ctl和epoll_wait三个函数。他们的作用分别为：
- create: 创建一个epoll实例
- ctl: 将要监听的一个fd注册到epoll实例
- wait: 等待监听的fd就绪

epoll实例用一个红黑树保存所有监听的fd，用一个ready list（双链表）保存就绪的fd。wait被调用
时，epoll实例会去查看有没有就绪的fd，如果没有，进程会把自己放到调度器的wait queue里，也就是
睡眠了。当有监听的fd就绪时，fd会通过一个callback函数把进程重新置回到runnable queue里，也就
是常说的唤醒。同时fd还会把自己放到epoll的ready list里，这样当进程唤醒后，就可以从ready list
里把fd取出来，做后续操作。

所以epoll并不是去轮询所以监听的fd的，因为那样会把cpu cycle浪费在轮询很多并没有就绪的fd上。
epoll采用的是一种异步阻塞（？这个存疑，还得再想一下）的方式，没有就绪的fd，就让出CPU；出现
就绪的fd，会主动唤醒睡眠的进程。

#### Level-trigger 和 Edge-trigger
- Level-trigger:
    只要监听的fd处在就绪状态，就会不断通知（唤醒）线程进行处理。这里问题是如果fd就绪了，但是
    其他资源并没有就绪，那么不断的通知就是一个消耗cpu cycle的事情。
- Edge-trigger:
    只在监听的fd发生状态改变（即非就绪->就绪）时，才会通知线程处理。这里通知变得更高效了，但
    是会造成问题，例如一个fd读就绪了，通知线程去读取数据，但是线程只读了部分数据，然后会发生
    什么呢？线程可能再也不会得到通知了，因为buffer里的数据没有被读完，所以仍然是就绪的状态，
    没有发生状态改变，即使buffer被写入更多的数据，仍然还是就绪的状态，不会通知线程。因此提高
    效率的代价就是编程者需要更小心的进行编程，保证不出现上述状态。
    
TODO: epoll还有一些其他的点，例如对内存的优化，edge-trigger如何正确编程等，留待以后学习...