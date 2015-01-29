---
author: admin
comments: true
date: 2013-08-05 09:04:25+00:00
layout: post
slug: unix%e4%b9%8blsof%e5%91%bd%e4%bb%a4
title: UNIX之lsof命令
wordpress_id: 655
categories:
- linux
tags:
- linux
- unix
---


  据说lsof命令很强大，就学习了一下。[这个网站](http://www.catonmat.net/blog/unix-utilities-lsof/)讲的最好,翻译如下：














  **lsof（list open files）**是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以lsof被称为unix系统检测的瑞士军刀。


















<!-- more -->




  

## 
    那怎么用呢?
  

接下来我们一个个来看。 
  
  

## 
    罗列出所有的文件设备情况.
  


{% codeblock %}
$ lsof
{% endcodeblock %}
 不带参数的运行 
  
  `lsof` 就是列出所有被进程打开的文件。 

## 
    查看文件被谁使用.
  


{% codeblock %}
$ lsof /path/to/file
{% endcodeblock %}
 如果把文件路径作为参数，lsof命令会把用到这个文件的所有进程都罗列出来。 也可以加多个路径在后面，它会把所涉及到这些文件的进程都罗列出来: 
{% codeblock %}
$ lsof /path/to/file1 /path/to/file2
{% endcodeblock %}
 
  


  

## 
    递归查找所有打开的文件
  


{% codeblock %}
$ lsof +D /usr/lib
{% endcodeblock %}
 使用+D参数，加路径，就是递归查找目录下所有被打开的文件. 当然，显然没有用 
  
  `grep`的如下命令快： 
{% codeblock %}
$ lsof | grep '/usr/lib'
{% endcodeblock %}
 

## 
    列出某用户的所有打开的端口.
  


{% codeblock %}
$ lsof -u root
{% endcodeblock %}
 -u参数就是把所有后面指定用户打开的文件列出来. 也可以列出好几个用户打开的，用逗号分隔: 
{% codeblock %}
$ lsof -u vcap,root
{% endcodeblock %}
 另外的方法就是使用 
  
  `-u` 参数两次: 
{% codeblock %}
$ lsof -u rms -u root
{% endcodeblock %}
 

## 
    通过程序名.
  


{% codeblock %}
$ lsof -c apache
{% endcodeblock %}
 
  
  `-c` 参数就是筛选出进程名为**apache**的。 同样的: 
{% codeblock %}
$ lsof | grep foo
{% endcodeblock %}
 跟这个是一样的。 
{% codeblock %}
$ lsof -c foo
{% endcodeblock %}
 打前半部分也可以进行查找 
{% codeblock %}
$ lsof -c apa
{% endcodeblock %}
 通过多次使用-c选项可以多个进程名称的匹配 
{% codeblock %}
$ lsof -c apache -c python
{% endcodeblock %}
 

## 
    如果你同时使用-u和-c参数.
  


{% codeblock %}
$ lsof -u pkrumins -c apache
{% endcodeblock %}
 其作用是“或”，即列出符合上述条件之一的. 
  
  

## 
    加上'-a'参数，就是and，两个条件共同满足的效果.
  


{% codeblock %}
$ lsof -a -u pkrumins -c bash
{% endcodeblock %}
 
  
  

## 
    列出不包含root的所有用户.
  


{% codeblock %}
$ lsof -u ^root
{% endcodeblock %}
 注意到 
  
  `^` 这个符号. 

## 
    通过进程号查找-p(PID).
  


{% codeblock %}
$ lsof -p 1
{% endcodeblock %}
 用逗号分隔可以添加多个要查找的进程号 
{% codeblock %}
$ lsof -p 450,980,333
{% endcodeblock %}
 
  
  

## 
    使用^符号对进程号也有排除功能.
  


{% codeblock %}
$ lsof -p ^1
{% endcodeblock %}
 
  
  

## 
    列出所有的网络连接
  


{% codeblock %}
$ lsof -i
{% endcodeblock %}
 
  
  `-i` 参数列出所有（TCP和UDP）连接的进程 

## 
    列出所有TCP连接.
  


{% codeblock %}
$ lsof -i tcp
{% endcodeblock %}
 
  
  

## 
    同理，列出所有UDP连接.
  


{% codeblock %}
$ lsof -i udp
{% endcodeblock %}
 
  
  

## 
    查看谁在使用该端口.
  


{% codeblock %}
$ lsof -i :25
{% endcodeblock %}
 参数 
  
  `:25` 和 `-i` 让 lsof命令查找使用TCP 或 UDP进行连接的端口号为25的进程. 也可以用：后跟服务的名字(可以在这里查看 `/etc/services`) : 
{% codeblock %}
$ lsof -i :smtp
{% endcodeblock %}
 

## 
    查看一个特定的UDP端口的使用者.
  


{% codeblock %}
$ lsof -i udp:53
{% endcodeblock %}
 同样的，TCP端口使用者查看命令: 
{% codeblock %}
$ lsof -i tcp:80
{% endcodeblock %}
 
  
  

## 
    查看所有用户hacker的网络连接.
  


{% codeblock %}
$ lsof -a -u hacker -i
{% endcodeblock %}
 
  
  `-a` 参数起到一个and的作用`-u` 表示用户 `-i` 表示网络连接. 

## 
    列出所有 NFS (Network File System)文件.
  


{% codeblock %}
$ lsof -N
{% endcodeblock %}
 很容易记住 
  
  `-N` 就是 **N**FS. 

## 
    列出 Unix 域套接字.
  


{% codeblock %}
$ lsof -U
{% endcodeblock %}
 
  
  

## 
    列出特定进程组的进程文件.
  


{% codeblock %}
$ lsof -g 1234
{% endcodeblock %}
 
  
  

## 
    根据文件描述符指定查找.
  


{% codeblock %}
$ lsof -d 2
{% endcodeblock %}
 查找一个范围0~2 
{% codeblock %}
$ lsof -d 0-2
{% endcodeblock %}
 There are also many special values, such as 
  
  `mem`, that lists memory-mapped files: 
{% codeblock %}
$ lsof -d mem
{% endcodeblock %}
 Or `txt` for programs loaded in memory and executing: 
{% codeblock %}
$ lsof -d txt
{% endcodeblock %}
 

## 
    根据条件输出所有的进程号.
  


{% codeblock %}
$ lsof -t -i
{% endcodeblock %}
 
  
  `-t` 参数就是只输出进程号，和 `-i` 参数一起用就是把有网络连接的进程号列出来. 杀死所有有网络连接的进程可以这么做： 
{% codeblock %}
$ kill -9 `lsof -t -i`
{% endcodeblock %}
 

## 
    -r参数就是repeat，重复执行.
  


{% codeblock %}
$ lsof -r 1
{% endcodeblock %}
 
  
  `-r` 参数就是重复，后面的数字1表示每隔1s重复执行一遍。用于监控活动的连接的话，这个连接很重要: 
{% codeblock %}
$ lsof -r 1 -u john -i -a
{% endcodeblock %}





