
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
so 我们可以利用此机制，来检测一个对象是否被GC，再关注对象的使用生命周期场景，即可知道对象是否存在内存泄漏。

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
2. 分析解析.hprof文件的机制

`LeakCannary优缺点`
LeakCannary泄漏时展示的数据更加直观

`LeakCanary`
实时监控 在Activity 或者Fragment destory 后，可检测Activity和Fragment是否被回收，以此判断它们是否存在泄露的情况
hprof文件剪裁 通过将所有基本类型数组替换为空数组（大小不变）
dump操作 因调用Debug.dumpHprofData()而带来的卡顿阻塞

缺点 hprof文件过大，导致传输失败率高。
因对性能影响较大，适用于开发环境。
优点 提供完善的内存泄露机制和详细日志，方便问题的定位。
		
`微信的Matrix`
实时监控 基于LeakCanary二次开发
增加了配置选项：根据不同的环境，配置不同的dump模式，及检测时间间隔；
增加了误报优化，支持重复Bitmap的检测。
hprof文件裁剪 剪除部分字符串和Bitmap以外实例对象中的buffer数组，裁剪收益可观。
dump操作 因调用Debug.dumpHprofData()而带来的卡顿阻塞

dump操作卡顿阻塞的问题根源：查看native代码可见，在进行Dump操作之前，会构造ScopedSuspendAll 对象，用来暂停所有的线程，然后再析构方法中恢复所有的线程。

`快手的KOOM`
解决dumpHprofData方法阻塞问题
解决方案：fork一个子进程，基于Copy-On-Write机制，在内存发送写入操作时，子进程拥有父进程的内存副本，再在子进程去执行 dumpHprofData 方法，而父进程则可以正常继续运行，在执行完dumpHprofData 方法后，直接退出关闭子进程，释放资源。

dump 



利用AndroidStudio的Lint静态扫描潜在的内存泄漏


工具使用
https://www.cnblogs.com/andy-songwei/p/11832280.html



Android开发重要知识点
https://www.cnblogs.com/andy-songwei/p/12893258.html