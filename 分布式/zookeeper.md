ZooKeeper 是一个开源的**分布式协调服务框架**，为分布式系统提供一致性服务。

可能单说这个有点官方了，那就先从分布式的cap说起吧，cap中，zk的实现思路是cp，即强一致性。当然并不是说人家就不保证高可用了，zk也是有高可用实现的，具体体现在哪呢？比如说超过半数以上同意就提交事务，这样做一方面提高效率，另一方面也增加了自己的容错性，下面展开说它里面的一些原理。

## zk的组成

zk可以说是一个文件系统+时间监听通知来组成的。

为什么说它是文件系统呢？它的结构是一个树状结构，访问节点通过当前节点的path路径即可访问，每个节点可以用来存储数据，当然不能过多，单节点存储数据上线是1MB，在zk中，节点分为几种类型，有持久性节点和临时目录节点，持久性和临时目录节点分别可以排序。

它的监听watcher机制，主要体现在每个节点都可以被别的节点监听，利用监听机制可以让zk完成很多事情，比如集群管理，分布式锁等。在dubbo中作为注册中心时，就可以利用监听机制完成dubbo服务节点的注册情况，每当节点下有数据更新时，monitor可以监听到，同时消费者和服务者也可以注册监听事件互相监听来完成服务之间的调用。

同时zk的watcher机制也可以实现分布式锁，分两种情况，一种是大家都创建都一个目录，当前面的任务执行完成后删除，然后后面的任务进来时如果发现这个节点已经存在了，则监听这个节点的删除事件，当删除时呢我再去抢占创建这个目录，但是这种方式容易产生羊群效应一般不建议这么使用；第二种方式可以创建固定目录节点，在该节点下创建临时顺序目录节点，当客户端建立链接创建时没问题，优先给你创建并排好序，每个节点监听它的前一个节点的删除事件即可。

### zk分布式锁和redis的实现有什么区别

最明显区别就是zk是天生支持阻塞式排队获取锁，并且效率很高，但redis只能通过客户端编码来实现，效率相对较低。

## zk的特点

1. 集群，zk是天生支持集群的，它要求集群数量是2n+1，这和它的过半提交机制相关。
2. 高可用，过半提交的情况下，如果有少于半数的机器宕机，是不影响系统正常运行。
3. 全局一致性，每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4. 请求顺序，zk可以保证客户端每个client的请求按顺序执行，这是follower节点内会维护一个先进先出的队列保证。
5. 数据更新原子性，数据的一致性前提条件，就是要保证全局数据操作的原子性，也就是一次数据操作，要么全部成功，要么失败。
6. 实时性，这是相对的，主要是说在一定时间范围内的数据实时。
7. 保证cp，即强一致性，和eureka不太一样，eureka保证的是高可用性，可以容忍一段时间内数据不一样，但保证最终一致性。
   
## zk的协议

zk的核心和zab协议，也就是原子广播，该协议能够很好地支持**崩溃修复**。

zab协议中，有三个角色：
1. Leader ：集群中 唯一的写请求处理者 ，能够发起投票（投票也是为了进行写请求）。
2. follower：可以理解为leader的跟随者，职责是主要可以负责查询请求，在写请求是转发给leader，最重要的，在选举是可以投票，有选举和被选举权。
3. observer：观察者。这个角色没有选举和被选举权利，只能用来完成查询请求，就是没有选举权和被选举权的 Follower ，它的存在是为了提交系统性能，一般生产不会使用该角色，除非生产环境某个运行阶段突发流量需要临时机器替补干活，才会用到它。

拥有这个三个角色可以用来干什么呢？分为两部分，**消息广播**和**崩溃恢复**。

### 协议中的提交标识

zk中所有的提议都会携带一个标识，这个标识是一个64位，高32位用来标识当前leader所代表的一个纪元，可以说是某一个leader的统治时期，低32位用来表示它内部的一个递增事务id，纪元和事务id都会在更换过程中递增发生变化，所以最大的往往是最新的。

### 消息广播

1. leader从客户端收到一个写请求
2. leader生成一个新的事务并为这个事务生成一个唯一的ZXID
3. leader将这个事务发送给所有的follows节点，将带有 zxid 的消息作为一个提案(proposal)分发给所有 follower。
4. follower节点将收到的事务请求加入到历史队列(history queue)中，当 follower 接收到 proposal，先将 proposal 写到硬盘，写硬盘成功后再向 leader 回一个 ACK,当leader收到大多数follower（超过一半）的ack消息，leader会向follower发送commit请求（leader自身也要提交这个事务）
5. 当follower收到commit请求时，会判断该事务的ZXID是不是比历史队列中的任何事务的ZXID都小，如果是则提交事务，如果不是则等待比它更小的事务的commit(保证顺序性)
6. Leader将处理结果返回给客户端

Leader处理写请求是通过上面的消息广播模式，实质上最后所有的zkServer都要执行写操作，这样数据才会一致。

而对于读请求，Leader/Follower/Observer都可直接处理读请求，从本地内存中读取数据并返回给客户端即可。由于处理读请求不需要各个服务器之间的交互，因此Follower/Observer越多，整体可处理的读请求量越大，也即读性能越好。

### 崩溃恢复

zk运行期间，可能会由于某些原因，导致leader宕机，当发生宕机后，zk集群会马上进入选举状态。

选举的过程中第一阶段每个阶段会把自己的myid，事务id广播给所有其他节点，每个节点收到后优先比较事务ID大的进行投票，其次在比较myid，myid是集群启动时配置的，肯定不一样，当投票超过一半同意某一个节点作为leader时，leader就产生了。

这时还没完，这个leader还没创建自己的纪元代号呢，这时这个选出来的准leader就会问这个从节点，你们之前接收到最大的纪元id是多少啊，还有你们的事务id最大是多少啊，给我说一下，然后拿到我这我自己比较一下，更新到最新事务id对应的数据状态，并记录历史日志，然后我将我的最新的这些状态同步给所有节点，当然也必须是大于一半follower节点提交即可，然后将follower节点给的这个纪元id加上1，表明我要开启一个新的时代，到这里基本上就完成了一次崩溃恢复的选举，集群进入广播模式正常接收请求。

# 脑裂问题什么情况下会产生

脑裂就是集群中出现多个leader的现象。

脑裂问题一般发生在leader假死，或者说明明leader还在，但过半从节点就是联系不上它，比如机房之间的通信问题，或者网络抖动等原因，这时leader还认为自己是leader，但由于过半的从节点认为它挂了，已经开始了新一轮的选举，所以会选举出一个新的leader，这时集群内有多个leader，就会导致数据不一致的现象产生，因为原leader的消息广播不会执行成功，但是它可以完成查询。

如何避免呢，比如说可以考虑follower检测leader是否存活增加别的通信模式，比如你检测心跳检测不到，那是否可以检测不到时在采用其他方式继续链接呢，或者设置大一些重试次数；或者也可以通过共享磁盘锁类似方式实现当心跳检测不到，原leader是否真的shutdown了，确保真的shutdown之后再进行选举。

当然，zk的过半机制也一定程度的预防了脑裂，比如a机房3个节点，b机房2个节点，如果leader是在a机房的话，即使与b机房不能取得链接，那b机房内的两个节点也不会选举出心的leader。

# epoch存在的意义是什么

它的意义很明显就是防止接收脑裂后，原leader的请求。因为新的leader它的epoch会加1，follwer会认为这个请求才是最新的。
