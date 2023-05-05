# java IO

已经建立在linux io模型的基础上

## java nio

主要基于linux的epoll实现

看这篇文章就够了，注意最后的那张图：[从jdk的nio到epoll源码与实现内幕全面解析 | HeapDump性能社区](https://heapdump.cn/article/2769345)



### 大致流程

- socket与事件绑定
- 内核中的eventpoll维护一棵红黑树，保存了所有文件描述符
- 中断到来，查询红黑树，回调函数将fd写到eventpoll的就绪队列中
- 唤醒epoll_wait，将事件拷贝到用户区的pollArray数组中



