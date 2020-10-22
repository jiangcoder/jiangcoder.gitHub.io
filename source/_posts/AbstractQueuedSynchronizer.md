title: AbstractQueuedSynchronizer
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---
## 一，AQS如何设置链表尾巴
compareAndGetTail(oldTail, node);
一般来说锁整个链表，改为只锁尾巴，通过cas操作，不需要把原来的整个链表上锁
## 二，CLH锁
AQS中的等待队列：是一个双向链表，并使用了“CLH锁”的思想实现等待队列
（1）CLH锁是一个自旋锁，能确保无饥饿性，提供先来先服务的公平性。

（2）CLH锁也是一种基于链表的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。

（3）在AQS中，等待队列中的每个节点也是类似的去轮训前继节点的状态。不过为了减少资源浪费，会加入线程waitting的优化。当然，这不是唯一的获取资源的方式，如果头部节点释放资源后，也会主动的去唤醒waitting状态的后继节点
前言
谈到并发，我们不得不说AQS(AbstractQueuedSynchronizer)，所谓的AQS即是抽象的队列式的同步器，内部定义了很多锁相关的方法，我们熟知的ReentrantLock、ReentrantReadWriteLock、CountDownLatch、Semaphore等都是基于AQS来实现的。
## 三，AQS实现原理
AQS中 维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。

这里volatile能够保证多线程下的可见性，当state=1则代表当前对象锁已经被占有，其他线程来加锁时则会失败，加锁失败的线程会被放入一个FIFO的等待队列中，比列会被UNSAFE.park()操作挂起，等待其他获取锁的线程释放锁才能够被唤醒。

另外state的操作都是通过CAS来保证其并发修改的安全性。

## 四，参考URL
https://blog.csdn.net/weixin_43314519/article/details/108160297