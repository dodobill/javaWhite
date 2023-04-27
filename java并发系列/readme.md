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

- 中断与不可中断

底层都是基于*parkEvent*的*park*和*unpark*实现

AQS与synchronized的不同在于

interrupt方法修改了标记位并唤醒相应的线程：

synchronized忽略标记位，继续尝试获取锁，失败后继续阻塞；

AQS的可中断方案，唤醒后判断标记位，若是因为中断唤醒的，则抛出InterruptException



- 共享锁与非共享锁



- 公平锁和非公平锁





- 多条件队列



