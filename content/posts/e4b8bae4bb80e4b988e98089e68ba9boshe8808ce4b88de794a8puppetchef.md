+++
author = "admin"
date = "2014-03-17T06:44:24Z"
slug = "2014/03/17/e4b8bae4bb80e4b988e98089e68ba9boshe8808ce4b88de794a8puppetchef"
title = "为什么选择Bosh而不用Puppet/Chef"
Categories = ["CloudComputing", "CloudFoundry", "IT"]
Tags = ["bosh", "cloud", "puppet"]
+++

总的来说，Puppet/Chef是配置管理（Configration Management）工具，Bosh是云管理（Cloud Orchestration）工具。





Bosh的功能包含了Puppet/Chef所有的功能，并且Bosh把IaaS和PaaS的管理工作结合了起来并且实现了自动化，节省了大量的工作。但是Bosh需要IaaS层提供API，并且需要专门为API编写Bosh适配的CPI（Cloud Provider Interface），故而部署和使用的要求较高。





与此同时，Puppet/Chef工具自2000年起就已开始被广泛使用，其标准化的配置流程也渐渐成为了业界标准。因为其小巧灵活的特性，所以使用要求较低。与Bosh相比，除了配置管理相关的工作以外，还需要人工的去做配置网络、管理虚拟机、版本控制管理等等其他工作，使用起来较为繁琐。





目前针对Cloud Foundry的部署，基本有两种大众化的解决方案。







  1. 使用Bosh，以及拥有Bosh CPI的IaaS平台。 


  2. 使用vagrant管理虚拟机，使用puppet/chef来部署。





而第2点则更多的用于在物理机上部署Cloud Foundry或者在个人开发机上部署CF环境。





另外，Stark & Wayne 公司的CEO Dr Nic Williams在介绍为什么使用bosh的时候也说道，Bosh为开发人员、测试人员、部署人员配置了一套统一的环境，免去了很多不必要的麻烦和工作，使得工作变得更加简单了。





相比之下，使用bosh进行部署确实如Dr Nic Williams所言，“It makes life easier !”





下面详细介绍一下两个工具。
<!-- more -->





# PUPPET简介





Puppet是基于Ruby的开源系统配置管理工具，依赖于C/S的部署架构。主要开发者是Luke Kanies，遵循GPL v2版权协议。





作为一款配置管理工具，Puppet除包含正常的配置管理工具的功能外，它还有跨平台的特性。Puppet的语法允许你创建一个单独脚本，用来在你所有的目标主机上建立一个用户。所有的目标主机会依次使用适用于本地系统的语法解释和执行这个模块。（举例：如果这个配置是在Red Hat服务器上执行，建立用户使用useradd命令；如果这个配置是在FreeBSD主机上执行，使用的是adduser命令。）





Puppet使用跨平台语言规范，管理配置文件、用户、软件包、系统服务等内容，在Puppet里这些内容都被看做是“资源”，每种资源都有对应的属性，如软件包有安装、不安装的属性，文件有权限属性等。Puppet的代码主要由这些资源和其属性组成。其代码化的好处：分享，保存，快速的恢复和部署。





## Puppet架构





![image](http://raw.github.com/wonderflow/pic/master/puppet.png)





Puppet是一个C/S架构的配置管理工具，在中央服务器上安装puppet-server软件包（被称作Puppet master）。在需要管理的目标主机上安装puppet客户端软件（被称作Puppet Client）。当客户端连接上Puppet master后，定义在Puppet master上的配置文件会被编译，然后在客户端上运行。每个客户端默认每半个小时和服务器进行一次通信，确认配置信息的更新情况。如果有新的配置信息或者配置信息已经改变，配置将会被重新编译并发布到各客户端执行。也可以在服务器上主动触发一个配置信息的更新，强制各客户端进行配置。如果客户端的配置信息被改变了，它可以从服务器获得原始配置进行校正。





同时，Puppet也是易于扩展的。定制软件包的支持功能和特殊的系统环境配置能够快速简单的添加进Puppet的安装程序中。作为一款开源的工具，Puppet社区正快速壮大，并且许多新的想法不断融入，促使开发、更新和模块每天都在呈现。





# Vagrant简介





Vagrant 是一个虚拟机管理工具，非常适合用于搭建开发环境。它提供了一套易用的配置规则，比人工操作或者使用虚拟机软件提供的命令行要方便，且可重复。Vagrant最大的用途就是为开发团队配置统一的开发环境，与不同平台的虚拟机供应商结合，无论开发人员用的是 Windows 还是 Mac，都可以跑一个一致的 Linux 开发环境。





目前用的最广的是开源的虚拟机VirtualBox。另外VMware等其他供应商的产品也支持，但是需要收费。





而与IaaS层的结合使用方面，并没有明确的官方支持，官方只是提供了插件的编写方式允许开发者自行编写插件。
目前开源社区Github上存在[AWS](https://github.com/mitchellh/vagrant-aws)、[CloudStack](https://github.com/klarna/vagrant-cloudstack)以及[OpenStack](https://github.com/FlaPer87/vagrant-openstack)的插件。





Vagrant启动虚拟机后，默认是跑在后台，并且不显示图形界面的，这时候需要用 ssh 连接虚拟机。





网络方面，Vagrant 的网络配置支持3种：







  * Forwarded ports:将虚拟机的某个端口绑定到本机端口。


  * Private networks:分配给虚拟机一个私有 ip，这样可以在本机上访问虚拟机的所有端口。


  * Public networks:让虚拟机暴露在真实的网络中，跟本机同等。





虚拟机支持的操作系统方面（vagrant官方），Vagrant 官方只提供了 Ubuntu LTS 的 box，需要打包 CentOS，OpenSUSE 之类的 box 需要自己动手。





总体来说，**Vagrant+puppet的部署方式并不适合大规模集群的部署，只是在没有Bosh出现的时候当成临时方案来使用，其中经历了太多不同的工具，可能遇到的问题以及得到技术支持的难度都相应增加。并且，Vagrant的使用一般都需要IaaS层提供接口，如果IaaS层提供了接口，那么就达到了使用Bosh的门槛，无需舍近求远。**





# BOSH简介





Bosh是用于发布（release engineering）、部署（deployment）以及维护大规模分布式服务(lifecycle management of large-scale distributed services)的一个开源工具集。





Bosh起先是因为CloudFoundry部署有需要而开发的，后来野心变大，成为了可以为大部分分布式系统提供自动化部署及生命周期管理的云任务编排（Cloud Orchestration）工具。





## Cloud Orchestration





任务编排（Orchestration）就是将服务中涉及的工具、流程、结构描述成一个可执行的流程，结合软硬件的接口，使之可以按照描述自动化、可重复地交付。具体到Cloud Orchestration，主要就是通过定义一系列的元数据来描述交付过程中涉及的环境管理(OS，network，storage等)、软件包管理、配置管理以及监控和升级操作，以便自动化和标准化地完成从申请资源到交付/升级服务的整个生命周期。





为什么要Cloud Orchestration呢？简单来说就是降低管理大规模集群和服务所需的人力和时间消耗，保证服务环境的一致性和可靠性。具体来说主要是面对以下三个不良现象：







  * 将IaaS当作单纯的VM生成器： 这个现象非常普遍，因为IaaS最基本的功能就是提供VM，然而其提供资源标准操作手段的价值被忽略了，API才是云服务的入口。



  * 多种角色贯穿整个环境管理过程： OS由安全人员指定，网络/存储归运维人员，数据库等管理归中间件人员配置。多种角色之间如果沟通不畅，很可能导致环境发生不可预知的错误。



  * 多种环境管理不一致： 开发/QA/产品环境由不同的团队采用不同的方法维护，多种环境管理不一致可能会导致服务运行环境不可预期，出现错误难以复现等问题。






针对以上三个问题，所以我们需要一个统一的手段去管理从VM生成到服务交付中的各个环节，控制服务的运行环境。





## Bosh架构





针对Bosh的架构，官方给出了一张非常漂亮的图。
![image](http://raw.github.com/wonderflow/pic/master/bosh.png)





BOSH在整体架构上跟Cloud Foundry很相似，采用NATS为消息中间件。





### CLI





CLI是用户客户端的一个命令终端，提供了BOSH的命令界面。所有bosh用户的命令都通过bosh_cli发出，有关CLI的完整命令介绍可以参见BOSH官方文档的BOSHCommand Line Interface部分。





### Director





Director组件是BOSH的核心，其作用是负责虚拟机的创建、部署和其他软件和服务生命期的管理。
![image](http://raw.github.com/wonderflow/pic/master/director-components.png)





有关Director的更详细的信息可以查看官方的[Director文档]((http://docs.cloudfoundry.com/docs/running/bosh/components/director.html))。





### Agent





Agent运行在已经创建出的VM上，每个BOSH创建的虚拟机都有一个Agent。它是BOSH在VM上的代理，通过从Director获取配置信息，对虚拟机分配不同的角色。比如，在部署cloudfoundry集群的时候，我需要添加一个DEA节点。这时BOSH先会创建并启动一个VM，其中也运行着BOSH的Agent。然后Director就会发送一个命令到VM的Agent，明确告知这个VM需要安装有关DEA的组件和软件包，以及详细的配置信息。Agent就会根据这些信息创建一个配置好的DEA节点，这样VM就分配好了角色。





### Health Monitor





HealthMonitor从Agent接收状态和生命周期信息，并且可以发送警告报警。这里的Health Monitor是一个简单的事件机制，当一个组件更新时，不会产生报警。





### Blobstore





Blobstore是用来存储用户上传的Release。用户通过CLI指令上传Release，通过Director将Release存储到Blobstore中。当你对上传的Release进行部署时，BOSH将在Blobstore中排列这些编译包和存储结果。当BOSH部署一个job到VM上时，Agent将从Blobstore中拿出指定的BOSH Jobs和相关的资源包。





BOSH也使用Blobstore作为大型有效载荷的中间存储，比如日志文件，超过Agent处理消息总线最大容量的输出。





现在有三种Blobstore支持BOSH：Atmos、S3、simple blobstore server。





### db





BOSH用来存储有关VM的元数据的数据库。





### IaaS interface





这里就是BOSH CPI（Cloud Provider Interface）的部分。CPI封装了一套IaaS的API，它使得BOSH能够通过对CPI的调用从而实际调用IaaS的API。CPI实际上间接的完成了Bosh的跨平台特性。





# BOSH与PUPPET/CHEF对比





![image](http://tiewei.github.io/images/bosh_features.png)





从上图可以看出，puppet/chef属于Configuration Management工具。





Configuration Mangagement工具包含的功能有：







  1. Provisioning: 标准化供应，提供统一的配置管理


  2. Templating: 模板化，提供一个基础模板，其余资源配置都与该版本相同


  3. Job Consistency: 一致性，保证了部署与配置管理工作的一致统一


  4. Packaging: 部署内容封装


  5. Machine Updates: 持续维护，可更新





除上述功能外，传统的Orchestration工具还包括：







  1. Discovery: 易于问题的及时发现和反馈


  2. Interdependence: 软硬件之间的结合，体现在这里就是PaaS与IaaS的结合。


  3. LifeCycle Management: 部署包的生命周期管理，包括从开发环境、测试环境到生产环境的统一化管理





Bosh还包括：







  1. Backup & Restore: 软件包的备份和存储功能


  2. Stemcell: 基础模板环境的配置和更改


  3. Cross Cloud Portability: 云之间可移植性强


  4. Monitor & Alert: 实时监控与反馈


  5. Networking: 自动化的网络管理


  6. Rolling Updates: 更新可回滚，使用git版本控制


  7. Storage Management: 软件包的存储管理功能。





由此可见，Bosh涵盖了所有Puppet功能，并且在包管理、网络管理以及服务器管理上提供了极为全面的解决方案。但是也正是因为Bosh涵盖的东西太过广泛，导致使用Bosh需要达到的起点较高必须要有IaaS层开发的接口，并且必须要有专为这些接口写的Bosh层接口CPI(cloud provider interface)存在才行。





# 其他工具对比





除Bosh以外，还有一些其他Cloud Orchestration工具。影响较大的包括 "AWS Cloud Formation" , "Ubuntu Juju" , "Openstack HEAT"。





AWS CloudFormation是亚马逊向开发人员和系统管理员提供的一种简便地创建和管理一批相关AWS资源的方法，并通过有序且可预测的方式对其进行资源配置和更新。用户可以通过AWS CloudFormation的示例模板或自己创建模板来介绍AWS资源以及应用程序运行时所需的任何相关依赖项或运行时参数。当设置完成后，用户可通过按受控制、可预测的方式修改和更新 AWS 资源，可像执行软件版本控制一样对AWS基础结构进行版本控制。





Juju 是 Ubuntu 旗下的一款工具，可以对云端的服务进行快速可靠的布署，包括可以扩展云端业务，因此管理员可以很容易地布署 Wordpress 博客系统、MongoDB 大系统管理系统、Ceph 分布式文件系统等。用户可以通过命令行和图形界面使用 Juju，图形界面被称为“Juju GUI”。Juju GUI界面基于网页，精美、对用户友好、直观，用户可以通过网页简单的操作完成复杂的功能。





HEAT是openstack的官方项目，其思想来自于 AWS 的 Cloud Formation，甚至 template 的 schema 都有兼容版本。对于Openstack而言， HEAT是其通向为客户创造价值而非只是温床(平台) 的入口， United Stack 的工程师甚至将其定位在 Devops2.0 工具。因为其能利用在 openstack 上的存储、计算、网络等所有资源，根据客户的需要定义一个完全的运行环境，形成了从VM到APP的闭环。





下图是一个综合的比较：
![image](http://raw.github.com/wonderflow/pic/master/current_solution_comparsion.png)





# 引用列表







  * [Cloud Orchestration In My View]((http://tiewei.github.io/devops/Cloud-Orchestration-In-My-View))


  * [Dive-into-BOSH-Overview]((http://tiewei.github.io/bosh/Dive-into-BOSH-Overview))


  * [Cloudfoundry的部署工具BOSH介绍]((http://blog.csdn.net/alan90121/article/details/8184849))


  * [开源自动化配置管理工具Puppet入门教程]((http://os.51cto.com/art/201011/232845_all.htm))


  * [Configuration Management](http://en.wikipedia.org/wiki/Configuration_management)


  * [Why Must I Use Cloud Foundry's Bosh? I just Learned Chef/Puppet!](http://www.infoq.com/presentations/cloud-foundry-bosh)



