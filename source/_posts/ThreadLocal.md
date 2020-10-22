title: ThreadLocal
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---ThreadLocal

## 一，ThreadLocal简介
ThreadLocalMap是Thread中的一个属性，
ThreadLocal.ThreadLocalMap
ThreadLocal不是为了解决多线程访问共享变量，而是为每个线程创建一个单独的变量副本，
提供了保持对象的方法和避免参数传递的复杂性。

## 二，ThreadLocal内存泄漏问题
每个thread中都存在一个map, map的类型是ThreadLocal.ThreadLocalMap. 
Map中的key为一个threadlocal实例. 这个Map的确使用了弱引用,不过弱引用只是针对key. 
每个key都弱引用指向threadlocal. 
当把threadlocal实例置为null以后,没有任何强引用指向threadlocal实例,
所以threadlocal将会被gc回收. 但是,我们的value却不能回收,
因为存在一条从current thread连接过来的强引用. 只有当前thread结束以后, 
current thread就不会存在栈中,强引用断开, Current Thread, Map, value将全部被GC回收。
所以得出一个结论就是只要这个线程对象被gc回收，就不会出现内存泄露，
但在threadLocal设为null和线程结束这段时间不会被回收的，就发生了我们认为的内存泄露。
其实这是一个对概念理解的不一致，也没什么好争论的。最要命的是线程对象不被回收的情况，
这就发生了真正意义上的内存泄露。比如使用线程池的时候，线程结束是不会销毁的，
会再次使用的就可能出现内存泄露 。
（在web应用中，每次http请求都是一个线程，tomcat容器配置使用线程池时会出现内存泄漏问题）
