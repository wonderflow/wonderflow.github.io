---
author: admin
comments: true
date: 2014-10-22 03:31:08+00:00
layout: post
slug: '%e5%8c%97%e4%ba%accontainer%e6%8a%80%e6%9c%af%e5%a4%a7%e4%bc%9a'
title: 北京container技术大会
wordpress_id: 1011
categories:
- CloudComputing
- IT
tags:
- container
- report
---

[可以看我的印象笔记共享，格式更加漂亮一些。](https://app.yinxiang.com/shard/s29/sh/db4cbf8e-eed7-45cd-89f0-1bca3a3512f3/e0188caf5e8b92181568798330f5356a)





此次去北京参加的container大会，主办方为csdn，实际会议主持为docker中文社区创始人马全一。会议时间为一天，从早上九点开始一直到晚上六点结束，共包含16个主题。虽说是container大会，但实际上基本围绕docker展开。期间也讲到了社区较为火热的IaaS平台openstack，以及PaaS平台Cloudfoundry，总的来说收获颇丰。





**会议流程：** ![会议流程](http://raw.githubusercontent.com/wonderflow/pic/master/container_con_flow.png)





以下我将按每个有收获的主题分别讲述我的思考和总结。





# Jrome（Hello Container Ops）





正如会议流程所示，第一个讲的是docker的开发工程师Jrome。我觉得他讲述了以下两点比较有意思： 1. docker container的应用场景： 1.1 Web applications 1.2 API backends 1.3 Databases (SQL, NoSQL) 1.4 Big data 1.5 Message queues 总的来说就是涵盖了几乎所有linux server下的工作负载，并且是跨linux不同版本的，据说后面还会加入windows的支持。 2. docker的优势/为什么用docker 2.1 非常轻，以容器做隔离性能损失极小。 2.2 平台无关性使开发、上线流程一体化，减轻了大量运维工作。 2.3 使用dockerfile对镜像高度可定制 2.4 部署、迁移，快速、稳定、简便 2.5 镜像的层级化使使管理非常方便，极易备份恢复等 2.6 共享宿主机使文件和信息共享变得非常方便(volume)





# 喻勇&孙宏亮（container技术在cloudfoundry中的演化）





关于这一项，宏亮学长的内容讲的很精彩，可以在实验室找机会给大家也分享一下这一块内容，在此就略过不表。





其中Frank(喻勇)除了介绍diego以外，还提到的几点让我感到非常有收获： 1. cloudfoundry现有的不足之处： 1.1 组件内部设计紧耦合,当时设计的时候就没有考虑到要再分割。 1.2 cloud_controller的负载过重，什么事情都要经过它 1.3 复杂的交互形成环状的依赖，使新功能不易添加，老的功能也不易维护。 1.4 Domain Specic，所有应用都是appname.[domain]，固定的模式使应用想要使用/扩展一个新域名变得非常苦难。 1.5 platform Specic，几乎只能稳定运行在linux ubuntu平台上 1.6 ruby编写的dea使应用处理效率低下 2. 容器技术影响未来PaaS的走向 2.1 从软件开发的角度看，开发测试和软件发布的流程都会发生较大的变化。产生诸如： * 产品化的container生命周期管理方案 * 多container复杂应用协同工作 * image host交互（类似github） * image的静态安全性





2.2 从云计算产业链来看： * SaaS层面，由于以container形式交付应用变得非常简便，就会出现更多针对云平台的应用; * PaaS层面，container使部署和维护变得简单，DevOps更加实际，将成为巨大的技术推动力，PaaS会更易用; * 而下一代的IaaS也将变得更加强大，可以从各种粒度上面根据需求分离硬件。





在我看来，docker作为容器一旦加入到IaaS层，那么应用可以直接在docker中运行，只要IaaS管理的好，PaaS的概念终将随之淡化。实际上它们之间的界限也就越来越不分明了。





# 刘永峰(Docker时代，公有云面临的挑战和机遇)





刘永峰是腾讯的产品经理，他指出了目前IaaS存在的问题： 1. 使用门槛高 2. 资源利用率不高 3. 迁移困难 4. 无统一的标准





而他所指的挑战是：Docker弥合了不同的IAAS之间的差异，降低IAAS 服务商用户粘性，使得跨云服务商迁移更加自由。 他所指的机遇是：Docker催生了云计算服务标准化: 1. 服务构建标准化（Dockerfile） 2. 服务交付形态标准化（容器） 3. 服务运行环境标准化（Docker引擎）





他说，**云计算的本质不是虚拟化，而是计算变成服务！**





他还认为，构建云端集成开发环境将是未来趋势。





# 李闻（Matrix：基于container的大规模集群操作系统）





介绍了一下Matrix的规模，从哲学的高度讲了一下产品开发要遵守的一些规范。





# 于顺治(Container在搜狐PaaS平台中的应用实践)





讲述了一下搜狐从用“沙盒”（即针对编程语言做特殊安全隔离的一类容器）到使用container的路程。





描述了container相较于沙盒的一些优势，上面已经介绍过一些。





# 马全一（构建 Docker 的开发环境）





作为主持人，马全一的报告讲的更多的是部署docker开发环境的一些技术上的流程，这些需要具体的实践才能有所收获。





# 蔡书(关于性能--Docker)





http://caishu.github.io/csdn2014/#/ 蔡书是红帽的软件架构师，它在会上讲述了提升docker性能的一些方法。 除了docker性能调优的一些参数以外，它还指出可以通过"tuned/tuned-adm"这样的系统工具进行性能调优。 另外，他还指出，docker的优势在于其灵活性，适用于 1. 软件分发； 2. 环境部署与迁移。





而想要获得较高性能的方法就是对container上的应用： 1. 大量同构应用的密集部署； 2. 重计算，轻端口的应用。





它还给出了一张非常精致的图片，展示了所有他说到的概念和工具，确实值得收藏： ![概念与工具](https://raw.githubusercontent.com/wonderflow/pic/master/docker-on-rhel7.png)





# 艾奇伟（Docker的安全性解析）





艾奇伟又名大鹰，他分析了docker关于安全性方面的设计，提出了docker的弱安全性（本来就没有为安全考虑）





指出安全性和性能有一定的矛盾，而较为安全的做法可能是在虚拟机中跑docker或其他container。





## 毛文波（网络虚拟化与SDN实现Docker连通）





关于SDN这个主题，宏亮学长的报告中提及较多。





我对网络并不了解，后面需要加强学习。





# 李明宇（和而不同——OpenStack&Docker）





介绍了openstack中docker的使用方式： 1. 把docker作为虚拟机同一层次的概念，使openstack不仅能管理虚拟机，也能管理docker。原理就是通过调用docker API来实现。 2. 使用Heat这个编排引擎来启动和管理docker镜像。





我之前也研究过一段时间Heat，所以会后向李明宇博士提出了我的困惑，即**Heat如何与虚拟机交互获得状态反馈**的问题。





李博士也表示没有特别好的办法，只能通过nova list查看虚拟机创建进度，通过nova的其他IP这样分门别类的查看其它内容的状态，但是想要进入虚拟机里面查看目前还没有好的方法。 这也是我目前研究Heat自动化部署的一个瓶颈。





相同的问题也问过网易的学长，他们则说没有尝试过使用Heat部署cloudfoundry这样较为重型的带有明确拓扑关系的集群。





# 总结





关于技术，以上便是我在container大会的收获。实际上，技术之外，企业人士对产品的敬业精神，对产品的深度思考等，都让我收获良多。我觉得平时我们除了在技术层面的探索以外，也要跳出技术的思维局限，站在更广阔角度思考问题。



