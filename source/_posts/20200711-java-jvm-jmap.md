---
title: 20200711-java-jvm-jmap
date: 2020-07-12 17:39:10
tags:
- JVM
- jmap
categories:
- Java
---
## 01 前言

在一次排查线上问题时，偶然发现 jmap -histo:live 会触发FULL GC，于是为了深入了解，对jmap的原理做了一次探索。


## 02 正文

### 测试环境
Linux CentOS7、jdk8、64位

### 源码排查

我们查看JMap源码

包名：sun.tools.jmap
类名：JMap.java

如图我们可以看到有两组参数
![jmap-图-1](http://wylbucket20190621.oss-cn-beijing.aliyuncs.com/photo/blog/20200712-jvm-jmap-01.jpg)



在JMap中搜索 LIVE_HISTO_OPTION 即 ”-histo:live“ 可以发现，它和 ”-histo“  区别是 调用 histo(p1, p2) 是 p2的值不同

![jmap-图-2](http://wylbucket20190621.oss-cn-beijing.aliyuncs.com/photo/blog/20200712-jvm-jmap-02.jpg)

跟进查看 histo 方法，发现在调用heapHisto 方法是参数有变化

```java
private static void histo(String var0, boolean var1) throws IOException {
    VirtualMachine var2 = attach(var0);
    InputStream var3 = ((HotSpotVirtualMachine)var2).heapHisto(new Object[] {var1 ? "-live" : "-all"});
    drain(var2, var3);
}
```

跟进查看 heapHisto 发现这个方法在是 sun.tools.attach.HotSpotVirtualMachine.java 类中。HotSpot就是当前归属于Oracle 公司，世界上使用最广泛的虚拟机。 HotSpotVirtualMachine 类继承的是 VirtualMachine  虚拟机类。

在这里我们能看到，像dumpHeap、heapHisto 都被封装 executeCommand 函数调用。

```java
public InputStream dumpHeap(Object... var1) throws IOException {
    return this.executeCommand("dumpheap", var1);
}

public InputStream heapHisto(Object... var1) throws IOException {
    return this.executeCommand("inspectheap", var1);
}

abstract InputStream execute(String var1, Object... var2) throws AgentLoadException, IOException;

private InputStream executeCommand(String var1, Object... var2) throws IOException {
    try {
        return this.execute(var1, var2);
    } catch (AgentLoadException var4) {
        throw new InternalError("Should not get here", var4);
    }
}
```

executeCommand 来自抽象的 execute方法，它的实现类在 sun.tools.attach.BsdVirtualMachine.java 中，BsdVirtualMachine是HotSpotVirtualMachine 的子类，在其虚拟机功能的基础上实现了一些socket读、写等操作，可以实现与虚拟机的功能调用。



到此，SDK部分就已经结束了，接下来我们看一下 HotSpot 的源码。

源码下载方式，[参考OSChina文章](https://my.oschina.net/u/2518341/blog/1931088)

在 HotSpot 源码中
jdk/src/hotspot/share/services/attachListener.cpp

与JDK中的方法映射
![jmap-图-5](http://wylbucket20190621.oss-cn-beijing.aliyuncs.com/photo/blog/20200712-jvm-jmap-05.jpg)

dump_heap 方法的实现
![jmap-图-3](http://wylbucket20190621.oss-cn-beijing.aliyuncs.com/photo/blog/20200712-jvm-jmap-03.jpg)

heap_inspection 方法的实现
![jmap-图-4](http://wylbucket20190621.oss-cn-beijing.aliyuncs.com/photo/blog/20200712-jvm-jmap-04.jpg)



通过上面我们可以看到，JDK 中的 dumpheap 和 inspectheap 分别对应着 dump_heap 和 heap_inspection 的方法。而在其方法中，都会执行FULL GC。

## 03 总结

jmap 在带 live 过滤参数时会执行FULL GC
```shell
jmap -histo:live pid 
jmap -dump:live,file=dump_001.bin pid
```

个人分析，这大概是因为需要过滤活跃的对象，而活跃的对象是时刻都在发生变化的，为了快速分离，jvm最简单的方法就是通过一次FULL GC清理掉不活跃的对象，然后导出。


