# zookeeper

## 基础

### 简介

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop的重要组件，CDH版本中更是使用它进行Namenode的协调控制。它是一个为分布式应用提供一致性服务的软件.

提供的功能包括：配置维护、名字服务、分布式同步、组服务等。

ZooKeeper的目标：是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

**原语：** 操作系统或计算机网络用语范畴。它是由若干条指令组成的，用于完成一定功能的一个过程。具有不可分割性，即原语的执行必须是连续的，在执行过程中不允许被中断。

ZooKeeper 是一个典型的**分布式数据一致性解决方案**，分布式应用程序可以基于 ZooKeeper 实现诸如**数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列**等功能。

### 由来

下面这段内容摘自《从 Paxos 到 ZooKeeper 》第四章第一节的某段内容，推荐大家阅读一下：

Zookeeper 最早起源于雅虎研究院的一个研究小组。在当时，研究人员发现，在雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行分布式协调，但是这些系统往往都存在分布式单点问题。

所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。

关于“ZooKeeper”这个项目的名字，其实也有一段趣闻。在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目)，雅虎的工程师希望给这个项目也取一个动物的名字。

时任研究院的首席科学家 Raghu Ramakrishnan 开玩笑地说：“在这样下去，我们这儿就变成动物园了！”

此话一出，大家纷纷表示就叫动物园管理员吧，因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了。

而 Zookeeper 正好要用来进行分布式环境的协调，于是，Zookeeper 的名字也就由此诞生了。

![img](https://img-blog.csdn.net/2018091122155573?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 使用场景

ZooKeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心。服务生产者将自己提供的服务注册到 ZooKeeper 中心，服务的消费者在进行服务调用的时候先到 ZooKeeper 中查找服务，获取到服务生产者的详细信息之后，再去调用服务生产者的内容与数据。

如下图所示，在 Dubbo 架构中 ZooKeeper 就担任了注册中心这一角色。

![img](https://img-blog.csdn.net/20180911221613235?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Dubbo 架构图

### 结合个人使用讲一下 ZooKeeper

在我自己做过的项目中，主要使用到了 ZooKeeper 作为 Dubbo 的注册中心(Dubbo 官方推荐使用 ZooKeeper 注册中心)。

另外在搭建 Solr 集群的时候，我使用  ZooKeeper 作为 Solr 集群的管理工具。

这时，ZooKeeper 主要提供下面几个功能：

- **集群管理：容错、负载均衡。**
- **配置文件的集中管理。**
- **集群的入口。**

我个人觉得在使用 ZooKeeper 的时候，最好是使用集群版的 ZooKeeper 而不是单机版的。

官网给出的架构图就描述的是一个集群版的 ZooKeeper 。通常 **3 台**服务器就可以构成一个 **ZooKeeper 集群**了。

#### 最好使用奇数台服务器构成 ZooKeeper 集群？

我们知道在 ZooKeeper 中 **Leader 选举算法**采用了 Zab 协议。Zab 核心思想是当多数 Server 写成功，则任务数据写成功：

> ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务）。

- 如果有 3 个 Server，则最多允许 1 个 Server 挂掉。
- 如果有 4 个 Server，则同样最多允许 1 个 Server 挂掉。

既然 3 个或者 4 个 Server，同样最多允许 1 个 Server 挂掉，那么它们的可靠性是一样的。

所以选择奇数个 ZooKeeper Server 即可，这里选择 3 个 Server。

### 优点

- **ZooKeeper 本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper 就能正常服务）。**

- **为了保证高可用，最好是以集群形态来部署 ZooKeeper，**这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的。

- **ZooKeeper 将数据保存在内存中，**这也就保证了 高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持 Znode 中存储的数据量较小的进一步原因）。

- **ZooKeeper 是高性能的。**在“读”多于“写”的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）

- **ZooKeeper 有临时节点的概念。**当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。

  而当会话终结时，瞬时节点被删除。持久节点是指一旦这个 ZNode 被创建了，除非主动进行 ZNode 的移除操作，否则这个 ZNode 将一直保存在 Zookeeper 上。

- **ZooKeeper 底层其实只提供了两个功能**：①管理（存储、读取）用户程序提交的数据；②为用户程序提交数据节点监听服务。

## 相关概念

### 会话（Session）

Session 指的是 ZooKeeper  服务器与客户端会话。在 ZooKeeper 中，一个客户端连接是指客户端和服务器之间的一个 TCP 长连接。

客户端启动的时候，首先会与服务器建立一个 TCP 连接，从第一次连接建立开始，客户端会话的生命周期也开始了。

通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能够向 Zookeeper 服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的 Watch 事件通知。

Session 的 sessionTimeout 值用来设置一个客户端会话的超时时间。

当由于服务器压力太大、网络故障或是客户端主动断开连接等各种原因导致客户端连接断开时，只要在 sessionTimeout 规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。

在为客户端创建会话之前，服务端首先会为每个客户端都分配一个 sessionID。

由于 sessionID 是 Zookeeper 会话的一个重要标识，许多与会话相关的运行机制都是基于这个 sessionID 的。

因此，无论是哪台服务器为客户端分配的 sessionID，都务必保证全局唯一。

### Znode

在谈到分布式的时候，我们通常说的“节点"是指组成集群的每一台机器。

然而，在 ZooKeeper 中，“节点"分为两类：

- 第一类同样是指构成集群的机器，**我们称之为\**机器节点\**。**
- 第二类则是指数据模型中的数据单元，**我们称之为数据节点一ZNode。**

ZooKeeper 将所有数据存储在内存中，数据模型是一棵树（Znode Tree)，由斜杠（/）的进行分割的路径，就是一个 Znode，例如/foo/path1。每个上都会保存自己的数据内容，同时还会保存一系列属性信息。

在 Zookeeper 中，Node 可以分为持久节点和临时节点两类。所谓持久节点是指一旦这个 ZNode 被创建了，除非主动进行 ZNode 的移除操作，否则这个 ZNode 将一直保存在 ZooKeeper 上。

而临时节点就不一样了，它的生命周期和客户端会话绑定，一旦客户端会话失效，那么这个客户端创建的所有临时节点都会被移除。

另外，ZooKeeper 还允许用户为每个节点添加一个特殊的属性：SEQUENTIAL。

一旦节点被标记上这个属性，那么在这个节点被创建的时候，ZooKeeper 会自动在其节点名后面追加上一个整型数字，这个整型数字是一个由父节点维护的自增数字。

#### 版本

在前面我们已经提到，Zookeeper 的每个 ZNode 上都会存储数据，对应于每个 ZNode，Zookeeper 都会为其维护一个叫作 Stat 的数据结构。

Stat 中记录了这个 ZNode 的三个数据版本，分别是：

- **version（当前 ZNode 的版本）**
- **cversion（当前 ZNode 子节点的版本）**
- **aversion（当前 ZNode 的 ACL 版本）**

### Watcher

Watcher（事件监听器），是 ZooKeeper 中的一个很重要的特性。

ZooKeeper 允许用户在指定节点上注册一些 Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是 ZooKeeper 实现分布式协调服务的重要特性。

**ACL**

ZooKeeper 采用 ACL（AccessControlLists）策略来进行权限控制，类似于  UNIX 文件系统的权限控制。

ZooKeeper 定义了 5 种权限，如下图：

![img](https://img-blog.csdn.net/20180911221856871?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 

其中尤其需要注意的是，CREATE 和 DELETE 这两种权限都是针对子节点的权限控制。

### ZooKeeper 特点

ZooKeeper 有哪些特点呢？具体如下：

- **顺序一致性：**从同一客户端发起的事务请求，最终将会严格地按照顺序被应用到 ZooKeeper 中去。
- **原子性：**所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。
- **单一系统映像：**无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。
- **可靠性：**一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。

### ZooKeeper 设计目标

**简单的数据模型**

ZooKeeper 允许分布式进程通过共享的层次结构命名空间进行相互协调，这与标准文件系统类似。

名称空间由 ZooKeeper 中的数据寄存器组成，称为 Znode，这些类似于文件和目录。

与为存储设计的典型文件系统不同，ZooKeeper 数据保存在内存中，这意味着 ZooKeeper 可以实现高吞吐量和低延迟。

![img](https://img-blog.csdn.net/20180911221937498?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 保证高可用

> **可构建集群**

为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的。 

客户端在使用 ZooKeeper 时，需要知道集群机器列表，通过与集群中的某一台机器建立 TCP 连接来使用服务。

客户端使用这个 TCP 链接来发送请求、获取结果、获取监听事件以及发送心跳包。如果这个连接异常断开了，客户端可以连接到另外的机器上。

ZooKeeper 官方提供的架构图：

![img](https://img-blog.csdn.net/20180911222117330?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上图中每一个 Server 代表一个安装 ZooKeeper 服务的服务器。组成 ZooKeeper 服务的服务器都会在内存中维护当前的服务器状态，并且每台服务器之间都互相保持着通信。

集群间通过 Zab 协议（Zookeeper Atomic Broadcast）来保持数据的一致性。

### 顺序访问

对于来自客户端的每个更新请求，ZooKeeper 都会分配一个全局唯一的递增编号。

这个编号反应了所有事务操作的先后顺序，应用程序可以使用 ZooKeeper 这个特性来实现更高层次的同步原语。这个编号也叫做时间戳—zxid（ZooKeeper Transaction Id）。

### 高性能

ZooKeeper 是高性能的。在“读”多于“写”的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）

ZooKeeper 集群角色介绍

**最典型集群模式：Master/Slave 模式（主备模式）。**在这种模式中，通常 Master 服务器作为主服务器提供写服务，其他的 Slave 服务器从服务器通过异步复制的方式获取 Master 服务器最新的数据提供读服务。

但是，在 ZooKeeper 中没有选择传统的 Master/Slave 概念，而是引入了Leader、Follower 和 Observer 三种角色。

如下图所示：

![img](https://img-blog.csdn.net/20180911222159806?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

ZooKeeper 集群中的所有机器通过一个 Leader 选举过程来选定一台称为 “Leader” 的机器。

Leader 既可以为客户端提供写服务又能提供读服务。除了 Leader 外，Follower 和  Observer 都只能提供读服务。

Follower 和 Observer 唯一的区别在于 Observer 机器不参与 Leader 的选举过程，也不参与写操作的“过半写成功”策略，因此 Observer 机器可以在不影响写性能的情况下提升集群的读性能。 

![img](https://img-blog.csdn.net/20180911222230900?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYWhhbzExODY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 常见面试题

## **谈下你对 Zookeeper 的认识？** 

ZooKeeper 是一个分布式的，开放源码的分布式应用程序协调服务。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

ZooKeeper 的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

## **Zookeeper 都有哪些功能？** 

**1. 集群管理**：监控节点存活状态、运行请求等；

**2. 主节点选举**：主节点挂掉了之后可以从备用的节点开始新一轮选主，主节点选举说的就是这个选举的过程，使用 Zookeeper 可以协助完成这个过程；

**3. 分布式锁**：Zookeeper 提供两种锁：独占锁、共享锁。独占锁即一次只能有一个线程使用资源，共享锁是读锁共享，读写互斥，即可以有多线线程同时读同一个资源，如果要使用写锁也只能有一个线程使用。Zookeeper 可以对分布式锁进行控制。

**4. 命名服务**：在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，提供者等信息。

## **谈下你对 ZAB 协议的了解？**

ZAB 协议是为分布式协调服务 Zookeeper 专门设计的一种支持崩溃恢复的原子广播协议。ZAB 协议包括两种基本的模式：崩溃恢复和消息广播。

当整个 Zookeeper 集群刚刚启动或者Leader服务器宕机、重启或者网络故障导致不存在过半的服务器与 Leader 服务器保持正常通信时，所有服务器进入崩溃恢复模式，首先选举产生新的 Leader 服务器，然后集群中 Follower 服务器开始与新的 Leader 服务器进行数据同步。当集群中超过半数机器与该 Leader 服务器完成数据同步之后，退出恢复模式进入消息广播模式，Leader 服务器开始接收客户端的事务请求生成事物提案来进行事务请求处理。

![img](https://pic1.zhimg.com/80/v2-d602bd259cf8784f85e3485c57973080_1440w.jpg)

## **Zookeeper 怎么保证主从节点的状态同步？** 

Zookeeper 的核心是原子广播机制，这个机制保证了各个 server 之间的同步。实现这个机制的协议叫做 Zab 协议。Zab 协议有两种模式，它们分别是恢复模式和广播模式。

**1. 恢复模式**

当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数 server 完成了和 leader 的状态同步以后，恢复模式就结束了。状态同步保证了 leader 和 server 具有相同的系统状态。

**2. 广播模式**

一旦 leader 已经和多数的 follower 进行了状态同步后，它就可以开始广播消息了，即进入广播状态。这时候当一个 server 加入 ZooKeeper 服务中，它会在恢复模式下启动，发现 leader，并和 leader 进行状态同步。待到同步结束，它也参与消息广播。ZooKeeper 服务一直维持在 Broadcast 状态，直到 leader 崩溃了或者 leader 失去了大部分的 followers 支持。

## **Zookeeper 有几种部署模式？**

Zookeeper 有三种部署模式：

\1. 单机部署：一台集群上运行；

\2. 集群部署：多台集群运行；

\3. 伪集群部署：一台集群启动多个 Zookeeper 实例运行。

## **说一下 Zookeeper 的通知机制？** 

client 端会对某个 znode 建立一个 watcher 事件，当该 znode 发生变化时，这些 client 会收到 zk 的通知，然后 client 可以根据 znode 变化来做出业务上的改变等。

## **集群中为什么要有主节点？** 

在分布式环境中，有些业务逻辑只需要集群中的某一台机器进行执行，其他的机器可以共享这个结果，这样可以大大减少重复计算，提高性能，于是就需要进行 leader 选举。

## **集群中有 3 台服务器，其中一个节点宕机，这个时候 Zookeeper 还可以使用吗？** 

可以继续使用，单数服务器只要没超过一半的服务器宕机就可以继续使用。

集群规则为 2N+1 台，N >0，即最少需要 3 台。

![img](https://pic1.zhimg.com/80/v2-bfaada4e6e1b7b36853a5122d8e5e6b8_1440w.jpg)

## **说一下两阶段提交和三阶段提交的过程？分别有什么问题？** 

**两阶段提交协议 2PC**

\1. 第一阶段（投票阶段）：

（1）协调者节点向所有参与者节点询问是否可以执行提交操作(vote)，并开始等待各参与者节点的响应；

（2）参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志。
（3）各参与者节点响应协调者节点发起的询问。如果参与者节点的事务操作实际执行成功，则它返回一个”同意”消息；如果参与者节点的事务操作实际执行失败，则它返回一个”中止”消息。

\2. 第二阶段（提交执行阶段）：
当协调者节点从所有参与者节点获得的相应消息都为”同意”时：

（1）协调者节点向所有参与者节点发出”正式提交(commit)”的请求；

（2）参与者节点正式完成操作，并释放在整个事务期间内占用的资源；

（3）参与者节点向协调者节点发送”完成”消息；

（4）协调者节点受到所有参与者节点反馈的”完成”消息后，完成事务。



**两阶段提交存在的问题：**


\1. 执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态；

\2. 参与者发生故障：协调者需要给每个参与者额外指定超时机制，超时后整个事务失败；

\3. 协调者发生故障：参与者会一直阻塞下去。需要额外的备机进行容错；

\4. 二阶段无法解决的问题：协调者再发出 commit 消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。



**三阶段提交协议 3PC**


与两阶段提交不同的是，三阶段提交有两个改动点：

\1. 引入超时机制。同时在协调者和参与者中都引入超时机制；

\2. 在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

也就是说，除了引入超时机制之外，3PC 把 2PC 的准备阶段再次一分为二，这样三阶段提交就有 CanCommit、PreCommit、DoCommit 三个阶段。

\1. CanCommit 阶段
3PC 的 CanCommit 阶段其实和 2PC 的准备阶段很像。协调者向参与者发送 commit 请求，参与者如果可以提交就返回 Yes 响应，否则返回 No 响应。

（1）事务询问：协调者向参与者发送 CanCommit 请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。

（2）响应反馈：参与者接到 CanCommit 请求之后，正常情况下，如果其自身认为可以顺利执行事务，则返回 Yes 响应，并进入预备状态。否则反馈 No。

\2. PreCommit 阶段
协调者根据参与者的反应情况来决定是否可以继续事务的 PreCommit 操作。根据响应情况，有以下两种可能：

假如协调者从所有的参与者获得的反馈都是 Yes 响应，那么就会执行事务的预执行。

（1）发送预提交请求：协调者向参与者发送 PreCommit 请求，并进入 Prepared 阶段。

（2）事务预提交：参与者接收到 PreCommit 请求后，会执行事务操作，并将 undo 和 redo 信息记录到事务日志中。
（3）响应反馈：如果参与者成功的执行了事务操作，则返回 ACK 响应，同时开始等待最终指令。

假如有任何一个参与者向协调者发送了 No 响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。
（1）发送中断请求：协调者向所有参与者发送 abort 请求。
（2）中断事务：参与者收到来自协调者的 abort 请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。

\3. doCommit 阶段
该阶段进行真正的事务提交，也可以分为以下两种情况。

3.1 执行提交

（1）发送提交请求：协调接收到参与者发送的 ACK 响应，那么他将从预提交状态进入到提交状态。并向所有参与者发送 doCommit 请求。

（2）事务提交：参与者接收到 doCommit 请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。

（3）响应反馈：事务提交完之后，向协调者发送 ACK 响应。

（4）完成事务：协调者接收到所有参与者的 ACK 响应之后，完成事务。

3.2 中断事务

协调者没有接收到参与者发送的 ACK 响应（可能是接受者发送的不是 ACK 响应，也可能响应超时），那么就会执行中断事务。

（1）发送中断请求：协调者向所有参与者发送 abort 请求。

（2）事务回滚：参与者接收到 abort 请求之后，利用其在阶段二记录的 undo 信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。

（3）反馈结果：参与者完成事务回滚之后，向协调者发送 ACK 消息。

（4）中断事务：协调者接收到参与者反馈的 ACK 消息之后，执行事务的中断。

**三阶段提交的问题：**

网络分区可能会带来问题。需要四阶段解决：四阶段直接调用远程服务的数据状态，确定当前数据一致性的情况。

## **Zookeeper 宕机如何处理？** 

Zookeeper 本身也是集群，推荐配置不少于 3 个服务器。Zookeeper 自身也要保证当一个节点宕机时，其他节点会继续提供服务。如果是一个 Follower 宕机，还有 2 台服务器提供访问，因为 Zookeeper 上的数据是有多个副本的，数据并不会丢失；如果是一个 Leader 宕机，Zookeeper 会选举出新的 Leader。

Zookeeper 集群的机制是只要超过半数的节点正常，集群就能正常提供服务。只有在 Zookeeper 节点挂得太多，只剩一半或不到一半节点能工作，集群才失效。所以：

3 个节点的 cluster 可以挂掉 1 个节点(leader 可以得到 2 票 > 1.5)

2 个节点的 cluster 就不能挂掉任何1个节点了(leader 可以得到 1 票 <= 1)

## 说下四种类型的数据节点 Znode？

\1. PERSISTENT：持久节点，除非手动删除，否则节点一直存在于 Zookeeper 上。

\2. EPHEMERAL：临时节点，临时节点的生命周期与客户端会话绑定，一旦客户端会话失效（客户端与 Zookeeper连接断开不一定会话失效），那么这个客户端创建的所有临时节点都会被移除。

\3. PERSISTENT_SEQUENTIAL：持久顺序节点，基本特性同持久节点，只是增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。

\4. EPHEMERAL_SEQUENTIAL：临时顺序节点，基本特性同临时节点，增加了顺序属性，节点名后边会追加一个由父节点维护的自增整型数字。

## **Zookeeper 和 Dubbo 的关系？**

Dubbo 的将注册中心进行抽象，是得它可以外接不同的存储媒介给注册中心提供服务，有 ZooKeeper，Memcached，Redis 等。

引入了 ZooKeeper 作为存储媒介，也就把 ZooKeeper 的特性引进来。首先是负载均衡，单注册中心的承载能力是有限的，在流量达到一定程度的时 候就需要分流，负载均衡就是为了分流而存在的，一个 ZooKeeper 群配合相应的 Web 应用就可以很容易达到负载均衡；资源同步，单单有负载均衡还不 够，节点之间的数据和资源需要同步，ZooKeeper 集群就天然具备有这样的功能；命名服务，将树状结构用于维护全局的服务地址列表，服务提供者在启动 的时候，向 ZooKeeper 上的指定节点 /dubbo/${serviceName}/providers 目录下写入自己的 URL 地址，这个操作就完成了服务的发布。 其他特性还有 Mast 选举，分布式锁等。

![img](https://pic4.zhimg.com/80/v2-5e88d88bc9fcaaeeb10c957fbbc7a9db_1440w.jpg)

# 数据一致性（Zab）

## ZAB 协议 & Paxos 算法

Paxos 算法可以说是  ZooKeeper 的灵魂了。但是，ZooKeeper 并没有完全采用 Paxos 算法 ，而是使用 ZAB 协议作为其保证数据一致性的核心算法。

另外，在 ZooKeeper 的官方文档中也指出，ZAB 协议并不像 Paxos 算法那样，是一种通用的分布式一致性算法，它是一种特别为 ZooKeeper 设计的崩溃可恢复的原子消息广播算法。

关于 ZAB 协议 & Paxos 算法需要讲和理解的东西太多了，推荐阅读下面两篇文章：

- **图解 Paxos 一致性协议：**

  http://blog.xiaohansong.com/2016/09/30/Paxos/

- **Zookeeper ZAB 协议分析：**

  http://blog.xiaohansong.com/2016/08/25/zab/

关于如何使用 ZooKeeper 实现分布式锁，可以查看下面这篇文章：

- **Zookeeper ZAB 协议分析：**

  https://blog.csdn.net/qiangcuo6087/article/details/79067136

## 什么是Zab协议？

Zab协议 的全称是 **Zookeeper Atomic Broadcast** （Zookeeper原子广播）。
 **Zookeeper 是通过 Zab 协议来保证分布式事务的最终一致性**。

1. Zab协议是为分布式协调服务Zookeeper专门设计的一种 **支持崩溃恢复** 的 **原子广播协议** ，是Zookeeper保证数据一致性的核心算法。Zab借鉴了Paxos算法，但又不像Paxos那样，是一种通用的分布式一致性算法。**它是特别为Zookeeper设计的支持崩溃恢复的原子广播协议**。
2. 在Zookeeper中主要依赖Zab协议来实现数据一致性，基于该协议，zk实现了一种主备模型（即Leader和Follower模型）的系统架构来保证集群中各个副本之间数据的一致性。
    这里的主备系统架构模型，就是指只有一台客户端（Leader）负责处理外部的写事务请求，然后Leader客户端将数据同步到其他Follower节点。

Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。

**Zab 协议的特性**：
 1）Zab 协议需要确保那些**已经在 Leader 服务器上提交（Commit）的事务最终被所有的服务器提交**。
 2）Zab 协议需要确保**丢弃那些只在 Leader 上被提出而没有被提交的事务**。

**模型图**

![image-20200305210649310](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210649310.png)

## Zab 协议实现的作用

1）**使用一个单一的主进程（Leader）来接收并处理客户端的事务请求**（也就是写请求），并采用了Zab的原子广播协议，将服务器数据的状态变更以 **事务proposal** （事务提议）的形式广播到所有的副本（Follower）进程上去。

2）**保证一个全局的变更序列被顺序引用**。
 Zookeeper是一个树形结构，很多操作都要先检查才能确定是否可以执行，比如P1的事务t1可能是创建节点"/a"，t2可能是创建节点"/a/bb"，只有先创建了父节点"/a"，才能创建子节点"/a/b"。

为了保证这一点，Zab要保证同一个Leader发起的事务要按顺序被apply，同时还要保证只有先前Leader的事务被apply之后，新选举出来的Leader才能再次发起事务。

3）**当主进程出现异常的时候，整个zk集群依旧能正常工作**。

## Zab协议原理

Zab协议要求每个 Leader 都要经历三个阶段：**发现，同步，广播**。

- **发现**：要求zookeeper集群必须选举出一个 Leader 进程，同时 Leader 会维护一个 Follower 可用客户端列表。将来客户端可以和这些 Follower节点进行通信。
- **同步**：Leader 要负责将本身的数据与 Follower 完成同步，做到多副本存储。这样也是提现了CAP中的高可用和分区容错。Follower将队列中未处理完的请求消费完成后，写入本地事务日志中。
- **广播**：Leader 可以接受客户端新的事务Proposal请求，将新的Proposal请求广播给所有的 Follower。

## Zab协议核心

Zab协议的核心：**定义了事务请求的处理方式**

1）所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被叫做 **Leader服务器**。其他剩余的服务器则是 **Follower服务器**。

2）Leader服务器 负责将一个客户端事务请求，转换成一个 **事务Proposal**，并将该 Proposal 分发给集群中所有的 Follower 服务器，也就是向所有 Follower 节点发送数据广播请求（或数据复制）

3）分发之后Leader服务器需要等待所有Follower服务器的反馈（Ack请求），**在Zab协议中，只要超过半数的Follower服务器进行了正确的反馈**后（也就是收到半数以上的Follower的Ack请求），那么 Leader 就会再次向所有的 Follower服务器发送 Commit 消息，要求其将上一个 事务proposal 进行提交。

**事务请求处理**

![image-20200305210609785](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210609785.png)

## Zab协议内容

Zab 协议包括两种基本的模式：**崩溃恢复** 和 **消息广播**

#### 协议过程

当整个集群启动过程中，或者当 Leader 服务器出现网络中弄断、崩溃退出或重启等异常时，Zab协议就会 **进入崩溃恢复模式**，选举产生新的Leader。

当选举产生了新的 Leader，同时集群中有过半的机器与该 Leader 服务器完成了状态同步（即数据同步）之后，Zab协议就会退出崩溃恢复模式，**进入消息广播模式**。

这时，如果有一台遵守Zab协议的服务器加入集群，因为此时集群中已经存在一个Leader服务器在广播消息，那么该新加入的服务器自动进入恢复模式：找到Leader服务器，并且完成数据同步。同步完成后，作为新的Follower一起参与到消息广播流程中。

#### 协议状态切换

当Leader出现崩溃退出或者机器重启，亦或是集群中不存在超过半数的服务器与Leader保存正常通信，Zab就会再一次进入崩溃恢复，发起新一轮Leader选举并实现数据同步。同步完成后又会进入消息广播模式，接收事务请求。

#### 保证消息有序

在整个消息广播中，Leader会将每一个事务请求转换成对应的 proposal 来进行广播，并且在广播 事务Proposal 之前，Leader服务器会首先为这个事务Proposal分配一个全局单递增的唯一ID，称之为事务ID（即zxid），由于Zab协议需要保证每一个消息的严格的顺序关系，因此必须将每一个proposal按照其zxid的先后顺序进行排序和处理。

## ZAB 协议两种基本的模式

ZAB 协议包括两种基本的模式。

1. 崩溃恢复
2. 消息广播

当整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进入恢复模式并选举产生新的 Leader 服务器。

当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该 Leader 服务器完成了状态同步之后，ZAB 协议就会退出恢复模式。

其中，所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和 Leader 服务器的数据状态保持一致。

当集群中已经有过半的 Follower 服务器完成了和 Leader 服务器的状态同步，那么整个服务框架就可以进人消息广播模式了。 

当一台同样遵守 ZAB 协议的服务器启动后加入到集群中时，如果此时集群中已经存在一个 Leader 服务器在负责进行消息广播。

**那么新加入的服务器就会自觉地进人数据恢复模式：**找到 Leader 所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。

正如上文介绍中所说的，ZooKeeper 设计成只允许唯一的一个 Leader 服务器来进行事务请求的处理。

Leader 服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议。

而如果集群中的其他机器接收到客户端的事务请求，那么这些非 Leader 服务器会首先将这个事务请求转发给 Leader 服务器。

## 消息广播

1）在zookeeper集群中，数据副本的传递策略就是采用消息广播模式。zookeeper中农数据副本的同步方式与二段提交相似，但是却又不同。二段提交要求协调者必须等到所有的参与者全部反馈ACK确认消息后，再发送commit消息。要求所有的参与者要么全部成功，要么全部失败。二段提交会产生严重的阻塞问题。

2）Zab协议中 Leader 等待 Follower 的ACK反馈消息是指“只要半数以上的Follower成功反馈即可，不需要收到全部Follower反馈”

**消息广播流程图**

![image-20200305210547651](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210547651.png)

#### 消息广播具体步骤

1）客户端发起一个写操作请求。

2）Leader 服务器将客户端的请求转化为事务 Proposal 提案，同时为每个 Proposal 分配一个全局的ID，即zxid。

3）Leader 服务器为每个 Follower 服务器分配一个单独的队列，然后将需要广播的 Proposal 依次放到队列中取，并且根据 FIFO 策略进行消息发送。

4）Follower 接收到 Proposal 后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向 Leader 反馈一个 Ack 响应消息。

5）Leader 接收到超过半数以上 Follower 的 Ack 响应消息后，即认为消息发送成功，可以发送 commit 消息。

6）Leader 向所有 Follower 广播 commit 消息，同时自身也会完成事务提交。Follower 接收到 commit 消息后，会将上一条事务提交。

**zookeeper 采用 Zab 协议的核心，就是只要有一台服务器提交了 Proposal，就要确保所有的服务器最终都能正确提交 Proposal。这也是 CAP/BASE 实现最终一致性的一个体现。**

**Leader 服务器与每一个 Follower 服务器之间都维护了一个单独的 FIFO 消息队列进行收发消息，使用队列消息可以做到异步解耦。 Leader 和 Follower 之间只需要往队列中发消息即可。如果使用同步的方式会引起阻塞，性能要下降很多。**

## 崩溃恢复

**一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。**

在 Zab 协议中，为了保证程序的正确运行，整个恢复过程结束后需要选举出一个新的 Leader 服务器。因此 Zab 协议需要一个高效且可靠的 Leader 选举算法，从而确保能够快速选举出新的 Leader 。

Leader 选举算法不仅仅需要让 Leader 自己知道自己已经被选举为 Leader ，同时还需要让集群中的所有其他机器也能够快速感知到选举产生的新 Leader 服务器。

崩溃恢复主要包括两部分：**Leader选举** 和 **数据恢复**

#### Zab 协议如何保证数据一致性

假设两种异常情况：
 1、一个事务在 Leader 上提交了，并且过半的 Folower 都响应 Ack 了，但是 Leader 在 Commit 消息发出之前挂了。
 2、假设一个事务在 Leader 提出之后，Leader 挂了。

要确保如果发生上述两种情况，数据还能保持一致性，那么 Zab 协议选举算法必须满足以下要求：

**Zab 协议崩溃恢复要求满足以下两个要求**：
 1）**确保已经被 Leader 提交的 Proposal 必须最终被所有的 Follower 服务器提交**。
 2）**确保丢弃已经被 Leader 提出的但是没有被提交的 Proposal**。

根据上述要求
 Zab协议需要保证选举出来的Leader需要满足以下条件：
 1）**新选举出来的 Leader 不能包含未提交的 Proposal** 。
 即新选举的 Leader 必须都是已经提交了 Proposal 的 Follower 服务器节点。
 2）**新选举的 Leader 节点中含有最大的 zxid** 。
 这样做的好处是可以避免 Leader 服务器检查 Proposal 的提交和丢弃工作。

#### Zab 如何数据同步

1）完成 Leader 选举后（新的 Leader 具有最高的zxid），在正式开始工作之前（接收事务请求，然后提出新的 Proposal），Leader 服务器会首先确认事务日志中的所有的 Proposal 是否已经被集群中过半的服务器 Commit。

2）Leader 服务器需要确保所有的 Follower 服务器能够接收到每一条事务的 Proposal ，并且能将所有已经提交的事务 Proposal 应用到内存数据中。等到 Follower 将所有尚未同步的事务 Proposal 都从 Leader 服务器上同步过啦并且应用到内存数据中以后，Leader 才会把该 Follower 加入到真正可用的 Follower 列表中。

#### Zab 数据同步过程中，如何处理需要丢弃的 Proposal

在 Zab 的事务编号 zxid 设计中，zxid是一个64位的数字。

其中低32位可以看成一个简单的单增计数器，针对客户端每一个事务请求，Leader 在产生新的 Proposal 事务时，都会对该计数器加1。而高32位则代表了 Leader 周期的 epoch 编号。

> epoch 编号可以理解为当前集群所处的年代，或者周期。每次Leader变更之后都会在 epoch 的基础上加1，这样旧的 Leader 崩溃恢复之后，其他Follower 也不会听它的了，因为 Follower 只服从epoch最高的 Leader 命令。

每当选举产生一个新的 Leader ，就会从这个 Leader 服务器上取出本地事务日志充最大编号 Proposal 的 zxid，并从 zxid 中解析得到对应的 epoch 编号，然后再对其加1，之后该编号就作为新的 epoch 值，并将低32位数字归零，由0开始重新生成zxid。

**Zab 协议通过 epoch 编号来区分 Leader 变化周期**，能够有效避免不同的 Leader 错误的使用了相同的 zxid 编号提出了不一样的 Proposal 的异常情况。

基于以上策略
 **当一个包含了上一个 Leader 周期中尚未提交过的事务 Proposal 的服务器启动时，当这台机器加入集群中，以 Follower 角色连上 Leader 服务器后，Leader 服务器会根据自己服务器上最后提交的 Proposal 来和 Follower 服务器的 Proposal 进行比对，比对的结果肯定是 Leader 要求 Follower 进行一个回退操作，回退到一个确实已经被集群中过半机器 Commit 的最新 Proposal**。

## 实现原理

**Zab 节点有三种状态**：

- Following：当前节点是跟随者，服从 Leader 节点的命令。
- Leading：当前节点是 Leader，负责协调事务。
- Election/Looking：节点处于选举状态，正在寻找 Leader。

代码实现中，多了一种状态：Observing 状态
 这是 Zookeeper 引入 Observer 之后加入的，Observer 不参与选举，是只读节点，跟 Zab 协议没有关系。

**节点的持久状态**：

- history：当前节点接收到事务 Proposal 的Log
- acceptedEpoch：Follower 已经接受的 Leader 更改 epoch 的 newEpoch 提议。
- currentEpoch：当前所处的 Leader 年代
- lastZxid：history 中最近接收到的Proposal 的 zxid（最大zxid）
  

#### Zab 的四个阶段

**1、选举阶段（Leader Election）**
 节点在一开始都处于选举节点，只要有一个节点得到超过半数节点的票数，它就可以当选准 Leader，只有到达第三个阶段（也就是同步阶段），这个准 Leader 才会成为真正的 Leader。

**Zookeeper 规定所有有效的投票都必须在同一个 轮次 中，每个服务器在开始新一轮投票时，都会对自己维护的 logicalClock 进行自增操作**。

每个服务器在广播自己的选票前，会将自己的投票箱（recvset）清空。该投票箱记录了所受到的选票。
 例如：Server_2 投票给 Server_3，Server_3 投票给 Server_1，则Server_1的投票箱为(2,3)、(3,1)、(1,1)。（每个服务器都会默认给自己投票）

前一个数字表示投票者，后一个数字表示被选举者。票箱中只会记录每一个投票者的最后一次投票记录，如果投票者更新自己的选票，则其他服务器收到该新选票后会在自己的票箱中更新该服务器的选票。

**这一阶段的目的就是为了选出一个准 Leader ，然后进入下一个阶段。**
 协议并没有规定详细的选举算法，后面会提到实现中使用的 Fast Leader Election。

**选举流程**

![image-20200305210455246](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210455246.png)

**2、发现阶段（Descovery）**
 在这个阶段，Followers 和上一轮选举出的准 Leader 进行通信，同步 Followers 最近接收的事务 Proposal 。
 一个 Follower 只会连接一个 Leader，如果一个 Follower 节点认为另一个 Follower 节点，则会在尝试连接时被拒绝。被拒绝之后，该节点就会进入 Leader Election阶段。

**这个阶段的主要目的是发现当前大多数节点接收的最新 Proposal，并且准 Leader 生成新的 epoch ，让 Followers 接收，更新它们的 acceptedEpoch**。

**发现流程**

![image-20200305210433513](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210433513.png)

### 3、同步阶段（Synchronization）

 **同步阶段主要是利用 Leader 前一阶段获得的最新 Proposal 历史，同步集群中所有的副本**。
 只有当 quorum（超过半数的节点） 都同步完成，准 Leader 才会成为真正的 Leader。Follower 只会接收 zxid 比自己 lastZxid 大的 Proposal。

**同步流程**

![image-20200305210338666](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210338666.png)

### **4、广播阶段（Broadcast）**

 到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 Leader 可以进行消息广播。同时，如果有新的节点加入，还需要对新节点进行同步。
 需要注意的是，Zab 提交事务并不像 2PC 一样需要全部 Follower 都 Ack，只需要得到 quorum（超过半数的节点）的Ack 就可以。

广播流程

![image-20200305210318037](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210318037.png)

## 协议实现

协议的 Java 版本实现跟上面的定义略有不同，选举阶段使用的是 Fast Leader Election（FLE），它包含了步骤1的发现指责。因为FLE会选举拥有最新提议的历史节点作为 Leader，这样就省去了发现最新提议的步骤。

实际的实现将发现和同步阶段合并为 Recovery Phase（恢复阶段），所以，Zab 的实现实际上有三个阶段。

#### Zab协议三个阶段：

1）**选举（Fast Leader Election）**
 2）**恢复（Recovery Phase）**
 3）**广播（Broadcast Phase）**

**Fast Leader Election（快速选举）**
 前面提到的 FLE 会选举拥有最新Proposal history （lastZxid最大）的节点作为 Leader，这样就省去了发现最新提议的步骤。**这是基于拥有最新提议的节点也拥有最新的提交记录**

- **成为 Leader 的条件：**
   1）选 epoch 最大的
   2）若 epoch 相等，选 zxid 最大的
   3）若 epoch 和 zxid 相等，选择 server_id 最大的（zoo.cfg中的myid）

节点在选举开始时，都默认投票给自己，当接收其他节点的选票时，会根据上面的 **Leader条件** 判断并且更改自己的选票，然后重新发送选票给其他节点。**当有一个节点的得票超过半数，该节点会设置自己的状态为 Leading ，其他节点会设置自己的状态为 Following**。

### 选举过程

![image-20200305210236881](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210236881.png)

### Recovery Phase（恢复阶段）

 这一阶段 Follower 发送他们的 lastZxid 给 Leader，Leader 根据 lastZxid 决定如何同步数据。这里的实现跟前面的 Phase 2 有所不同：Follower 收到 TRUNC 指令会终止 L.lastCommitedZxid 之后的 Proposal ，收到 DIFF 指令会接收新的 Proposal。

history.lastCommitedZxid：最近被提交的 Proposal zxid
 history.oldThreshold：被认为已经太旧的已经提交的 Proposal zxid

![image-20200305210215002](/Users/wangchong/Library/Application Support/typora-user-images/image-20200305210215002.png)

恢复阶段

