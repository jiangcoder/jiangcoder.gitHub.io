title: Java引用之强软弱虚
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---Java引用之强软弱虚
## 一，强引用
默认的引用就是强引用，NormalReference
## 二，软引用
SoftReference，当有一个对象(字节数组)被一个软引用所指向的时候，只有系统内存不够的时候，才会回收，
非常适合做缓存使用
## 三，弱引用
弱引用只要遇到GC就会回收
```java
    ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
    String str = new String("abc");
    SoftReference<String> softReference = new SoftReference<>(str, referenceQueue);

    str = null;
    // Notify GC
    System.gc();

    System.out.println(softReference.get()); // abc

    Reference<? extends String> reference = referenceQueue.poll();
    System.out.println(reference); //null

```
## 四，虚引用
虚引用顾名思义，就是形同虚设。与其他几种引用都不同，虚引用并不会决定对象的生命周期。
如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
应用场景：
虚引用主要用来跟踪对象被垃圾回收器回收的活动。
虚引用与软引用和弱引用的一个区别在于：

虚引用必须和引用队列(ReferenceQueue)联合使用。
当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，
把这个虚引用加入到与之关联的引用队列中。