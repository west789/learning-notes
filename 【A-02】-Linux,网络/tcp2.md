# TCP2

## 重传机制

> TCP针对数据包丢失的情况，会用重传机制解决

- 超时重传
- 快速重传
- SACK
- D-SACK

### 超时重传

在发送数据时，设定一个定时器，没有收到对方的ACK确认应答报文，就会重发该数据，也就是常说的超时重传

- 数据包丢失
- ACK丢失

![](.\photo\646.webp)

> 超时重传的时间应该设置多少?

RTT(Round-Trip Time 往返时延)

![](.\photo\往返时延.webp)

超时重传是以**RTO**表示

> RTO较短或较长会有什么问题?

- RTO较短: 包没有丢失就重发了包，造成包重复，增加网络拥塞，浪费资源
- RTO较长：重发慢，丢了好长才发，没有效率，性能差

根据以上得知: RTO的值应该略大于报文往返RTT的值

RTO是一个动态变化的值

**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设置为先前置的两倍。两次超时，说明网络环境差，不宜频繁重发**

### 快速重传

不以时间为驱动，以数据为驱动

三次同样的ACK，就会触发快速重传机制

### SACK

选择性确认

在TCP头部的选项字段加一个SACK的标识，可以将缓存地图发送给发送方，这样发送方就知道重发哪些数据

### D-SACK

使用SACK来告诉发送方有哪些数据被重复接收了

- ACK丢包
- 网络延时

## 滑动窗口

为每个包确认应答的缺点，包的往返时延越长，通信效率就越低

TCP引入窗口的概念，窗口大小指的是:无需确认应答，就可以发送数据的最大值

> 窗口大小由哪一方决定

由接收方决定，发送方发送的数据不能超过窗口大小，否则接收方就无法正常收到数据

> 发送方的滑动窗口

![](.\photo\发送方滑动窗口.png)



- #1 已发送并且已经收到ACK确认的数据
- #2 已发送还未收到ACK确认的数据
- #3 未发送但是可以发送的数据
- #4 未发送不可以发送的数据

> 程序是如何表示这四个部分的

![](.\photo\程序表示发送滑动窗口.png)

- SND.WND: 表示发送窗口的大小，由接收方决定

- SND.UNA: 是一个绝对指针，指向已发送还未确认的第一个字节，#2第一个字节

- SND.NXT:是一个绝对指针,指向未发送可发送的第一个字节，#3第一个字节

- 指向#4 的第一个字节是一个相对指针，需要SND.UNA指针加上SND.WND大小的偏移量

  可用窗口的大小:

  **可用窗口大小=SND.WND-(SND.NXT-SND.UNA)**

> 接收方的滑动窗口

![](.\photo\接收方滑动窗口.png)

分为三个部分：

- #1+#2，已成功接收并已确认
- #3，未收到数据可以接收的
- #4，未收到数据不可以接收的

RCV.WND指的是接收窗口大小，会通告给发送方

RCV.NXT指向#3的第一个字节，期望从发送方发送过来的下一个字节的学历噩耗

> 接收窗口和发送窗口的大小是相等的吗

并不是完全相等，接收窗口的大小约等于发送窗口的大小。因为中间传送窗口的大小存在时延，所以是约等于。

## 流量控制

TCP提供了一种机制让发送方根据接收方的实际处理能力来控制发送的数据量，这就是流量控制

### 操作系统缓冲区与滑动窗口的关系

发送窗口和接收窗口中存放的字节数，都是放在操作系统内存缓冲区中的，而操作系统的缓冲区会被操作系统调整。

### 窗口关闭

如果窗口大小为0，就会阻止发送方给接收方传递数据，直到窗口变为非0为止，这就是窗口关闭

> 窗口关闭潜在危险

接收方向发送方通告窗口大小，是通过ACK报文来通告的

<img src=".\photo\窗口关闭潜在危险.png" style="zoom: 67%;" />

> TCP是如何解决窗口关闭，潜在死锁现象

只要TCP连接一方收到零窗口通知，就启动持续计时器

如果持续计时器超时，就会发送窗口探测报文，而对方在确认这个探测报文时，给出自己的窗口大小。

- 如果窗口大小仍然为0，则收到报文的一方就重新启动计时器
- 如果非0，则打破死锁局面，可以继续发送数据

探测次数一般为3次，每次30-60s，如果3次过后仍然是0，则有的会发送RST中断连接

### 糊涂窗口综合症

如果接收方腾出几个字节并告诉发送方现在有几个字节，发送方义无反顾发送这几个字节，这就是糊涂窗口综合症

解决糊涂窗口综合症现象分别从发送方和接收方解决问题：

- 让接收方不通告小窗口给发送方
- 让发送方不发送小数据

> 怎么让接收方不通告小窗口

当窗口大小小于 min(MSS. 缓存空间/2)，就会向发送方通告窗口为0，阻止了发送方再发数据过来

> 怎么让发送方避免发送小数据

使用Nagle算法，满足以下条件任意一条才可以发送数据:

- 窗口大小>=MSS 或是数据大小>=MSS
- 收到之前发送数据的ack回包

Nagle算法是默认打开的，对于一些需要小数据包交互的场景的程序，比如telnet,ssh这样交互性比较强的程序，需要关闭Nagle算法

可以在Socket设置TCP_NODELAY选项来关闭这个算法

## 拥塞控制

### 为什么要有拥塞控制

拥塞控制的目的是避免发送方的数据填满整个网络

流量控制是避免发送方的数据填满接收方的缓存

网络出现拥堵的时候，如果继续发送大量的数据包，可能会导致数据包时延丢失等， 这是TCP就会重传数据，但是一重传就会导致网络的负担更重，于是就会导致更严重的时延和丢包，造成恶性循环

定义了一个**拥塞窗口**的概念

> 什么是拥塞窗口？和发送窗口什么关系

拥塞窗口(**CWND**)是发送方维护的一个状态变量，根据网络的拥塞程度动态变化

swnd = min(cwnd, rwnd)

变化规则是：

- 只要网络中没有出现拥塞，cwnd就会增大
- 出现拥塞，cwnd就会减小

> 怎么知道当前网络是否出现了拥塞呢

只要发送方没有在规定时间内收到ASK应答报文，也就是发生了超时重传，就会认为网络出现了拥塞

> 拥塞控制有哪些控制算法

- 慢启动
- 拥塞 避免
- 拥塞发生
- 快速恢复

### 慢启动

TCP刚建立连接时，首先有个慢启动的过程，慢启动的意思就是一点点提高发送数据包的数量。这个算法的一个规则：当发送方每收到一个ACK，拥塞窗口的大小cwnd就会加1.

![](.\photo\慢启动.png)

可以看出，慢启动的算法，发包的个数是**指数性增长**

> 慢启动涨到什么时候是个头呢

有一个叫**慢启动门限**(ssthresh)的状态变量

- 当 cwnd < ssthresh 时，使用慢启动算法
- 当cwnd >= ssthresh时，就会使用拥塞避免算法

### 拥塞避免算法

条件: cwnd >= ssthresh

一般来说，ssthresh大小是**65536**字节

进入拥塞避免算法后，规则是: **每收到一个ACK时，cwnd增加1/cwnd**

![](.\photo\拥塞避免算法.png)

原本的慢启动算法的指数增长变成了线性增长

一直增长着，网络就会慢慢进入了拥塞状况，于是就会出现丢包现象，这是就需要对丢失的数据包进行重传。

当触发了重传机制，也就进入了**拥塞发生算法**

### 拥塞发生

> 发生超时重传的拥塞发生算法

- ssthresh 设为cwnd/2
- cwnd重置为1

![](.\photo\拥塞发生-超时重传.png)