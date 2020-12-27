---
layout: post
title: 系统调试的十八般武器(1)  --- linux命令
categories: [操作系统]
tags: [system debug linux]
description: 系统调试的一些常用的手段，本文主要总结系统调试中运用到的一些linux的命令，根据这些命令可以获取什么调试信息
---

tomorrow.cyz@gmail.com 


开发一款终端设备，比如手机、IOT、嵌入式设备等，都会涉及到大量的系统调试工作。

很多情况下，出现的问题不仅仅是一个应用或者一个系统模块的问题，而是应用与系统，应用与应用，系统中不同的模块之间，系统中用户态和内核态
之间的协作，设备驱动与内核的问题，这类问题需要从整个系统层面进行分析，所以称之为系统调试。

对于深度依赖系统能力的App，比如音视频应用，浏览器应用，超级App等，在解决App在不同系统、不同机型上出现的问题的时候，系统调试也非常重要。

一个工程师的系统调试能力，基本上能够反馈工程师的领域深度和广度，尤其是广度。

这个系统调试的十八般武器系列，是地瓜在日常系统调试的时候用到的一些手段。

除了一些RTOS，目前主流的系统是Linux系统。这些系统上一般都会embed一些linux命令，常见的比如集成busybox。用好这些常用linux命令，
我们可以获取很多程序、内核运行相关的信息。这些命令在设备上可以运行，所以我们称之为target上的命令。

除了target上的命令，有些host上的命令，比如交叉编译环境里面gcc目录下的工具(如addr2line），也可以给我们很大帮助。

对于不同的linux发行版， linux命令的格式不一定一样，本文所有的target上的命令，都以busybox的实现为基准。host上的命令，除非特别说明，
以交叉工具链(arm-linux-gcc)的命令为准。

# ps
ps是Process Status的缩写，用来显示系统中执行命令瞬间进程的状态。

在系统debug中，`ps aux`和`ps -elf`是比较常用的两个命令，通常还会结合grep做filter。

此外，`ps -Lf` 命令可以用于查看一个进程的线程

## `ps aux`以BSD的格式显示进程信息

```
# ps aux |grep test
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        15   0.0  0.1  26516  6380 ?        Ss   Dec25   0:58 /usr/bin/test
```
各列的意义如下

USER    用户名 

PID     进程ID

%CPU    进程占用的CPU百分比

%MEM    进程占用的内存百分比

VSZ     进程使用的虚拟内存量(KB)

RSS     进程实际占用的内存大小(KB)

TTY     进程在哪个终端运行，若与终端无关，显示?

STAT    进程的状态

TIME    进程实际使用CPU运行的时间

COMMAND 进程运行命令的名称和参数

其中STAT常见的状态字符有

D      无法中断的休眠状态（通常 IO 的进程）； 

R      正在运行可中在队列中可过行的； 

S      处于休眠状态； 

T      停止或被追踪； 

W      进入内存交换 （从内核2.6开始无效）； 

X      死掉的进程 （基本很少见）； 

Z      僵尸进程； 

<      优先级高的进程 

N      优先级较低的进程 

L      有些页被锁进内存； 

s      进程的领导者（在它之下有子进程）； 

l      多线程，克隆线程（使用 CLONE_THREAD, 类似 NPTL pthreads）； 

\+      位于后台的进程组；

## `ps -elf` 以标准格式显示进程信息
```
# ps -elf |grep mpd
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S system    2690     1  0  80   0 - 105135 SyS_ep Dec25 ?       00:00:00 /usr/bin/mpd
```
各列的意义如下

F       Flags (octal and additive) associated with the process

S       进程状态,注意这个S和ps aux中的STAT不一样

UID     进程的用户ID

PID     进程ID

PPID    进程的父进程ID

C       Process Utilization(进程占用CPU的百分比)

PRI     进程优先级，值越高优先级越低

NI      Nice值

ADDR    进程的地址

SZ      The size in blocks of the core image of the process（进程image以blocks为单位的大小)

WCHAN   The event for which the process is waiting or sleeping(进程等待的事件)

STIME   进程开始事件

TTY     进程在哪个终端运行，若与终端无关，显示? 

TIME    进程累积执行时间

CMD     进程运行命令的名称和参数 

## `ps -Lf $PID`可以用来查看本进程的所有线程信息
```
# ps -LF 2176
UID        PID  PPID   LWP  C NLWP    SZ   RSS PSR STIME TTY      STAT   TIME CMD
media     2176     1  2176  0   10 37069  4896   3 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2251  0   10 37069  4896   1 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2252  0   10 37069  4896   1 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2253  0   10 37069  4896   4 Dec25 ?        Ssl    2:41 /usr/bin/mdpd -f
media     2176     1  2254  0   10 37069  4896   3 Dec25 ?        Ssl    9:02 /usr/bin/mdpd -f
media     2176     1  2255  0   10 37069  4896   1 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2256  0   10 37069  4896   1 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2257  0   10 37069  4896   4 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2258  0   10 37069  4896   4 Dec25 ?        Ssl    0:00 /usr/bin/mdpd -f
media     2176     1  2259  0   10 37069  4896   2 Dec25 ?        Ssl    4:34 /usr/bin/mdpd -f
```
如上，LWP就是线程ID

## 参数定义
在busybox上，ps命令的参数可以在target上执行`ps --help all`查看。在桌面linux发行版上，则可以尝试man ps。

如下是busybox v1.20.2的输出:
```
# ps --help all

Usage:
 ps [options]

Basic options:
 -A, -e               all processes
 -a                   all with tty, except session leaders
  a                   all with tty, including other users
 -d                   all except session leaders
 -N, --deselect       negate selection
  r                   only running processes
  T                   all processes on this terminal
  x                   processes without controlling ttys

Selection by list:
 -C <command>         command name
 -G, --Group <gid>    real group id or name
 -g, --group <group>  session or effective group name
 -p, --pid <pid>      process id
     --ppid <pid>     select by parent process id
 -s, --sid <session>  session id
 -t, t, --tty <tty>   terminal
 -u, U, --user <uid>  effective user id or name
 -U, --User <uid>     real user id or name

  selection <arguments> take either:
    comma-separated list e.g. '-u root,nobody' or
    blank-separated list e.g. '-p 123 4567'

Output formats:
 -F                   extra full
 -f                   full-format, including command lines
  f, --forest         ascii art process tree
 -H                   show process hierarchy
 -j                   jobs format
  j                   BSD job control format
 -l                   long format
  l                   BSD long format
 -M, Z                add security data (for SELinux)
 -O <format>          preloaded with default columns
  O <format>          as -O, with BSD personality
 -o, o, --format <format>
                      user defined format
  s                   signal format
  u                   user-oriented format
  v                   virtual memory format
  X                   register format
 -y                   do not show flags, show rrs vs. addr (used with -l)
     --context        display security context (for SELinux)
     --headers        repeat header lines, one per page
     --no-headers     do not print header at all
     --cols, --columns, --width <num>
                      set screen width
     --rows, --lines <num>
                      set screen height
Show threads:
  H                   as if they where processes
 -L                   possibly with LWP and NLWP columns
 -m, m                after processes
 -T                   possibly with SPID column

Miscellaneous options:
 -c                   show scheduling class with -l option
  c                   show true command name
  e                   show the environment after command
  k,    --sort        specify sort order as: [+|-]key[,[+|-]key[,...]]
  L                   list format specifiers
  n                   display numeric uid and wchan
  S,    --cumulative  include some dead child process data
 -y                   do not show flags, show rss (only with -l)
 -V, V, --version     display version information and exit
 -w, w                unlimited output width

        --help <simple|list|output|threads|misc|all>
                      display help and exit
```
# pstree
pstree 显示进程树。

在linux中，并没有真正的线程，用进程来模拟线程，也就是所谓的轻量级进程(LWP)。因此pstree可以用来分析一个进程内部的线程情况。

可以使用`pstree -p $pid`来看指定pid的进程树。进程树信息里面包含线程ID和线程名，可以帮助在分析log的时候将线程id对应到代码中具体的线程上。

# top
top命令可以实时显示系统中各个进程的实时资源占用情况，包括CPU，memory等，经常用于分析性能相关问题。

busybox的top显示和linux 桌面版本的top显示有些细微的区别，桌面版本可以自行google。

```
Mem: 37824K used, 88564K free, 0K shrd, 0K buff, 23468K cached
CPU:   0% usr   0% sys   0% nic  60% idle   0% io  38% irq   0% sirq
Load average: 0.00 0.09 0.26 1/50 1081
  PID  PPID USER     STAT   VSZ %MEM CPU %CPU COMMAND
 1010     1 root     S     2464   2%   0   8% -/sbin/getty -L ttyS0 115200 vt10
 1081  1079 root     R     2572   2%   0   1% top
    5     2 root     RW<      0   0%   0   1% [events/0]
 1074   994 root     S     7176   6%   0   0% sshd: root@ttyp0
 1019     1 root     S    13760  11%   0   0% /SecuriWAN/mi
  886     1 root     S     138m 112%   0   0% /usr/bin/rstpd 51234  <== 112% MEM?!?
 1011   994 root     S     7176   6%   0   0% sshd: root@ttyp2
  994     1 root     S     4616   4%   0   0% /usr/sbin/sshd
 1067  1030 root     S     4572   4%   0   0% ssh passive
  932     1 root     S     4056   3%   0   0% /sbin/ntpd -g -c /etc/ntp.conf
 1021     1 root     S     4032   3%   0   0% /SecuriWAN/HwClockSetter
  944     1 root     S     2680   2%   0   0% dbus-daemon --config-file=/etc/db
 1030  1011 root     S     2572   2%   0   0% -sh
 1079  1074 root     S     2572   2%   0   0% -sh
    1     0 root     S     2460   2%   0   0% init
  850     1 root     S     2460   2%   0   0% syslogd -m 0 -s 2000 -b 2 -O /var
  860     1 root     S     2460   2%   0   0% klogd -c 6
  963     1 root     S     2184   2%   0   0% /usr/bin/vsftpd /etc/vsftpd.conf
    3     2 root     SW<      0   0%   0   0% [ksoftirqd/0]
  823     2 root     SWN      0   0%   0   0% [jffs2_gcd_mtd6]
```

# addr2line
# strings
# 参考
## [The Fundamentals Of Sound](https://www.youtube.com/watch?v=gAQ79rHFRfE)
