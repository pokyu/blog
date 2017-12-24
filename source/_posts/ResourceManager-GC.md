---
title: ResourceManager GC
date: 2017-08-31 22:24:50
type: "categories"
tags: "YARN"
categories: "Hadoop"
---



### 现象
在系统运行高峰期，YARN的RM无法登录或登录界面现实特别慢。应用执行也特别慢。

### 分析与解决方案
根据经验，系统RM无法登录，那么有可能是RM进程有问题，所以查看RM进行日志。
查看RM的GC日志resourcemanager-omm-20170214200940-pid13297-gc.log.8，发现大量的FULL GC。
```
2017-02-16T09:53:27.389+0800: 135826.463: [GC (Allocation Failure) 2017-02-16T09:53:27.389+0800: 135826.463: [ParNew: 114712K->10438K(118016K), 0.0059636 secs] 1844453K->1740908K(2084096K), 0.0062153 secs] [Times: user=0.09 sys=0.01, real=0.00 secs]
```
出现FULL GC说明RM内存已用光，无可分配资源，故无法RM出现hang住状态。
解决方案：
  - 调整RM的最大可用内存限制
  - 重启YARN

### 详解知识点
在YARN的GC日志中GC分为三种：普通GC，CMS GC，FULL GC
#### Minor GC：
```
2017-02-16T09:53:26.409+0800: 135825.482: [GC (Allocation Failure) 2017-02-16T09:53:26.409+0800: 135825.482: [ParNew: 114914K->9992K(118016K), 0.0119158 secs] 1843767K->1739516K(2084096K), 0.0122177 secs] [Times: user=0.12 sys=0.00, real=0.01 secs] 16T09:53:26.409+0800:
2017-02-16T09:53:26.425+0800: 135825.498: [GC (CMS Initial Mark) [1 CMS-initial-mark: 1729524K(1966080K)] 1739924K(2084096K), 0.0034278 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
```
在GC日志中此正常情况下会进出现此日志。其中1844453K->1740908K(2084096K) 内存总:2084096K 已用内存从1844453K释放到1740908K
2017-02-16T09:53:26.409+0800：： GC发生的时间；
135825.482：：GC开始，相对JVM启动的相对时间，单位是秒；
GC：：区别Minor GC、CMS GC、Full GC的标识，这次代表的是Minor GC；
Allocation Failure：： MinorGC的原因，在这个case里边，由于年轻代不满足申请的空间，因此触发了MinorGC;
ParNew ：：收集器的名称，它预示了年轻代使用一个并行的 mark-copy stop-the-world 垃圾收集器；
114914K->9992K：：收集前后年轻代的使用情况；
(118016K)：：整个年轻代的容量；
0.0119158 secs：：回收操作所用时间；
1843767K->1739516K：：收集前后整个堆的使用情况；
(2084096K)：：整个堆的容量；
0.0122177 secs：：ParNew收集器标记和复制年轻代活着的对象所花费的时间（包括和老年代通信的开销、对象晋升到老年代时间、垃圾收集周期结束一些最后的清理对象等的花销）；
[Times: user=0.12 sys=0.00, real=0.01 secs]：：GC事件在不同维度的耗时，具体的用英文解释起来更加合理:
  - user ： Total CPU time that was consumed by Garbage Collector threads during this collection
  - sys ： Time spent in OS calls or waiting for system event
  - real ： Clock time for which your application was stopped. With Parallel GC this number should be close to (user time + system time) divided by the number of threads used by the Garbage Collector. In this particular case 8 threads were used. Note that due to some activities not being parallelizable, it always exceeds the ratio by a certain amount.

#### CMS GC：
```
2017-02-16T09:53:25.805+0800: 135824.878: [CMS-concurrent-sweep-start]
2017-02-16T09:53:26.161+0800: 135825.234: [CMS-concurrent-sweep: 0.356/0.356 secs] [Times: user=0.46 sys=0.01, real=0.36 secs] 
2017-02-16T09:53:26.161+0800: 135825.234: [CMS-concurrent-reset-start]
2017-02-16T09:53:26.169+0800: 135825.243: [CMS-concurrent-reset: 0.009/0.009 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
2017-02-16T09:53:26.934+0800: 135826.007: [CMS-concurrent-mark: 0.481/0.505 secs] [Times: user=4.27 sys=0.44, real=0.50 secs] 
2017-02-16T09:53:26.934+0800: 135826.008: [CMS-concurrent-preclean-start]
2017-02-16T09:53:26.964+0800: 135826.038: [CMS-concurrent-preclean: 0.028/0.030 secs] [Times: user=0.04 sys=0.00, real=0.03 secs] 
2017-02-16T09:53:26.964+0800: 135826.038: [CMS-concurrent-abortable-preclean-start]
```
当系统繁忙时，会出现CMS GC，此时说明系统已经非常繁忙了，内存不足了。CMS的目标是尽量减少应用的暂停时间，减少full gc发生的几率。

#### FULL GC：
```
2017-02-15T10:37:23.957+0800: 53139.489: [Full GC (Allocation Failure) 2017-02-15T10:37:23.957+0800: 53139.490: [CMS: 1966079K->1966079K(1966080K), 4.4891712 secs] 2084073K->1970966K(2084096K), [Metaspace: 67007K->67007K(1110016K)], 4.4894022 secs] [Times: user=4.49 sys=0.00, real=4.49 secs]
```
如果出现FULL GC，那么说明系统已经出现问题。4.4891712 secs表示整个JVM都停顿了4.48秒。

#### jstat查看GC
```
jstat -gcutil <pid> <时间间隔（ms）>
例如：jstat -gcutil 15743 3000
```
运行结果如下：
```
[omm@HDM03 rm]$ jstat -gcutil 15743 3000
       S0     S1         E     O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
100.00   0.00  66.79  63.13  98.55  96.57  27560  330.003   78       2.388  332.391
 73.12   0.00  15.32  63.35  98.55  96.57  27562  330.028    78       2.388  332.416
  0.00  54.64  29.29  63.35  98.55  96.57  27563  330.036    78       2.388  332.424
 55.43   0.00  57.75  63.35  98.55  96.57  27564  330.044    78       2.388  332.432
```
S0    ：： Heap上的 Survivor space 0 区已使用空间的百分比（年轻代中第一个survivor（幸存区）已使用的占当前容量百分比）
S1    ：： Heap上的 Survivor space 1 区已使用空间的百分比 （年轻代中第二个survivor（幸存区）已使用的占当前容量百分比）
E      ：： Heap上的 Eden space 区已使用空间的百分比 （年轻代中Eden（伊甸园）已使用的占当前容量百分比）
O      ：： Heap上的 Old space 区已使用空间的百分比 （old代已使用的占当前容量百分比）
P       ：： Perm space 区已使用空间的百分比（perm代已使用的占当前容量百分比）
YGC  ：： 从应用程序启动到采样时发生 Young GC 的次数 
YGCT：：从应用程序启动到采样时 Young GC 所用的时间(单位秒) 
FGC  ：： 从应用程序启动到采样时发生 Full GC 的次数 （从应用程序启动到采样时old代(全gc)gc次数）
FGCT：：从应用程序启动到采样时 Full GC 所用的时间(单位秒) （从应用程序启动到采样时old代(全gc)gc所用时间(s)）
GCT  ：： 从应用程序启动到采样时用于垃圾回收的总时间(单位秒) 
通过FGC我们可以发现系统是否发生FULL GC和FULL GC的频率 

```
关于GC的详细介绍请参考《深入理解 Java 垃圾回收机制》
```
