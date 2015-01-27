---
layout: post
title: "Oracle Java buildpack制作"
date: 2015-01-27 10:42:08 +0800
comments: true
categories: 
---

CloudFoundry提供的javabuildpack默认采用openjdk，因为授权的原因，并没有给出官方的Oracle jdk的buildpack。

但是Oracle jdk 使用范围广泛，用到的人很多，所以Cloudfoundry的文档中给出了如何制作buildpack的一系列[提示](https://github.com/cloudfoundry/java-buildpack/blob/master/docs/jre-oracle_jre.md)。

在本来CF[官方提供的java_buildpack](https://github.com/cloudfoundry/java-buildpack)的基础上，制作自己的buildpack很简单，分为四步：

1. 下载Oracle 官方的JRE包[tar格式](http://download.oracle.com/otn-pub/java/jdk/7u60-b19/jre-7u60-linux-x64.tar.gz) 然后把这个包放到一个网页能访问到的地方。可以自己架设一个tomcat，然后上传。公司内网我目前搭的地址是: http://10.10.101.181/
2. 上传index.yml到同样的位置。index.yml主要用于写版本号对应的下载地址，随便点开一个pivatol的[tomcat/index.yml](http://download.run.pivotal.io/tomcat/index.yml)看一下就明白。
3. 配置java_buildpack项目中的[config/oracle_jre.yml](https://github.com/cloudfoundry/java-buildpack/blob/master/config/oracle_jre.yml)文件。在`repository_root: ""`这一览中加上你上传的oracle jre地址。
4. 配置java_buildpack项目中的[config/components.yml](https://github.com/cloudfoundry/java-buildpack/blob/master/config/components.yml)。把其中oracle jdk那一栏原来的注释去掉，把openjdk注释起来。即告诉buildpack使用oracle jdk。

至此，我们的工作就基本完成了。

然后按照官方 java_buildpack的流程执行：

```
bundle install
bundle exec rake package OFFLINE=true
...
Creating build/java-buildpack-offline-cfd6b17.zip
```

把生成的`java-buildpack-offline-cfd6b17.zip`上传的CF集群上。

```
root@cf-one:~/java-buildpack# cf create-buildpack oracle_java_pack ./build/java-buildpack-offline-abe37f7.zip 0
Creating buildpack oracle_java_pack...
OK
Uploading buildpack oracle_java_pack...
OK

root@cf-one:~/java-buildpack# cf buildpacks
Getting buildpacks...

buildpack          position   enabled   locked   filename   
java_buildpack     2          true      false    java-buildpack-v2.1.2.zip   
ruby_buildpack     3          true      false    buildpack_ruby_v46-245-g2fc4ad8.zip   
nodejs_buildpack   4          true      false    buildpack_nodejs_v8-177-g2b0a5cf.zip   
oracle_java_pack          1          true      false    java-buildpack-offline-abe37f7.zip
```

最后就可以使用了。

```
cf push appname -p /dir/appname.war -b oracle_java_pack
```


**参考内容：**

https://github.com/cloudfoundry/java-buildpack
https://github.com/cloudfoundry/java-buildpack/blob/master/docs/jre-oracle_jre.md#configuration
https://github.com/cloudfoundry/java-buildpack/tree/master/config
https://github.com/cloudfoundry/java-buildpack/blob/master/docs/extending-repositories.md

