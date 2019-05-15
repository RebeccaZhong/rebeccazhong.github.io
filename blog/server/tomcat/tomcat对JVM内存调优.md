tomcat对JVM内存调优

---

# JVM内存划分
![堆内存划分](https://raw.githubusercontent.com/RebeccaZhong/markdown-photos/master/%E4%B8%93%E4%B8%9A%E6%8A%80%E8%83%BD/Java/JVM/JVM%E5%A0%86%E5%86%85%E5%AD%98%E5%88%92%E5%88%86.png)

# 堆内存分配

- 堆内存初始值分配: `-Xms`，默认是物理内存的1/64；
- 堆内存最大值分配: `-Xmx`，默认是物理内存的1/4。

默认空余堆内存小于`40%`时，JVM就会增大堆直到`-Xmx`的最大限制；空余堆内存大于`70%`时，JVM会减少堆直到`-Xms`的最小限制。因此服务器一般设置`-Xms`、`-Xmx` 相等以避免在每次GC 后调整堆的大小。

# 非堆内存分配

- 非堆内存初始值分配: `-XX:PermSize`，默认是物理内存的1/64；
- 非堆内存最大值分配: `-XX:MaxPermSize`，默认是物理内存的1/4。


**说明：**
1. 如果 `-Xmx` 不指定或者指定偏小，应用可能会导致 `java.lang.OutOfMemoryError` 错误，此错误来自JVM，不是Throwable的，无法用try...catch捕捉。

2. 如果`-XX:MaxPermSize`设置过小会导致`java.lang.OutOfMemoryError: PermGen space` 就是内存益出。

**为什么会内存益出？**
- 这一部分内存用于存放`Class`和`Meta`的信息，`Class`在被 Load的时候被放入`PermGen space`区域，它和存放`Instance`的Heap区域不同。
- GC(Garbage Collection)不会在主程序运行期对`PermGen space`进行清理，所以如果你的APP会LOAD很多CLASS 的话,就很可能出现`PermGen space`错误。
这种错误常见在web服务器对 JSP 进行`pre compile` 的时候。

# JVM参数说明

|JVM参数 | 说明 |
---|---|---
|`-server` | 一定要作为第一个参数，在多个CPU时性能佳
|`-Xms` | java Heap初始大小。 默认是物理内存的1/64。
|`-Xmx` | java heap最大值。建议均设为物理内存的一半。不可超过物理内存。
|`-XX:PermSize` | 设定内存的永久保存区初始大小，缺省值为64M。
|`-XX:MaxPermSize` | 设定内存的永久保存区最大 大小，缺省值为64M。
|`-XX:SurvivorRatio=2` | 生还者池的大小,默认是2，如果垃圾回收变成了瓶颈，您可以尝试定制生成池设置
|`-XX:NewSize` | 新生成的池的初始大小。 缺省值为2M。
|`-XX:MaxNewSize` |  新生成的池的最大大小。   缺省值为32M。如果 JVM 的堆大小大于 1GB，则应该使用值：-XX:newSize=640m -XX:MaxNewSize=640m -XX:SurvivorRatio=16，或者将堆的总大小的 50% 到 60% 分配给新生成的池。调大新对象区，减少Full GC次数。
|`-XX:+AggressiveHeap` | 会使得 Xms没有意义。这个参数让jvm忽略Xmx参数,疯狂地吃完一个G物理内存,再吃尽一个G的swap。 [参考](https://stackoverflow.com/questions/1152507/get-jvm-to-grow-memory-demand-as-needed-up-to-size-of-vm-limit)
|`-Xss` | 每个线程的Stack大小，“-Xss 15120” 这使得JBoss每增加一个线程（thread)就会立即消耗15M内存，而最佳值应该是128K,默认值好像是512k.
|`-verbose:gc` | 现实垃圾收集信息
|`-Xloggc:gc.log` | 指定垃圾收集日志文件
|`-Xmn` | young generation的heap大小，一般设置为Xmx的3、4分之一
|`-XX:+UseParNewGC` |  缩短minor收集的时间
|`-XX:+UseConcMarkSweepGC` | 缩短major收集的时间 此选项在Heap Size 比较大而且Major收集时间较长的情况下使用更合适。
|`-XX:userParNewGC` | 可用来设置并行收集【多核CPU】
|`-XX:ParallelGCThreads` | 可用来增加并行度【多核CPU】
|`-XX:UseParallelGC` | 设置后可以使用并行清除收集器【多核CPU】

具体在`tomcat`的 catalina 的文件中进行配置：
```
JAVA_OPTS="-server -Xms1024m -Xmx8192m -XX:PermSize=256M -XX:MaxPermSize=1024m -Dfile.encoding=utf-8"
```


# 讲在最后

另外需要考虑的是Java提供的垃圾回收机制。虚拟机的堆大小决定了虚拟机花费在收集垃圾上的时间和频度。

收集垃圾可以接受的速度与应用有关，应该通过分析实际的垃圾收集的时间和频率来调整。如果堆的大小很大，那么完全垃圾收集就会很慢，但是频度会降低。如果你把堆的大小和内存的需要一致，完全收集就很快，但是会更加频繁。

调整堆大小的的目的是最小化垃圾收集的时间，以在特定的时间内最大化处理客户的请求。在基准测试的时候，为保证最好的性能，要把堆的大小设大，保证垃圾收集不在整个基准测试的过程中出现。

如果系统花费很多的时间收集垃圾，请减小堆大小。一次完全的垃圾收集应该不超过 3-5 秒。如果垃圾收集成为瓶颈，那么需要指定代的大小，检查垃圾收集的详细输出，研究垃圾收集参数对性能的影响。

一般说来，你应该使用物理内存的 80% 作为堆大小。当增加处理器时，记得增加内存，因为分配可以并行进行，而垃圾收集不是并行的。

**PS:** 下面是我的IDEA对JVM调优之后的配置：
```
-Xms2048m
-Xmx3072m
-XX:NewRatio=3
-Xss16m
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:ConcGCThreads=4
-XX:ReservedCodeCacheSize=240m
-XX:+AlwaysPreTouch
-XX:+TieredCompilation
-XX:+UseCompressedOops
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Dfile.encoding=utf-8
```


参考：

- [tomcat内存配置及配置参数详解](https://www.cnblogs.com/oskyhg/p/6549877.html)
- [内存溢出之Tomcat内存配置](https://blog.csdn.net/crazy_kis/article/details/7535932)
