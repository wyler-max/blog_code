---
title: JVM 命令行工具
date: 2020-07-11 17:03:57
tags:
- JVM
- Linux命令
categories:
- Java
---
### jstat 

jstat 监控的内容有：类装载、内存、垃圾收集、jit编译的信息

查看gc信息，间隔0.25s，共4次
```shell
jstat -gc 7201 250 4
```
如果对每列指标所代表的的意义有所困惑，可以使用  `man jstat` 查看帮助手册，然后搜索 
`-gc option` 就得到如下信息：
```
Garbage-collected heap statistics

S0C: Current survivor space 0 capacity (kB).
S1C: Current survivor space 1 capacity (kB).
S0U: Survivor space 0 utilization (kB).
S1U: Survivor space 1 utilization (kB).

EC: Current eden space capacity (kB).
EU: Eden space utilization (kB).

OC: Current old space capacity (kB).
OU: Old space utilization (kB).

MC: Metaspace capacity (kB).
MU: Metacspace utilization (kB).

CCSC: Compressed class space capacity (kB).
CCSU: Compressed class space used (kB).
        
YGC: Number of young generation garbage collection events.
YGCT: Young generation garbage collection time.
        
FGC: Number of full GC events.
FGCT: Full garbage collection time.

GCT: Total garbage collection time.
```


### jmap 

jmap查看堆内存
```shell
jmap -heap pid
```

查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象
```shell
jmap -histo:live pid 
或者
jmap -dump:live,file=dump_001.bin pid
```
上面操作会触发FULL GC，运维必备。

在排查线上问题是，先用jmap把进程内存使用情况dump到文件中，然后用jhat分析查看
1. 对进程内存dump
`jmap -dump:format=b,file=/tmp/dump.hprof pid ` 

2. dump出来的文件可以用MIT Memory Analyzer Tool、VisualVM等工具查看，这里用jhat查看
`jhat -port 9998 /tmp/dump.hprof`

3. 在浏览器中通过ip:port查看 默认端口7000

### jstack

jstack 查看Java进程内的线程堆栈信息
```shell
jstack pid
```

### jinfo

实时查看与调整虚拟机的各项参数
```shell
jinfo pid
```
