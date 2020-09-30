+++
author = "admin"
date = "2013-09-03T11:34:14Z"
slug = "2013/09/03/linux-magic-key-sysrq"
title = "Linux Magic Key -- SysRq"
Categories = ["linux"]
+++



  * SysRq 键是什么 ?




{% codeblock %}
`It is a ‘magical’ key combo you can hit which the kernel will respond to
regardless of whatever else it is doing, unless it is completely locked up.
即linux为内核设定的一系列关键键，让内核快速做出反应，跳过一些常规的过程。

* 如何开启神奇的SysRq键?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
你需要配置 /proc/sys/kernel/sysrq 这个文件。
默认情况下，文件的值是1，即开启所以sysrq请求的功能。所有/proc/sys/kernel/sysrq文件值对应的功能如下：



{% codeblock %}

0 – disable sysrq completely
1 – enable all functions of sysrq
>1 – bitmask of allowed sysrq functions (see below for detailed function
description):
2 – enable control of console logging level
4 – enable control of keyboard (SAK, unraw)
8 – enable debugging dumps of processes etc.
16 – enable sync command
32 – enable remount read-only
64 – enable signalling of processes (term, kill, oom-kill)
128 – allow reboot/poweroff
256 – allow nicing of all RT tasks

{% endcodeblock %}



可以使用如下命令进行设置：



{% codeblock %}

echo "number" >/proc/sys/kernel/sysrq

{% endcodeblock %}





<!-- more -->


* 如何使用?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



{% codeblock %}

echo ‘key’ > /proc/sysrq-trigger

{% endcodeblock %}



在x86机器上使用组合键：’ALT-SysRq-‘.
注意：有的键盘不显示’SysRq’,那就是’Print Screen’键。

* 有哪些Magic Key?
`
{% endcodeblock %}



{% codeblock %}

‘b’ – Will immediately reboot the system without syncing or unmounting
your disks.

‘c’ – Will perform a system crash by a NULL pointer dereference.
A crashdump will be taken if configured.

‘d’ – Shows all locks that are held.

‘e’ – Send a SIGTERM to all processes, except for init.

‘f’ – Will call oom_kill to kill a memory hog process.

‘g’ – Used by kgdb (kernel debugger)

‘h’ – Will display help (actually any other key than those listed
here will display help. but ‘h’ is easy to remember

‘i’ – Send a SIGKILL to all processes, except for init.

‘j’ – Forcibly "Just thaw it" – filesystems frozen by the FIFREEZE ioctl.

‘k’ – Secure Access Key (SAK) Kills all programs on the current virtual
console. NOTE: See important comments below in SAK section.

‘l’ – Shows a stack backtrace for all active CPUs.

‘m’ – Will dump current memory info to your console.

‘n’ – Used to make RT tasks nice-able

‘o’ – Will shut your system off (if configured and supported).

‘p’ – Will dump the current registers and flags to your console.

‘q’ – Will dump per CPU lists of all armed hrtimers (but NOT regular
timer_list timers) and detailed information about all
clockevent devices.

‘r’ – Turns off keyboard raw mode and sets it to XLATE.

‘s’ – Will attempt to sync all mounted filesystems.

‘t’ – Will dump a list of current tasks and their information to your
console.

‘u’ – Will attempt to remount all mounted filesystems read-only.

‘v’ – Forcefully restores framebuffer console
‘v’ – Causes ETM buffer dump [ARM-specific]

‘w’ – Dumps tasks that are in uninterruptable (blocked) state.

‘x’ – Used by xmon interface on ppc/powerpc platforms.
Show global PMU Registers on sparc64.

‘y’ – Show global CPU Registers [SPARC-64 specific]

‘z’ – Dump the ftrace buffer

’0′-’9′ – Sets the console log level, controlling which kernel messages
will be printed to your console. (’0′, for example would make
it so that only emergency messages like PANICs or OOPSes would
make it to your console.)

{% endcodeblock %}






  * 我们能用来干嘛呢?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
unraw(r) is very handy when your X server or a svgalib program crashes.





sak(k) (Secure Access Key) 可以用来保证你输入密码的时候，是在一个初始化的终端里面，
而不是木马环境中，因为它会把所有该终端下的进程都杀死。.
注意这个最初的作用只是去除终端下已经存在的进程而已，不能真当成SAK来用。
It seems others find it useful as (System Attention Key) which is
useful when you want to exit a program that will not let you switch consoles.
(For example, X or a svgalib program.)





reboot(b) is good when you’re unable to shut down. But you should also
sync(s) and umount(u) first.





**crash(c) 可以用来作为系统的宕机机制.**





sync(s) is great when your system is locked up, it allows you to sync your
disks and will certainly lessen the chance of data loss and fscking. Note
that the sync hasn’t taken place until you see the “OK” and “Done” appear
on the screen. (If the kernel is really in strife, you may not ever get the
OK or Done message…)





umount(u) is basically useful in the same ways as sync(s). I generally sync(s),
umount(u), then reboot(b) when my system locks. It’s saved me many a fsck.
Again, the unmount (remount read-only) hasn’t taken place until you see the
“OK” and “Done” message appear on the screen.





The loglevels ’0′-’9′ are useful when your console is being flooded with
kernel messages you do not want to see. Selecting ’0′ will prevent all but
the most urgent kernel messages from reaching your console. (They will
still be logged if syslogd/klogd are alive, though.)





term(e) and kill(i) are useful if you have some sort of runaway process you
are unable to kill any other way, especially if it’s spawning other
processes.





“just thaw it(j)” is useful if your system becomes unresponsive due to a frozen
(probably root) filesystem via the FIFREEZE ioctl.





参考自：[https://www.kernel.org/doc/Documentation/sysrq.txt](https://www.kernel.org/doc/Documentation/sysrq.txt)



