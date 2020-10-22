title: Java之Buffer
toc: true
categories: java
tags:
  - java
thumbnail: /logo/java.jpg
---
## 一，Buffer概念
Buffer 类是 java.nio 的构造基础。
一个 Buffer 对象是固定数量的数据的容器，其作用是一个存储器，或者分段运输区.
每个非布尔原始数据类型都有一个缓冲区类.
子类有
```java
    ByteBuffer
    CharBuffer
    IntBuffer
    LongBuffer
    DoubleBuffer
    FloatBuffer
    ShortBuffer
    
```
## 二，缓存区的四个属性
```java
    private int mark = -1;//一个备忘位置。标记在设定前是未定义的(undefined)。使用场景是，假设缓冲区中有 10 个元素，position 目前的位置为 2(也就是如果get的话是第三个元素)，现在只想发送 6 - 10 之间的缓冲数据，此时我们可以 buffer.mark(buffer.position())，即把当前的 position 记入 mark 中，然后 buffer.postion(6)，此时发送给 channel 的数据就是 6 - 10 的数据。发送完后，我们可以调用 buffer.reset() 使得 position = mark，因此这里的 mark 只是用于临时记录一下位置用的
    private int position = 0;//下一个要被读或写的元素的索引。位置会自动由相应的 get() 和 put() 函数更新。 这里需要注意的是positon的位置是从0开始的
    private int limit;//缓冲区的第一个不能被读或写的元素。缓冲创建时，limit 的值等于 capacity 的值。假设 capacity = 1024，我们在程序中设置了 limit = 512，说明，Buffer 的容量为 1024，但是从 512 之后既不能读也不能写，因此可以理解成，Buffer 的实际可用大小为 512
    private int capacity;//缓冲区能够容纳的数据元素的最大数量。这一容量在缓冲区创建时被设定，并且永远不能被改变。
```
## 三，flip
flip() :写模式转到读模式
clear()或者compact():重新从Buffer中读取数据
such as :
```java
import java.nio.ByteBuffer;
public class CreateBuffer {
    public static  void main(String args[]) {
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        buffer.put((byte) 'a');
        buffer.put((byte) 'b');
        buffer.put((byte) 'c');

        //调用flip之后，读写指针指到缓存头部，并且设置了最多只能读出之前写入的数据长度(而不是整个缓存的容量大小)
        buffer.flip();
        System.out.println((char) buffer.get());//a
        System.out.println((char) buffer.get());//b
        System.out.println((char) buffer.get());//c
        //重新从Buffer中读取数据
        buffer.compact();
        System.out.println((char) buffer.get());//a
        System.out.println((char) buffer.get());//b
        System.out.println((char) buffer.get());//c        
    }
}
```