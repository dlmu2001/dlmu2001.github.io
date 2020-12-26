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

`ps aux`以BSD的格式显示进程信息

```
# ps aux |grep test
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        15   0.0  0.1  26516  6380 ?        Ss   Dec25   0:58 /usr/bin/test
```
各列的意义如下
USER    用户名 &emsp;
PID     进程ID &emsp;
%CPU    进程占用的CPU百分比 &emsp;
%MEM    进程占用的内存百分比 &emsp;
VSZ     进程使用的虚拟内存量(KB) &emsp;
RSS     进程实际占用的内存大小(KB) &emsp;
TTY     进程在哪个终端运行，若与终端无关，显示? &emsp;
STAT    进程的状态 &emsp;
TIME    进程实际使用CPU运行的时间 &emsp;
COMMAND 进程运行命令的名称和参数 &emsp;

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
+      位于后台的进程组；

`ps -elf` 以标准格式显示进程信息
```
# ps -elf |grep mpd
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S system    2690     1  0  80   0 - 105135 SyS_ep Dec25 ?       00:00:00 /usr/bin/mpd
```
各列的意义如下
&emsp;&emsp; F       Flags (octal and additive) associated with the process
&emsp;&emsp; S       进程状态,注意这个S和ps aux中的STAT不一样
&emsp;&emsp; UID     进程的用户ID
&emsp;&emsp; PID     进程ID
&emsp;&emsp; PPID    进程的父进程ID
&emsp;&emsp; C       Process Utilization(进程占用CPU的百分比)
&emsp;&emsp; PRI     进程优先级，值越高优先级越低
&emsp;&emsp; NI      Nice值
&emsp;&emsp; ADDR    进程的地址
&emsp;&emsp; SZ      The size in blocks of the core image of the process（进程image以blocks为单位的大小)
&emsp;&emsp; WCHAN   The event for which the process is waiting or sleeping(进程等待的事件)
&emsp;&emsp; STIME   进程开始事件
&emsp;&emsp; TTY     进程在哪个终端运行，若与终端无关，显示? 
&emsp;&emsp; TIME    进程累积执行时间
&emsp;&emsp; CMD     进程运行命令的名称和参数 

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


# top
# addr2line
# strings
# 参考
## [The Fundamentals Of Sound](https://www.youtube.com/watch?v=gAQ79rHFRfE)
