---
layout: post
title: "数据收集工具的设计与最佳实践"
date: 2017-10-18 10:48:47 +0800
comments: true
categories: 
  - logkit
  - bigdata
  - qiniu
---


【本文首先发布于InfoQ】[《数据收集工具的设计与最佳实践》](http://www.infoq.com/cn/articles/data-collection-tool)

笔者之前在[《七牛大数据平台的演进与大数据分析实践》](http://www.infoq.com/cn/articles/qiniu-big-data-platform-evolution-and-analysis)中提到了已经开源的数据收集工具logkit。本文将深入介绍数据收集的设计思路以及大数据收集背后的细节，为大家提供大数据实战中第一步数据采集的最佳实践。


## 数据收集工具对比

目前社区就不乏存在大量优秀的数据收集工具，如有名的Elastic Stack(Elasticsearch、Logstash、Kibana)中的[Logstash](https://github.com/elastic/logstash)；CNCF基金会里面有名的[Fluentd](https://github.com/fluent/fluentd)；InfluxData公司TICK Stack中的[Telegraf](https://github.com/influxdata/telegraf)；Google 出品为Kubernetes定制的[cAdvisor](https://github.com/google/cadvisor)；Apache基金会中的顶级项目[Flume](https://github.com/apache/flume)。除了早期诞生的诸如Fluentd、Flume等项目，其他项目都是为特定的平台业务定制而成，然后在随后的开源中不断进化，变得更为通用。所以针对特定业务，量身定制一款数据收集工具，是一个较为普遍的需求，也是出现如此众多“轮子”的主要原因。

让我们先来看看这几种知名开源数据收集工具有哪些特点。

* [Flume](https://flume.apache.org/): 毋庸置疑，在流式数据处理的场景中，Flume绝对是开源产品中的不二选择。其架构分为Source、Channel、Sink三部分，分别负责从上游服务端获取数据、暂存数据、以及解析并发送到下游。Flume尤以灵活的扩展性和强大的容错处理能力著称，非常适合在大数据量的情况下做数据解析、中转以及上下游适配的工作。
另一方面，Flume也有一些缺陷，如解析与发送都耦合在Sink模块，用户在编写Sink插件时不得不编写解析的逻辑，无法复用一些常规的解析方式；依赖JVM运行环境，作为服务端程序可以接受，但是部署和运行一个数据收集客户端程序则变得相对笨重；Flume的配置融合了Channel部分，基本配置并不简单，用户想要用起来需要的前置知识较多。

* [Logstash](https://www.elastic.co/products/logstash): 随着Elastic Stack的广受热捧，Logstash自然也达到了技术圈家喻户晓的程度，而Logstash本身的强大功能也确实名副其实，其架构分为Inputs、Filters以及Outputs三部分。Inputs作为所有输入端的集合，包含了各类数据输入插件；Filters包括解析与数据转换两部分的插件集合，其中就包含了大名鼎鼎的Grok解析方式，几乎可以解析所有类型的数据；Outputs则是输出端的集合。毫无疑问，Logstash几乎是使用Elastic Stack方案时作为数据收集的唯一选择。
但是同样的，Logstash也是基于JVM运行的，作为一个客户端程序相对较重；当你希望把它作为一个agent收集一些机器的基本信息时，它也无能为力。于是除了Logstash之外Elastic Stack家族中又增加了beats这个成员，但是如果仅仅选择beats，其功能又太过单薄。

* [Fluentd](https://www.fluentd.org/blog/unified-logging-layer): Fluentd也是收据收集界的老牌重量级选手，如果你玩容器、玩Kubernetes，那么就一定听说过CNCF(Cloud Native Computing Foundation)，而Fluentd就是其中的一员，成为了容器圈子里多数人日志收集处理的首选。Fluentd的架构与Logstash相似，也大致分为输入、输出以及中间的处理层，但是与之不同的是，其中间的处理层除了包括常规的filter(解析)以及buffer(数据暂存)以外，还包含了一个routing(路由)的功能，这是Fluentd的一大特色。routing功能使得Fluentd能够将自己称为一个统一的日志管理中间层，将所有的数据输入和输出管理起来，按照需求将输入的数据路由到一个或多个输出端。这是一个非常先进的设计，其他类似的开源软件往往要写多份配置文件才能达到这个效果。
Fluentd由CRuby实现，性能表现优良但依赖ruby环境；相较于前面两者，Fluentd插件支持相对较少；其配置也过于复杂，使用门槛较高。

* [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)/[cAdvisor](https://github.com/google/cadvisor): 这两款均是Go语言编写的针对系统信息数据收集的开源工具，其侧重点在metric收集，相较于通用的日志收集和处理，其功能面较窄，但是性能方面均表现优异。Telegraf配合influxdb，可以让你对机器各个维度的信息了如指掌；而cAdvisor更是Kubernetes的亲儿子，处理容器资源信息几无敌手。
但是这两款工具并无意于发力通用数据收集的功能，功能上可能无法满足一些日志收集的场景。

了解了以上这些开源软件的特点后，下面我们开始深入介绍构建一款数据收集工具会遇到哪些设计与挑战，以此为你的业务量身定制。

## 数据收集工具设计

### 架构设计

主流的数据收集工具其主架构都分为reader、parser，以及sender三部分，如图1所示。除了这三个日志收集常规组成部分，还应该包含可选的模块，如基于解析过后的数据转换(filter/transformer)以及数据暂存管道(Channel/Buffer)。为了尽可能复用，每个组成部分都应该是插件式的，可以编写不同类型插件并且灵活的组装。Channel/Buffer部分也应该提供基于内存或者基于磁盘的选择。

![数据收集架构][1]
图1 数据收集架构设计

对于Reader、Parser、Sender等插件共同组装的业务数据收集单元，我们称之为一个运行单元(Runner)，数据收集工具必须可以同时运行多个Runner，且每个Runner可以支持更新。

更新可以通过多种方式实现，最常规的是手动更新配置然后重启；更好的设计是支持热更新，不需要重启，自动识别配置文件的变化；还可以设计一个漂亮的web界面做配置的变更，以此引导用户使用，解决数据收集配置复杂用户使用门槛高的难题。所以在整体之上还应该构建一个简单的API层，支持web界面的功能。

### 语言选择

数据收集属于轻量级的agent服务，一般选择的语言为C/C++或者近年来特别火热的Go，而Go语言已经成为这类数据收集工具编写的大众选择，如Logstash新开发的一个beats工具，Telegraf，cAdvisor等等，均使用Go语言开发。

社区已经有很多文章描述使用Go语言的好处，在此也就不再赘述。总体而言用Go语言开发门槛较低，性能优良，支持跨多种操作系统平台，部署也极为简便。

### 分模块设计
#### 数据读取模块（Reader）

顾名思义，数据读取模块负责从不同数据源中读取数据，设计Reader模块的时候，必须支持插件式数据源接入，且将接口设计的足够简单，方便大家一同贡献更多的读取数据源驱动。

自定义数据源，最基本的只需要实现如下两个方法即可。

```
ReadLine() string
SyncMeta() error
```

从数据来源上分类，数据读取大致可分为从文件读取、从数据存储服务端读取以及从消息队列中读取三类。

每一类Reader均在发送成功后通过SyncMeta()函数记录读取的位置，保证数据不会因为runner意外中断而丢失。

**从文件读取数据**最为常见，针对文件的不同rotate方式，有不同的读取模式，主要分为三类：

* **file模式**：使用file模式的经典日志存储方式类似于nginx的日志rotate方式，日志名称是固定的，如`access.log`，rotate时直接move成新的文件如`access.log.1`，新的数据仍然写入到`access.log`。即我们永远只针对`access.log`这一个固定名称的文件进行收集。而检测文件是否rotate的标志是文件的inode号，在windows下则是fd的属性编号。当文件rotate后，则从文件头开始读取。

* **dir模式**：使用dir模式的经典日志存储方式为整个文件夹下存储单个服务的业务日志，文件夹下的日志通常有统一前缀，后缀为时间戳，根据日志的大小rotate到新的文件。如配置的文件夹为logdir，下面的文件为 `logdir/a.log.20170621`, `logdir/a.log.20170622`, `logdir/a.log.20170623`,...。每次分割后新命名文件并以时间戳为后缀，并且该文件夹下只有这一个服务。dir模式首先会对文件夹下文件整体排序，依次读取各个文件，读完最后一个文件后会查找时间(文件ctime)更新文件并重新排序，依次循环。dir模式应该将多个文件数据串联起来。即数据读取过程中a.log.20170621中最后一行的下一行就是a.log.20170622的第一行。该模式下自然还包括诸如文件前缀匹配、特定后缀忽略等功能。

* **tailx模式**：以通配的路径模式读取，读取所有被通配符匹配上的日志文件，对于单个日志文件使用file模式不断追踪日志更新，例如匹配路径的模式串为 `/home/*/path/*/logdir/*.log*`, 此时会展开并匹配所有符合该表达式的文件，并持续读取所有有数据追加的文件。每隔一定时间，重新获取一遍模式串，添加新增的文件。

除此之外，还应支持包括多文件编码格式支持、读取限速等多种功能。

**从数据存储服务中读取数据**，可以采用时间戳策略，在诸如MongoDB、MySQL中记录的数据，包含一个时间戳的字段，每次读取数据均按这个时间戳字段排序，以此获得新增的数据或者数据更新。另一方面，需要为用户设计类似定时器等策略，方便用户可以多次运行，不断同步收集服务器中的数据。

**从消息队列中读取数据**，这个最为简单，直接从消息队列中消费数据即可。注意记录读取的Offset，防止数据丢失。

#### 解析模块(Parser)

解析模块应该负责将数据源中读取的数据解析到对应的字段及类型，目前常见的解析器包括：

1. csv parser: 按照分隔符解析成对应字段和类型，分隔符可以自定义，如常见的制表符(`\t`)、空格(` `)、逗号(`,`)等等。
2. json parser: 解析json格式的数据，json是一种自带字段名称及类型的序列化协议，解析json格式仅需反序列化即可。
3. 基于正则表达式(grok) parser: Logstash grok解析非常强大，但是它并不指定类型，而Telegraf做了一个增强版的grok 解析器，除了基本的正则表达式和字段名称，还能标志数据类型，基本上能标志数据类型的grok解析器已经是一个完备的数据解析器了，可以解析几乎所有数据。当然，类型解析是相对复杂的功能，可能涉及到具体业务，如时间类型等。
4. raw parser: 将读取到的一行数据作为一个字段返回，简单实用。
5. nginx/apache parser: 读取nginx/apache等常见配置文件，自动生成解析的正则表达式，解析nginx/apache日志。

除了以上几种内置的解析器，同Reader一样，你也需要实现自定义解析器的插件功能，而Parser极为简单，只需要实现最基本的Parse方法即可。

```
Parse(lines []string) (datas []sender.Data, err error)
```

每一种Parser都是插件式的结构，可以复用，并任意选择。在不考虑解析性能的情况下，上述几种解析器基本可以满足所有数据解析的需求，将一行行数据解析为带有Schema(具备字段名称及类型)的数据。但是当你希望对某个字段做操作时，纯粹的解析器可能不够用。于是作为补充，数据收集工具还需要提供Transformer/Filter的功能。

#### Transformer

Transformer是Parser的补充，针对字段进行数据变化。

举例来说，如果你有个字段想做字符串替换，比如将所有字段名称为"name"的数据中，值为"Tom"的数据改为"Tim"，那么可以添加一个字符串替换的Transformer，针对"name"这个字段做替换。

又比如说，你的字段中有个"IP"，你希望将这个IP解析出运营商、城市等信息，那么你就可以添加一个Transformer做这个IP信息的转换。

当然，Transformer应该可以多个连接到一起连动合作。

设计Transformer模块是一件有趣而富有挑战的事情，这涉及到tranformer功能多样性带来的3个问题。

1. 多样的功能必然涉及到多样的配置，如何将不同的配置以优雅而统一的方式传达到插件中？
2. 多样的功能也涉及到不同功能的描述，如何将功能描述以统一的形式表达给用户，让用户选择相应的配置？
3. 如何将上述两个问题尽可能简单的解决，让用户编写Transformer插件时关注尽可能少的问题？

这里我们留个悬念，感兴趣的朋友可以阅读[logkit Transformer相关](https://github.com/qiniu/logkit/tree/develop/transforms)的源码寻求答案，笔者后续也会在logkit的wiki页面中描述。


#### Channel

经过解析和变换完成后的数据，就认为已经是处理好的，此时就会进入待发送队列，即我们的Channel部分。Channel的好坏决定了一个数据收集发送工具的性能及可靠程度，是数据收集工具中最具技术含量的一环。

数据收集工具，顾名思义，就是将数据收集起来，再发送到指定位置，而为了将性能最优化，我们必须把收集和发送解耦，中间提供一个缓冲带，而Channel就是负责这个数据暂存的地方。有了Channel，读取和发送就解耦了，可以利用多核优势，多线程发送数据，提高数据吞吐量。

![ft_sender][2]
图2 ft sender

一种设计思路就是 把整个Channel，包括多线程发送做成一个框架，封装成一个特殊的sender，我们称这个特殊的sender为"ft sender"。其架构如图2所示，ft sender与其他sender一样也提供了Send()方法，解析完毕后的数据调用Send方法实际上就是将数据传入到Channel中，然后再由ft sender处理多线程发送的逻辑，将从队列中取出的数据交由实际的sender多线程发送。

同时需要为用户提供磁盘和内存两大队列方式选择。

如果追求最高的可靠性，那么就使用磁盘队列，数据会暂存到本地磁盘中，发送后自动删除，即使突然宕机也不怕丢失数据。

如果追求更高的性能，可以使用内存队列，其实现方式就是Go语言的Channel机制，稳定而简单，在关停过程中，也需要将Channel中的数据落地到磁盘，在随后重新启动时载入，正常使用过程中也没有数据丢失的风险。得益于Go语言的同步Channel机制，你甚至可以把内存队列的大小设置为0，以此实现多线程发送，这样使用内存队列即使宕机，也没有了数据丢失的风险。

除了正常的作为待发送数据的等待队列以外，Channel如下一些非常有趣而实用的功能。

##### 错误数据筛选

并不是所有解析完毕的数据发送到服务端就一定是正确的，有时服务端指定的数据格式和解析完毕的格式存在出入，或者数据中含有一些非法字符等情况，则数据不能发送成功，此时，如果一批数据中只有一条这样错误的数据，就很容易导致这一整批都失败。

错误数据筛选的作用就在于，把这一整批数据中对的数据筛选出来，留下错误的数据，将正确的发送出去。

做法很简单，当发送时遇到服务端返回存在格式错误的数据时，将这一批数据平均拆分为两批（二分），再放入队列，等待下次发送。再遇到错误时则再拆分，这样不断二分，直到一个批次中只有一条数据，且发送失败，那我们就找到了这个错误数据，可以选择丢弃或记录。

借助队列，我们很容易就能将错误数据筛选出来。

##### 包拆分（大包拆小包）

包拆分的由来是服务端不可能无限制开放每个批次数据传输的大小，出于服务器性能、传输性能等原因，总有会有一些限制。

当一个批次的数据过大时，就会导致传输失败。此时的做法与错误筛选的方法相似，只要将包二分即可，如果还是太大就再次二分，以此类推。

##### 限速

相信限速的功能最容易理解，数据统统经过Channel，那么只要限制这个Channel传输介质的速度即可。例如磁盘队列，只需要限制磁盘的IO读写速度；内存队列则限制队列大小以此达到限速的目的。

常见的流量控制的算法有[漏桶算法(Leaky bucket)](https://en.wikipedia.org/wiki/Leaky_bucket)和[令牌桶算法(Token bucket)](https://en.wikipedia.org/wiki/Token_bucket)两种，比较推荐采用令牌桶算法实现该功能，感兴趣的朋友可以阅读一下[logkit 的 rateio 包](https://github.com/qiniu/logkit/tree/develop/rateio)。


#### Sender

Sender的主要作用是将队列中的数据发送至Sender支持的各类服务，一个最基本的实现同样应该设计的尽可能简单，理论上仅需实现一个Send接口即可。

```
Send([]Data) error
```

那么实现一个发送端有哪些注意事项呢？

1. **多线程发送**：多线程发送可以充分利用CPU多核的能力，提升发送效率，这一点我们在架构设计中已经设计了ft sender作为框架解决了该问题。
2. **错误处理与等待**：服务端偶尔出现一些异常是很正常的事情，此时就要做好不同错误情况的处理，不会因为某个错误而导致程序出错，另外一方面，一旦发现出错应该让sender等待一定时间再发送，设定一个对后端友好的变长错误等待机制也非常重要。一般情况下，可以采用随着连续错误出现递增等待时间的方法，直到一个最顶峰（如`10s`），就不再增加，当服务端恢复后再取消等待。
3. **数据压缩发送**：带宽是非常珍贵的资源，通常服务端都会提供gzip压缩的数据接收接口，而sender利用这些接口，将数据压缩后发送，能节省大量带宽成本。
4. **带宽限流**：通常情况下数据收集工具只是机器上的一个附属程序，主要资源如带宽还是要预留给主服务，所以限制sender的带宽用量也是非常重要的功能，限流的方法就可以采用前面Channel一节提到的令牌桶算法。
5. **字段填充(UUID/timestamp)**：通常情况下收集的数据信息可能不是完备的，需要填充一些信息进去，如全局唯一的UUID、代表收集时间的timestamp等字段，提供这些字段自动填充的功能，有利于用户对其数据做唯一性、时效性等方面的判断。
6. **字段别名**：解析后的字段名称中经常会出现一些特殊字符，如"$","@"等符号，如果发送的服务端不支持这些特殊字符，就需要提供一些重命名的功能，将这些字段映射到一个别的名称。
7. **字段筛选**：未必解析后的所有字段数据都需要发送，这时候提供一个字段筛选的功能，可以方便用户选择去掉一些无用的字段，也可以节省传输的成本。也可以在Transformer中也提供类似[`discard transformer`](https://github.com/qiniu/logkit/wiki/%5Bdiscard%5DTransformer)的功能，将某个字段去掉。
8. **类型转换**：类型转换是一个说来简单但是做起来非常繁琐的事情，不只是纯粹的整型转换成浮点型，或者字符串转成整型这么简单，还涉及到你发送到的服务端支持的一些特殊类型，如`date`时间类型等，更多的类型转换实际上相当于最佳实践，能够做好这些类型转换，就会让用户体验得到极大提升。
9. **简单、简单、简单**：除了上述这些，剩下的就是尽可能的让用户使用简单。比如假设我们要写一个 mysql sender，mysql的数据库和表如果不存在，可能数据会发送失败，那就可以考虑提前创建；又比如数据如果有更新了，那么就需要将针对这些更新的字段去更新服务的Schema等等。

logkit针对上述这些要点都做了大量细节的改进和优化。


#### Metrics

除了基本的自定义的数据收集，数据收集工具作为一个机器的agent，还可以采集机器的基本数据，例如CPU、内存、网络等信息，通过这些信息，可以全面掌控机器的状态。

具体的内容也可以参考logkit文档：[Runner之系统信息采集配置](https://github.com/qiniu/logkit/wiki/Runner%E4%B9%8B%E7%B3%BB%E7%BB%9F%E4%BF%A1%E6%81%AF%E9%87%87%E9%9B%86%E9%85%8D%E7%BD%AE)。

至此，一个完整的数据收集工具的设计要点都已经介绍完。

而我们开源的logkit，就是完全按照这样的设计实现的，我们集合了多种开源数据收集工具的优点，聚焦易用性，致力于打造产品级别的开源软件。


## 数据收集工具--logkit

logkit(https://github.com/qiniu/logkit)是七牛大数据团队开源的一个通用的日志收集工具，可以从多种不同的数据源中采集数据，并对数据进行一系列的解析、变换、裁减，最终发送到多种不同的数据下游，其中就包括了[七牛的大数据平台Pandora](https://pandora-docs.qiniu.com)。除了基本的数据采集、解析以及发送功能之外，logkit集合了多种同类开源软件的优势，涵盖了容错、并发、热加载、断点续传等高级功能，更提供了页面方便用户配置、监控以及管理自己的数据采集业务，是一款产品级别的开源软件。

目前支持的数据源包括：

1. File Reader: 读取文件中的日志数据，如 nginx/apache 服务日志文件、业务日志等。
2. Elasticsearch Reader: 全量导出Elasticsearch中的数据。
3. MongoDB Reader: 同步MongoDB中的数据。
4. MySQL Reader: 同步MySQL中的数据。
5. MicroSoft SQL Server Reader: 同步Microsoft SQL Server中的数据。
6. Kafka Reader: 导出Kafka中的数据。
7. Redis Reader: 导出Redis中的数据。

目前支持发送到的服务包括： Pandora、ElasticSearch、InfluxDB、MongoDB以及本地文件五种。近期还会支持发送到Kafka以及发送到某个HTTP地址这两种。

1. Pandora Sender: 发送到[Pandora(七牛大数据处理平台)](https://pandora-docs.qiniu.com)服务端。
2. Elasticsearch Sender: 发送到Elasticsearch服务端。
3. File Sender: 发送到本地文件。
4. InfluxDB Sender: 发送到InfluxDB服务端。
5. MongoDB Sender: 后发送到MongoDB服务端。

而在这已经实现的有限的几个发送端中，我们是这么设计的使用场景。

* 如果收集数据是为了监控，那么可以使用InfluxDB Sender，发送到开源的InfluxDB服务端，实现实时的数据监控。
* 如果收集数据是为了搜索查看，那么可以使用Elasticsearch Sender，发送到开源的Elasticsearch服务端，进行日志查询。
* 如果收集数据是为了计量统计或者其他一些涵盖复杂的增删改查需求的场景，那么就可以使用MongoDB Sender，在本地对数据进行聚合，再发送到开源的MongoDB服务中。

而发送到[七牛的Pandora服务](https://pandora-docs.qiniu.com)中，不仅涵盖了上述全部的场景，还提供了大量可以快速发掘数据价值的实际应用场景的使用模板，并且可以利用七牛成本较低的云存储服务对数据进行持久备份。

目前logkit支持的收集端和发送端并不多，我们非常欢迎大家来贡献更多的收集/发送端。

### 量身定制

再回过头来聊聊量身定制的话题，本文描述了一个数据收集工具打造的完整过程，我们深知量身定制一个数据收集工具有多么重要，而量身定制也是logkit的一大特点。logkit架构中的每个组成部分(Reader、Parser、Sender、Transformer、Channel等)都是一个GO语言的package，你完全可以用logkit中的包，自己写主函数，定制一个你专属的收集工具。我们甚至给出了一个这样的[代码案例](https://github.com/qiniu/logkit/tree/develop/samples)，方便你亲自去实践。

### 优势

总体而言，logkit有如下优势：

* GO语言编写，性能优良，资源消耗低，跨平台支持。
* Web支持，提供**页面**对数据收集、解析、发送过程可视化
* 插件式架构，扩展性强，使用灵活，易于复用。
* 定制化能力强，可以仅使用部分logkit的包，以此定制你的专属收集工具。
* 配置简单，易于上手，可通过**页面**进行操作管理
* 纯天然中文支持，没有汉化烦恼
* 功能全面，涵盖了包括grok解析、metric收集、字段变化(transform)在内的多种开源软件特点
* 生态全面，数据发送到七牛的Pandora大数据平台支持包括时序数据库、日志检索以及压缩永久存储等多种数据落地方案。

下面让我们来实践一下，看看logkit实战中是什么样子。

## logkit实战

### 下载

编译完后的logkit是一个Go的二进制包，你可以在[logkit的Download页面](https://github.com/qiniu/logkit/wiki/Download)找到对应操作系统的Release版本。

也可以参照[logkit源码编译指南](https://github.com/qiniu/logkit/wiki/%E6%9C%AC%E5%9C%B0%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91%E6%8C%87%E5%8D%97)，从代码层面定制自己的专属logkit。

### 启动

logkit部署非常简单，将这个binary放在系统PATH路径中就算部署完成了，推荐使用诸如supervisor等进程管理工具进行管理。

启动logkit工具，可以使用默认的配置文件，执行如下命令即可。

```
logkit -f logkit.conf
```

配置文件中默认开启了3000端口，初次使用可以通过浏览器打开logkit配置页面，配置runner并调试你的读取、解析和发送方式，浏览器访问的地址就是 http://127.0.0.1:3000 。

### 配置

![logkit 首页][3]
图3 logkit首页

打开网址后看到如图3所示的logkit配置助手首页，在这个页面会清晰的展示你目前所有的logkit Runner运行状态，包括读取速率、发送速率、成功/失败数据条数，以及一些错误日志。你还可以在这里修改和删除Runner。

点击左上角【增加Runner】按钮，可以添加新的logkit Runner。

![Reader][4]
图4 数据源Reader配置

如图4所示，新增Runner的第一步就是配置数据源，为了尽可能方便用户，logkit将绝大多数选项都预设了默认值，用户只需要根据页面提示填写黄色的必填项即可。

按页面步骤依次配置数据源、解析方式、以及发送方式。

![Parser][5]
图5 解析数据

如图5 所示，在配置解析方式的页面还可以根据配置尝试解析样例数据，这个页面可以根据你的实际数据非常方便的调试你的解析方式。

![Transformer][6]
图6 字段变化Transformer

如图6所示，除了解析以外，你还可以针对解析出来的某个字段内容做数据变换（Transform），即我们上一节中描述的Transformer，你可以像管道一样多个拼接Transformer，做多重字段变化。

最后配置完发送方式，你可以在如图7所示的页面做二次确认。

![确认添加][7]
图7 确认并添加页

在二次确认的页面中，你可以直接修改表达内容也可以返回上一步修改，最终点击添加Runner即可生效。

到这里，一个复杂的数据收集工作就完成了，怎么样，就是这么简单，快来实际体验一下吧！


  [1]: http://oet7by1sp.bkt.clouddn.com/logkit_arch.png
  [2]: http://oet7by1sp.bkt.clouddn.com/ft_sender.png
  [3]: https://raw.githubusercontent.com/qiniu/logkit/develop/resources/logkitnewconfig1.png
  [4]: https://raw.githubusercontent.com/qiniu/logkit/develop/resources/logkitnewconfig2.png
  [5]: https://raw.githubusercontent.com/qiniu/logkit/develop/resources/logkitnewconfig3.png
  [6]: https://raw.githubusercontent.com/qiniu/logkit/develop/resources/logkitnewconfig5.png
  [7]: https://raw.githubusercontent.com/qiniu/logkit/develop/resources/logkitnewconfig4.png