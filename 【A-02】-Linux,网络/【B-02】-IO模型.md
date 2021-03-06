# IO模型

![](photo\IO模型.png)



> 什么是IO模型?

所谓IO模型指的是出现IO等待时，进程的状态以及处理数据的方式

<img src="photo\IO过程.webp" style="zoom: 80%;" />

> 五种IO模型

- Blocking I/O

![](photo\阻塞IO.webp)

- Non-Blocking I/O

![](photo\非阻塞IO.webp)

- IO多路复用

![](photo\IO多路复用.webp)

- 信号驱动I/O

![](photo\信号驱动IO.webp)

- 异步I/O

![](photo\异步Io.webp)

> 同步IO与异步IO,阻塞与非阻塞之分

阻塞、非阻塞、信号驱动、IO多路复用都是同步IO模型，只有异步IO模型也是异步的。

其中首先清楚两个概念，数据准备阶段：数据从磁盘或其他硬件复制到Kernel buffer的过程; 数据复制阶段：数据从Kernel buffer复制到app buffer的过程。阻塞也就是体现在数据复制阶段的，这一阶段必须要有cpu的参与，进程会被阻塞，而异步Io会一直等待直到数据在appbuffer复制完成，数据复制阶段也是不阻塞的。

看上去异步IO很好，但是数据复制阶段肯定需要cpu的参与，因此应用进程需要和数据复制阶段争抢cpu，并发量高的情况下，连接数会越多，cpu争抢情况就越严重，异步函数返回成功的信号就越慢，如果不能很好处理这个问题，异步IO模型也不一定就好。

> select /poll /epoll

![](photo\select过程.webp)

(1).首先通过FD_ZERO宏函数初始化描述符集合。图中每个小方格表示一个文件描述符。
(2).通过FD_SET宏函数创建描述符集合，此时集合中的文件描述符都被打开，也就是稍后要被select()监控的对象。
(3).使用select()函数监控描述符集合。当某个文件描述符满足就绪条件时，select()函数返回集合中满足条件的数量。图中标黄色的小方块表示满足就绪条件的描述符。
(4).通过FD_ISSET宏函数遍历整个描述符集合，并将满足就绪条件的描述符发送给进程。同时，使用FD_CLR宏函数将满足就绪条件的描述符从集合中移除。
(5).进入下一个循环，继续使用FD_SET宏函数向描述符集合中添加新的待监控描述符。然后重复(3)、(4)两个步骤。

如果使用简单的伪代码来描述：

```
FD_ZEROfor() {    FD_SET()    select()    if(){        FD_ISSET()        FD_CLR()
    }    writen()
}
```

以上所说只是一种需要循环监控的示例，具体如何做却是不一定的。不过从中也能看出这一系列的流程。

**epoll**比poll()、select()先进，考虑以下几点，自然能看出它的优势所在：

(1).epoll_create()创建的epoll实例可以随时通过epoll_ctl()来新增和删除感兴趣的文件描述符，不用再和select()每个循环后都要使用FD_SET更新描述符集合的数据结构。
(2).在epoll_create()创建epoll实例时，还创建了一个epoll就绪链表list。而epoll_ctl()每次向epoll实例添加描述符时，还会注册该描述符的回调函数。当epoll实例中的描述符满足就绪条件时将触发回调函数，被移入到就绪链表list中。
(3).当调用epoll_wait()进行监控时，它只需确定就绪链表中是否有数据即可，如果有，将复制到用户空间以被进程处理，如果没有，它将被阻塞。当然，如果监控的对象设置为非阻塞模式，它将不会被阻塞，而是不断地去检查。

也就是说，epoll的处理方式中，根本就无需遍历描述符集合。