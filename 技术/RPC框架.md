# RPC

RPC全称是[远程过程调用](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8)（Remote Procedure Call），RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。简单来讲，就是我们从一台机器（客户端）传递参数调用另外一个机器（服务端）的函数或方法（或称为服务）并得到返回结果，通过自定义的传输协议和数据协议来实现RPC，从这个层面来讲，HTTP也可以算做一种RPC的实现。

RPC框架负责屏蔽底层的传输方式（TCP或者UDP）、序列化方式、以及通信细节。实际使用中，并不需要关心底层通信细节和调用过程，让业务端专注于业务代码的实现。国内大家熟知的PRC框架，阿里的HSF和Dubbo(开源)。

### 为什么使用RPC不使用HTTP

> http 协议相较于自定义tcp报文协议，增加的开销在于连接的建立与断开。
>
> http协议是支持连接池复用的，也就是建立一定数量的连接不断开，并不会频繁的创建和销毁连接。
>
> http也可以使用protobuf这种二进制编码协议对内容进行编码，因此二者最大的区别还是在传输协议上。

#### 效率问题

RPC框架可以使用自定义的TCP报文协议

通用定义的http1.1协议的tcp报文包含太多废信息，即使编码协议也就是body是使用二进制编码协议，报文元数据也就是header头的键值对却用了文本编码，非常占字节数。

那么假如我们使用自定义tcp协议的报文如下

![img](https://pic3.zhimg.com/50/v2-89c905b0806577471aa7789a25ac0d44_hd.jpg)![img](https://pic3.zhimg.com/80/v2-89c905b0806577471aa7789a25ac0d44_1440w.jpg)

报头占用的字节数也就只有16个byte，极大地精简了传输内容。

#### 服务的可用性和效率

- 所谓的效率优势是针对http1.1协议来讲的，http2.0协议已经优化编码效率问题，像grpc这种rpc库使用的就是http2.0协议。这么来说吧http容器的性能测试单位通常是kqps，自定义tpc协议则通常是以10kqps到100kqps为基准

- 成熟的rpc库相对http容器，更多的是封装了“服务发现”，"负载均衡"，“熔断降级”一类面向服务的高级特性。可以这么理解，rpc框架是面向服务的更高级的封装。如果把一个http servlet容器上封装一层服务发现和函数代理调用，那它就已经可以做一个rpc框架了。

- 良好的rpc调用是面向服务的封装，针对服务的可用性和效率等都做了优化。单纯使用http调用则缺少了这些特性。

### 核心设计

前面提到了RPC的核心目标：主要是解决分布式系统中服务之间的调用问题。

其实，走到这一步涉及的知识体系非常的多：要求对通信、远程调用、消息机制等有深入的理解和掌握，要求的都是从理论、硬件级、操作系统级以及所采用的语言的实现都有清楚的理解。

### 三个核心角色

1)服务提供者（Server）

对外提供后台服务，将自己的服务信息，注册到注册中心。

2)注册中心（Registry）

用于服务端注册远程服务以及客户端发现服务。

目前主要的注册中心可以借由 zookeeper，eureka，consul，etcd 等开源框架实现。

比如：阿里的Dubbo就是采用zookeeper实现注册中心。

3)服务消费者（Client）

从注册中心获取远程服务的注册信息，然后进行远程过程调用。

### 调用过程

> 一个完整的RPC架构里面包含了四个核心的组件，分别是Client ,Server,Client Stub以及Server Stub，这个Stub大家可以理解为存根。

**客户端（Client）：**服务的调用方。

**服务端（Server）：**真正的服务提供者。

**客户端存根（client stub）：**存放服务端的地址消息，再将客户端的请求参数打包成网络消息，然后通过网络远程发送给服务方。

**服务端存根（Server Stub）**：接收客户端发送过来的消息，将消息解包，并调用本地的方法

1）服务调用方（客户端）调用以本地调用方式调用服务；

2）客户端存根 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；在Java里就是序列化的过程

3）客户端存根 找到服务地址，并将消息通过网络发送到服务端；

4）服务端存根 收到消息后进行解码,在Java里就是反序列化的过程；

5）服务端存根 根据解码结果调用本地的服务；

6）本地服务执行处理逻辑；

7）本地服务将结果返回给server stub（服务端存根）；

8）server stub（服务端存根）将返回结果打包成消息，Java里的序列化；

9）server stub（服务端存根）将打包后的消息通过网络并发送至消费方

10）client stub（客户端存根）接收到消息，并进行解码, Java里的反序列化；

11）服务调用方（client）得到最终结果。

RPC框架的目标就是要2~10这些步骤都封装起来。

### 原理

**1.建立通信**

首先，要解决通讯的问题，主要是通过在客户端和服务器之间建立TCP连接，远程过程调用的所有交换的数据都在这个连接里传输。

**2.服务寻址**

1）服务注册

首先需要把服务注册到服务中心。其实就是在注册中心进行一个登记，注册中心存储了该服务的IP、端口、调用方式(协议、序列化方式)等。在zookeeper中，进行服务注册，实际上就是在zookeeper中创建了一个znode节点，该节点存储了上面所说的服务信息。

2）服务发现

服务消费者在第一次调用服务时，会通过注册中心找到相应的服务的IP地址列表，并缓存到本地，以供后续使用。当消费者调用服务时，不会再去请求注册中心，而是直接通过负载均衡算法从IP列表中取一个服务提供者的服务器调用服务。

3）注册服务

可靠的寻址方式（主要是提供服务的发现）是RPC的实现基石，比如可以zookeeper来实现注册服务等等。

服务提供者启动后主动向服务（注册）中心注册机器ip、端口以及提供的服务列表。

服务消费者启动时向服务（注册）中心获取服务提供方地址列表，可实现软负载均衡和Failover。

提供者需要定时向注册中心发送心跳，一段时间未收到来自提供者的心跳后，认为提供者已经停止服务，从注册中心上摘取掉对应的服务等等。

使用zookeeper管理服务节点配置

![img](https://img-blog.csdn.net/20171219190935495)

RPC服务往平台化的方向发展, 会屏蔽掉更多的服务细节(服务的IP地址集群, 集群的扩容和迁移), 只暴露服务接口，是的客户端和服务端解耦，两者的交互通过ConfigServer(MetaServer)的中介角色来搭线。借助Zookeeper来扮演该角色, server扮演发布者的角色, 而client扮演订阅者的角色。

Zookeeper是分布式应用协作服务. 它实现了paxos的一致性算法, 在命名管理/配置推送/数据同步/主从切换方面扮演重要的角色。 其数据组织类似文件系统的目录结构:

![img](https://img-blog.csdn.net/20171219191101341)

每个节点被称为znode, 为znode节点依据其特性, 又可以分为如下类型:
　　1) PERSISTENT: 永久节点
　　2) EPHEMERAL: 临时节点, 会随session(client disconnect)的消失而消失
　　3) PERSISTENT_SEQUENTIAL: 永久节点, 其节点的名字编号是单调递增的
　　4) EPHEMERAL_SEQUENTIAL: 临时节点, 其节点的名字编号是单调递增的
　　**注**: 临时节点不能成为父节点

Watcher观察模式, client可以注册对节点的状态/内容变更的事件回调机制. 其Event事件的两类属性需要关注下:
　　1) KeeperState: Disconnected,SyncConnected,Expired
　　2) EventType: None,NodeCreated,NodeDeleted,NodeDataChanged,NodeChildrenChanged

1、RPC服务端:
作为具体业务服务的RPC服务发布方, 对其自身的服务描述由以下元素构成：

- root:用来群不同的业务系统；
- namespace: 命名空间，来区分不同的业务； 
- service: 服务接口, 采用发布方的类全名来表示，可以用来区分不同的服务模块；
- version: 版本号

运行服务端后，我们可以看见zookeeper注册了多个服务地址。

![img](https://img-blog.csdn.net/20171219194457205)

**3.网络传输**

数据传输采用什么协议，数据该如何序列化和反序列化，序列化协议包含: 如基于文本编码的 xml json，也有二进制编码的 protobuf hessian等。

**4.NIO通信**

当前很多RPC框架都直接基于netty这一IO通信框架，比如阿里巴巴的HSF、dubbo，Hadoop Avro，推荐使用Netty 作为底层通信框架。

**5.服务调用**

比如：B机器进行本地调用（通过代理Proxy）之后得到了返回值，此时还需要再把返回值发送回A机器，同样也需要经过序列化操作，然后再经过网络传输将二进制数据发送回A机器，而当A机器接收到这些返回值之后，则再次进行反序列化操作

总之，要实现一个RPC不算难，难的是实现一个高性能高可靠的RPC框架，后续将结合Dubbo的实现一起再探讨。

### 流行的RPC框架

目前流行的开源RPC框架还是比较多的。下面重点介绍三种：

**gRPC**是Google最近公布的开源软件，**基于最新的HTTP2.0协议**，并支持常见的众多编程语言。 我们知道HTTP2.0是基于二进制的HTTP协议升级版本，目前各大浏览器都在快马加鞭的加以支持。 这个RPC框架是基于HTTP协议实现的，底层使用到了Netty框架的支持。

**Thrift**是Facebook的一个开源项目，主要是一个跨语言的服务开发框架。**Thrift是基于TCP协议的**。它有一个代码生成器来对它所定义的IDL定义文件自动生成服务代码框架。用户只要在其之前进行二次开发就行，对于底层的RPC通讯等都是透明的。不过这个对于用户来说的话需要学习特定领域语言这个特性，还是有一定成本的。

**Dubbo**是阿里集团开源的一个极为出名的RPC框架。**Dubbo在Socket编程方面是用的Netty来实现，使用自定义报文的tcp协议。**协议和序列化框架都可以插拔是及其鲜明的特色。同样的远程接口是基于Java Interface，并且依托于spring框架方便开发。可以方便的打包成单一文件，独立进程运行，和现在的微服务概念一致。

### 

## Thrift

### 介绍

Thrift是一个跨语言的服务部署框架，最初由Facebook于2007年开发，2008年进入Apache开源项目。

Thrift通过IDL（Interface Definition Language，接口定义语言）来定义RPC（Remote Procedure Call，远程过程调用）的接口和数据类型。用来进行可扩展且跨语言的服务的开发。它结合了功能强大的软件堆栈和代码生成引擎，以构建在 C++, Java, Go,Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, and OCaml 这些编程语言间无缝结合的、高效的服务,并由生成的代码负责RPC协议层和传输层的实现。

简单的来说就是各种不同的语言，根据统一定义好的Thrift规范文件通过Thrift软件生成各自对应的文件，并使用这些文件来达到相互通信的目的。

Thrift是基于TCP协议的

Thrift中所有的传输层协议的基类是TTransport

### 工程结构

- thrift/和compiler/包含C++实现的Thrift编译器。
- lib/ 包含Thrift软件库的实现, 细分为不同语言的实现。
- cpp/，go/，java/，php/，py/ ，rb/ … test/包含Thrift示例文件以及跨目标语言的测试代码。
- tutorial/包含一个如何使用Thrift开发软件的教程。 

### 优点

Thrift的优点有很多，比如省流量（二进制传输）、性能高（使用Socket通信）、跨语言、数据结构统一。

### 缺点

数据结构需要提前定义，一旦发生变化，相关方都需要重新编译等。

### Thrift工作原理

**2、Thrift的RPC调用过程**

远程过程调用（RPC）的调用方和被调用方不在同一个进程内，甚至都不在同一台机子上，因此远程过程调用中，必然涉及网络传输；假设有下列代码

![](/Users/wangchong/Pictures/Typora/rpc调用代码.jpg)

#### **Thrift框架的远程过程调用的工作过程**

1、通过IDL定义一个接口的thrift文件，然后通过thrift的多语言编译功能，将接口定义的thrift文件翻译成对应的语言版本的接口文件；

2、Thrift生成的特定语言的接口文件中包括客户端部分和服务器部分；

3、客户端通过接口文件中的客户端部分生成一个Client对象，这个客户端对象中包含所有接口函数的存根实现，然后用户代码就可以通过这个Client对象来调用thrift文件中的那些接口函数了，但是，客户端调用接口函数时实际上调用的是接口函数的本地存根实现，如图3.2中的箭头1所示；

4、接口函数的存根实现将调用请求发送给thrift服务器端，然后thrift服务器根据调用的函数名和函数参数，调用实际的实现函数来完成具体的操作，如图3.2中的箭头2所示；

5、Thrift服务器在完成处理之后，将函数的返回值发送给调用的Client对象；如图3.2中的箭头3所示；

6、Thrift的Client对象将函数的返回值再交付给用户的调用函数，如图3.2中的箭头4所示；

#### **说明**

1、本地函数的调用方和被调方在同一进程的地址空间内部，因此在调用时cpu还是由当前的进行所持有，只是在调用期间，cpu去执行被调用函数，从而导致调用方被卡在那里，直到cpu执行完被调用函数之后，才能切换回来继续执行调用之后的代码；

2、RPC在调用方和被调用方一般不在一台机子上，它们之间通过网络传输进行通信，一般的RPC都是采用tcp连接，如果同一条tcp连接同一时间段只能被一个调用所独占，这种情况与[1]中的本地过程更为相似，这种情况是同步调用，很显然，这种方式通信的效率比较低，因为服务函数执行期间，tcp连接上没有数据传输还依然被本次调用所霸占；另外，这种方式也有优点：实现简单。

3、在一些的RPC服务框架中，为了提升网络通信的效率，客户端发起调用之后不被阻塞，这种情况是异步调用，它的通信效率比同步调用高，但是实现起来比较复杂。

#### **Thrift源码分析**

　　源码分析主要分析thrift生成的java接口文件，并以TestThriftService.java为例，以该文件为线索，逐渐分析文件中遇到的其他类和文件；在thrift生成的服务接口文件中，共包含以下几部分：

　　1、异步客户端类AsyncClient和异步接口AsyncIface，本节暂不涉及这些异步操作相关内容；

　　2、同步客户端类Client和同步接口Iface，Client类继承自TServiceClient，并实现了同步接口Iface；Iface就是根据thrift文件中所定义的接口函数所生成；Client类是在开发Thrift的客户端程序时使用，Client类是Iface的客户端存根实现， Iface在开发Thrift服务器的时候要使用，Thrift的服务器端程序要实现接口Iface。

　　3、Processor类，该类主要是开发Thrift服务器程序的时候使用，该类内部定义了一个map，它保存了所有函数名到函数对象的映射，一旦Thrift接到一个函数调用请求，就从该map中根据函数名字找到该函数的函数对象，然后执行它；

　　4、参数类，为每个接口函数定义一个参数类，例如：为接口getInt产生一个参数类：getInt_args，一般情况下，接口函数参数类的命名方式为：接口函数名_args;

　　5、返回值类，每个接口函数定义了一个返回值类，例如：为接口getInt产生一个返回值类：getInt_result，一般情况下，接口函数返回值类的命名方式为：接口函数名_result;

　　参数类和返回值类中有对数据的读写操作，在参数类中，将按照协议类将调用的函数名和参数进行封装，在返回值类中，将按照协议规定读取数据。

　　Thrift调用过程中，Thrift客户端和服务器之间主要用到传输层类、协议层类和处理类三个主要的核心类，这三个类的相互协作共同完成rpc的整个调用过程。在调用过程中将按照以下顺序进行协同工作：

　　1、将客户端程序调用的函数名和参数传递给协议层（TProtocol），协议层将函数名和参数按照协议格式进行封装，然后封装的结果交给下层的传输层。此处需要注意：要与Thrift服务器程序所使用的协议类型一样，否则Thrift服务器程序便无法在其协议层进行数据解析；

　　2、传输层（TTransport）将协议层传递过来的数据进行处理，例如传输层的实现类TFramedTransport就是将数据封装成帧的形式，即“数据长度+数据内容”，然后将处理之后的数据通过网络发送给Thrift服务器；此处也需要注意：要与Thrift服务器程序所采用的传输层的实现类一致，否则Thrift的传输层也无法将数据进行逆向的处理；

　　3、Thrift服务器通过传输层（TTransport）接收网络上传输过来的调用请求数据，然后将接收到的数据进行逆向的处理，例如传输层的实现类TFramedTransport就是将“数据长度+数据内容”形式的网络数据，转成只有数据内容的形式，然后再交付给Thrift服务器的协议类（TProtocol）；

　　4、Thrift服务端的协议类（TProtocol）将传输层处理之后的数据按照协议进行解封装，并将解封装之后的数据交个Processor类进行处理；

　　5、Thrift服务端的Processor类根据协议层（TProtocol）解析的结果，按照函数名找到函数名所对应的函数对象；

　　6、Thrift服务端使用传过来的参数调用这个找到的函数对象；

　　7、Thrift服务端将函数对象执行的结果交给协议层；

　　8、Thrift服务器端的协议层将函数的执行结果进行协议封装；

　　9、Thrift服务器端的传输层将协议层封装的结果进行处理，例如封装成帧，然后发送给Thrift客户端程序；

　　10、Thrift客户端程序的传输层将收到的网络结果进行逆向处理，得到实际的协议数据；

　　11、Thrift客户端的协议层将数据按照协议格式进行解封装，然后得到具体的函数执行结果，并将其交付给调用函数；

　　上述过程如下图

![](/Users/wangchong/Pictures/Typora/Thrift的各类协同工作.jpg)

上图的客户端协议类和服务端协议类都是指具体实现了TProtocol接口的协议类，在实际开发过程中二者必须一样，否则便不能进行通信；同样，客户端传输类和服务端传输类是指TTransport的子类，二者也需保持一致；

在上述开发thrift客户端和服务器端程序时需要用到三个类：传输类（TTransport）、协议接口（TProtocol）和处理类（Processor），其中TTransport是抽象类，在实际开发过程中可根据具体清空选择不同的实现类；TProtocol是个协议接口，每种不同的协议都必须实现此接口才能被thrift所调用。例如TProtocol类的实现类就有TBinaryProtocol等；在Thrift生成代码的内部，还需要将待传输的内容封装成消息类TMessage。处理类（Processor）主要在开发Thrift服务器端程序的时候使用。

##### 1、TMessage

Thrift在客户端和服务器端传递数据的时候（包括发送调用请求和返回执行结果），都是将数据按照TMessage进行组装，然后发送；

TMessage包括三部分：

- 消息的名称

  > 消息名称为字符串类型

- 消息的序列号

  > 消息的序列号为32位的整形

- 消息的类型

> 消息的类型为byte类型

消息的类型共有如下17种：

```java
public final classTType {

     public staticfinal byte STOP  =0;

     public staticfinal byte VOID  =1;

     public staticfinal byte BOOL  =2;

     public staticfinal byte BYTE  =3;

     public staticfinal byte DOUBLE = 4;

     public staticfinal byte I16   =6;

     public staticfinal byte I32   =8;

     public staticfinal byte I64   =10;

     public staticfinal byte STRING = 11;

     public staticfinal byte STRUCT = 12;

     public staticfinal byte MAP   =13;

     public staticfinal byte SET   =14;

     public staticfinal byte LIST  =15;

     public staticfinal byte ENUM  =16;

}
```

　　Byte共可表示0~255个数字，这里只是用了前17个

##### **2、传输类（TTransport）**

　　传输类或其各种实现类，都是对I/O层的一个封装，可更直观的理解为它封装了一个socket，不同的实现类有不同的封装方式，例如TFramedTransport类，它里面还封装了一个读写buf，在写入的时候，数据都先写到这个buf里面，等到写完调用该类的flush函数的时候，它会将写buf的内容，封装成帧再发送出去；

　　TFramedTransport是对TTransport的继承，由于tcp是基于字节流的方式进行传输，因此这种基于帧的方式传输就要求在无头无尾的字节流中每次写入和读出一个帧，TFramedTransport是按照下面的方式来组织帧的：每个帧都是按照4字节的帧长加上帧的内容来组织，帧内容就是我们要收发的数据，如下：

 +---------------+---------------+

 |  4字节的帧长  |  帧的内容    |

 +---------------+---------------+

TTransport这个基类有哪些common的抽象函数：

isOpen:用户判断底层传输链路是否是ready的；
open:用于打开底层的传输链路；
close:用于关闭底层传输链路；
read:用于从链路中读取数据；
write:用于往链路中写入数据；
flush:用于将内存中的buffer数据写到链路中；
getBufferPosition:返回链路底层buffer数据当前read位置；
getBuffer:用于返回底层buffer数据
getBytesRemainingInBuffer:用于返回当前底层buffer中还有多少数据没有读取；
consumeBuffer：从底层buffer数据中读取一些数据；

##### Socket通信

Thrift的客户端代码同样需要服务器开头的那两步：添加三个jar包和生成的java接口文件TestThriftService.java。

```java
	 m_transport = new TSocket(THRIFT_HOST, THRIFT_PORT,2000);
	 TProtocol protocol = new TBinaryProtocol(m_transport);
	 TestThriftService.Client testClient = new TestThriftService.Client(protocol);
	
	 try {
		 m_transport.open();
		 
		 String res = testClient.getStr("test1", "test2");
		 System.out.println("res = " + res);
		 m_transport.close();
	} catch (TException e){
			// TODO Auto-generated catch block
			e.printStackTrace();
	}
```
代码2.4

由代码2.4可以看到编写客户端代码非常简单，只需下面几步即可：

[1]创建一个传输层对象（TTransport），具体采用的传输方式是TFramedTransport，要与服务器端保持一致，即：

m_transport =new TFramedTransport(newTSocket(THRIFT_HOST,THRIFT_PORT, 2000));

这里的THRIFT_HOST, THRIFT_PORT分别是Thrift服务器程序的主机地址和监听端口号，这里的2000是socket的通信超时时间；

[2]创建一个通信协议对象（TProtocol），具体采用的通信协议是二进制协议，这里要与服务器端保持一致，即：

TProtocolprotocol =new TBinaryProtocol(m_transport);

[3]创建一个Thrift客户端对象（TestThriftService.Client），Thrift的客户端类TestThriftService.Client已经在文件TestThriftService.java中，由Thrift编译器自动为我们生成，即：

TestThriftService.ClienttestClient =new TestThriftService.Client(protocol);

[4]打开socket，建立与服务器直接的socket连接，即：

m_transport.open();

[5]通过客户端对象调用服务器服务函数getStr，即：

String res = testClient.getStr("test1","test2");

System.out.println("res = " +res);

[6]使用完成关闭socket，即：

m_transport.close();

这里有以下几点需要说明：

[1]在同步方式使用客户端和服务器的时候，socket是被一个函数调用独占的，不能多个调用同时使用一个socket，例如通过m_transport.open()打开一个socket，此时创建多个线程同时进行函数调用，这时就会报错，因为socket在被一个调用占着的时候不能再使用；

[2]可以分时多次使用同一个socket进行多次函数调用，即通过m_transport.open()打开一个socket之后，你可以发起一个调用，在这个次调用完成之后，再继续调用其他函数而不需要再次通过m_transport.open()打开socket；

### 使用流程

- 将thrift文件拷贝到 /usr/local/bin 路径下

  > 从官方网址（[Apache Thrift - Download](https://link.zhihu.com/?target=http%3A//thrift.apache.org/download)）下载对应的安装包并进行安装

- 终端执行`chmod 777 /usr/local/bin/thrift`增加权限

- 执行 thrift -version 看下版本号，我用的是0.9.2

- 写thrift代码

  ![image-20200219172653455](/Users/wangchong/Library/Application Support/typora-user-images/image-20200219172653455.png)

  - 配置命名空间

    ```java
    namespace java ngcc.im.session.client
    ```

  - 编写代码

    ```c++
    include "base.thrift"
    namespace java ngcc.im.session.client
    
    /*结构体*/
    struct UserDto {
        1: optional i64 userId,
        2: optional string token,
        3: optional i32 status,
        4: optional i32 currentConversationCount
    }
    
    struct DialogDto {
        1: optional i64 toUserId,
        2: optional string toUserName,
        3: optional i64 startTime,
        4: optional i64 endTime,
        5: optional string conversationId,
        6: optional i64 conversationShortId,
        7: optional i32 type,
        8: optional i32 status
        9: optional i64 fromUserId,
        10: optional string fromUserName,
        11: optional i64 id
    }
    
    /*请求协议*/
    struct GetUserTokenRequest {
        1: optional i64 userId,
        255: required base.Base base
    }
    
    struct GetUserTokenResponse {
        1: optional UserDto userDto,
        255: required base.BaseResp baseResp
    }
    
    struct GetUserStatusRequest {
        1: optional i64 userId,
        255: required base.Base base
    }
    
    struct GetUserStatusResponse {
        1: optional UserDto userDto,
        255: required base.BaseResp baseResp
    }
    
    struct GetUserStatusBatchRequest {
        1: optional list<i64> userIdList,
        255: required base.Base base
    }
    
    struct GetUserStatusBatchResponse {
        1: optional list<UserDto> userDtoList,
        255: required base.BaseResp baseResp
    }
    
    struct CreateDialogRequest {
        1: optional DialogDto dialogDto,
        255: required base.Base base
    }
    
    struct CreateDialogResponse {
        1: optional DialogDto dialogDto,
        255: required base.BaseResp baseResp
    }
    
    struct FinishDialogRequest {
        1: optional i64 dialogId,
        255: required base.Base base
    }
    
    struct FinishDialogResponse {
        1: optional DialogDto dialogDto,
        255: required base.BaseResp baseResp
    }
    
    struct GetCurrentDialogRequest {
        1: optional i64 userId,
        255: required base.Base base
    }
    
    struct GetCurrentDialogResponse {
        1: list<DialogDto> dialogDtoList,
        255: required base.BaseResp baseResp
    }
    
    struct GetFinishedDialogRequest {
        1: optional i64 userId,
        255: required base.Base base
    }
    
    struct GetFinishedDialogResponse {
        1: list<DialogDto> dialogDtoList,
        255: required base.BaseResp baseResp
    }
    
    struct LoginUserRequest {
        1: optional i64 userId
        255: required base.Base base
    }
    
    struct LoginUserResponse {
        1: optional i64 userId
        255: required base.BaseResp baseResp
    }
    
    struct LogoutUserRequest {
        1: optional i64 userId
        255: required base.Base base
    }
    
    struct LogoutUserResponse {
        1: optional i64 userId
        255: required base.BaseResp baseResp
    }
    
    service ImSessionThrift {
        //获取UserToken
        GetUserTokenResponse getUserToken(1:GetUserTokenRequest request)
        //获取用户状态
        GetUserStatusResponse getUserStatus(1:GetUserStatusRequest request)
        //批量获取用户状态
        GetUserStatusBatchResponse getUserStatusBatch(1:GetUserStatusBatchRequest request)
        //创建会话
        CreateDialogResponse createDialog(1:CreateDialogRequest request)
        //结束会话
        FinishDialogResponse finishDialog(1:FinishDialogRequest request)
        //获取当前会话列表
        GetCurrentDialogResponse getCurrentDialog(1:GetCurrentDialogRequest request)
        //获取已结束会话列表
        GetFinishedDialogResponse getFinishedDialog(1:GetFinishedDialogRequest request)
        //用户上线
        LoginUserResponse loginUser(1:LoginUserRequest request)
        //用户下线
        LogoutUserResponse logoutUser(1:LogoutUserRequest request)
    }
    ```

  - 引入依赖

    ```xml
    <dependency>
        <groupId>org.apache.thrift</groupId>
        <artifactId>libthrift</artifactId>
    </dependency>
    ```

- 去项目里编译compile就行了

  - 直接compile

  ![image-20200219171649803](/Users/wangchong/Library/Application Support/typora-user-images/image-20200219171649803.png)

  - 执行命令，生成Thrift文件（生成之后将文件复制到项目目录）

    ```bash
    thrift -r -gen java ../UserService.thrift
    ```

  就会在指定文件自动编写代码

  ![image-20200219172905823](/Users/wangchong/Library/Application Support/typora-user-images/image-20200219172905823.png)

### Thrift语法

#### 数据类型

> Thrift类型系统包括预定义基本类型，用户自定义结构体，容器类型，异常和服务定义

##### 基本类型

- bool: 布尔类型，占一个字节
- byte: 有符号字节
- i16：16位有符号整型
- i32：32位有符号整型
- i64：64位有符号整型
- double：64位浮点数
- string：未知编码或者二进制的字符串

注意：thrift不支持无符号整形，因为很多目标语言不存在无符号整形（比如java）

##### 容器类型

`List<t1>`：一系列t1类型的元素组成的有序列表，元素可以重复

`Set<t1>`：一些t1类型的元素组成的无序集合，元素唯一不重复

`Map<t1,t2>`：key/value对，key唯一

容器中的元素类型可以是除service以外的任何合法的thrift类型，包括结构体和异常类型

##### 结构体

> Thrift结构体在概念上同c语言的结构体类似，在面向对象语言中，thrift结构体将被转化为类。

##### 异常

> 异常在语法和功能上类似于结构体，只是异常使用关键字exception而不是struct关键字来声明。但它在语义上不同于结构体—当定义一个RPC服务时，开发者可能需要声明一个远程方法抛出一个异常。

##### 服务

> Thrift中服务定义的方式和语法等同于面向对象语言中定义接口。Thrift编译器会产生实现接口的client和server桩。

##### 类型定义

```java
typedef i32 MyInteger   //末尾没有逗号
typedef Tweet ReTweet   //struct可以使用typedef
```

##### 枚举类型

可以像C/C++那样定义枚举类型，如：

```c++
enum TweetType {
 
    TWEET,       //编译器默认从0开始赋值

    RETWEET = 2, //可以赋予某个常量

    DM = 0xa,  //允许常量是十六进制整数

    REPLY

}        //末尾没有逗号
 
struct Tweet {
 
    1: required i32 userId;

    2: required string userName;

    3: required string text;

    4: optional Location loc;

    5: optional TweetType tweetType = TweetType.TWEET // 给常量赋缺省值时，使用常量的全称

    16: optional string language = "english"

}
```

不同于protocol buffer，thrift不支持枚举类嵌套，枚举常量必须是32位正整数

#### 注释

Thrift支持shell注释风格、C/C++语言中的单行或多行注释风格

```java
# This is a valid comment.
 
/*
* This is a multi-line comment.
* Just like in C.
*/
 
// C++/Java style single-line comments work just as well.
```

#### 命名空间

Thrift中的命名空间同C++中的namespace和java中的package类似，它们均提供了一种组织（隔离）代码的方式。因为每种语言均有自己的命名空间定义方式（如python中有module），thrift允许开发者针对特定语言定义namespace：

```c++
namespace cpp com.example.project  // a 
namespace java com.example.project // b
```

#### 文件包含

Thrift允许文件包含，需要使用thrift文件名作为前缀访问被包含的对象，如：

```c
include "tweet.thrift"           // thrift文件名需要双引号包含，末尾没有逗号或者分号
 
struct TweetSearchResult {
		1: list<tweet.Tweet> tweets; // 注意tweet前缀
}
```

#### 常量

Thrift允许用户定义常量，复杂的类型和结构体可以使用JSON形式表示：

```
const i32 INT_CONST = 1234;    // 分号是可选的，支持十六进制赋值
 
const map<string,string> MAP_CONST = {"hello": "world", "goodnight": "moon"}
```

#### 定义结构体

```c
struct Tweet {
 
    1: required i32 userId;                  // 每一个域都有一个唯一的正整数标识符

    2: required string userName;             // 每个域可以标识为required或者																									optional（也可以不注明）
    3: required string text;

    4: optional Location loc;                // 结构体可以包含其他结构体

    16: optional string language = "english" // 域可以有缺省值

}
 
struct Location {                            // 一个thrift中可以定义多个结构体，																								 并存在引用关系
    1: required double latitude;

    2: required double longitude;
 
}
```

规范的struct定义中的每个域均会使用required或者optional关键字进行标识。

如果required标识的域没有赋值，thrift将给予提示。

如果optional标识的域没有赋值，该域将不会被序列化传输。如果某个optional标识域有缺省值而用户没有重新赋值，则该域的值一直为缺省值。
与service不同，结构体不支持继承，即，一个结构体不能继承另一个结构体。

#### 定义服务

在流行的序列化/反序列化框架（如protocol buffer）中，thrift是少有的提供多语言间RPC服务的框架。
Thrift编译器会根据选择的目标语言为server产生服务接口代码，为client产生桩代码。

```c
//“Twitter”与“{”之间需要有空格！！！
service Twitter {
// 方法定义方式类似于C语言中的方式，它有一个返回值，一系列参数和可选的异常
// 列表. 注意，参数列表和异常列表定义方式与结构体中域定义方式一致.

    void ping(),                    // 函数定义可以使用逗号或分号标识结束
  
    bool postTweet(1:Tweet tweet);  // 参数可以是基本类型或者结构体，参数只能是只读																				的（const），不可以作为返回值
    TweetSearchResult searchTweets(1:string query); // 返回值可以是基本类型或者																												 结构体
    // ”oneway”标识符表示client发出请求后不必等待回复（非阻塞）直接进行下面的操作，
    // ”oneway”方法的返回值必须是void
		oneway void zip() // 返回值可以是void
}
```

注意，函数中参数列表的定义方式与struct完全一样

Service支持继承，一个service可使用extends关键字继承另一个service。

## Dubbo

### **Dubbo是什么？**

Dubbo是阿里巴巴开源的基于 Java 的高性能 RPC 分布式服务框架，现已成为 Apache 基金会孵化项目。

面试官问你如果这个都不清楚，那下面的就没必要问了。

> 官网：[http://dubbo.apache.org](https://link.zhihu.com/?target=http%3A//dubbo.apache.org)

### **为什么要用Dubbo？**

随着互联网的发展，网站的应用规模不断扩大，常规的垂直架构已经无法应，分布式服务架构势在必行，亟需一个治理系统架构的方案。

　　1)单一架构，当网站流量很小，我们将所有的功能都部署到一起，减少部署节点和成本。此时，用于简化增删改工作量，ORM是关键

　　2)垂直架构，当访问逐渐增大，单一机器的速度显然不理想，将应用拆成几个不相干的应用，以便提升效率。此时，用于加速前端访问，MVC是关键。

　　3)分布式服务架构，当垂直应用越来越多，应用之间的交互不可避免。将核心的业务抽取出来，作为独立的服务，使前端应用可以快速响应多变的市场需求。此时，提高业务的复用整合的RPC框架是关键。

　　4)当我们服务越来越多，容量评估以及小服务资源浪费的问题逐渐展现出来。此时就需要一个调度中心基于访问压力实时的去管理集群容量，提高集群利用率。此时用于提高集群利用率和资源问题soa是关键。

因为是阿里开源项目，国内很多互联网公司都在用，已经经过很多线上考验。内部使用了 Netty、Zookeeper，保证了高性能高可用性。

使用 Dubbo 可以将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，可用于提高业务复用灵活扩展，使前端应用能更快速的响应多变的市场需求。

下面这张图可以很清楚的诠释，最重要的一点是，分布式架构可以承受更大规模的并发流量。



![img](https://pic4.zhimg.com/80/v2-74de60b0d8b345c9f7556ef21cf0810b_1440w.jpg)



下面是 Dubbo 的服务治理图。

![img](https://pic2.zhimg.com/80/v2-6da957a755d37928916bb6e84afbb599_1440w.jpg)



### dubbo为什么需要和zookeeper结合使用，zookeeper在dubbo体系中起到什么作用？  

　　dubbo是一个prc远程服务调用框架，需要一个注册中心去管理每个服务的集群。zookeeper在dubbo中扮演一个注册中心的角色（当然也可以不选择zookeeper）,zookeeper用来注册服务和进行负载均衡。

　　详述：哪一个服务由哪一个机器来提供，必须让调用者知道。也就是ip地址和服务名对应关系。也可以把这种对应关系通过硬编码的方式加在调用者的业务中，但是一旦提供的服务挂掉调用者无法知晓。zookeeper通过心跳机制来检测并将挂掉的机器从列表中删除。可以在不更改代码的情况下通过添加机器来解决高并发。

### Dubbo 和 Spring Cloud 有什么区别？

两个没关联，如果硬要说区别，有以下几点。

1）通信方式不同

Dubbo 使用的是 RPC 通信，而 Spring Cloud 使用的是 HTTP RESTFul 方式。

2）组成部分不同

组件 | Dubbo | Spring Cloud ---|---|--- 服务注册中心 | Zookeeper | Spring Cloud Netflix Eureka 服务监控 | Dubbo-monitor | Spring Boot Admin 断路器 | 不完善 | Spring Cloud Netflix Hystrix 服务网关 | 无 | Spring Cloud Netflix Gateway 分布式配置 | 无 | Spring Cloud Config 服务跟踪 | 无 | Spring Cloud Sleuth 消息总线 | 无 | Spring Cloud Bus 数据流 | 无 | Spring Cloud Stream 批量任务 | 无 | Spring Cloud Task ... | ... | ...

### **支持的协议，推荐用哪种？**

- dubbo://（推荐）
- rmi://
- hessian://
- http://
- webservice://
- thrift://
- memcached://
- redis://
- rest://

### **需要 Web 容器吗？**

不需要，如果硬要用 Web 容器，只会增加复杂性，也浪费资源。

### **内置了哪几种服务容器？**

- Spring Container
- Jetty Container
- Log4j Container

Dubbo 的服务容器只是一个简单的 Main 方法，并加载一个简单的 Spring 容器，用于暴露服务。

### **有哪几种节点角色？**

节点 | 角色说明 ---|--- Provider | 暴露服务的服务提供方 Consumer | 调用远程服务的服务消费方 Registry | 服务注册与发现的注册中心 Monitor | 统计服务的调用次数和调用时间的监控中心 Container | 服务运行容器

### **画一画服务注册与发现的流程图**

![img](https://pic3.zhimg.com/80/v2-4e1b887d19f03d131d9969c36d19e98e_1440w.jpg)



该图来自 Dubbo 官网，供你参考，如果你说你熟悉 Dubbo, 面试官经常会让你画这个图，记好了。

### **Dubbo默认使用什么注册中心，还有别的选择吗？**

推荐使用 Zookeeper 作为注册中心，还有 Redis、Multicast、Simple 注册中心，但不推荐。

### **Dubbo有哪几种配置方式？**

1）Spring 配置方式 2）Java API 配置方式

### **Dubbo 核心的配置有哪些？**

我曾经面试就遇到过面试官让你写这些配置，我也是蒙逼。。

配置 | 配置说明 ---|--- dubbo:service | 服务配置 dubbo:reference | 引用配置 dubbo:protocol | 协议配置 dubbo:application | 应用配置 dubbo:module | 模块配置 dubbo:registry | 注册中心配置 dubbo:monitor | 监控中心配置 dubbo:provider | 提供方配置 dubbo:consumer | 消费方配置 dubbo:method | 方法配置 dubbo:argument | 参数配置

配置之间的关系见下图。



![img](https://pic2.zhimg.com/80/v2-b9ede3a7c70677cc460e74b75147010d_1440w.jpg)



### **在 Provider 上可以配置的 Consumer 端的属性有哪些？**

1）timeout：方法调用超时 2）retries：失败重试次数，默认重试 2 次 3）loadbalance：负载均衡算法，默认随机 4）actives 消费者端，最大并发调用限制

### **Dubbo启动时如果依赖的服务不可用会怎样？**

Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，默认 check="true"，可以通过 check="false" 关闭检查。

### **Dubbo推荐使用什么序列化框架，你知道的还有哪些？**

推荐使用Hessian序列化，还有Duddo、FastJson、Java自带序列化。

### **Dubbo默认使用的是什么通信框架，还有别的选择吗？**

Dubbo 默认使用 Netty 框架，也是推荐的选择，另外内容还集成有Mina、Grizzly。

### **16、Dubbo有哪几种集群容错方案，默认是哪种？**

集群容错方案 | 说明 ---|--- Failover Cluster | 失败自动切换，自动重试其它服务器（默认） Failfast Cluster | 快速失败，立即报错，只发起一次调用 Failsafe Cluster | 失败安全，出现异常时，直接忽略 Failback Cluster | 失败自动恢复，记录失败请求，定时重发 Forking Cluster | 并行调用多个服务器，只要一个成功即返回 Broadcast Cluster | 广播逐个调用所有提供者，任意一个报错则报错

### **Dubbo有哪几种负载均衡策略，默认是哪种？**

负载均衡策略 | 说明 ---|--- Random LoadBalance | 随机，按权重设置随机概率（默认） RoundRobin LoadBalance | 轮询，按公约后的权重设置轮询比率 LeastActive LoadBalance | 最少活跃调用数，相同活跃数的随机 ConsistentHash LoadBalance | 一致性 Hash，相同参数的请求总是发到同一提供者

### **注册了多个同一样的服务，如果测试指定的某一个服务呢？**

可以配置环境点对点直连，绕过注册中心，将以服务接口为单位，忽略注册中心的提供者列表。

### **Dubbo支持服务多协议吗？**

Dubbo 允许配置多协议，在不同服务上支持不同协议或者同一服务上同时支持多种协议。

### **当一个服务接口有多种实现时怎么做？**

当一个接口有多种实现时，可以用 group 属性来分组，服务提供方和消费方都指定同一个 group 即可。

### **服务上线怎么兼容旧版本？**

可以用版本号（version）过渡，多个不同版本的服务注册到注册中心，版本号不同的服务相互间不引用。这个和服务分组的概念有一点类似。

### **Dubbo可以对结果进行缓存吗？**

可以，Dubbo 提供了声明式缓存，用于加速热门数据的访问速度，以减少用户加缓存的工作量。

### **Dubbo服务之间的调用是阻塞的吗？**

默认是同步等待结果阻塞的，支持异步调用。

Dubbo 是基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小，异步调用会返回一个 Future 对象。

异步调用流程图如下。



![img](https://pic4.zhimg.com/80/v2-e310fca48b875bd334e0eec6d53d0e3b_1440w.jpg)



### **Dubbo支持分布式事务吗？**

目前暂时不支持，后续可能采用基于 JTA/XA 规范实现，如以图所示。



![img](https://pic2.zhimg.com/80/v2-fd1722167a4e4042406fea8c8abef3d1_1440w.jpg)



### **Dubbo telnet 命令能做什么？**

dubbo 通过 telnet 命令来进行服务治理，具体使用看这篇文章《[dubbo服务调试管理实用命令](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/0OSXVundismrRSR4mG-kEQ)》。

> telnet localhost 8090

### **Dubbo支持服务降级吗？**

Dubbo 2.2.0 以上版本支持。

### **Dubbo如何优雅停机？**

Dubbo 是通过 JDK 的 ShutdownHook 来完成优雅停机的，所以如果使用 kill -9 PID 等强制关闭指令，是不会执行优雅停机的，只有通过 kill PID 时，才会执行。

### **服务提供者能实现失效踢出是什么原理？**

服务失效踢出基于 Zookeeper 的临时节点原理。

### **如何解决服务调用链过长的问题？**

Dubbo 可以使用 Pinpoint 和 Apache Skywalking(Incubator) 实现分布式服务追踪，当然还有其他很多方案。

### **服务读写推荐的容错策略是怎样的？**

读操作建议使用 Failover 失败自动切换，默认重试两次其他服务器。

写操作建议使用 Failfast 快速失败，发一次调用失败就立即报错。

### **Dubbo必须依赖的包有哪些？**

Dubbo 必须依赖 JDK，其他为可选。

### **Dubbo的管理控制台能做什么？**

管理控制台主要包含：路由规则，动态配置，服务降级，访问控制，权重调整，负载均衡，等管理功能。

### **说说 Dubbo 服务暴露的过程。**

Dubbo 会在 Spring 实例化完 bean 之后，在刷新容器最后一步发布 ContextRefreshEvent 事件的时候，通知实现了 ApplicationListener 的 ServiceBean 类进行回调 onApplicationEvent 事件方法，Dubbo 会在这个方法中调用 ServiceBean 父类 ServiceConfig 的 export 方法，而该方法真正实现了服务的（异步或者非异步）发布。

### **Dubbo 停止维护了吗？**

2014 年开始停止维护过几年，17 年开始重新维护，并进入了 Apache 项目。

### **Dubbo 和 Dubbox 有什么区别？**

Dubbox 是继 Dubbo 停止维护后，当当网基于 Dubbo 做的一个扩展项目，如加了服务可 Restful 调用，更新了开源组件等。

### **你还了解别的分布式框架吗？**

别的还有 Spring cloud、Facebook 的 Thrift、Twitter 的 Finagle 等。

### **Dubbo 能集成 Spring Boot 吗？**

可以的，项目地址如下。

> [https://github.com/apache/incubator-dubbo-spring-boot-project](https://link.zhihu.com/?target=https%3A//github.com/apache/incubator-dubbo-spring-boot-project)

### **在使用过程中都遇到了些什么问题？**

Dubbo 的设计目的是为了满足高并发小数据量的 rpc 调用，在大数据量下的性能表现并不好，建议使用 rmi 或 http 协议。

### **你读过 Dubbo 的源码吗？**

要了解 Dubbo 就必须看其源码，了解其原理，花点时间看下吧，网上也有很多教程，后续有时间我也会在公众号上分享 Dubbo 的源码。

### **你觉得用 Dubbo 好还是 Spring Cloud 好？**

扩展性的问题，没有好坏，只有适合不适合，不过我好像更倾向于使用 Dubbo, Spring Cloud 版本升级太快，组件更新替换太频繁，配置太繁琐，还有很多我觉得是没有 Dubbo 顺手的地方……

