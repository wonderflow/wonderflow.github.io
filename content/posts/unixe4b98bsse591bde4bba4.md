+++
author = "admin"
date = "2013-08-06T02:31:59Z"
slug = "2013/08/06/unixe4b98bsse591bde4bba4"
title = "UNIX之ss命令"
Categories = ["IT", "linux"]
Tags = ["linux", "ss", "unix"]
+++

ss命令用来进行统计socket连接的统计。与netstat命令相似。 这个命令之前不常用，甚至没怎么听过，直到有一天不小心想要输入ssh的时候打了ss就回车，结果出来意想不到的好东西，让我决定去学习一下。



<!-- more -->



**SS**能提供的信息包括：







  1. TCP连接 


  2. UDP连接 


  3. 已经建立的 ssh/ftp/http/https连接 


  4. 本地进程连向X服务器的连接 


  5. 筛选状态(例如connected, synchronized, SYN-RECV, SYN-SENT,TIME-WAIT)、地址和端口 


  6. 所有在FIN-WAIT-1状态下的TCP连接。





大多数linux发行版都涵盖ss命令以及其他一些工具，熟悉这些命令有助于多网络状态的掌握。





下面罗列出一些常用的方法。




{% codeblock %}

$ ss -s #列出所有的socket连接（-s summary） 
$ ss -l #列出所有打开的网络端口 
$ ss -l | grep 80 #与grep命令一起用把80端口的socket连接信息查出来 
$ ss -p #列出使用了socket的进程 
$ ss -t -a #列出所有TCP连接 
$ ss -u -a #列出所有udp连接 
$ ss -o state established '(dport = :smtp or sport = :smtp )' #列出所有已建立的SMTP连接
$ ss -o state established '(dport = :http or sport = :http )' #列出所有已建立的http连接 
$ ss -x src /tmp/.X11-unix/* #列出所有本地进程连向Xserver的连接 
$ ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 192.168.1/24 #列出所有在fin-wait-1状态的TCP连接，查看他们的计时器 
#关于fin-wait-1，是发出了连接，还没有得到对方的握手信号的状态， 
#这时连接不接收应用的发送请求，只接收服务器的反馈信号， 
#直到计时器超时 

{% endcodeblock %}




关于TCP状态的筛选：




{% codeblock %}

#IPV4 
$ ss -4 state [筛选器] 
#IPV6 
$ ss -6 state [筛选器] 

{% endcodeblock %}




筛选器有以下几种







  * established 


  * syn-sent 


  * syn-recv 


  * fin-wait-1


  * fin-wait-2


  * time-wait


  * closed


  * close-wait


  * last-ack


  * listen 


  * closing


  * all : All of the above states 


  * connected : All the states except for listen and closed 


  * synchronized : All the connected states except for syn-sent 


  * bucket : Show states, which are maintained as minisockets, i.e. time-wait and syn-recv. 


  * big : Opposite to bucket state. 例如查看已经关闭的IPV4连接的状态： 




{% codeblock %}
$ ss -4 state closing
{% endcodeblock %}




根据远程IP地址和端口查看连接信息




{% codeblock %}
 
$ ss dst 192.168.0.1 
$ ss dst 192.168.0.1:http 

{% endcodeblock %}




查看本地的地址和端口的连接信息





$ ss src 192.168.0.121 
$ ss src 192.168.0.121:http 仅通过端口来查看连接 远端的参数是dport 本地的参数是sport




{% codeblock %}
 
$ ss dport = :http 
$ ss sport = :http 
$ ss sport = :80 

{% endcodeblock %}




除了等于外还有其他操作符匹配方式： 
**<= or le** : Less than or equal to port 
**>= or ge** : Greater than or equal to port 
**== or eq** : Equal to port 
**!= or ne** : Not equal to port 
**< or gt **: Less than to port 
**> or lt** : Greater than to port 
**Note**: le, gt, eq, ne etc. are use in unix shell and are accepted as well.




{% codeblock %}
 
#注意特殊符号要转义字符 
$ ss dport \> :1024 
$ ss \(sport = :http or sport = :https \) 

{% endcodeblock %}




另外，我之前因为一直使用netstat , 后来发现ss几乎涵盖了netstat的功能，常常困惑着两个有什么不同。





下面是我找到的比较好的答案：





ss command is included in iproute2 package and is the substitute of the command netstat.ss is used to dump socket statistics. It allows showing information similar to netstat. It can display more TCP and state informations than other tools. It is a new, incredibly useful and faster (as compare to netstat) tool for tracking TCP connections and sockets.





**Resources from**：[ss: Display Linux TCP / UDP Network and Socket Information](http://www.cyberciti.biz/tips/linux-investigate-sockets-network-connections.html)



