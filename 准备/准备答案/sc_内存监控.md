
关于LeakCanary的原理
https://youuupeng.github.io/2020/05/15/LeakCanary%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90/

https://zhuanlan.zhihu.com/p/73675401

https://www.cnblogs.com/huansky/p/15489365.html


内存监控
https://juejin.cn/post/6844904090179207176


LeakCanary是基于一个叫做ObjectWatcher Android库。

`初始化`
LeakCanary2.0 通过ContentProvider的随进程启动实例化的特性，实现无感知初始化，初始化即完成了ObjectWatcher对Activity及Fragment的注册监听。

- 注册在Manifest文件中的ContentProvider，会在应用启动时，由ActivityThread创建并初始化。借助ContentProvider特性，LeakCanary完成了初始化。

`JVM引用队列机制`
除强引用外的，其他三个引用，可以与引用队列联合使用，如果软/弱/虚引用的对象被GC回收，JVM会将这个软/弱/虚引用加入与之关联的引用队列中。
so 我们可以利用此机制，来检测一个对象是否被GC，再关注对象所在模块的生命周期场景，即可知道对象是否存在内存泄漏。

`检测内存泄漏的核心原理`
RefWatcher.watch()创建了一个KeyedWeakReference用于去观察对象。
在Activity执行完onDestory()之后，将Activity放入WeakReference中，然后将
将弱引用与引用队列相关联，如果弱引用持有的对象被GC回收了，那么JVM就会把这个弱引用加入到引用队列referenceQueue中。即如果KeyedWeakReference持有的Activity对象被GC回收，就会被加入到引用队列中。从referenceQueque中查看是否有没有该对象，如果没有，执行gc，再次查看，还是没有的话则判断发生内存泄露了。


LeakCanary工作步骤

1. ObjectWatcher监听Activity/Fragment生命周期

2. 检测内存泄漏：在onDestory时，创建弱引用指向Activity对象，并关联引用队列，启动后台进程去检测内存泄漏。time later，从引用队列中读取activity的引用，如读不到，可能是发生了泄漏，触发gc，time later，再读取，如果还读不到，即发生了内存泄漏。

4. 到达阈值后Dump
Dump java堆到一个.hprof文件<堆转储存文件>

3. 分析解析
LeakCanary使用Shark解析分析.hprof文件，查找出阻止泄漏对象被垃圾回收的引用链：leaktrace，也称为`GCRoot到泄漏对象的最短强引用路径`
 
这里有两处核心点
1. 检测内存泄漏的机制
2. 分析解析.hprof文件的机制 怎么得到GCRoot的引用路径？

`LeakCanary特点`
实时监控 在Activity 或者Fragment destory 后，可检测Activity和Fragment是否被回收，以此判断它们是否存在泄露的情况
hprof文件剪裁 通过将所有基本类型数组替换为空数组（大小不变）
dump操作 因调用Debug.dumpHprofData()而带来的卡顿阻塞

缺点 hprof文件过大，导致传输失败率高。
因对性能影响较大，适用于开发环境。
优点 LeakCannary泄漏时展示的数据更加直观，方便问题的定位。
		
`微信的Matrix`
- 监控原理 基于LeakCanary二次开发，监控原理一致
- 相比LeakCanary 
    1. 增加了配置选项：根据不同的环境，配置不同的dump模式，及检测时间间隔；
    2. 增加了误报优化
    > 1. 支持Bitmap重复检测。[在剪裁hprof文件时，保留Bitmap，对 Bitmap的buffer数组做一次MD5操作来判断是否重复。]
    > 2. 多次检测相同的可疑对下，才认为泄露对象。

- hprof文件裁剪 剪除`部分字符串`和`Bitmap以外实例对象`中的`buffer数组`，裁剪收益可观。
- dump操作 因调用Debug.dumpHprofData()而带来的卡顿阻塞

dump操作卡顿阻塞的问题根源：查看native代码可见，在进行Dump操作之前，会构造ScopedSuspendAll 对象，用来暂停所有的线程，然后再析构方法中恢复所有的线程。

`美团的Probe`
非开源

`快手的KOOM`
解决dumpHprofData方法阻塞问题
解决方案：fork一个子进程，基于Copy-On-Write机制，在内存发送写入操作时，子进程拥有父进程的内存副本，再在子进程去执行 dumpHprofData 方法，而父进程则可以正常继续运行，在执行完dumpHprofData 方法后，直接退出关闭子进程，释放资源。

dump 



利用AndroidStudio的Lint静态扫描潜在的内存泄漏


工具使用
https://www.cnblogs.com/andy-songwei/p/11832280.html



Android开发重要知识点
https://www.cnblogs.com/andy-songwei/p/12893258.html


堆内存不足
Android中最常见的OOM就是Java堆内存不足，对于堆内存不足导致的OOM问题，发生Crash时的堆栈信息往往只是“压死骆驼的最后一根稻草”，它并不能有效帮助我们准确地定位到问题。

堆内存分配失败，通常说明进程中大部分的内存已经被占用了，且不能被垃圾回收器回收，一般来说此时内存占用都存在一些问题，例如内存泄漏等。要想定位到问题所在，就需要知道进程中的内存都被哪些对象占用，以及这些对象的引用链路。而这些信息都可以在Java内存快照文件中得到，调用Debug.dumpHprofData(String fileName)函数就可以得到当前进程的Java内存快照文件（即HPROF文件）。所以，关键在于要获得进程的内存快照，由于dump函数比较耗时，在发生OOM之后再去执行dump操作，很可能无法得到完整的内存快照文件。

于是Probe对于线上场景做了内存监控，在一个后台线程中每隔1S去获取当前进程的内存占用（通过Runtime.getRuntime.totalMemory()-Runtime.getRuntime.freeMemory()计算得到），当内存占用达到设定的阈值时（阈值根据当前系统分配给应用的最大内存计算），就去执行dump函数，得到内存快照文件。[可能此时并没有内存泄漏，只是内存占用较高]

在得到内存快照文件之后，我们有两种思路，一种想法是直接将HPROF文件回传到服务器，我们拿到文件后就可以使用分析工具进行分析。另一种想法是在用户手机上直接分析HPROF文件，将分析完得到的分析结果回传给服务器。但这两种方案都存在着一些问题，下面分别介绍我们在这两种思路的实践过程中遇到的挑战和对应的解决方案。

