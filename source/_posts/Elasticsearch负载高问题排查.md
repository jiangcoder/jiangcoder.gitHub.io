1. 我们Elasticsearch集群近几日负载飙高，排查思路记录如下；

2. 查看日志信息，确定是否存在异常情况

```java
cd ${ES_HOME}/logs
tail -100f ES.log
```

3. 频繁Full GC往往会引起负载飙高，故查看ES集群GC 情况，

使用命令：
```java
 jstat -gcutil <pid> <period> <times>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200320205733385.png)
近一个多月，服务共Full GC700余次，平均每次Full GC耗时90ms，符合预期，排除Full GC问题导致负载飙高；
4. 找到ES中占用CPU的线程ID

```java
  top -Hp PID
```
如找到ES进程中2291线程较费CPU
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200320210618219.png)

5. 将得到的线程id，转化为16进制

```java
printf %x 2291
输出结果：8f3
```

7. 使用jstack分析线程状态
jstack命令主要用于调试java程序运行过程中的线程堆栈信息

```java
jstack PID  >> pid.txt
```
8.查看pid.txt文件，分析线程对应的堆栈信息
由步骤6得到 16进制线程“8f3”

```java
vim pid.txt
//查找线程“8f3”对应信息
```
内容如下

```java
"elasticsearch[data-es12][write][T#3]" #94 daemon prio=5 os_prio=0 tid=0x00007f3254017800 nid=0x8f3 waiting for monitor entry [0x00007f2d37a7a000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at org.elasticsearch.index.translog.TranslogWriter.syncUpTo(TranslogWriter.java:342)
        - waiting to lock <0x000000055ff5f220> (a java.lang.Object)
        at org.elasticsearch.index.translog.Translog.ensureSynced(Translog.java:797)
        at org.elasticsearch.index.translog.Translog.ensureSynced(Translog.java:818)
        at org.elasticsearch.index.engine.InternalEngine.ensureTranslogSynced(InternalEngine.java:489)
        at org.elasticsearch.index.shard.IndexShard$5.write(IndexShard.java:2782)
        at org.elasticsearch.common.util.concurrent.AsyncIOProcessor.processList(AsyncIOProcessor.java:107)
        at org.elasticsearch.common.util.concurrent.AsyncIOProcessor.drainAndProcess(AsyncIOProcessor.java:99)
        at org.elasticsearch.common.util.concurrent.AsyncIOProcessor.put(AsyncIOProcessor.java:82)
        at org.elasticsearch.index.shard.IndexShard.sync(IndexShard.java:2804)
        at org.elasticsearch.action.support.replication.TransportWriteAction$AsyncAfterWriteAction.run(TransportWriteAction.java:355)
        at org.elasticsearch.action.support.replication.TransportWriteAction$WritePrimaryResult.<init>(TransportWriteAction.java:151)
        at 
```

该线程正处于堵塞状态。

9，原因分析
从ES索引数据创建机制说起
写请求首先先写到内存一份数据，然后写translog，默认每1秒进行refresh，将索引写入到文件系统；
该ES集群是ELK日志查询服务，特点读多写少，我们TB级数据，上千个索引，refresh_interval参数1s，在日志服务中显然不太合适，是时候调整下该参数配置了。

10，参数调整

```java
http://ip:port/logstash*/_settings PUT
{
  "settings": {
    "refresh_interval": "5m"
  }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200320212733184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FjbTM2NQ==,size_16,color_FFFFFF,t_70)

11，问题解决
