## zookeeper 核心原理

- zookeeper的由来

  1. 分布式架构各个节点的数据一致性
  2. 怎么保证任务只在一个节点上执行
  3. 如果server1挂了，其他节点如何发现并接替任务
  4. 存在共享资源 互斥性 安全性

- zookeeper的设计猜想

  1. 如何防止单点故障

     集群方案（leader，follower） 分担请求流量（高可用）

  2. 数据一致性（必须要有leader）

     有中心化节点（leader,master）

     无中心化节点（redis-cluster）

     leader负责协调和数据同步若没有leader则每个结点都可以接收请求了，那么数据同步的复杂度就会很高。

  3. leader挂了怎么办？数据如何恢复？

     选举机制（zab）。数据恢复。

  4. 如何去保证数据一致性？（分布式事物）

     强一致性，弱一致性  2PC（二阶提交）

- zookeeper中集群角色

  1. leader
     - 事务请求的唯一调度者和处理者。保证集群事务处理的顺序性
     - 集群内各服务器的调度者
  2. follower
     - 处理客户端的非事务请求、转发事务请求到leader服务器
     - 参与事务请求的投票若过半才能通知leader去commit数据还是rollback数据（需要leader发起之后follower才可以投票）
     - leader投票选举
  3. observer
     - 不参与任何投票（事务还是leader都不参与）
     - 与follower类似只参与非事务处理

- zookeeper的事务提交

  提交事务请求转发给leader，leader对每个follower结点进行事务写入的一个申请请求，每个follower结点返回一个ack告诉leader是否可以进行事务操作，若操作数**过半**则对所有的follower结点进行commit否则所有的follower结点rollback操作。这里也解释了为什么结点数量为2N+1 （不包括observer结点）

  ```mermaid
  graph LR
  client[客户端]--> |1.发起请求|follower1[Follower]
  follower1[Follower] --> |2.转发请求到leader|leader[Leader]
  leader[Leader] -->|3.询问结点是否可以提交事务|follower2[Follower]
  leader[Leader] -->|3.询问结点是否可以提交事务|follower1[Follower]
  
  follower1[Follower] -->|4.发送ack到leader|leader[Leader]
  follower2[Follower] -->|4.发送ack到leader|leader[Leader]
  
  leader[Leader] -->|5.提交或者回滚|follower1[Follower]
  leader[Leader] -->|5.提交或者回滚|follower2[Follower]
  leader[Leader] -->|6.同步数据|observer[Observer]
  
  follower1[Follower] -->  |7.返回响应|client[客户端]
  
  F[2PC示意图]
  ```

- ZAB协议

  - 崩溃恢复的原子广播协议主要实现数据一致性。

  1. 原子广播（消息广播模式）

     - leader收到消息请求后会给消息赋予一个全局唯一的64位自增ID，叫做zxid.
     - leader准备了一个FIFO队列将带有zxid的消息作为一个提案分发给每个follower
     - 当follower收到proposal之后会先把proposal写入磁盘，写入成功之后再向leader返回一个ack
     - 当leader收到合法数量（过半，投票是所有非observer结点都要参与）的ack之后则leader会向这些follower发送commit命令，同时也会在本地执行（之前的操作没在leader执行操作）
     - 当follower收到消息的commit命令则会提交消息

     leader的投票过程不需要observer的ack也就不需要它去参与投票的过程。但是observer和leader必须保持数据一致。

  2. 崩溃恢复

     - 问题引入:
       1. 当leader失去过半的follower联系
       2. 当leader挂了 

     - 需要解决的问题：
       1. 已经处理的消息不能丢失（所有follower收到commit之前leader挂了会导致部分提交部分没有提交）
       2. 被丢弃的消息不能再次重现（leader刚收到消息请求生成proposal就挂了，经过恢复模式重新选举leader这个proposal是需要被删除的）

       ```mermaid
       graph LR
       a1[a1] --> b1[b1]
       ```

       

