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









