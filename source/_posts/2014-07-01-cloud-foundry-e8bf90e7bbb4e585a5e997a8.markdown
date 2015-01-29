---
author: admin
comments: true
date: 2014-07-01 06:05:03+00:00
layout: post
slug: cloud-foundry-%e8%bf%90%e7%bb%b4%e5%85%a5%e9%97%a8
title: Cloud Foundry 运维入门
wordpress_id: 959
categories:
- CloudFoundry
tags:
- guide
---

@(CF V2相关)[guide]





之前写过一个[Guide for Cloud Foundry New Teamer](http://wonderflow.info/archives/590)。不过似乎已经有些过时，那会实验室主要是针对的CF v1进行的研究，现在已经全面进入V2时代了。所以更新一下关于CloudFoundry运维的一些内容。如果有时间也可再回头看看V1的那个帖子，希望能有所帮助。





# 部署





关于部署，目前使用的工具一般有两种，[BOSH](http://docs.cloudfoundry.org/bosh/)和[cf_nise_installer](https://github.com/yudai/cf_nise_installer)。BOSH适用于集群安装，cf_nise_installer适用于单节点安装。下面主要以cf_nise_installer的安装方法为主描述部署的运维流程。





cf_nise_installer实际上就是一大堆shell脚本建立起来的项目，对shell脚本熟悉的人打开上面的链接就可以看到整个部署的流程。





第一步就是安装运行环境。





通过cf_nise_installer中的这段[install.sh](https://github.com/yudai/cf_nise_installer/blob/master/scripts/install.sh)脚本的代码我们可以看到它的安装流程如下：




{% codeblock %}
./scripts/install_ruby.sh
source ~/.profile
./scripts/clone_nise_bosh.sh
./scripts/clone_cf_release.sh
./scripts/install_environemnt.sh
./scripts/install_cf_release.sh

{% endcodeblock %}






  * 安装ruby：cf_nise_installer使用的是rbenv这个ruby安装工具。同样比较有名的ruby安装工具还有rvm，这两个工具任选一个即可，如果是使用某个固定的ruby版本的话，建议使用[源码安装](http://scoop.simplyexcited.co.uk/2012/03/02/install-ruby-1-9-3-p125-from-source/)。


  * 下载[nise_bosh](https://github.com/nttlabs/nise_bosh)项目：实际上cf_nise_installer是基于nise_bosh的一个脚本，真正执行安装的就是nise_bosh这个项目。nise_bosh是一个基于bosh的项目，把bosh关于IaaS层的内容去除，保留了虚拟机上组建安装的内容。所以使用nisebosh无需IaaS层的API支持，只需要虚拟机即可安装。


  * 制作cf_release：cf_release是cf源码经过编译后的内容。从github上clone下来的[cf_release](https://github.com/cloudfoundry/cf-release)，checkout到指定版本[git使用简易入门](http://wonderflow.info/wp-content/uploads/2013/08/chinese_git_pocket_wonderflow.pdf)，然后执行`git submodule update --init --recursive`把子模块submodule下载下来，再执行`bosh create release`命令，就可以得到一个完整的cf_release，当然，这里面又是漫长的下载。实验室已经制作了几个常用的cf_release版本，在[内网](http://10.10.103.102/releases/)可以下载。


  * 安装环境，这里执行的其实就是[nise_bosh/bin/init](https://github.com/nttlabs/nise_bosh/blob/master/bin/init)这个脚本，打开就可以看到下载了很多基础的运行时环境，以及监控使用的monit工具。


  * 安装cf组件。到这里就是真正的安装cf组件了。安装的命令很短，东西都在配置文件里面了。可以打开[manifests/template](https://github.com/yudai/cf_nise_installer/blob/master/manifests/template.yml)查看一下。默认的域名都设置成了`192.168.10.10.xip.io`，执行[ generate_deploy_manifest.sh](https://github.com/yudai/cf_nise_installer/blob/master/scripts/generate_deploy_manifest.sh)脚本可以更改域名和密码，不过需要在环境变量中导入`export NISE_DOMAIN=[你的域名]` `export NISE_PASSWORD=[你的密码]`





**关于域名** 架设本地域名解析服务器，如bind9之类的，然后在/etc/resolv.conf下面的nameserver加上本地域名解析服务器IP即可。在域名解析服务器上加上一条域名对应组件机器的IP就可以顺利用域名访问集群了。





**关于一次正常的连接** [用户访问域名]->[域名解析服务器解析出IP]->[Haproxy组件收到请求转发]->[Gorouter接到请求]->[DEA/CC最终处理]





# 集群使用





## 安装CF命令行工具





下载并安装[cf_cli](https://github.com/cloudfoundry/cli/releases) `dpkg -i ***.deb`。新版cf命令行的命令有比较大的变化，可以使用`cf --help`看一下。





## 制作离线java_buildpack





cloudfoundry从V168以后，就不在cf-release里面放入buildpack了，也就是云应用的运行时环境。每次应用上传都要去pivotal的网站上下buildpack，速度比较慢，所以就涉及到了制作离线的buildpack。官方的[java_buildpack](https://github.com/cloudfoundry/java-buildpack)就提供了制作离线包的功能。Clone下来以后执行如下步骤就制作成功了一个zip包。




{% codeblock %}
bundle install
bundle exec rake package OFFLINE=true
...
Creating build/java-buildpack-offline-cfd6b17.zip

{% endcodeblock %}




然后使用cf命令上传离线的buildpack，参数的意思可以从help中查看。




{% codeblock %}
root@cf-one:~/java-buildpack# cf create-buildpack test_pack ./build/java-buildpack-offline-abe37f7.zip 0
Creating buildpack test_pack...
OK
Uploading buildpack test_pack...
OK

root@cf-one:~/java-buildpack# cf buildpacks
Getting buildpacks...

buildpack          position   enabled   locked   filename   
java_buildpack     2          true      false    java-buildpack-v2.1.2.zip   
ruby_buildpack     3          true      false    buildpack_ruby_v46-245-g2fc4ad8.zip   
nodejs_buildpack   4          true      false    buildpack_nodejs_v8-177-g2b0a5cf.zip   
test_pack          1          true      false    java-buildpack-offline-abe37f7.zip   


{% endcodeblock %}




## 使用CF-CLI命令行上传应用





如果用户名密码都是默认的话，使用流程基本如下：





**target**




{% codeblock %}
root@cf-one:~/yd/cf_nise_installer# cf api --skip-ssl-validation api.test4.sel
Setting api endpoint to api.test4.sel...
OK
API endpoint: https://api.test4.sel (API version: 2.2.0)
Not logged in. Use 'cf login' to log in.

{% endcodeblock %}




**login**




{% codeblock %}
root@cf-one:~/yd/cf_nise_installer# cf login
API endpoint: https://api.test4.sel
Email> admin
Password> 
Authenticating...
OK
Targeted org DevBox
Select a space (or press enter to skip):
Space> 

API endpoint: https://api.test4.sel (API version: 2.2.0)
User:         admin
Org:          DevBox
Space:        No space targeted, use 'cf target -s SPACE'

{% endcodeblock %}




**org**




{% codeblock %}
root@cf-one:~/yd/cf_nise_installer# cf create-org sun
Creating org sun as admin...
OK
TIP: Use 'cf target -o sun' to target new org
root@cf-one:~/yd/cf_nise_installer# cf target -o sun
API endpoint: https://api.test4.sel (API version: 2.2.0)
User:         admin
Org:          sun
Space:        No space targeted, use 'cf target -s SPACE'

{% endcodeblock %}




**space**




{% codeblock %}
root@cf-one:~/yd/cf_nise_installer# cf create-space test
Creating space test in org sun as admin...
OK
Assigning role SpaceManager to user admin in org sun / space test as admin...
OK
Assigning role SpaceDeveloper to user admin in org sun / space test as admin...
OK

TIP: Use 'cf target -o sun -s test' to target new space
root@cf-one:~/yd/cf_nise_installer# cf create-space test
Creating space test in org sun as admin...
OK
Assigning role SpaceManager to user admin in org sun / space test as admin...
OK
Assigning role SpaceDeveloper to user admin in org sun / space test as admin...
OK
TIP: Use 'cf target -o sun -s test' to target new space

root@cf-one:~/yd/cf_nise_installer# cf target -o sun -s test
API endpoint: https://api.test4.sel (API version: 2.2.0)
User:         admin
Org:          sun
Space:        test

{% endcodeblock %}




**Push APP**




{% codeblock %}
root@cf-one:~# cf push test -p helloworldWeb.war -b test_pack
Creating app test in org sun / space test as admin...
OK

Creating route test.test4.sel...
OK

Binding test.test4.sel to test...
OK

Uploading test...
Uploading app files from: helloworldWeb.war
Uploading 2.6K, 10 files
OK

Starting app test in org sun / space test as admin...
OK
-----> Downloaded app package (4.0K)
-----> Java Buildpack Version: abe37f7 (offline) | https://github.com/cloudfoundry/java-buildpack.git#abe37f7
-----> Downloading Open Jdk JRE 1.7.0_60 from http://download.run.pivotal.io/openjdk/lucid/x86_64/openjdk-1.7.0_60.tar.gz (found in cache)
       Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.1s)
-----> Downloading Tomcat Instance 7.0.54 from http://download.run.pivotal.io/tomcat/tomcat-7.0.54.tar.gz (found in cache)
       Expanding Tomcat to .java-buildpack/tomcat (0.1s)
-----> Downloading Tomcat Lifecycle Support 2.1.0_RELEASE from http://download.run.pivotal.io/tomcat-lifecycle-support/tomcat-lifecycle-support-2.1.0_RELEASE.jar (found in cache)
-----> Downloading Tomcat Logging Support 2.1.0_RELEASE from http://download.run.pivotal.io/tomcat-logging-support/tomcat-logging-support-2.1.0_RELEASE.jar (found in cache)
-----> Uploading droplet (38M)

0 of 1 instances running, 1 starting
1 of 1 instances running

App started

Showing health and status for app test in org sun / space test as admin...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: test.test4.sel

     state     since                    cpu    memory         disk   
#0   running   2014-06-19 08:10:08 AM   0.0%   170.5M of 1G   96M of 1G 

root@cf-one:~# curl test.test4.sel




## 
  Hello World!







{% endcodeblock %}




至此，就成功上传了一个应用了。





## 查看应用状态




{% codeblock %}
root@cf-one:~# cf apps
Getting apps in org sun / space test as admin...
OK

name   requested state   instances   memory   disk   urls   
test   started           1/1         1G       1G     test.test4.sel  

{% endcodeblock %}




## 重启一个应用




{% codeblock %}
oot@cf-one:~# cf restart test
Stopping app test in org sun / space test as admin...
OK
Starting app test in org sun / space test as admin...
OK
0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running
App started

Showing health and status for app test in org sun / space test as admin...
OK
requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: test.test4.sel

     state     since                    cpu    memory       disk   
#0   running   2014-06-19 08:19:38 AM   0.0%   170M of 1G   96M of 1G  

{% endcodeblock %}




# 集群维护





目前集群的维护都是使用的monit工具进行简单的组件监控。更多的维护还需要查看组件的日志。组件日志一般在`/var/vcap/sys/log`中





## 查看集群组件状态




{% codeblock %}

root@cf-one:~# /var/vcap/bosh/bin/monit summary
The Monit daemon 5.2.4 uptime: 44m 

Process 'nats'                      running
Process 'nats_stream_forwarder'     running
Process 'cloud_controller_ng'       running
Process 'cloud_controller_worker_local_1' running
Process 'cloud_controller_worker_local_2' running
Process 'nginx_ccng'                running
Process 'cloud_controller_worker_1' running
Process 'cloud_controller_clock'    running
Process 'uaa'                       running
Process 'uaa_cf-registrar'          running
Process 'haproxy'                   running
Process 'gorouter'                  running
Process 'warden'                    running
Process 'dea_next'                  running
Process 'dir_server'                running
Process 'dea_logging_agent'         running
Process 'loggregator'               running
Process 'loggregator_trafficcontroller' running
Process 'etcd'                      running
Process 'hm9000_listener'           running
Process 'hm9000_fetcher'            running
Process 'hm9000_analyzer'           running
Process 'hm9000_sender'             running
Process 'hm9000_metrics_server'     running
Process 'hm9000_api_server'         running
Process 'hm9000_evacuator'          running
Process 'hm9000_shredder'           running
Process 'postgres'                  running
System 'system_cf-one'              running

{% endcodeblock %}




## 重启某个组件




{% codeblock %}
root@cf-one:~# monit restart postgres

{% endcodeblock %}




## 重启所有组件




{% codeblock %}
root@cf-one:~# monit restart all

{% endcodeblock %}


