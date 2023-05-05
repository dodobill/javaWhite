# Netty IO

**主要解决问题**

已经存在java io为什么还设计了netty，即解决了java io的哪些问题？



看这篇就够了：[(九)Java网络编程无冕之王-这回把大名鼎鼎的Netty框架一网打尽！ - 掘金 (juejin.cn)](https://juejin.cn/post/7176869085521641509)



**思维导图**

![v2-bd1aade8739efcfd403d31dc037da0dd_720w](D:\workstation\就业方向\实习\java八股\javaWhite\javaIO系列\pic\v2-bd1aade8739efcfd403d31dc037da0dd_720w.png)



## 概述

两个关键词：

- 异步
- 事件驱动，类java nio



## 为什么使用netty

NIO的主要问题是：

- NIO的类库和API繁杂，学习成本高，你需要熟练掌握Selector、ServerSocketChannel、SocketChannel、ByteBuffer等。
- 需要熟悉Java多线程编程。这是因为NIO编程涉及到Reactor模式，你必须对多线程和网络编程非常熟悉，才能写出高质量的NIO程序。
- 臭名昭著的epoll bug。它会导致Selector空轮询，最终导致CPU 100%。直到JDK1.7版本依然没得到根本性的解决。



### epoll bug

- 由于socket出现中断，epoll触发特殊事件POLLHUP和POLLERR

- select由于特殊事件唤醒
- select 不对特殊事件进行处理
- 不断空转导致cpu 100%



## 架构图



![v2-8552db7ceabc450d9e0eb8db689155d6_720w](D:\workstation\就业方向\实习\java八股\javaWhite\javaIO系列\pic\v2-8552db7ceabc450d9e0eb8db689155d6_720w.webp)



## 核心组件

### 启动器

类似于Channel和socket的概念，上表格：

![屏幕截图 2023-05-04 165544](D:\workstation\就业方向\实习\java八股\javaWhite\javaIO系列\pic\屏幕截图 2023-05-04 165544.png)



### 事件组

EventLoopgroup和EventLoop

对Selector的封装

**继承了ScheduledExecutorService，周期执行任务**



#### EventLoopGroup

类似于一个线程池；



#### EventLoop

EventLoopGroup中的一个线程负责一个EventLoop；

一个EventLoop维护一个selector，托管多个连接。



#### 小结

从这里可以看出，如果使用Java原生NIO，很难自己开发该性能的server，比如：

- 多线程处理一个selector的事件
- 不同线程处理不同事件，连接和io分开

但是netty已经实现了以上的功能



## Netty中的增强通道

主要是支持了异步。

通过回调函数的方式。

**问题？为什么异步会提升性能，举个例子：**



### ChannelFuture，Netty-Future，JDK-Future之间的关系

channelFuture继承自netty-future；

netty-future继承自jdk-future



## 通道处理器Handler

- 入站处理器
- 出站处理器

### 入站处理器和出站处理器

...



### pipline处理器链表

一个处理器称为handler；

handler添加到通道上后称为channelHandler；

所有channelHandler串起来形成channelPipeline；



**pipline是一个双向链表**



## Netty重构后的缓冲区ByteBuf

- 可以使用堆内存创建ByteBuf
- 可以使用本地内存创建ByteBuf



os不能直接操作堆，因为GC会移动对象；

因此os对堆的读写都需要经过直接内存跳转；



### 使用堆外内存的优点

- 减少垃圾回收的工作量
- 减少堆到内存的拷贝开销



### 使用堆外内存的注意事项

为了避免堆外内存耗尽物理空间，需要指定

-XX:MaxDirectMemorySize。

当到达阈值时进行full gc，回收堆外内存。



### Netty对Buffer做了哪些增强

- 池化技术
- 动态扩容技术
- 零拷贝



#### 缓存池化技术

**应用场景：**Netty默认使用本地内存，由于本地内存不是os分配给jvm的。因此需要向os额外申请，开销自然大。

如果频繁创建销毁本地内存，开销很大。

另一方面，如果没有及时释放，可以发生内存溢出。



**因此netty使用池化技术**

注：Android默认采用非池化，而其它系统：Linux，Mac，Windows默认开启。



**最好通过以下参数控制**

- `-Dio.netty.allocator.type=unpooled`：关闭池化技术
- `-Dio.netty.allocator.type=pooled`：开启池化技术



#### ByteBuf动态扩容机制

**应用场景：**java nio的使用复杂

- 创建缓冲区
- put往缓冲区中写
- flip()转为读模式
- get从缓冲区中读
- clear, compact清空缓冲区



**netty 动态缓冲区的设计**

- 初始容量
- 最大容量
- readerIndex：读取指针，读取一部分数据，指针移动
- writerIndex：写指针，写一部分，指针移动



#### Netty 读写API

**应用场景：**

noi-buffer设计不合理：如果要往缓冲区中写入不同类型的数据：

- 手动转换成byte
- new 一个新的子实现



**Netty的做法 **

nio为不同类型实现不同的类；

netty为不同的类型实现不同的方法



#### ByteBuf的内存回收

内存什么时候还给内存池。

netty-bytebuf恰恰使用了引用计数法。不怕循环引用？



**一般来说：**

netty 中的ByteBuf会在ChannelPipeline的head和tail中释放；

但是如果在传递的过程中，将bytebuf转换成了其他没有实现==ReferenceCounted==接口的类，该方法会失效；

因此，在确定不需要bytebuf的时候，一定要手动release。





## 零拷贝

零拷贝不是不需要经过拷贝，而是减少拷贝的次数



### linux实现零拷贝的方法

- MMAP共享内存+write系统函数
- sendfile
- 硬件支持
- splice

具体实现请参考该博客



### Netty中零拷贝的实现

体现

- 使用了堆外内存
- 文件传输，底层调用sendfile
- 程序级的零拷贝
  - 拆解bytebuf
  - 多个bytebuf合并



#### 拆分缓冲区

拆分后的使用原来的缓冲区

需要手动增加引用计数

不支持扩容



**duplicate的零拷贝**

完全克隆，读写指针独立，支持自动扩容



#### 合并缓冲区

