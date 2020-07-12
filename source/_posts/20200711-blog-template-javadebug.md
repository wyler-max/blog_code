---
title: 问题博客模板-Java
date: 2020-07-11 16:36:43
tags:
- template
categories:
- blog
---
### 01. 问题现象 

1. 一般情况是收到报警信息，服务出现错误和异常
2. 严重点是服务功能直接不可用

### 02. 环境说明 

1. 服务器OS、OS版本
2. 服务部署情况
3. 服务功能
4. 相关依赖

### 03. 问题排查 

1. 检查服务状态和健康检查
2. 查看日志：业务access/error日志、服务stdout/stderr日志、gc日志、系统日志/var/log/message 等
3. 查询服务器资源占用情况：top/htop、dmesg、jstat 等
4. 导出各种堆栈信息（保护现场很重要！）

```shell

1、打印系统负载快照
top -b -n 2 > ./top.txt
top -H -n 1 -p pid > ./pid_top.txt

2、cpu升序打印进程对应线程列表
ps -mp pid -o THREAD,tid,time | sort -rn > ./pid_threads.txt

3、查看tcp连接数 (最好多次采样)：
lsof -p pid > ./pid_lsof.txt

4、查看线程信息 (最好多次采样)
ps -mp pid -o THREAD,tid,time | sort -rn > ./pid_threads.txt

5、查看堆内存占用概况
jmap -heap pid > ./pid_jmap_heap.txt

6、查看堆中对象的统计信息
map -histo pid|head -n 100 > ./pid_jmap_histo.txt

7、生产对堆快照Heap dump
#堆中所有
jmap -dump:format=b,file=./pid_jmap_heapdump.hprof
#堆中存活对象（会产生full gc）
jmap -dump:live,format=b,file=./pid_live_jmap_heapdump.hprof

8、查看GC统计信息
jstat -gcutil pid > ./pid_jstat_gc.txt

```

### 04. 问题分析 

1. 分析cpu，内存占用情况，尤其是高耗能的进程、线程情况
2. 分析gc情况（Minor GC、Major GC、Full GC）
3. 分析JVM各个区的情况 Eden Space、s1/s2、Old Gen、MetaSpace
4. 分析问题进程、线程的堆栈信息
5. 分析堆快照

### 05. 原因探索和验证 

大胆假设，小心求证

### 06. 解决方案 

### 07. 参考文章

1. 资源连接地址
2. 参考博客地址，对原创者的尊重
