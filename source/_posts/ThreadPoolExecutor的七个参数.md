title: ThreadPoolExecutor的七个参数
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---
## ThreadPoolExecutor的七个参数
corePoolSize：
线程池的基本大小，即在没有任务需要执行的时候线程池的大小，并且只有在工作队列满了的情况下才会创建超出这个数量的线程。
maximumPoolSize：
线程池中允许的最大线程数，线程池中的当前线程数目不会超过该值。如果队列中任务已满，并且当前线程个数小于maximumPoolSize，那么会创建新的线程来执行任务。
keepAliveTime
当线程空闲时间达到keepAliveTime，该线程会退出，直到线程数量等于corePoolSize。如果allowCoreThreadTimeout设置为true，则所有线程均会退出直到线程数量为0。
workQueue
ArrayBlockingQueue、LinkedBlockingQueue
threadFactory
线程工厂，自定义线程名，指定线程优先级
handler
拒绝策略
有4中拒绝策略
1，Abort:抛异常
2，Discard:扔掉，不抛异常
3，DiscardOldest：扔掉排队时间最久的
4，CallerRuns：调用者处理服务

## 线程池执行步骤
当线程数小于核心线程数时，创建线程。
当线程数大于等于核心线程数，且任务队列未满时，将任务放入任务队列。
当线程数大于等于核心线程数，且任务队列已满
若线程数小于最大线程数，创建线程
若线程数等于最大线程数，抛出异常，拒绝任务

## 具体方法

```java
 public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)

```

## 线程池设置大小建议
N(threads) = N(cpu)*U(cpu,期望利用率) * (1+w/c)