+++
author = "admin"
date = "2013-10-14T15:20:42Z"
slug = "2013/10/14/rubye78988e79a84crash_recoverye887aae58aa8e58c96e6b58be8af95e5b7a5e585b7"
title = "Ruby版的crash_recovery自动化测试工具"
Categories = ["linux", "Ruby"]
+++

高可靠性（High availability）是个相当有意义课题，在现有大型集群机器上，有着广泛的应用。如今，针对这个命题的实现，各大公司都提供了方方面面的方式。针对高可靠性的测试，也就有了意义。





正好近期在学ruby，于是就尝试着用ruby做了一个crash——recovery测试工具。





工具分为两块，第一块是破坏虚拟机，使虚拟机突然宕机。第二块就是杀虚拟机中相应的进程。
最终都是要看进行了这些破坏以后，系统是否能够自动恢复。





破坏虚拟机，使其突然宕机，用到的是我之前博客中写到的方法[《Linux Magic Key — SysRq》](http://wonderflow.info/archives/714)，使用"echo c > /proc/sysrq-trigger"命令让虚拟机突然奔溃。
 <!-- more -->





既然提到了自动化，自然就是远程执行该命令了。在ruby中，远程执行命令的方式为使用net/ssh包。然后把“echo c..”写在一个shell脚本中，传输过去。然后在后台执行该脚本。





<





pre>
require ‘net/ssh’
require ‘net/scp’
def vm_dump_thread(host)
begin
Net::SSH.start(host,’root’,:password=>"password",:timeout=>5) do |ssh|
puts "success to setup. #{host}"
puts ssh.scp.upload!(‘sysdump.sh’,’.’)
ssh.exec("bash sysdump.sh &")
end
rescue Timeout::Error
puts " Connection Time out."
rescue Errno::ECONNREFUSED
puts " Connection refused"
end
end





<





pre>
本以为这样的效果就很好了。但不幸的是，ruby的线程会不断的等待





<





pre>
ssh.exec("bash sysdump.sh &")





<





pre>
返回结果。导致整个线程都卡死在这里，程序完全无法运行下去。





然后以为采取多线程的方式，可以解决问题：




{% codeblock %}

thread = []
puts list.size()
list.each do |host|
thread << Thread.new{ vm_dump_thread(host) }
end
thread.each do |s|
s.join
end

{% endcodeblock %}




但是ruby有一个诡异的地方，多线程开始跑了以后，主线程就完全被占用了。这真是一个奇怪的设定，导致我觉得ruby的多线程很难用。





不过还好之前学了一下[eventmachine](http://wonderflow.info/archives/624)的用法。所以就把多线程套在了eventmachine外面。





eventmachine是ruby中一个拥有20个线程池的事件触发型的轻量级通信模块。





我的做法就是设置心跳检查，，每隔2s检查一下所有要被杀死的虚拟机的心跳是否正常，若所有虚拟机的心跳都已经停止，就把所有开出来的线程停止，即停止eventmachine。





至于心跳怎么检查，只要连一下对方虚拟机的ssh端口看看是不是通着就可以了。




{% codeblock %}

def heartbeat_test(host)
puts host+" hearbeat…"
num = 0;
begin
Timeout::timeout(3){
TCPSocket.new(host,22)
}
rescue Timeout::Error
num = 1
puts host+" timed out. Thread killed."
end
return num
end

def dumpVM(list)
EM.run do
EM.defer do
thread = []
puts list.size()
list.each do |host|
thread << Thread.new{ vm_dump_thread(host) }
end
thread.each do |s|
s.join
end
end
EM.add_periodic_timer(2) do
num = 0
list.each do |host|
num += heartbeat_test(host)
end
puts "heartbeats "+num.to_s
if num == list.size()
puts "new loop begain"
EM.stop
end
end
end
end

{% endcodeblock %}




完成到上面这些，基本上第一块使虚拟机定时宕机的任务也就完成了。





至于第二部分，就相对简单很多了，因为所需要杀死的进程，都在“/var/vcap/sys/run/”路径下。只要把这个路径下的所有pid找到，然后使用如下命令即可。




{% codeblock %}

kill -9 [pid]

{% endcodeblock %}




值得一提的是，ruby中需要设置一下当前的工作目录就是文件所在目录，通过一行代码解决。





<





pre>
$:.unshift File.dirname(**FILE**)





<





pre>





整个工具的github地址[https://github.com/wonderflow/crash_recovery](https://github.com/wonderflow/crash_recovery)。



