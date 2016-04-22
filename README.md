# 1.  通用概念

例子："我"烧水(注意：这里的主语是"我"，也就是我们讨论的who)
## 1.1. 同步异步
### 1.1.1.   例子
关注点：是否需要"我主动"去看水烧开没(注意，"我"和"主动"这两个词)

- 同步：**"我"要主动**看水烧开没
- 异步：**不需要"我"主动**去看水烧开，水壶响了通知"我"

### 1.1.2.   概念
抽象概念：事情执行者对事情结果的获取机制（who：事情执行者）

- 同步：事情执行者**主动**获取事情结果
- 异步：事情执行者**被动**获取事情结果

## 1.2. 阻塞非阻塞
### 1.2.1.   例子
关注点：水烧开前，"我"能不能去做其他事情

- 阻塞：
  1. 水没烧开，我一直等着（全程阻塞）
  2. 我去干其他事情了，但是隔一会儿来看一次水烧开没（来看的时候阻塞）
- 非阻塞：
  水没烧开，"我"去拖地了,拖完地又看了会儿电视

### 1.2.2.   概念
抽象概念：事情执行者得到事情结果前能不能去做其他事情（who：**事情执行者**）

- 阻塞：事情执行者得到结果前不能去做其他事情
- 非阻塞：事情执行者得到结果前可以去做其他事情

## 1.3. 总结
1. 同步一定是会发生阻塞的，只是阻塞的时间长短问题。
   - 我主动看水烧开（同步），水烧开前我一直等着（阻塞）：全程阻塞
   - 我主动看水烧开（同步），隔一会儿来看水烧开（看的过程是阻塞）：局部阻塞
2. 异步一定是非阻塞的：
   - 不需要我主动去看水烧开（异步），烧水过程中我可以去做任何事情（非阻塞）。
   
# 2.  网络编程层面

网络编程层面主要涉及的就是IO，所以，这里主要讲IO。这块儿其实在《UNIX网络编程卷1 第6章 IO复用》中讲的非常清楚了，建议大家看一下，我引用其中的一些关键点。具体的可以找书去看。

## 2.1. POSIX定义

- 同步I/O操作：导致请求进程阻塞，直到I/O操作完成；
- 异步I/O操作：不导致请求进程阻塞。

## 2.2. IO模型

**根据POXIX的定义，下面图中的五种IO模型，其实前四种都是同步IO。**
[![](http://img.blog.csdn.net/20160422115157700)](http://img.blog.csdn.net/20160422115157700)
[![](http://img.blog.csdn.net/20160422115206497)](http://img.blog.csdn.net/20160422115206497)

## 2.3. 总结

**注意：这里的who是进程**

### 2.3.1.   同步IO

1. 进程**主动**查看关心的IO事件是否就绪，就绪时再执行相关操作
   - 注意：进程的阻塞发生在select/epoll/poll等待事件时
2. 进程**主动**执行读写
   - 注意：进程的阻塞发生在数据未到达（设备未收到数据），或者未就绪（内核空间和用户空间的数据copy未完成）
   
### 2.3.2.   异步IO
异步IO是需要底层操作系统内核做支持的，比如linux的AIO， windows的IOCP。
 
1. 进程**不用主动**去查看关心的IO事件，只需要把关心的IO事件、数据buffer以及要绑定的回调函数或者signal直接告诉内核，内核发现IO时间就绪时，直接调用回调函数或者发送绑定的signal
   - 注意：IO事件相关的数据，由内核直接将数据copy到用户空间（异步读），或者从用户空间copy至内核空间（异步写）
2. 进程**被动**执行数据操作
   - 注意：进程收到内核通知后，在1中绑定的数据buffer里就已经有数据了，进程直接对数据进行相关处理即可。
