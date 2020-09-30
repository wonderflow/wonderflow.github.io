+++
author = "admin"
date = "2013-12-15T15:48:24Z"
slug = "2013/12/15/e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f"
title = "'《MapReduce Design Patterns》读书笔记——浅谈Map/Reduce设计模式 '"
Categories = ["CloudComputing", "IT", "Reading Notes"]
Tags = ["hadoop"]
+++

# 概述


MapReduce是一个基于Hadoop的算法框架。本文将从Hadoop开始介绍，然后重点讲述可用于Hadoop上的Map/Reduce设计模式。


# Hadoop简介




## Hadoop历史


Hadoop最早起源于Apache Nutch，该项目始于2002年，是Apache Lucence的子项目之一。该项目的出现源于两篇论文，一篇是2003年发表的“关于谷歌分布式文件系统”（NDFS：Nutch Distributed File System），描述了谷歌搜索引擎网页相关数据存储架构，解决Nutch遇到的网页抓取和索引过程中产生的超大文件存储需求问题。一篇是2004年发表的“关于谷歌分布式计算框架MapReduce”，描述了谷歌内部最重要分布式计算框架，该框架可用于处理海量网页索引问题。由于NDFS和MapReduce在Nutch引擎中有着良好的应用，所以它们于2006年2月被分离出来成为一套独立的软件，命名为Hadoop。


## Hadoop功能与优势


近年来，随着现代社会信息的飞速增长，大数据与云计算的概念已经越来越火。预计到2020年，互联网中产生的数字信息将会有三分之一的内容驻留在云平台中交由云平台处理。如何高效的存储和管理这些数据就成为了我们亟需解决的问题，这时Hadoop系统的优势就体现出来了。Hadoop通过三个方面高效的解决了云平台数据存储与管理的问题。一、它采用分布式存储方式HDFS（Hadoop Distributed File System）来提高读写速度和扩大存储容量；二、它采用MapReduce计算框架，分割数据进行分发到分布式文件系统中进行处理，然后再把处理过后的数据进行整合，保证了数据处理的高速；三、它采用存储冗余数据的方式来保证数据的安全性。用户可以轻松的架构并使用Hadoop系统，它主要具有四个优点：高可靠性、高扩展性、高效性、高容错性。
<!-- more -->


## Hadoop的构成


经过多年发展，hadoop已经成为以MapReduce和HDFS为核心的很多个项目的集合，包括Common、Avro、Chukwa、Hive、HBase等都对hadoop提供了互补性服务或在核心层上提供了更高层的服务。在主要讲解MapReduce之前，让我们先简要的看一下各项目的功能。

Core：一套分布式文件系统以及支持Map-Reduce的计算框架；

Avro：定义了一种用于支持大数据应用的数据格式，并为这种格式提供了不同的编程语言的支持；

HDFS：Hadoop分布式文件系统；

Map/Reduce**：**是一个使用简易的软件框架，基于它写出来的应用程序能够运行在由上千个商用机器组成的大型集群上，并以一种可靠容错的方式并行处理上T级别的数据集；

ZooKeeper：是高可用的和可靠的分布式协同（coordination ）系统；

Pig：建立于 Hadoop Core之上为并行计算环境提供了一套数据工作流语言和执行框架；

Hive：是为提供简单的数据操作而设计的下一代分布式数据仓库。它提供了简单的类似SQL的语法的HiveQL语言进行数据查询；

Hbase：建立于 Hadoop Core之上提供一个可扩展的数据库系统。


# Map/Reduce简介


MapReduce 是Google 公司的核心计算模型，它将运行于大规模集群上的复杂的并行计算过程高度地抽象为两个函数：Map 和Reduce。Hadoop 中的MapReduce 是一个使用简易的软件框架，基于它写出来的应用程序能够运行在由上千台商用机器组成的大型集群上，并以一种可靠容错的方式并行处理上T 级别的数据集，实现了Hadoop 在集群上的数据和任务的并行计算与处理。

一个Map/Reduce 作业（Job）通常会把输入的数据集切分为若干独立的数据块，由Map任务（Task）以完全并行的方式处理它们。框架会先对Map 的输出进行排序，然后把结果输

入给Reduce 任务。通常作业的输入和输出都会被存储在文件系统中。整个框架负责任务的

调度和监控，以及重新执行已经失败的任务。

通常，Map/Reduce 框架和分布式文件系统是运行在一组相同的节点上的，也就是说，计算节点和存储节点在一起。这种配置允许框架在那些已经存好数据的节点上高效地调度任务，这样可以使整个集群的网络带宽得到非常高效的利用。

Map/Reduce 框架由一个单独的Master JobTracker 和集群节点上的Slave TaskTracker 共同组

成。Master 负责调度构成一个作业的所有任务，这些任务分布在不同的slave上。Master 监控它们的执行情况，并重新执行已经失败的任务，而Slave 仅负责执行由Master 指派的任务。

在Hadoop 上运行的作业需要指明程序的输入/ 输出位置（路径），并通过实现合适的接

口或抽象类提供Map 和Reduce 函数。同时还需要指定作业的其他参数，构成作业配置（Job

Configuration）。在Hadoop 的JobClient 提交作业（JAR 包/ 可执行程序等）和配置信息给

JobTracker 之后，JobTracker 会负责分发这些软件和配置信息给slave 及调度任务，并监控它们的执行，同时提供状态和诊断信息给JobClient。


# Map/Reduce计算模型


在Google 发表论文时，MapReduce 的最大成就是重写了Google 的索引文件系统。而现在，谁也不知道它还会取得多大的成就。MapReduce 被广泛地应用于日志分析、海量数据排序、在海量数据中查找特定模式等场景中。但是，我们终究不得不承认，MapReduce终究不是万能的，它有自身使用的特定场景。


## JOBS


Hadoop MapReduce Jobs可以切分成一系列运行于分布式集群中的map和reduce任务，每个任务只运行全部数据的一个指定的子集，以此达到整个集群的负载平衡。Map任务通常加载，解析，转换，过滤数据，每个reduce处理map输出的一个子集。Reduce任务会去map任务端获取中间数据来完成分组，聚合。用这样一种简明直接的范式，从简单的数值聚合到复杂的join操作和笛卡尔操作。


## INPUT


MapReduce 的输入是hdfs上存储的一系列文件集。在hadoop中，这些文件被一种定义了如何分割一个文件成分片的input format来分割，一个分片是一个文件基于字节的可以被一个map任务加载的一个块。


## MAP


每个map任务被分为以下阶段：record reader，mapper，combiner，partitioner。Map任务的输出叫中间数据，包括keys和values，发送到reduce端。运行map任务的节点会尽量选择数据所在的节点。这种情况下，不会出现网络传输，在本地节点就可以完成计算。


### Record reader


Record reader会把根据input fromat生成输入分片翻译成records。Record reader的目的是把数据解析成记录，而不是解析数据本身。它把数据以键值对的形式传递给mapper。通常情况下键是偏移量，值是这条记录的整个字节块。


### Map


Map阶段，会对每个从record reader处理完的键值对执行用户代码，这些键值对又叫中间键值对。键和值的选择不是任意的，并且对MapReduce job的成功非常重要。键会用来分组，值是reducer端用来分析的数据。这本书会在设计模式方面提供大量的细节去解释键值对的选择。设计模式之间一个主要的区别是键值对的语义。


### Combiner


Combiner 是一个map阶段聚合数据的部件，相当于一个局部reducer，它是可选的。它根据用户提供的方法在一个mapper范围内根据中间键去聚合值。例如：数的总和是各个部分数量的和，你可以先计算中间的数目，最后再把所有中间数目加起来。很多情况下，这样能减少数据的网络传输量。发送（hello world，1）三次很显然要比发送（hello world，3）需要更多的网络传输字节量。Combiners可以被广泛的模式替换。很多Hadoop开发者忽视combiner，但能获得更好的性能。我们需要指出的是哪一种模式用combiner有好处，哪一种不能用combiner。Combiner不会保证总会执行，所以它是一个整体逻辑。


### Partitioner


Partitioner会获取从mapper（或combiner）来的键值对，并分割成分片，每个reducer一个分片。默认用哈希值，典型使用md5sum。然后partitioner根据reduce的个数执行取余运算：key.hashCode() % (number of reducers)。这样能随即均匀的根据key分发数据到reduce，但仍然要保证不同mapper的相同key要到同一个reduce。Partitioner也可以自定义，使用更高级的样式，例如排序。然而，更改partitioner很少用。Partitioner的每个map的数据会写到本地磁盘，并等待对应的reducer检测，拿走数据。


## REDUCE


Reduce任务分为以下阶段：shuffle，sort，reduce，output format。


### Shuffle and sort


Reduce任务开始于shuffle和sort阶段。这一阶段获取partitioner的输出文件，并下载到reduce运行的本地机器。这些分片数据会根据key合并，排序成一个大的数据文件。排序的目的是让相同的key相邻，方便在reduce阶段值得迭代处理。这一阶段不能自定义，由框架自动处理。需要做的只是key的选择和可以自定义个用于分组的比较器。


### Reduce


Reduce 任务会把分组的数据作为输入并对每个key组执行reduce方法代码。方法会传递key和可以相关的所有值得迭代集合。很多的处理会在这个方法里执行，也就会有很多的模式。一旦reduce方法完成，会发送0或多个键值对到output format。跟map一样，不同的reduce依据不同的逻辑情形而不同。


### Output format


Output format会把reduce阶段的输出键值对根据record writer写到文件里。默认用tab分割键值对，用换行分割不同行。这里也可以自定义为更丰富的输出格式，最后，数据被写到hdfs，你可以自定义输出格式。


# Map/Reduce设计模式


从上一章描述的mapreduce计算模型可以看出，mapreduce主要接受的是键值对，对大型键值对进行排序聚合等操作。下面让我们来看看有哪些模型可以应用于mapreduce。


## Summarization Patterns（摘要模式）


随着每天都有更多的数据加载进系统，数据量变得很庞大。这一章专注于对你的数据顶层的，概括性意见的设计模式，从而使你能扩展思路，但可能对局部数据是不适用的。概括性的分析都是关于对相似数据的分组和执行统计运算，创建索引，或仅仅为了计数。

通过分组数据集计算聚合排序是一种快速获取结果的好方法。例如,你可能想按某种规则计算出所存的钱的总数，或者按人口计算人们在互联网花费的平均时长。对于新的数据集，你可以开始用这些分析类型帮你计算出数据中什么东西有趣或唯一，和哪些需要仔细研究。

摘要模式主要有数值聚合、反向索引、用计数器计数三种具体的表现形式。


### 数值聚合模式


数值聚合模式是一个用于计算细节上的数据的统计值的通用模式。

**数值聚合模式使用所需条件：******



	
  * 处理数值类型数据或做计数。

	
  * 数据能根据指定字段分组


**数值聚合模式各组件功能图：[![1](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/1.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/1.jpg)******

**用法案例：**

1、  Word count。这也是讲解mapreduce被用到的最多的一个例子。通常的使用方式就是统计出现的单词的频率，相同类型单词聚合计数等。

2、  Record count。一种常用的根据特定时间周期（周，日，时等）获取数据流量规律的分析方法。

3、  Min/max/count。一种计算最小，最大值，或特定事件总和的分析。例如，用户第一次发帖时间，最后一次发帖时间，和一段时间内发帖的总数。


### 反向索引模式


反向索引即根据数据集生成索引，用于快速搜索或数据的富集能力。

**反向索引模式使用所需条件：**反向索引用在搜索查询时需要快速响应的情况下。查询的结果需要能被预先处理并放入数据库。

**反向索引模式各组件功能图：******

[![2](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/2.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/2.jpg)

**用法案例：**

Stackoverflow是一个技术类问答社区，上面经常引用wikipedia上面的一些信息作为评论提出。现在我们要在wikipedia网站的页面上添加所有引用到该wiki页面的stackoverflow的评论链接。

解决这个问题的做法就是在stackoverflow上预先进行检索，对每一条引用到wikipedia的评论都输出评论的链接和wikipedia的链接，得出键值对。然后对键值对进行聚合即可。


### 用计数器计数


这种模式利用了MapReduce框架本身的计数器功能在map端做全局的计算，不做任何输出。

**用计数器计数适用的情况：******



	
  * 在大数据集上收集计数或求和。

	
  * 创建的计数器个数较小，两位数以内。


**计数器计数模式各组件功能图：[![3](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/3.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/3.jpg)******

使用计数器能很快的完成计算，因为数据仅仅在map中处理，没有输出要写。性能主要取决于执行的map的个数和处理每条记录花费的时间。

相信这种模式的用法在看完模式的描述后就自然明白了。


## Filtering Patterns（过滤模式）


这种模式不会改变原来的记录。它的目的是找到一个数据的子集，或者更小，例如取前十条,或者很大，例如结果去重。这种过滤器模式跟前面章节的不同是，从更小的粒度认识数据，例如特殊用户生成的记录，或文本中用得最多的前10个动词。简单的说，过滤器允许你更清楚的看清数据，像在显微镜下一样。也可以认为是搜索的一种形式。如果你对找出所有有着特殊信息的记录感兴趣，你就可以过滤出不匹配搜索条件的记录。

抽样，一种通用的过滤程序，是指取出数据的一个样本，比如，取某字段值最高的几个，或随机取几个记录。取样能用来得到更小的，仍具有代表性的子数据集，在其上面进行分析，就不必处理庞大的数据集。很多机器学习算法在大数据集上运行低效，所以需要建模使其可运行在更小的子数据集。

子样本也可以用于开发目的。简单的抽取前一千条记录不是好的抽样方法，因为取的数据相似并不能代表整个数据集。一个均匀的抽样能够提供更好的数据的视图，并允许你的程序或分析能在现实数据上完成，甚至取样非常小。

根据过滤方式的不同，过滤模式下存在很多不同的模式，例如：filtering, Bloom filtering, top ten, and distinct。


### Filtering模式


最基本的模式，filtering。为其它具体过滤模式充当一个抽象模式。Filtering基于某种条件评估每一条记录，决定它的去留。

目的就在于过滤出我们感兴趣的数据并保存起来。

考虑用一个评估方法f，处理记录，如果返回true，保留，如果返回false，丢掉。

**过滤模式各组件功能图：[![4](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/4.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/4.jpg)******

**用法案例：******

**Closer view of data******

过滤出记录中有共性或有趣的子数据集，做进一步检查。例如，杭州移动对国际通话的数据中，只关心从杭州发出的去电。

**_Tracking a thread of events_******

抽取一系列连续事件作为大数据集的案例研究。例如，你可以通过分析apache web服务器的日志了解特殊用户怎样与你的网站交互。一个特殊用户的行为可能遍布于其它行为中，所以很难找出发生了什么。靠根据用户的ip过滤，就能够得到特殊用户行为更好的视图。

**Distributed grep******

Grep，一个使用正则表达式找出感兴趣的文本行的强大的工具，很容易并行应用正则表达式匹配每一行并输出。

**Data cleansing******

数据有时是脏的，或者很难理解，未完成，或错误的格式。数据可能丢失了字段，date类型的字段不能格式化成date，或二进制数据的随即字节可能存在。Filtering能检验每条记录是否结构良好，并去除任何垃圾数据。

**Simple random samping******

如果你想得到数据集的一个简单随机采样，可以使用filtering，用一个评估方法随即返回true或false。在简单随即采样中，数据集中的每条记录都有相同的概率被选中。可以根据源数据的数量算出百分比，得到要返回的记录的数量。例如，数据集有一万亿，你想得到一百万数据，那么评估方法应该每一百万次返回一次true，因为一百万个一百万是一万亿。

**_Removing low scoring data_******

如果你能根据排序给数据评分，你可以根据不满足某一临界条件来过滤数据。如果之前已经知道有些数据对分析也没意义，可以给这些记录评比较低的分。


## Data Organization Patterns（数据重组模式）


在很多组织结构方面，Hadoop和其它MapReduce使用案例仅仅是大数据分析平台上一片数据的处理。数据通常被转换成跟其它系统有良好接口的形式，同样，数据也可能从原来状态转成一种新的状态，从而使MapReduce分析更容易。数据重组模式讲述的就是对数据进行重组，分区、分片、排序等。


### 分层结构模式


结构分层模式会根据数据创建新的不同结构的记录。把基于行的数据转换成有层次的格式，例如，json xml。

**适用场景：******



	
  * 你的数据靠很多外键相联系

	
  * 你的数据是结构化的基于行的


**分层结构模式各组件功能图：[![5](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/5.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/5.jpg)******

**用法案例：******

_**Pre-joining data**_****

数据是杂乱的结构化数据集，为了分析，也很容易把数据组合成更复杂的对象。通过分层结构模式预处理，你设置好数据来充分利用分析nosql模型的优势。

_**Preparing data for HBase or MongoDB**_****

Hbase是很自然的存储这类数据的方式。所以可以用这种方法把数据搞到一起，作为加载到hbase或mongoDB的准备工作。创建hbase表，然后通过MapReduce执行大量导入是很高效的。


### 分区模式


分区模式是把记录移动到目录里（分片，分区，或箱）。但不关心记录的顺序。目的是获取数据集中相似的记录并把它们分区放入不重复的，更小的数据集。

**适用场景：******

这种模式最主要的是要预知分区的数量。例如，如果对一周的每天分区，可以得到7个分区。

可以通过运行一个分析获得能决定分区数量的信息。例如，有一段时间戳范围的数据，但不知道具体范围，跑个job计算出数据的具体范围。

这种模式利用partitioner分割数据。没有确切的分区逻辑。需要做的只是通过自定义partitioner决定一条记录去哪个分区。

**分区模式各组件功能图[![6](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/6.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/6.jpg)**

**用法案例：******

**_Partition pruning by continuous value_******

如果有某连续变量的排序，例如日期或数值，每次只会关注某一条件的数据的子集。把数据分成一块一块可以使job只加载相关的数据。

**_Partition pruning by category_******

这里不是根据连续的变量，而是某些已知类型，例如国家，地区号码，或语言。

**Sharding******

系统架构要分割数据，例如不同的磁盘，你需要把数据放进这些存在的分片中。


### 全局排序模式


全局排序模式关注数据中记录之间的顺序，实现全局排序。

**适用场景：**排序key是可比较的

**组件功能图**：全局排序无法用一张图简单的表达其组件的功能和运行模式，所以我们来描述一下这个算法。

**全局排序的算法：******

算法分为两部分：1、分析决定分割范围；2、对分割后的范围进行排序。

即运行两次mapreduce程序，并且mapreduce自定义的内容不同。

第一部分，对数据进行随机取样。分区是跟据这个随机样本进行的。原理是，**能把随机样本均匀分割的分区也应该能把大数据集均匀分割**。

**算法第一步的结构如下：******

确定整个数据集的记录数并算出对要分析的数据取多少百分比的样本是合理的。例如，计划跑1000个reducer的排序，取样10万条记录，应该均匀分区。假设有10亿条记录，相除，采样率就是0.0001，意味着0.1%的记录会在分析阶段运行。

mapper做一个简单的随机取样。当划分记录时，排序key作为输出key，使数据到reducer时看起来是排序的。这里不关心记录本身。

我们只使用一个reducer。收集排序的key进入一个排序列表，然后精简列表数据得到数据范围的边界，形成一个分区文件。

**第二步则相对简单，使用自定义****partitioner****的****MapReduce****程序，结构如下：******

mapper抽取排序key，跟分析阶段方式相同。

自定义的partitioner用于加载分区文件。然后获取这个分区文件的数据范围，来决定每条记录被发到哪个reducer。

reducer比较简单，Shuffle和sort做了繁重的工作。Reduce只是简单的把值输出。Reducer的个数应该等于分区的个数。


### 混乱洗排模式


混洗模式和全局排序在效果上是相反的，但接下来都会关注记录的排序。目的有两种，一种是混洗数据达到隐藏的目的；另一种是把数据随机分布，用于可重复的随机采样。

混洗模式的做法非常简单：

Map生成一个随机数对应每一条记录。

每条记录随机发送到reducer，每个reducer根据随机产生的key排序，产生那个reducer上的随机顺序。


## Join Patterns（关系数据连接模式）


数据遍布于各处，本身也很有价值。当我们合起来分析这些数据集时会发现一些有趣的关系。这就是join模式可以使用的地方。Join可以用一个小的引用集合让数据更丰富，或者过滤出或选择出指定的一系列类型。

在关系型数据库中，join操作可以使用简单的命令完成，数据库引擎会处理所有工作。对我们不幸的是，MapReduce中的join不会这么简单。MapReduce每次操作一个键值对，一般来自相同的输入。我们现在会处理至少两个输入数据集而且很可能有不同的结构，所以我们需要知道记录来自哪个数据集，以便正确的处理。一般情况下，join操作之前不会过滤数据，所以一些join操作需要把每个输入的字节都发送到reduce阶段，网络传输较繁重。例如，拿一个1T的数据跟另一个1T的数据做join，至少需要2T的网络流量-而且是在实际的join逻辑之前做的。

基于所有以上复杂的原因，要从几个不同的方式中选出一个最好的方式。由于框架只是简单的分解成map和reduce任务，有很多工作需要手动处理，有很多事情要考虑。当你了解了这些可能性，问题就是什么时候用什么模式。不管用什么MapReduce操作，网络流量都是非常重要的资源，join更会大量的使用它。让网络传输更有效率是值得考虑的，网络优化是这些模式中不同的地方。

**操作流程**：

mapper准备要join的数据，并从记录中抽取外键，当做输出key，整条记录作为输出值。输出值要标记上属于哪个数据集，

reducer收集每个输入组的值到临时列表，执行期望的join操作。例如标记为A的记录存储在代表A的列表，标记为B的记录存储在代表B的列表。然后迭代两个集合的所有记录join到一起。对于内连接，两个列表都不是空的就输出join后的数据。对于外连接，空列表也会跟非空列表join。反连接会确保只有一个列表是空的。非空列表记录跟空列表一块写出来。

[![7](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/7.jpg)](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/7.jpg)

**通常连接模式会跟过滤模式联合使用，以用来过滤掉大量重复无用的连接键值对。******


## Metapatterns（元模式）


元模式不是用于解决某个具体问题的，而是处理模式之间关系的。可以理解为“模式的模式”。首先讨论的是job链，把几个模式联合起来解决复杂的，有多个阶段要处理的问题。第二个是job 合并，用相同的MapReduce job执行多个分析的优化，达到一箭多雕的目的。

**Job chaining****（任务链）******

很多人发现用单独一个MapReduce job不能解决一个问题，需要把一连串的job联合起来运行，一些需要其它job的输出作为输入。一旦你开始熟悉用一些列MapReduce job解决问题时，你就真正进入hadoop使用的殿堂了。

任务链是一个较难处理的过程，因为它不是MapReduce 框架里确定的特性。像hadoop这样的系统设计成处理一个MapReduce job很容易，但处理一个有多个阶段要执行的job需要大量的工作量。要考虑到，某一阶段出错的job，要清除掉中间输出。

使用MapReduce的一个缺陷是数据太小没必要分布式运行。如果你认为链接两个job是正确的选择，要考虑第一个job有多少输出量。如果有大量的输出数据，尽量使用第二个MapReduce job。很多时候，Job的输出文件很小就可以在单节点上高效的执行。这两种方式是：或者在job完成后，在驱动代码里通过文件系统加载数据，或者用某种脚本封装在一起。

幸运的是，目前已经有很多成熟的框架用来处理任务链的事务流了。


# 总结


MapReduce正在飞速发展，新的功能和新的系统也会随着时间应运而生，但是我相信，本文谈到的设计模式，必将随MapReduce的存在而长久留存下去。掌握mapreduce的设计模式就等于在大数据的时代中掌握了一把利器。

附上pdf版本：[浅谈MapReduce设计模](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/浅谈MapReduce设计模式.pdf)[式](https://wonderflow.info/images/2013-12-15-e3808amapreduce-design-patternse3808be8afbbe4b9a6e7ac94e8aeb0-e6b585e8b088mapreducee8aebee8aea1e6a8a1e5bc8f/浅谈MapReduce设计模式.pdf)

参考文献：《Hadoop实战(第2版)》《MapReduce Design Patterns》
