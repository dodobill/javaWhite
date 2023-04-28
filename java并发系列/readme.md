# 大纲

1. cpu层面的多线程
2. 操作系统层面的多线程
3. java 层面的多线程

主要解决，jvm如何依靠操作系统提供的方法实现并发





## CPU如何处理多线程







## java中Object

### wait

link：[Java的wait()、notify()学习三部曲之一：JVM源码分析-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/891845)

基于cas操作，通过调用park方法和unpark方法

park方法和unpark方法基于系统调用：

- pthread_cond_wait
- pthread_cond_signal



### wait怎样与linux 的系统调用结合起来





### Synchronized锁膨胀的原理



synchronized支持编译器的优化：

- 锁消除

将无竞争的代码移除到同步代码块外

- 锁粗化

如果检测到连续加锁，则扩大同步代码块的范围



偏向锁--轻量级锁--自旋锁--重量级锁

参考：[深入分析synchronized原理和锁膨胀过程(二) - 掘金 (juejin.cn)](https://juejin.cn/post/6844903805121740814#heading-16)

[从jvm源码看synchronized - unbelievableme - 博客园 (cnblogs.com)](https://www.cnblogs.com/kundeg/p/8422557.html)



![169a9246f01f2cb6_tplv-t2oaga2asx-zoom-in-crop-mark_3024_0_0_0](D:\workstation\就业方向\实习\java八股\javaWhite\java并发系列\pic\169a9246f01f2cb6_tplv-t2oaga2asx-zoom-in-crop-mark_3024_0_0_0.jpg)





### 重量级锁的释放以及其他线程的唤醒

![1053518-20180206155148216-1870687856](D:\workstation\就业方向\实习\java八股\javaWhite\java并发系列\pic\1053518-20180206155148216-1870687856.png)

- 根据QMode从cxq中取目标加入到EntryList中去
- 从EntryList中取出目标线程进行唤醒



## AQS

link:[java - AQS源码分析看这一篇就够了 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000022918705)



### AQS 队列主要搞清楚几种状态的作用



- CANCELLED

线程因为特殊情况退出阻塞，此时应该退出锁的竞争，于是再次将状态为置成取消

队列中设置成取消的节点会被处理掉

- SIGNAL

队列中的线程在自旋的过程中必须要找到前面有一个Signal状态的线程才能安心睡眠；

节点前方找不到一个signal状态的线程，则当前线程睡眠后可能无法被唤醒。

- CONDITION
- PROPAGATE

共享锁用到的标志位



### AQS框架图

![82077ccf14127a87b77cefd1ccf562d3253591](D:\workstation\就业方向\实习\java八股\javaWhite\java并发系列\pic\82077ccf14127a87b77cefd1ccf562d3253591.png)







## ReentrantLock

### ReentrantLock如何与系统调用结合起来

依赖LockSupport

- pthread_mutex_lock
- pthread_mutex_unlock

- pthread_cond_wait
- pthread_cond_signal



执行park操作

- 获取mutex
- 执行wait，释放mutex
- 阻塞



执行unpark

- 获取mutex
- 执行signal，唤醒阻塞的线程
- 释放mutex





## AQS的锁方案与Synchronized的区别

### 中断与不可中断

底层都是基于*parkEvent*的*park*和*unpark*实现

AQS与synchronized的不同在于

interrupt方法修改了标记位并唤醒相应的线程：

synchronized忽略标记位，继续尝试获取锁，失败后继续阻塞；

AQS的可中断方案，唤醒后判断标记位，若是因为中断唤醒的，则抛出InterruptException



### 共享锁与非共享锁

基于synchronized的方案每次只允许一个线程访问同步代码块

基于AQS的共享锁允许多个读进程访问共享资源

**实现共享锁，直接读行不行？不行，读时不允许排他锁的写**

#### 有哪些共享锁

- Semaphore
- CountdownLatch
- 可重入读写锁中的读锁



#### CountDownLatch实现原理

主要考究countDown()和await（）

**想问两个问题？**

- countDown时，若state已经为0？

直接返回false

- countDownLatch是一次性的吗

是一次性的，state只在初始化时设置一次



**countDown** 

- 自己实现的tryReleaseShare
  - state--
  - 判断是否需要唤醒
- 若需要唤醒，则执行AQS的doReleaseShare
  - 取队头的进行唤醒



**await**

- 通过state判断是否能获取锁
- 拿不到锁进入等待队列
  - 不断尝试
  - 拿到锁后，以传播的方式唤醒后续的线程==这点与排他锁不一样==



#### 读写锁

*看这篇就够了*：[AQS系列（三）- ReentrantReadWriteLock读写锁的加锁 - 淡墨痕 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zzq6032010/p/12037854.html)

一个state表示两种锁的状态

高16位和低16位表示读写锁的状态



**读锁获取**

- 判断有没有排他锁
- 判断应不应该阻塞
  - 不应该阻塞，则直接获取锁
  - 应该阻塞：公平锁判断空闲同步队列；防止写线程饿死
- 若应该阻塞
  - 如果该线程之前已经拿到过读锁了，则还有机会竞争
  - 如果该线程之是第一次尝试拿取读锁，则失败

以上是try的逻辑

try失败后，加入队列进入队列同步阶段



### 公平锁和非公平锁

**主要关注公平锁与非公平锁的区别**

公平锁多了一个hasQueuedPredecessors()方法判断是否有线程在等待





### 多条件队列

看这篇文章就够了

[Condition 从使用到实现原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97292945)

- 创建node加入到条件队列==注意不是同步队列==

- 只有signal 才能见到条件队列中的节点

- 加入到节点中后，挂起
  - 被signal唤醒，已经被加入到同步队列中了，退出循环，调用acquireQueue尝试获取锁，和获取锁的流程一样
  - 被中断唤醒，不获取锁而是抛出中断异常

- 获取锁后继续执行









