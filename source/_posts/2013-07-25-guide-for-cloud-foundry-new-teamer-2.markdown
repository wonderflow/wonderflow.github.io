---
author: admin
comments: true
date: 2013-07-25 05:41:20+00:00
layout: post
slug: guide-for-cloud-foundry-new-teamer-2
title: Guide for Cloud Foundry New Teamer
wordpress_id: 590
categories:
- CloudFoundry
tags:
- cloudfoundry
- ruby
---

## Preparation





1、阅读以下博客： 
了解架构，阅读“[深入CloudFoundry一周年](http://qing.blog.sina.com.cn/2294942122/88ca09aa33003k2j.html)”





2、请准备好一个linux的客户端，如果没有请申请一个虚拟机，并安装好ubuntu10.04（windows下使用远程连接的方式是使用putty软件。）





3、如果不熟悉linux，请阅读linux相关命令，初步了解。





## Start







  1. 定位cloudfoundry集群： 请使用$ curl [your domain]。若显示出正常信息则初步表面集群配置正确，否则请找帮你配置集群的人。 


  2. 配置linux源，提高下载速度。(百度一个ubuntu cn的源) 




{% codeblock %}
$ /etc/apt/sources.list $ sudo apt-get update 
{% endcodeblock %}




  1. 安装git，为了下载一个基本应用，github上面的。 




{% codeblock %}
$ sudo apt-get install git
{% endcodeblock %}




  1. 安装maven，为了打包成war包，期间阅读下maven的功能，不求甚解。 




{% codeblock %}
$ sudo apt-get install maven2
{% endcodeblock %}




  1. 用git下载cloud foundry基本样例。(请注册github，并百度相关用法，[github简单应用](https://github.com/wonderflow/cloudfoundry-samples/tree/master/hello-spring-mysql) )




{% codeblock %}
$ git clone [http://***.git]
{% endcodeblock %}




  1. cd到相应根目录，执行maven打包 




{% codeblock %}
$ mvn clean package
{% endcodeblock %}




  1. 把war包上传，需要一个干净的目录，里面只有一个war包 




{% codeblock %}
$ mkdir newdir $ mv *.war newdir/
{% endcodeblock %}




  1. 在继续下去之前，可能你需要安装ruby以及vmc等相关工具。 ！！建议【安装的版本ruby1.9.3】！！ [install_ruby_guide](http://ruby-china.org/wiki/install_ruby_guide)（请一定要有耐心,到了这一部时间极长) 




{% codeblock %}
$ rvm install 1.9.3 --with-readline-dir=$rvm_path/usr # 安装 Ruby 1.9.3 
{% endcodeblock %}




尤其是出现 it may takes time....的时候，要等很久。 但是如果等了很久还是那样就，没关系的。




{% codeblock %}

 #安装完后查看下rails包是否正常。 
 $ gem list #如果是空的，执行 
 $ rvm use 1.9.3 
 #然后安装vmc 
 $ gem install vmc
 
{% endcodeblock %}




<!-- more --> 
9. 按下述执行




{% codeblock %}

  $ cd [只有war包的目录] 
  $ vmc target [your domain] 
  $ vmc register 
  >emai：　[your_email_address] 
  >password: [your password] 
  >confirm password:[your password] 
  $ vmc login 
  >emai：　[your_email_address] 
  >password: [your password]
  $ vmc push 
  Name> test1 
  Instances> 1 
  1: spring 
  2: other 
  Framework> spring 
  1: java 
  2: java7 
  3: other 
  Runtime> 1 
  1: 64M 
  2: 128M 
  3: 256M 
  4: 512M 
  5: 1G 
  6: 2G 
  7: 4G 
  8: 8G 
  9: 16G 
  Memory Limit> 512M 
  Creating spring-mysql... OK 
  1: spring-mysql.cf1.load 
  2: none 
  Domain> test1.cf.test 
  [根据给你部署集群的人-配置的不同而不同] 
  Updating spring-mysql... OK 
  Create services for application?> y 
  Name> mysql-spring 
  Creating service mysql-spring... OK 
  
{% endcodeblock %}




后面的操作全部默认（包括save configuration 选择n也是）





完成后得到一个网址，可以访问。（就是之前给你确定的Domain）





## 相关资料





[AB test](http://studiogang.blog.51cto.com/505887/386852)； 
[示例应用的解析](http://blog.csdn.net/yurunsheng/article/details/7978240)； 
[vmc的简单说明](http://blog.csdn.net/dengsilinming/article/details/7516655)； 
[vmc项目主页](https://github.com/cloudfoundry/vmc)；





## Some tips related





### 关于BOSH：





登陆到bosh，ssh root@[ip address] （you will know the password if you have the authentication）





安装自己相应的配置文件。




{% codeblock %}

$ bosh status #查看配置情况，顺便看到yml配置文件的地址 
$ bosh deployment [/[youraddress]/deployments/cf/cf_test_2.yml] #添加自己的配置文件。 
$ bosh vms #查看组件整体情况 
$ bosh restart XXX #重启某组件

{% endcodeblock %}




### rvm管理工具的一些用法。




{% codeblock %}

$ bash -l #bash重启 
$ rvm use 1.9.2 #使用某个版本 
$ rvm 1.9.2 --default 
$ gem list #查看gem包 
$ gem -v 
$ ruby -v #查看当前使用的版本

{% endcodeblock %}




Gem卸载、安装相关，以vmc为例。





先卸载当前版本，再安装0.5.1版本




{% codeblock %}

$ gem uninstall vmc 
$ gem install vmc --version=0.5.1 检查nameserver是否正常。 
$ vim /etc/resolv.conf 

{% endcodeblock %}




vmc登录出现一些问题，使用-t选项，可以看到cloudcontroller里面的代码。




{% codeblock %}

$ vmc register -t #可以看到cloudcontroller里面相关代码。 #登录到集群cloudcontroller相关模块查看cloudcontroller 的yml文件。 
$ ssh [root@10.10.102.165] #请使用你自己集群cloudcontroller的文件 
$ vi /var/vcap/jobs/cloud_controller/config/cloud_controller.yml #之前遇到一个https不能使用的问题，改成http即可。

{% endcodeblock %}




**others：you may ask the guys nearby，if you are clever, they will glad to help you。**





## Learn Ruby





一、 阅读一本ruby语言的相关电子书，建立初步概念，网上很多。
  [《Ruby从入门到精通》](http://ishare.iask.sina.com.cn/f/23034819.html) [《Ruby语言入门教程》](http://wenku.baidu.com/view/dd769633a32d7375a4178059.html)





二、如果有脚本语言的基础，阅读：[“快速读懂ruby代码问答”](http://www.btsmth.com/show_topic.php?en_name=Programming&gid=156911)





三、 看别人高手写的小代码，[Codeforces](http://codeforces.com/)





四、Cloud Foundry 源码：[@github](https://github.com/cloudfoundry)





## Components List





**LB**（load balance）:负载均衡器，对进来的url请求进行负载均衡的分发，分发到Router，使用nginx。





**Router**：对所有进来的Request进行路由，包括nginx和router两个组件。目前使用的nginx是带LUA语言扩展的第二版，它里面加入了url查询和统计逻辑。处理了原来ruby写脚本处理转发的瓶颈。从架构图上可以看出，从router路由出去的数据，很大一部分是流向了DEA，还有一部分则是流向了CloudController组件。





**DEA** （Droplet Execution Agent）:它负责把上传的应用制作并存储成CloudFoundry能使用的Droplet，在应用开始执行后，dea把droplet 复制成很多份实例去运行。同时它还进行一个对应用程序的容器作用，承载应用程序在其中孤立运行，对系统进行保护。最后它给router一个信息让应用对应的request访问到这里。





**CC** （Cloud Controller）： Cloud Controller是CloudFoundry的管理模块。主要工作包括： a) 对apps的增删改读； b) 启动、停止应用程序（通过DEA）； c) Stagingapps（把apps打包成一个droplet，通过Stager）； d) 修改应用程序运行环境，包括instance、mem等等； e) 管理service，包括service与app的绑定等； f) Cloud环境的管理； g) 修改Cloud的用户信息（通过UAA，ACM）； h) 查看CloudFoundry，以及每一个app的log信息。





**CCDB** （CloudController Database）：cc的数据库，用来存储用户的应用信息，在启动应用与创建instance的时候会用到。





**UAA**（User Account and Authentication）：是一个用户认证的模块，负责验证用户的信息。





**UAADB**（User Account and Authentication Database）：uaa的数据库，用来存储用户的信息，同样在登陆的过程中可以测试。





**Health Manager**: 从各个DEA里面拿到运行信息，然后进行统计分析，报告等。统计数据会与CloudController的设定指标进行比对，并提供Alert等。这些信息会发送到dashboard显示出来。





**Service**：这个模块是很多个服务的合集，由多个节点共同构成，包括所有支持的数据库等。 『mongodb，mysql ，postgresql ，rabbit ，redis ，service_gateway』 
大致服务流程如下： 
a) 开发人员通过VMC创建一个Service; 
b) VMC把其转为Restful接口到CloudController； 
c) CloudController通过Restful接口发送创建命令到Service Gateway； 
d) ServiceGateway通过NATS发送provision命令道Service Node； e) ServiceNode创建Service； 
f) Service Node把返回创建的Service访问方法回Service Gateway；
g) ServiceGateway返回访问方法到CloudController； 
h) CloudController把创建的Service访问放法注入到应用运行环境； 
i) 最终用户可以直接访问其服务。





**NATS** (Message bus)：CloudFoundry的架构是基于消息发布与订阅的，联系各模块的就是nats的组件。也就是说它是cloudfoundry的神经系统。 所以nats是一款基于EventMachine、使用“发布--订阅”机制的轻量级消息中间件。 『CloudFoundry作为一个弹性设计的，多模块的分布式系统，且支持模块自发现，错误自检，保证模块间低耦合。其根本原理就是基于消息发布订阅机制构建。每台服务器上的每个模块会根据自己的消息类别，向Message Bus发布多个消息主题；而同时也向自己需要交互的模块，按照需要的信息内容的消息主题订阅消息。』





**Dashboard**: 用于显示系统内部各情况的板报。





**Opentsdb**: 负责存储dashboard所要显示各类信息的数据库。





**Collector**: a) 发现各组件，创建varz信息（组件的各种信息，随组件的不同提供的信息也不同） b) 收集各组件提供的varz信息 c) 处理varz信息并存储到spentsdb





**Learn More by Yourself**: 
http://group.google.com 【bosh-users】 【cloudfoundry】 and so on





——Sun Jianbo 2013年7月25日



