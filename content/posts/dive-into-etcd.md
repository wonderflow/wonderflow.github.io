+++
title = "Dive into etcd"
date = "2015-02-02T10:57:55Z"
slug = "2015/02/02/dive-into-etcd"
Categories = []
+++

# etcd：从应用场景到实现原理的全方位解读

随着CoreOS和Kubernetes等项目在开源社区日益火热，它们项目中都用到的etcd组件作为一个高可用、强一致性的服务发现存储仓库，渐渐为开发人员所关注。在云计算时代，如何让服务快速透明地接入到计算集群中，如何让共享配置信息快速被集群中的所有机器发现，更为重要的是，如何构建这样一套高可用、安全、易于部署以及响应快速的服务集群，已经成为了迫切需要解决的问题。etcd为解决这类问题带来了福音，本章将从etcd的应用场景开始，深入解读etcd的实现方式，以供开发者们更为充分地享用etcd所带来的便利。

## 1 etcd经典应用场景
etcd是什么？很多人对这个问题的第一反应可能是，它是一个键值存储仓库，却没有重视官方定义的后半句，用于配置共享和服务发现。
> **A highly-available key value store for shared configuration and service discovery.**

实际上，etcd作为一个受到Zookeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更具有以下4个特点{![引自Docker官方文档]}。
<!--more-->


* 简单：基于HTTP+JSON的API让你用curl命令就可以轻松使用。
* 安全：可选SSL客户认证机制。
* 快速：每个实例每秒支持一千次写操作。
* 可信：使用Raft算法充分实现了分布式。


随着云计算的不断发展，分布式系统中涉及的问题越来越受到人们重视。受阿里中间件团队对[ZooKeeper典型应用场景一览](http://jm-blog.aliapp.com/?p=1232)一文的启发{![部分案例引自此文。]}，我根据自己的理解也总结了一些etcd的经典使用场景。值得注意的是，分布式系统中的数据分为控制数据和应用数据。使用etcd的场景处理的数据默认为控制数据，对于应用数据，只推荐处理数据量很小，但是更新访问频繁的情况。


### 1.1 场景一：服务发现
服务发现（Service Discovery）要解决的是分布式系统中最常见的问题之一，即在同一个分布式集群中的进程或服务如何才能找到对方并建立连接。从本质上说，服务发现就是想要了解集群中是否有进程在监听udp或tcp端口，并且通过名字就可以进行查找和连接。要解决服务发现的问题，需要有下面三大支柱，缺一不可。

- **一个强一致性、高可用的服务存储目录**。基于Raft算法的etcd天生就是这样一个强一致性高可用的服务存储目录。
- **一种注册服务和监控服务健康状态的机制**。用户可以在etcd中注册服务，并且对注册的服务设置`key TTL`，定时保持服务的心跳以达到监控健康状态的效果。
- **一种查找和连接服务的机制**。通过在etcd指定的主题下注册的服务也能在对应的主题下查找到。为了确保连接，我们可以在每个服务机器上都部署一个proxy模式的etcd，这样就可以确保能访问etcd集群的服务都能互相连接。

图1所示为服务发现示意图。
![enter image description here][1]

图1 服务发现示意图

下面我们来看一下服务发现对应的具体应用场景。

* **微服务协同工作架构中，服务动态添加**。随着Docker容器的流行，多种微服务共同协作，构成一个功能相对强大的架构的案例越来越多。透明化的动态添加这些服务的需求也日益强烈。通过服务发现机制，在etcd中注册某个服务名字的目录，在该目录下存储可用的服务节点的IP。在使用服务的过程中，只要从服务目录下查找可用的服务节点进行使用即可。
微服务协同工作如图2所示。
![enter image description here][2]

图2 微服务协同工作


* **PaaS平台中应用多实例与实例故障重启透明化**。PaaS平台中的应用一般都有多个实例，通过域名，不仅可以透明地对多个实例进行访问，而且还可以实现负载均衡。但是应用的某个实例随时都有可能故障重启，这时就需要动态地配置域名解析（路由）中的信息。通过etcd的服务发现功能就可以轻松解决这个动态配置的问题，如图33所示。

![enter image description here][3]

图3 云平台多实例透明化


###  1.2 场景二：消息发布与订阅
在分布式系统中，最为适用的组件间通信方式是消息发布与订阅机制。具体而言，即构建一个配置共享中心，数据提供者在这个配置中心发布消息，而消息使用者则订阅他们关心的主题，一旦相关主题有消息发布，就会实时通知订阅者。通过这种方式可以实现分布式系统配置的集中式管理与实时动态更新。

* **应用中用到的一些配置信息存放在etcd上进行集中管理**。这类场景的使用方式通常是这样的：应用在启动的时候主动从etcd获取一次配置信息，同时，在etcd节点上注册一个Watcher并等待，以后每次配置有更新的时候，etcd都会实时通知订阅者，以此达到获取最新配置信息的目的。
* **分布式搜索服务中，索引的元信息和服务器集群机器的节点状态信息存放在etcd中**，供各个客户端订阅使用。使用etcd的`key TTL`功能可以确保机器状态是实时更新的。
* **分布式日志收集系统**。这个系统的核心工作是收集分布在不同机器上的日志。收集器通常按照应用（或主题）来分配收集任务单元，因此可以在etcd上创建一个以应用（或主题）命名的目录P，并将这个应用（或主题）相关的所有机器ip，以子目录的形式存储在目录P下，然后设置一个递归的etcd Watcher，递归式地监控应用（或主题）目录下所有信息的变动。这样就实现了在机器IP（消息）发生变动时，能够实时通知收集器调整任务分配。
* **系统中信息需要动态自动获取与人工干预修改信息请求内容的情况。**通常的解决方案是对外暴露接口，例如JMX接口，来获取一些运行时的信息或提交修改的请求。而引入etcd之后，只需要将这些信息存放到指定的etcd目录中，即可通过HTTP接口直接被外部访问。

![enter image description here][4]

图4 消息发布与订阅


###  1.3 场景三：负载均衡
在`场景一`中也提到了负载均衡，本文提及的负载均衡均指软负载均衡。在分布式系统中，为了保证服务的高可用以及数据的一致性，通常都会把数据和服务部署多份，以此达到对等服务，即使其中的某一个服务失效了，也不影响使用。这样的实现虽然会导致一定程度上数据写入性能的下降，但是却能实现数据访问时的负载均衡。因为每个对等服务节点上都存有完整的数据，所以用户的访问流量就可以分流到不同的机器上。

* **etcd本身分布式架构存储的信息访问支持负载均衡**。etcd集群化以后，每个etcd的核心节点都可以处理用户的请求。所以，把数据量小但是访问频繁的消息数据直接存储到etcd中也是个不错的选择，如业务系统中常用的**二级代码表**。二级代码表的工作过程一般是这样，在表中存储代码，在etcd中存储代码所代表的具体含义，业务系统调用查表的过程，就需要查找表中代码的含义。所以如果把二级代码表中的小量数据存储到etcd中，不仅方便修改，也易于大量访问。
* **利用etcd维护一个负载均衡节点表**。etcd可以监控一个集群中多个节点的状态，当有一个请求发过来后，可以轮询式地把请求转发给存活着的多个节点。类似KafkaMQ，通过Zookeeper来维护生产者和消费者的负载均衡。同样也可以用etcd来做Zookeeper的工作。

![fuz][5]

图5 负载均衡

###  1.4 场景四：分布式通知与协调
这里讨论的分布式通知与协调，与消息发布和订阅有些相似。两者都使用了etcd中的Watcher机制，通过注册与异步通知机制，实现分布式环境下不同系统之间的通知与协调，从而对数据变更进行实时处理。实现方式通常为：不同系统都在etcd上对同一个目录进行注册，同时设置Watcher监控该目录的变化（如果对子目录的变化也有需要，可以设置成递归模式），当某个系统更新了etcd的目录，那么设置了Watcher的系统就会收到通知，并作出相应处理。

* **通过etcd进行低耦合的心跳检测**。检测系统和被检测系统通过etcd上某个目录关联而非直接关联起来，这样可以大大减少系统的耦合性。
* **通过etcd完成系统调度**。某系统有控制台和推送系统两部分组成，控制台的职责是控制推送系统进行相应的推送工作。管理人员在控制台做的一些操作，实际上只需要修改etcd上某些目录节点的状态，而etcd就会自动把这些变化通知给注册了Watcher的推送系统客户端，推送系统再做出相应的推送任务。
* **通过etcd完成工作汇报**。大部分类似的任务分发系统，子任务启动后，到etcd来注册一个临时工作目录，并且定时将自己的进度进行汇报（将进度写入到这个临时目录），这样任务管理者就能够实时知道任务进度。

![enter image description here][6]

图6 分布式协同工作

###  1.5 场景五：分布式锁
因为etcd使用Raft算法保持了数据的强一致性，某次操作存储到集群中的值必然是全局一致的，所以很容易实现分布式锁。锁服务有两种使用方式，一是保持独占，二是控制时序。

* **保持独占**，即所有试图获取锁的用户最终只有一个可以得到。etcd为此提供了一套实现分布式锁原子操作CAS（`CompareAndSwap`）的API。通过设置`prevExist`值，可以保证在多个节点同时创建某个目录时，只有一个成功，而该用户即可认为是获得了锁。
* **控制时序**，即所有试图获取锁的用户都会进入等待队列，获得锁的顺序是全局唯一的，同时决定了队列执行顺序。etcd为此也提供了一套API（自动创建有序键），对一个目录建值时指定为`POST`动作，这样etcd会自动在目录下生成一个当前最大的值为键，存储这个新的值（客户端编号）。同时还可以使用API按顺序列出所有当前目录下的键值。此时这些键的值就是客户端的时序，而这些键中存储的值可以是代表客户端的编号。

![enter image description here][7]

图7 分布式锁

###  1.6 场景六：分布式队列
分布式队列的常规用法与场景五中所描述的分布式锁的控制时序用法类似，即创建一个先进先出的队列，保证顺序。

另一种比较有意思的实现是**在保证队列达到某个条件时再统一按顺序执行**。这种方法的实现可以在/queue这个目录中另外建立一个/queue/condition节点。

* condition可以**表示队列大小**。比如一个大的任务需要很多小任务就绪的情况下才能执行，每次有一个小任务就绪，就给这个condition数字加1，直到达到大任务规定的数字，再开始执行队列里的一系列小任务，最终执行大任务。
* condition可以**表示某个任务在不在队列**。这个任务可以是所有排序任务的首个执行程序，也可以是拓扑结构中没有依赖的点。通常，必须执行这些任务后才能执行队列中的其他任务。
* condition还可以**表示其它的一类开始执行任务的通知**。可以由控制程序指定，当condition出现变化时，开始执行队列任务。

![enter image description here][8]
图8 分布式队列

###  1.7 场景七：集群监控与Leader竞选
通过etcd来进行监控实现起来非常简单并且实时性强，用到了以下两点特性。

1. 前面几个场景已经提到Watcher机制，当某个节点消失或有变动时，Watcher会第一时间发现并告知用户。
2. 节点可以设置`TTL key`，比如每隔30s向etcd发送一次心跳使代表该节点仍然存活，否则说明节点消失。

这样就可以第一时间检测到各节点的健康状态，以完成集群的监控要求。

另外，使用分布式锁，可以完成Leader竞选。对于一些长时间CPU计算或者使用IO操作，只需要由竞选出的Leader计算或处理一次，再把结果复制给其他Follower即可，从而避免重复劳动，节省计算资源。

Leader应用的经典场景是**在搜索系统中建立全量索引**。如果每个机器分别进行索引的建立，不但耗时，而且不能保证索引的一致性。通过在etcd的CAS机制竞选Leader，由Leader进行索引计算，再将计算结果分发到其它节点。

![enter image description here][9]
图9 Leader竞选


### 1.8 场景八：为什么用etcd而不用Zookeeper？
阅读了[“ZooKeeper典型应用场景一览”](http://jm-blog.aliapp.com/?p=1232)一文的读者可能会发现，etcd实现的这些功能，Zookeeper都能实现。那么为什么要用etcd而非直接使用Zookeeper呢？<sup>4</sup>

相较之下，Zookeeper有如下缺点。

1. 复杂。Zookeeper的**部署维护复杂**，管理员需要掌握一系列的知识和技能；而Paxos强一致性**算法也是素来以复杂**难懂而闻名于世；另外，Zookeeper的**使用也比较复杂**，需要安装客户端，官方只提供了java和C两种语言的接口。
2. Java编写。这里不是对Java有偏见，而是Java本身就偏向于**重型应用**，它会引入大量的依赖。而运维人员则普遍希望机器集群尽可能简单，维护起来也不易出错。
3. **发展缓慢**。Apache基金会项目特有的[“Apache Way”](http://www.infoworld.com/article/2612082/open-source-software/has-apache-lost-its-way-.html)在开源界饱受争议，其中一大原因就是由于基金会庞大的结构以及松散的管理导致项目发展缓慢。

而etcd作为一个后起之秀，其优点也很明显。

1. **简单**。使用Go语言编写**部署简单**；使用HTTP作为接口**使用简单**；使用Raft算法保证强一致性让用户**易于理解**。
2. **数据持久化**。etcd默认数据一更新就进行持久化。
3. **安全**。etcd支持SSL客户端安全认证。

最后，etcd作为一个年轻的项目，正在高速迭代和开发中，这既是一个优点，也是一个缺点。优点在于它的未来具有无限的可能性，缺点是版本的迭代导致其使用的可靠性无法保证，无法得到大项目长时间使用的检验。然而，目前CoreOS、Kubernetes和Cloudfoundry等知名项目均在生产环境中使用了etcd，所以总的来说，etcd值得你去尝试。


 
1：https://github.com/coreos/etcd 

2：http://jm-blog.aliapp.com/?p=1232

3：http://progrium.com/blog/2014/07/29/understanding-modern-service-discovery-with-docker/

4：http://devo.ps/blog/zookeeper-vs-doozer-vs-etcd

## 2 深度解析etcd
上一节中，我们概括了许多etcd的经典场景，这一节，我们将从etcd的架构开始，深入到源码中解析etcd。 

### 2.1 etcd架构

![enter image description here][10]
图10 etcd架构图

从etcd的架构图中我们可以看到，etcd主要分为四个部分。

* HTTP Server： 用于处理用户发送的API请求以及其它etcd节点的同步与心跳信息请求。
* Store：用于处理etcd支持的各类功能的事务，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，是etcd对用户提供的大多数API功能的具体实现。 
* Raft：Raft强一致性算法的具体实现，是etcd的核心。
* WAL：Write Ahead Log（预写式日志），是etcd的数据存储方式。除了在内存中存有所有数据的状态以及节点的索引以外，etcd还通过WAL进行持久化存储。WAL中，所有的数据提交前都会事先记录日志。Snapshot是为了防止数据过多而进行的状态快照；Entry则表示存储的具体日志内容。

通常，一个用户的请求发送过来，会经由HTTP Server转发给Store进行具体的事务处理，如果涉及到节点的修改，则交给Raft模块进行状态的变更、日志的记录，然后再同步给别的etcd节点以确认数据提交，最后进行数据的提交，再次同步。


### 2.2 etcd2.0.0区别于0.4.6的重要变更列表

* 获得了IANA认证的端口，2379用于客户端通信，2380用于节点通信，与原先的（4001  peers/7001 clients）共用。
* 每个节点可监听多个广播地址。监听的地址由原来的一个扩展到多个，用户可以根据需求实现更加复杂的集群环境，如一个是公网IP，一个是虚拟机（容器）之类的私有IP。
* etcd任意节点均可代理访问Leader节点的请求，所以如果你可以访问任何一个etcd节点，那么就可以对整个集群进行读写操作，而不需要刻意关注网络的拓扑结构。
* etcd集群和集群中的节点都有了自己独特的ID。这样就防止出现配置混淆，不是本集群的其他etcd节点发来的请求将被屏蔽。
* etcd配置信息在集群启动时就完全固定，这样有助于用户确认集群大小，正确地配置和启动集群。
* 运行时节点变更 (Runtime Reconfiguration)。用户不需要重启 etcd 服务即可实现对 etcd 集群结构进行变更，可以动态操作。
* 重新设计和实现了Raft算法，使得运行速度更快，更容易理解，包含更多测试代码。
* Raft日志现在是严格的只能向后追加、预写式日志系统，并且在每条记录中都加入了CRC校验码。
* 启动时使用的_etcd/* 关键字不再暴露给用户
* 废弃使用集群自动调整功能的standby模式，standby模式会使得用户维护集群更困难。
* 新增Proxy模式，不加入到etcd一致性集群中，纯粹进行代理转发。
* ETCD_NAME（-name）参数目前是可选的，不再用于唯一标识一个节点。
* 摒弃通过配置文件配置 etcd 属性的方式，你可以通过设置环境变量的方式代替。
* 通过自发现方式启动集群时要求用户提供集群大小，这样有助于确定集群实际启动的节点数量。

### 2.3 etcd概念词汇表

* Raft：etcd所采用的保证分布式系统强一致性的算法。
* Node：一个Raft状态机实例。
* Member： 一个etcd实例。它管理着一个Node，并且可以为客户端请求提供服务。
* Cluster：由多个Member构成可以协同工作的etcd集群。
* Peer：对同一个etcd集群中另外一个Member的称呼。
* Client： 向etcd集群发送HTTP请求的客户端。
* WAL：预写式日志，etcd用于持久化存储的日志格式。
* snapshot：etcd防止WAL文件过多而设置的快照，存储etcd数据状态。
* Proxy：etcd的一种模式，为etcd集群提供反向代理服务。
* Leader：Raft算法中通过竞选而产生的处理所有数据提交的节点。
* Follower：竞选失败的节点作为Raft中的从属节点，为算法提供强一致性保证。
* Candidate：当Follower超过一定时间接收不到Leader的心跳时转变为Candidate开始Leader竞选。
* Term：某个节点成为Leader到下一次竞选开始的时间周期，称为一个Term。
* Index：数据项编号。Raft中通过Term和Index来定位数据。

### 2.4 集群化应用实践

etcd作为一个高可用键值存储系统，天生就是为集群化而设计的。由于Raft算法在做决策时需要多数节点的投票，所以etcd一般部署集群推荐奇数个节点，推荐的数量为3、5或者7个节点构成一个集群。

#### 2.4.1 集群启动

etcd有三种集群化启动的配置方案，分别为静态配置启动、etcd自身服务发现、通过DNS进行服务发现。

**根据启动环境，你可以选择不同的配置方式。值得一提的是，这也是新版etcd区别于旧版的一大特性，它摒弃了使用配置文件进行参数配置的做法，转而使用命令行参数或者环境变量来配置参数。**

##### (1). 静态配置

这种方式比较适用于离线环境。在启动整个集群之前，你如果已经预先清楚所要配置的集群大小，以及集群上各节点的地址和端口信息，那么启动时，你就可以通过配置`initial-cluster`参数进行etcd集群的启动。

在每个etcd机器启动时，配置环境变量或者添加启动参数的方式如下。

    ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380"
    ETCD_INITIAL_CLUSTER_STATE=new


参数方法：

    -initial-cluster 
    infra0=http://10.0.1.10:2380,http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
     -initial-cluster-state new


**值得注意的是**，`-initial-cluster`参数中配置的url地址必须与各个节点启动时设置的`initial-advertise-peer-urls`参数相同。（`initial-advertise-peer-urls`参数表示节点监听其他节点同步信号的地址）

如果你所在的网络环境配置了多个etcd集群，为了避免意外发生，最好使用`-initial-cluster-token`参数为每个集群单独配置一个token认证。这样就可以确保每个集群和集群的成员都拥有独特的ID。

综上所述，如果你要配置包含3个etcd节点的集群，那么你在三个机器上的启动命令分别如下所示。

    $ etcd -name infra0 -initial-advertise-peer-urls http://10.0.1.10:2380 \
      -listen-peer-urls http://10.0.1.10:2380 \
      -initial-cluster-token etcd-cluster-1 \
      -initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
      -initial-cluster-state new

    $ etcd -name infra1 -initial-advertise-peer-urls http://10.0.1.11:2380 \
      -listen-peer-urls http://10.0.1.11:2380 \
      -initial-cluster-token etcd-cluster-1 \
      -initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
      -initial-cluster-state new

    $ etcd -name infra2 -initial-advertise-peer-urls http://10.0.1.12:2380 \
      -listen-peer-urls http://10.0.1.12:2380 \
      -initial-cluster-token etcd-cluster-1 \
      -initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
      -initial-cluster-state new


在初始化完成后，etcd还提供动态增、删、改etcd集群节点的功能，这个需要用到`etcdctl`命令进行操作。

##### (2). etcd自发现模式

通过自发现的方式启动etcd集群需要事先准备一个etcd集群。如果你已经有一个etcd集群，首先你可以执行如下命令设定集群的大小，假设为3.


    $ curl -X PUT https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83/_config/size -d value=3


然后你要把这个url地址`https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83`作为`  -discovery`参数来启动etcd。节点会自动使用`https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83`目录进行etcd的注册和发现服务。

所以最终你在某个机器上启动etcd的命令如下。


    $ etcd -name infra0 -initial-advertise-peer-urls http://10.0.1.10:2380 \
      -listen-peer-urls http://10.0.1.10:2380 \
      -discovery https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83


如果你在本地没有可用的etcd集群，etcd官网提供了一个可以用公网访问的etcd存储地址。
你可以通过如下命令得到etcd服务的目录，并把它作为` -discovery`参数使用。


    $ curl https://discovery.etcd.io/new?size=3
    https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de


同样的，当你完成了集群的初始化后，这些信息就失去了作用。当你需要增加节点时，需要使用`etcdctl`来进行操作。

为了安全，在每次启动新的etcd集群时，请务必使用新的discovery token进行注册。
另外，如果你初始化时启动的节点超过了指定的数量，多余的节点会自动转化为proxy模式的etcd。

##### (3). DNS自发现模式

etcd还支持使用DNS SRV记录进行启动。关于DNS SRV记录如何进行服务发现，可以参阅[RFC2782](http://tools.ietf.org/html/rfc2782)，所以，你要在DNS服务器上进行相应的配置。

* 开启DNS服务器上SRV记录查询，并添加相应的域名记录，使得查询到的结果类似如下。


    $ dig +noall +answer SRV _etcd-server._tcp.example.com
    _etcd-server._tcp.example.com. 300 IN   SRV 0 0 2380 infra0.example.com.
    _etcd-server._tcp.example.com. 300 IN   SRV 0 0 2380 infra1.example.com.
    _etcd-server._tcp.example.com. 300 IN   SRV 0 0 2380 infra2.example.com.


* 分别为各个域名配置相关的A记录指向etcd核心节点对应的机器IP，使得查询结果类似如下。


    $ dig +noall +answer infra0.example.com infra1.example.com infra2.example.com
    infra0.example.com. 300 IN  A   10.0.1.10
    infra1.example.com. 300 IN  A   10.0.1.11
    infra2.example.com. 300 IN  A   10.0.1.12


做好了上述两步DNS的配置，就可以使用DNS启动etcd集群了。配置DNS解析的url参数为`-discovery-srv`，其中某一个节点地启动命令如下。


    $ etcd -name infra0 \
    -discovery-srv example.com \
    -initial-advertise-peer-urls http://infra0.example.com:2380 \
    -initial-cluster-token etcd-cluster-1 \
    -initial-cluster-state new \
    -advertise-client-urls http://infra0.example.com:2379 \
    -listen-client-urls http://infra0.example.com:2379 \
    -listen-peer-urls http://infra0.example.com:2380


当然，你也可以直接把节点的域名改成IP来启动。

#### 2.4.2 关键部分源码解析

etcd的启动是从主目录下的`main.go`开始的，然后进入`etcdmain/etcd.go`，载入配置参数。如果被配置为proxy模式，则进入startProxy函数，否则进入startEtcd，开启etcd服务模块和http请求处理模块。

在启动http监听时，为了与集群其他etcd机器（peers）保持连接，均采用了`transport.NewTimeoutListener`启动方式，在超过指定时间没有获得响应时就会出现超时错误。而在监听client请求时，采用的是`transport.NewKeepAliveListener`，有助于连接的稳定。

在`etcdmain/etcd.go`中的setupCluster函数可以看到，对于不同的etcd参数，启动集群的方法略有不同，但是最终需要的就是一个IP与端口构成的字符串。

在静态配置的启动方式中，集群的所有信息都已经给出，所以直接解析用逗号隔开的集群url信息就好了。

DNS发现的方式与静态配置启动类似，会预先发送一个tcp的SRV请求，先查看`etcd-server-ssl._tcp.example.com`下是否有集群的域名信息，如果没有找到，则去查看`etcd-server._tcp.example.com`。根据找到的域名，解析出对应的IP和端口，即集群的url信息。

较为复杂是etcd式的自发现启动。首先用自身单个的url构成一个集群，然后在启动的过程中根据参数进入`discovery/discovery.go`源码的`JoinCluster`函数。在启动时使用的etcd的token地址中，包含了集群大小（size）信息，所以集群的启动本质上是一个不断监测与等待的过程。启动的第一步就是在这个借用的etcd的token目录下注册自身的信息，然后再监测token目录下所有节点的数量，如果数量没有达到指定值，则循环等待。否则结束等待，进入后续启动过程。

配置etcd过程中通常要用到两种url地址容易混淆，一种用于etcd集群同步信息并保持连接，通常称为peer-urls；另外一种用于接收用户端发来的HTTP请求，通常称为client-urls。

* `peer-urls`：通常监听的端口为`2380`（老版本使用的端口为`7001`），包括所有已经在集群中正常工作的所有节点的地址。
* `client-urls`：通常监听的端口为`2379`（老版本使用的端口为`4001`），为适应复杂的网络环境，新版etcd监听客户端请求的url从原来的1个变为现在可配置的多个。这样etcd就可以配合多块网卡同时监听不同网络下的请求。


#### 2.4.3 运行时节点变更

etcd集群启动完毕后，可以在运行的过程中对集群进行重构，包括核心节点的增加、删除、迁移、替换等。**运行时重构使得etcd集群无须重启即可改变集群的配置，这也是新版etcd区别于旧版包含的新特性。**

只有当集群中多数节点正常的情况下，你才可以进行运行时的配置管理。因为配置更改的信息也会被etcd当成一个信息存储和同步，如果集群多数节点损坏，集群就失去了写入数据的能力。所以在配置etcd集群数量时，强烈推荐至少配置3个核心节点，配置数目越多，可用性越强。

##### (1). 节点迁移、替换

当你节点所在的机器出现硬件故障，或者节点出现如数据目录损坏等问题，导致节点永久性的不可恢复时，就需要对节点进行迁移或者替换。当一个节点失效以后，必须尽快修复，因为etcd集群正常运行的必要条件是集群中多数节点都正常工作。

迁移一个节点需要进行四步操作：

* 暂停正在运行着的节点程序进程
* 把数据目录从现有机器拷贝到新机器
* 使用api更新etcd中对应节点指向机器的url记录更新为新机器的ip
* 使用同样的配置项和数据目录，在新的机器上启动etcd。


##### (2). 节点增加

增加节点可以让etcd的高可用性更强。举例来说，如果你有3个节点，那么最多允许1个节点失效；当你有5个节点时，就可以允许有2个节点失效。同时，增加节点还可以让etcd集群具有更好的读性能。因为etcd的节点都是实时同步的，每个节点上都存储了所有的信息，所以增加节点可以从整体上提升读的吞吐量。

增加一个节点需要进行两步操作：

* 在集群中添加这个节点的url记录，同时获得集群的信息。
* 使用获得的集群信息启动新etcd节点。

##### (3). 节点移除
有时你不得不在提高etcd的写性能和增加集群高可用性上进行权衡。Leader节点在提交一个写记录时，会把这个消息同步到每个节点上，当得到多数节点的同意反馈后，才会真正写入数据。所以节点越多，写入性能越差。在节点过多时，你可能需要移除其中的一个或多个。
移除节点非常简单，只需要一步操作，就是把集群中这个节点的记录删除，则对应机器上的该节点就会自动停止。

##### (4). 强制性重启集群
当集群超过半数的节点都失效时，就需要通过手动的方式，强制性让某个节点以自己为Leader，利用原有数据启动一个新集群。

此时你需要进行两步操作。

* 备份原有数据到新机器。
* 使用`-force-new-cluster`和备份的数据重新启动节点

**注意：强制性重启是一个迫不得已的选择，它会破坏一致性协议保证的安全性（如果操作时集群中尚有其它节点在正常工作，就会出错），所以在操作前请务必要保存好数据。**


### 2.5 Proxy模式

**Proxy模式也是新版etcd的一个重要变更**，etcd作为一个反向代理把客户的请求转发给可用的etcd集群。这样，你就可以在每一台机器都部署一个Proxy模式的etcd作为本地服务，如果这些etcd Proxy都能正常运行，那么你的服务发现必然是稳定可靠的。

![enter image description here][11]

图11 Proxy模式示意图

所以Proxy并不是直接加入到符合强一致性的etcd集群中，也同样的，Proxy并没有增加集群的可靠性，当然也没有降低集群的写入性能。

那么，为什么要有Proxy模式而不是直接增加etcd核心节点呢？实际上，etcd每增加一个核心节点（peer），都会给Leader节点增加一定程度的负担（包括网络、CPU和磁盘负载），因为每次信息的变化都需要进行同步备份。增加etcd的核心节点固然可以让整个集群具有更高的可靠性，但是当其数量达到一定程度以后，增强可靠性带来的好处就变得不那么明显，反倒是降低了集群写入同步的性能。因此，增加一个轻量级的Proxy模式etcd节点是对直接增加etcd核心节点的一个有效代替。

熟悉0.4.6这个旧版本etcd的用户会发现，Proxy模式实际上取代了原先的Standby模式。Standby模式具备转发代理的功能。此外，在核心节点因为故障导致数量不足时，还会从Standby模式转为核心节点。而当故障节点恢复时，若etcd的核心节点数量已经达到预设值，则前述节点会再次转为Standby模式。

但是在新版etcd中，只在最初启动etcd集群的过程中，若核心节点的数量已经满足要求，自动启用Proxy模式，反之则并未实现。主要原因如下。

- etcd是用来保证高可用的组件，因此它所需要的系统资源（包括内存、硬盘和CPU等）都应该得到充分保障。任由集群的自动变换随意地改变核心节点，无法让机器保证性能。所以etcd官方鼓励大家在大型集群中为运行etcd准备专有机器集群。
- 因为etcd集群是支持高可用的，部分机器故障并不会导致功能失效。所以在机器发生故障时，管理员有充分的时间对机器进行检查和修复。
- 自动转换使得etcd集群变得更为复杂，尤其是在如今etcd支持多种网络环境的监听和交互的情况下。在不同网络间进行转换，更容易发生错误，导致集群不稳定。

基于上述原因，目前Proxy模式有转发代理功能，而不会进行角色转换。

#### 关键部分源码解析

从代码中可以看到，Proxy模式的本质就是起一个http代理服务器，把客户发到这个服务器的请求转发给别的etcd节点。

etcd目前支持读写皆可和只读两种模式。默认情况下是读写皆可，就是把读、写两种请求都进行转发。而只读模式只转发读的请求，对所有其他请求返回501错误。

值得注意的是，在etcd集群化启动时，除了因为设置`proxy`参数作为Proxy模式启动之外，如果节点注册自身信息的时候监测到集群的实际节点数量已经符合要求，那么也会退化为Proxy模式。


### 2.6 数据存储

etcd的存储分为内存存储和持久化（硬盘）存储两部分，内存中的存储除了顺序化地记录下所有用户对节点数据变更的记录外，还会对用户数据进行索引、建堆等方便查询的操作。而持久化则使用预写式日志（WAL：Write Ahead Log）进行记录存储。

在WAL的体系中，所有的数据在提交之前都会进行日志记录。在etcd的持久化存储目录中，有两个子目录。一个是WAL，存储着所有事务的变化记录；另一个则是snapshot，用于存储某一个时刻etcd所有目录的数据。通过WAL和snapshot相结合的方式，etcd可以有效地进行数据存储和节点故障恢复等操作。

也许你会有这样的疑问，既然已经在WAL实时存储了所有的变更，为什么还需要snapshot呢？原因是这样的，随着使用量的增加，WAL存储的数据会急剧增加，为了防止磁盘空间不足，etcd默认每10000条记录做一次snapshot，经过snapshot以后的WAL文件就可以删除。通过API可以查询的历史etcd操作默认为1000条。

首次启动时，etcd会把启动的配置信息存储到`data-dir`参数指定的数据目录中。配置信息包括本地节点ID、集群ID和初始时集群信息。用户需要避免etcd从一个过期的数据目录中重新启动，因为使用过期的数据目录启动的节点会与集群中的其他节点产生不一致（如：之前已经记录并同意Leader节点存储某个信息，重启后又向Leader节点申请这个信息）。所以，为了最大化集群的安全性，一旦有任何数据损坏或丢失的可能性，你就应该把这个节点从集群中移除，然后加入一个不带数据目录的新节点。

#### （1）预写式日志（WAL）

WAL最大的作用是记录了整个数据变化的全部历程。在etcd中，所有数据的修改在提交前，都要先写入到WAL中。使用WAL进行数据的存储使得etcd拥有两个重要功能。

* **故障快速恢复**： 当你的数据遭到破坏时，就可以通过执行所有WAL中记录的修改操作，快速从最原始的数据恢复到数据损坏前的状态。
* **数据回滚（undo）/重做（redo）**：因为所有的修改操作都被记录在WAL中，在需要回滚或重做时，只需要反向或正向执行日志中的操作即可。

##### WAL与snapshot在etcd中的命名规则

在etcd的数据目录中，WAL文件以`$seq-$index.wal`的格式存储。最初始的WAL文件是`0000000000000000-0000000000000000.wal`，表示是所有WAL文件中的第0个，初始的Raft状态编号为0。运行一段时间后可能需要进行日志切分，把新的条目放到一个新的WAL文件中。

假设，当集群运行到Raft状态为20，需要进行WAL文件的切分时，则下一份WAL文件就会变为`0000000000000001-0000000000000021.wal`。如果在10次操作后又进行了一次日志切分，那么后一次的WAL文件名会变为`0000000000000002-0000000000000031.wal`。可以看到`-`符号前面的数字是每次切分后自增1，而`-`符号后面的数字则是根据实际存储的Raft起始状态来定。

snapshot的存储命名则比较容易理解，以`$term-$index.wal`格式进行命名存储。term和index就表示存储snapshot时数据所在的Raft节点状态，当前的任期编号以及数据项位置信息。


#### （2） 关键部分源码解析

从代码逻辑中可以看到，WAL有两种模式，读（read）模式和数据添加（append）模式，两者是互斥的。一个新创建的WAL文件处于append模式，并且不会进入到read模式。一个本来存在的WAL文件被打开的时候必然是read模式，只有在所有记录都被读完的时候，才能进入append模式，进入append模式后也不会再进入read模式。这样做有助于保证数据的完整与准确。

集群在进入到`etcdserver/server.go`的`NewServer`函数准备启动一个etcd节点时，会检测是否存在以前的遗留WAL数据。

etcd从v0.4.6升级到v2.0.0，它数据格式存储的格式也变化了。检测的第一步是查看snapshot文件夹下是否有符合规范的文件，若检测到snapshot格式是v0.4的，则调用函数升级到v0.5。从snapshot中获得集群的配置信息，包括token、其他节点的信息等等，然后载入WAL目录的内容，从小到大进行排序。根据snapshot中得到的term和index，找到WAL紧接着snapshot下一条的记录，然后向后更新，直到所有WAL包的entry都已经遍历完毕，Entry记录到ents变量中存储在内存里。此时WAL就进入append模式，为数据项添加进行准备。

当WAL文件中数据项内容过大达到设定值（默认为10000）时，会进行WAL的切分，同时进行snapshot操作。这个过程可以在`etcdserver/server.go`的`snapshot`函数中看到。所以，实际上数据目录中有用的snapshot和WAL文件各只有一个，默认情况下etcd会各保留5个历史文件。


### 2.7 Raft

新版Etcd中，Raft包就是对Raft一致性算法的具体实现。关于Raft算法的讲解，网上已经有很多文章，有兴趣的读者可以去阅读一下[Raft算法论文](https://ramcloud.stanford.edu/raft.pdf)，非常精彩。本文不再对Raft算法进行详细描述，而是结合etcd，针对算法中一些关键内容以问答的形式进行讲解。Raft算法的相关术语参见概念词汇表一节。

#### （1） Raft常见问答一览

* **Raft中一个Term（任期）是什么意思？**
在Raft算法中，从时间上讲，一个任期即从某一次竞选开始到下一次竞选开始。从功能上讲，如果Follower接收不到Leader节点的心跳信息，就会结束当前任期，变为Candidate发起竞选，有助于Leader节点故障时集群的恢复。
发起竞选投票时，任期值小的节点不会竞选成功。如果集群不出现故障，那么一个任期将无限延续下去。而投票出现冲突则有可能直接进入下一任再次竞选。

![enter image description here][12]
图12 Term示意图

* **Raft状态机是怎样切换的？**
Raft刚开始运行时，节点默认进入Follower状态，等待Leader发来心跳信息。若等待超时，则状态由Follower切换到Candidate进入下一轮Term发起竞选，等到收到集群多数节点的投票时，该节点转变为Leader。Leader节点有可能出现网络等故障，导致别的节点发起投票成为新Term的Leader，此时原先的老Leader节点会切换为Follower。Candidate在等待其它节点投票的过程中如果发现别的节点已经竞选成功成为Leader了，也会切换为Follower节点。

![enter image description here][13]

图13 Raft状态机

* **如何保证最短时间内竞选出Leader，防止竞选冲突？**
在Raft状态机一图中可以看到，在Candidate状态下， 有一个times out，这里的times out时间是个随机值，也就是说，每个机器成为Candidate以后，超时发起新一轮竞选的时间是各不相同的，这就会出现一个时间差。在时间差内，如果Candidate1收到的竞选信息比自己发起的竞选信息Term值大（即对方为新一轮Term），并且新一轮想要成为Leader的Candidate2包含了所有提交的数据，那么Candidate1就会投票给Candidate2。这样就保证了只有很小的概率会出现竞选冲突。

* **如何防止别的Candidate在遗漏部分数据的情况下发起投票成为Leader？**
Raft竞选的机制中，使用随机值决定超时时间，第一个超时的节点就会提升Term编号发起新一轮投票，一般情况下别的节点收到竞选通知就会投票。但是，如果发起竞选的节点在上一个Term中保存的已提交数据不完整，节点就会拒绝投票给它。通过这种机制就可以防止遗漏数据的节点成为Leader。

* **Raft某个节点宕机后会如何？**
通常情况下，如果是Follower节点宕机，且剩余可用节点数量超过总节点数的一半，集群可以几乎不受影响地正常工作。如果是Leader节点宕机，那么Follower节点会因为收不到心跳而超时，发起竞选获得投票，成为新一轮Term的Leader，继续为集群提供服务。**需要注意的是；etcd目前没有任何机制会自动去变化整个集群的总节点数量**，即如果没有人为地调用API，etcd宕机后的节点仍然被计算在总节点数中，任何请求被确认需要获得的投票数都是这个总数的一半以上。

![enter image description here][14]
**图14 节点宕机**

* **为什么Raft算法在确定可用节点数量时不需要考虑拜占庭将军问题？**
拜占庭将军问题中提出，允许n个节点宕机还能提供正常服务的分布式架构，需要的总节点数量为3n+1，而Raft只需要2n+1就可以了。其主要原因在于，拜占庭将军问题中存在数据欺骗的现象，而etcd中假设所有的节点都是诚实的。etcd在竞选前需要告诉别的节点自身的Term编号以及前一轮Term最终结束时的index值，这些数据都是准确的，其他节点可以根据这些值决定是否投票。另外，etcd严格限制Leader到Follower这样的数据流向保证数据一致不会出错。

* **用户从集群中哪个节点读写数据？**
Raft为了保证数据的强一致性，所有的数据流向都是一个方向，从Leader流向Follower，即所有Follower的数据必须与Leader保持一致，如果不一致则会被覆盖。也就是说，所有用户更新数据的请求都最先由Leader获得并保存下来，然后通知其他节点将其保存，等到大多数节点反馈时再把数据提交。一个已提交的数据项才是Raft真正稳定存储下来的数据项，不再被修改，最后再把提交的数据同步给其他Follower。因为每个节点都有Raft已提交数据准确的备份（最坏的情况也只是已提交数据还未完全同步），所以任何一个节点都可以处理读请求。

* **etcd实现的Raft算法性能如何？**
单实例节点支持每秒1000次数据写入。随着节点数目的增加，数据同步会因为网络延迟越来越慢；而读性能则会随之提升，因为每个节点都能处理用户的读请求。



#### （2） 关键部分源码解析
在etcd代码中，Node作为Raft状态机的具体实现，是整个算法的关键，也是了解算法的入口。

在etcd中，对Raft算法的调用如下，你可以在`etcdserver/raft.go`中的`startNode`找到：


    storage := raft.NewMemoryStorage()
    n := raft.StartNode(0x01, []int64{0x02, 0x03}, 3, 1, storage)


通过这段代码可以了解到，Raft在运行过程记录数据和状态都是保存在内存中，而代码中`raft.StartNode`启动的Node就是Raft状态机Node。启动了一个Node节点后，Raft会做如下事项。

首先，你需要把从集群的其他机器上收到的信息推送到Node节点，你可以在`etcdserver/server.go`中的`Process`函数看到。


    func (s *EtcdServer) Process(ctx context.Context, m raftpb.Message) error {
      if m.Type == raftpb.MsgApp {
        s.stats.RecvAppendReq(types.ID(m.From).String(), m.Size())
      }
      return s.node.Step(ctx, m)
    }


检测发来请求的机器是否是集群中的节点，自身节点是否是Follower，把发来请求的机器作为Leader，具体对Node节点信息的推送和处理则通过`node.Step()`函数实现。

其次，你需要把日志项存储起来，在你的应用中执行提交的日志项，然后把完成信号发送给集群中的其它节点，再通过`node.Ready()`监听等待下一次任务执行。有一点非常重要，你必须确保在你发送完成消息给其他节点之前，你的日志项内容已经确切稳定地存储下来了。

最后，你需要保持一个心跳信号`Tick()`。Raft有两个很重要的地方用到超时机制：心跳保持和Leader竞选。需要用户在其Raft的Node节点上周期性地调用Tick()函数，以便为超时机制服务。

综上所述，整个Raft节点的状态机循环类似如下所示：


    for {
        select {
        case <-s.Ticker:
            n.Tick()
        case rd := <-s.Node.Ready():
            saveToStorage(rd.State, rd.Entries)
            send(rd.Messages)
            process(rd.CommittedEntries)
            s.Node.Advance()
        case <-s.done:
            return
        }
    }


而这个状态机真实存在的代码位置为`etcdserver/server.go`中的`run`函数。

对状态机进行状态变更（如用户数据更新等）时将调用`n.Propose(ctx, data)`函数，在存储数据时，会先进行序列化操作。获得大多数其他节点的确认后，数据会被提交，保存为已提交状态。
  
之前提到etcd集群的启动如果使用自发现方式，需要借助别的etcd集群或者DNS，而启动完毕后这些`外力`就不需要了。etcd会把自身集群的信息作为状态存储起来。所以要变更自身集群节点数量实际上也需要像用户数据变更那样添加数据条目到Raft状态机中。上述功能由`n.ProposeConfChange(ctx, cc)`实现。当集群配置信息变更的请求同样得到大多数节点的确认反馈后，再进行配置变更的正式操作，代码如下。


    var cc raftpb.ConfChange
    cc.Unmarshal(data)
    n.ApplyConfChange(cc)


注意：为了避免不同etcd集群消息混乱，ID需要确保唯一性，不能重复使用旧的token数据作为ID。

### 2.8 Store

顾名思义，Store这个模块就像一个商店一样把etcd已经准备好的各项底层支持加工起来，为用户提供五花八门的API支持，处理用户的各项请求。要理解Store，就要从etcd的API入手。打开[etcd的API列表](https://github.com/coreos/etcd/blob/master/Documentation/api.md)，我们可以看到如下API，均系对etcd存储的键值进行的操作，亦即Store提供的内容。API中提到的目录（Directory）和键（Key），上文中也可能称为etcd节点（Node）。

* 为etcd存储的键赋值

        curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world"
        {
            "action": "set",
            "node": {
                "createdIndex": 2,
                "key": "/message",
                "modifiedIndex": 2,
                "value": "Hello world"
            }
        }

 反馈的内容含义如下：

  - action: 刚刚进行的动作名称。
  - node.key: 请求的HTTP路径。etcd使用一个类似文件系统的方式来反映键值存储的内容。
  - node.value: 刚刚请求的键所存储的内容。
  - node.createdIndex: etcd节点每次发生变化时，该值会自动增加。除了用户请求外，etcd内部运行（如启动、集群信息变化等）也可能会因为对节点有变动而引起该值的变化。
  - node.modifiedIndex: 类似node.createdIndex，能引起modifiedIndex变化的操作包括set, delete, update, create, compareAndSwap and compareAndDelete。

* 查询etcd某个键存储的值

        curl http://127.0.0.1:2379/v2/keys/message

* 修改键值：与创建新值几乎相同，但是反馈时会有一个`prevNode`值反应了修改前存储的内容。

        curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello etcd"


* 删除一个值

        curl http://127.0.0.1:2379/v2/keys/message -XDELETE

* 对一个键进行定时删除：etcd中对键进行定时删除，设定一个ttl值，当这个值到期时键就会被删除。反馈的内容会给出expiration项告知超时时间，ttl项告知设定的时长。

        curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=5


* 取消定时删除任务

        curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl= -d prevExist=true


* 对键值修改进行监控：etcd提供的这个API让用户可以监控一个值或者递归式地监控一个目录及其子目录的值，当目录或值发生变化时，etcd会主动通知。

        curl http://127.0.0.1:2379/v2/keys/foo?wait=true

* 对过去的键值操作进行查询：类似上面提到的监控，在其基础上指定过去某次修改的索引编号，就可以查询历史操作。默认可查询的历史记录为1000条。

        curl 'http://127.0.0.1:2379/v2/keys/foo?wait=true&waitIndex=7'

* 自动在目录下创建有序键。在对创建的目录使用`POST`参数，会自动在该目录下创建一个以createdIndex值为键的值，这样就相当于根据创建时间的先后进行了严格排序。该API对分布式队列这类场景非常有用。

        curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job1
        {
            "action": "create",
            "node": {
                "createdIndex": 6,
                "key": "/queue/6",
                "modifiedIndex": 6,
                "value": "Job1"
            }
        }


* 按顺序列出所有创建的有序键。

        curl -s 'http://127.0.0.1:2379/v2/keys/queue?recursive=true&sorted=true'


* 创建定时删除的目录：就跟定时删除某个键类似。如果目录因为超时被删除了，其下的所有内容也自动超时删除。

        curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true

* 刷新超时时间。

        curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true -d prevExist=true


* 自动化CAS（Compare-and-Swap）操作：etcd强一致性最直观的表现就是这个API，通过设定条件，阻止节点二次创建或修改。即用户的指令被执行当且仅当CAS的条件成立。条件有以下几个。
  - prevValue 先前节点的值，如果值与提供的值相同才允许操作。
  - prevIndex 先前节点的编号，编号与提供的校验编号相同才允许操作。
  - prevExist 先前节点是否存在。如果存在则不允许操作。这个常常被用于分布式锁的唯一获取。

 假设先进行了如下操作：设定了foo的值。
 `curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=one`然后再进行操作：`curl http://127.0.0.1:2379/v2/keys/foo?prevExist=false -XPUT -d value=three`就会返回创建失败的错误。
 
* 条件删除（Compare-and-Delete）：与CAS类似，条件成立后才能删除。

* 创建目录

        curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d dir=true

* 列出目录下所有的节点信息，最后以`/`结尾。还可以通过recursive参数递归列出所有子目录信息。

        curl http://127.0.0.1:2379/v2/keys/

* 删除目录：默认情况下只允许删除空目录，如果要删除有内容的目录需要加上`recursive=true`参数。

        curl 'http://127.0.0.1:2379/v2/keys/foo_dir?dir=true' -XDELETE

* 创建一个隐藏节点：命名时名字以下划线`_`开头默认就是隐藏键。

        curl http://127.0.0.1:2379/v2/keys/_message -XPUT -d value="Hello hidden world"


相信看完这么多API，相信读者已经对Store的工作内容有了基本的了解。它对etcd下存储的数据进行加工，创建出如文件系统般的树状结构供用户快速查询。它有一个`Watcher`用于节点变更的实时反馈，还需要维护一个`WatcherHub`对所有`Watcher`订阅者进行通知的推送。同时，它还维护了一个由定时键构成的小顶堆，快速返回下一个要超时的键。最后，所有这些API的请求都以事件的形式存储在事件队列中等待处理。

## 3 总结

通过从应用场景到源码分析的一系列回顾，我们了解到etcd并不是一个简单的分布式键值存储系统。它解决了分布式场景中最为常见的一致性问题，为服务发现提供了一个稳定高可用的消息注册仓库，为以微服务协同工作的架构提供了无限的可能。相信在不久的将来，通过etcd构建起来的大型系统会越来越多。

## 4 参考文献

* https://github.com/coreos/etcd
* https://groups.google.com/forum/#!topic/etcd-dev/wmndjzBNdZo
* http://jm-blog.aliapp.com/?p=1232
* http://progrium.com/blog/2014/07/29/understanding-modern-service-discovery-with-docker/
* http://devo.ps/blog/zookeeper-vs-doozer-vs-etcd/
* http://jasonwilder.com/blog/2014/02/04/service-discovery-in-the-cloud/
* http://www.infoworld.com/article/2612082/open-source-software/has-apache-lost-its-way-.html
* http://en.wikipedia.org/wiki/WAL
* http://www.infoq.com/cn/articles/coreos-analyse-etcd
* http://www.activestate.com/blog/2014/05/service-discovery-solutions
* https://ramcloud.stanford.edu/raft.pdf

  [1]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129001.jpg
  [2]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129002.jpg
  [3]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129003.jpg
  [4]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129004.jpg
  [5]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129005.jpg
  [6]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129006.jpg
  [7]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0130000.jpg
  [8]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129008.jpg
  [9]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129009.jpg
  [10]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129010.jpg
  [11]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129011.jpg
  [12]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129012.jpg
  [13]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129013.jpg
  [14]: http://cdn4.infoqstatic.com/resource/articles/etcd-interpretation-application-scenario-implement-principle/zh/resources/0129014.jpg
