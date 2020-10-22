1.raft是一个实现了解决分布式一致性问题的协议

2.分布式环境下的每个节点有三种状态：

　　follower（跟随者）
　　candidate（候选人）
　　leader（领导者）

3.所有的节点开始都是follower状态

   一旦他们不能检测到leader就能成为candidate

   candidate会请求其他节点投票，得到大多是节点的投票就会成为leader，这个过程叫做Leader Election

4.节点的每个改变都会增加节点的日志条目，提交之后leader会复制日志条目到从节点上，在大多数从节点成功增加日志条目之后，提交成功，leader值更新，leader通知从节点已经更新成功，从节点更新数据，这个过程叫做Log Replication。

5.Leader Election

Raft有两个timeout设置来控制选举
第一个是election timeout
　　election timeout指的是follower等待成为candidate的时间，在150ms到300ms之间
　　election timeout一个follower成为一个开始一个election term
　　投票给自己，请求其他节点投票，在得到大多数节点投票之后，重置election timeout，成为leader
第二个是HearBeat timeout
　　Append Entries 在HeartBeat timeout时发送
6.Log Replication
 Log Replicaiton是通过HeartBeat timeout发送Append Entries来完成的

　　客户端发送更新请求

　　leader增加日志条目

　　在下个HeartBeat时，日志变化发给follower

　　在大多数从节点成功之后，leader提交数据并响应客户端

　　下次HeartBeat Timeout之后，从节点数据更新

7，如何选主过程，可以参考学习该动画
https://raft.github.io/raftscope/index.html
