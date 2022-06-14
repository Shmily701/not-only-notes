前言
性能优化是一个永不过时的话题，对于移动端来说也是如此。即使现在的终端设备更新速度日新月异，但用户总是希望你的应用能越快越好，秒启动，流畅无违和的操作，及时的操作响应等等。
移动端性能优化是一个很泛的话题，以笔者目前的认知水平来说，大概可分以下几个方面：

内存优化
UI 渲染优化
IO 优化
弱网优化
电量优化

对于一般的应用来说，UI 渲染优化 和 内存优化 是两个必须要做的事情，因为这两个方面是日常开发比较常见的，同时对于用户来说，也是更直接的影响。比如，如果内存出现问题，常常会导致长时间内存占用过高，继而导致内存抖动，更极端的情况会出现 OOM 异常，导致应用奔溃。如果 UI 绘制渲染出现问题，常常会出现丢帧的情况，对于用户来说，就是卡顿，操作不流畅。
关于本文
关于 内存优化 这个话题，网上有很多相关的文章，会告诉你怎么去减少内存占用等等，所以这篇文章不会涉及这些知识，正像标题所说的一样，偏向于怎么去做内存监控。文章会分成上下两篇，这是第一篇，主要是调研现有的方案，包括微信开源的 Martix 中的 Matrix-Android-ResourceCanary，美团的 Probe：Android线上OOM问题定位组件，还有开源的 LeakCanary。第二篇，会尝试去开发一个内存监控组件。
技术难点
我们先把相关技术难点先提出来，再带着问题去看看上面列举的这些方案的解决思路。


hprof 文件裁剪
hprof 是堆内存快照文件，可以方便的定位内存问题，但有一点不可忽视的问题，就是文件过大。hprof 文件动则 70M，80M 多，有的应用可能会超过 100M，对于测试环境来说，这个可能不是问题，但如果在生产环境的话，对于用户设备来说，这会是个不小的负担（不管是流量或者电量来说），而且如果涉及日志回捞，那上传成功率可能会大打折扣。再者，当我们在设备上进行分析操作时，内存的占用也会处于更高的水平线，这可能会加剧应用当前的内存问题，甚至可能会导致 OOM。


实时监控
我们需要监控哪些对象，比如 Activity，Fragment 或者是其他的对象，检查频率又该是怎么样的，比如一分钟检查一次，或者是一天检查一次等等。对于测试环境来说，我们可以采用更高的监控等级，监控更多的对象，使用更高的检查频率，生产环境则需要更为保守的方案。


优化 dump 操作
Android 设备上，要 dump 当前内存快照，一般会调用 Debug.dumpHprofData() 方法，接入过 LeakCanary 的同学会知道，当执行 dump 操作时，常常会导致应用停顿几秒，在生产环境执行 dump 操作的话，如果时间过长，可能会给用户带来糟糕的体验，甚至可能会触发 ANR。


可能还有其他问题，但暂时还没考虑到，后续再补充。
调研方案
网上能搜集到方案可能很多，这里我们选取几个比较有参考价值的：

LeakCanary
微信的 Matrix
美团的 Probe

技术更新非常快，后续有其他更好的方案，再继续更新。
LeakCanary
LeakCanary 可能是 Android 同学非常熟悉的开源项目了，它有很多优点，现在已经开发了 2.0 了，使用新的 hrpof 分析工具 shark 代替原来的 haha，对于上面提出的难点，LeakCanary 主要做了 hprof 文件裁剪 和 实时监控。
hprof 文件裁剪
LeakCanary 用 shark 组件来实现裁剪 hprof 文件功能，在 shark-cli 工具中，我们可以通过添加 strip-hprof 选项来裁剪 hprof 文件，它的实现思路是：通过将所有基本类型数组替换为空数组（大小不变）。
执行以下命令：
shark-cli strip-hprofHPROF_FILE_PATH
复制代码相关源码：
when (record) {                                                  
  is BooleanArrayDump -> BooleanArrayDump(                       
      record.id, record.stackTraceSerialNumber,                  
      BooleanArray(record.array.size)                            
  )                                                              
  is CharArrayDump -> CharArrayDump(                             
      record.id, record.stackTraceSerialNumber,                  
      CharArray(record.array.size) {                              
        '?'                                                      
      }                                                          
  )                                                              
  is FloatArrayDump -> FloatArrayDump(                           
      record.id, record.stackTraceSerialNumber,                  
      FloatArray(record.array.size)                              
  )                                                              
  is DoubleArrayDump -> DoubleArrayDump(                         
      record.id, record.stackTraceSerialNumber,                  
      DoubleArray(record.array.size)                             
  )                                                              
  is ByteArrayDump -> ByteArrayDump(                             
      record.id, record.stackTraceSerialNumber,                  
      ByteArray(record.array.size)                               
  )                                                              
  is ShortArrayDump -> ShortArrayDump(                           
      record.id, record.stackTraceSerialNumber,                  
      ShortArray(record.array.size)                              
  )                                                              
  is IntArrayDump -> IntArrayDump(                               
      record.id, record.stackTraceSerialNumber,                  
      IntArray(record.array.size)                                
  )                                                              
  is LongArrayDump -> LongArrayDump(                             
      record.id, record.stackTraceSerialNumber,                  
      LongArray(record.array.size)                               
  )                                                              
  else -> {                                                      
    record                                                       
  }                                                              
}                                                                
复制代码实验下来，这个裁剪的收益很小，72M 的文件大概只裁剪了 62KB，可能这个功能不是为了裁剪文件大小，而是其他目的，有知道的同学可以告知下。
实时监控
LeakCanary 主要是为测试环境开发，它会在 Activity 或者 Fragment 的 destory 生命周期后，可以检测 Activity 和 Fragment 是否被回收，来判断它们是否存在泄露的情况。更多相关知识，可以参考笔者之前的文章：LeakCanary2 源码分析。
微信的 Matrix
微信开源的 Matrix 组件是个非常不错的项目，其中关于内存监控部分是 ResourceCanary，和 LeakCanary 一样， Matrix 主要也是在 hprof 文件裁剪 和 实时监控 这两方面做了一些优化。
hprof 文件裁剪
Matrix 的裁剪思路主要是将除了部分字符串和 Bitmap 以外实例对象中的 buffer 数组。之所以保留 Bitmap 是因为 Matirx 有个检测重复 Bitmap 的功能，会对 Bitmap 的 buffer 数组做一次 MD5 操作来判断是否重复。
@Override                                                                                                              
public void visitHeapDumpPrimitiveArray(int tag, ID id, int stackId, int numElements, int typeId, byte[] elements) {   
    final ID deduplicatedID = mBmpBufferIdToDeduplicatedIdMap.get(id);                                                 
    // Discard non-bitmap or duplicated bitmap buffer but keep reference key.                                          
    if (deduplicatedID == null || !id.equals(deduplicatedID)) {                                                        
        if (!mStringValueIds.contains(id)) {                                                                           
            skipData += elements.length;                                                                               
            return;                                                                                                    
        }                                                                                                              
    }                                                                                                                  
    super.visitHeapDumpPrimitiveArray(tag, id, stackId, numElements, typeId, elements);                                
}                                                                                                                      
复制代码Martix 的裁剪收益还是比较可观的，原文件 92M 裁剪后只有 18M。
实时监控
Matrix 是基于 LeakCanary 上进行二次开发，所以监控原理基本是一致的，主要增加了一些误报的优化，比如：


多次检测到相同的可疑对象，才认定为泄露对象，参数可配置。
if (destroyedActivityInfo.mDetectedCount < mMaxRedetectTimes                                
        || !mResourcePlugin.getConfig().getDetectDebugger()) {                              
    // Although the sentinel tell us the activity should have been recycled,                
    // system may still ignore it, so try again until we reach max retry times.             
    MatrixLog.i(TAG, "activity with key [%s] should be recycled but actually still \n"      
                    + "exists in %s times, wait for next detection to confirm.",            
            destroyedActivityInfo.mKey, destroyedActivityInfo.mDetectedCount);              
    continue;                                                                               
}                                                                                           
复制代码

增加一个哨兵对象，用于判断是否有 GC 操作，因为调用 Runtime.getRuntime().gc() 只是建议虚拟机进行 GC 操作，并不一定会进行。
if (sentinelRef.get() != null) {                                                       
    // System ignored our gc request, we will retry later.                             
    MatrixLog.d(TAG, "system ignore our gc request, wait for next detection.");        
    return Status.RETRY;                                                               
}                                                                                      
复制代码

避免重复检测相同的对象
if (sentinelRef.get() != null) {                                                       
    // System ignored our gc request, we will retry later.                             
    MatrixLog.d(TAG, "system ignore our gc request, wait for next detection.");        
    return Status.RETRY;                                                               
}                                                                                      
复制代码

Matrix 虽然是基于 LeakCnary，但额外增加了一些配置选项，可以用于生产环境，比如 dump 模式，支持手动触发 dump，自动 dump，和不进行 dump，可以根据不同的环境，使用不同的模式。
public enum DumpMode {                            
    NO_DUMP, AUTO_DUMP, MANUAL_DUMP, SILENCE_DUMP 
}                                                 
复制代码除此之前，还有检测时间间隔等等。
还有一点就是，Matrix 是将分析 hprof 操作独立出来的，可以使用 resource-canary-analyzer 去执行分析操作。
美团的 Probe
Probe 不是开源的，所以只能通过相关的文章 Probe：Android线上OOM问题定位组件 和一些其他手段去了解。
Probe 同样是在 hprof 文件裁剪 和 实时监控 这两方面做了优化。

注意事项⚠️：对 Probe 组件的研究纯粹只是出于学习目的，请不要用做其他用途。

hprof 文件裁剪
与 Matrix 不同的是，Probe 支持在设备上进行分析 hprof 操作，所以它不仅考虑了 hprof 文件上传成功率，还考虑加载 hprof 带来的内存占用高问题。不管是 LeakCanary 还是 Matrix，在裁剪的处理上，都是先将原始的 hprof 文件 dump 出来以后，过滤掉某些数据后，再重新写入到新的 hprof 文件。而 Probe 的处理则更加有创意，它通过 native hook 了虚拟机写入 hprof 文件的操作，在这里先过滤了某些数据，最终生成的 hprof 文件就是裁剪后的了。在不考虑可能会有的兼容性问题，这个方案要比其他的方案要高效，因为它解决了 hprof 加载的内存问题。
美团在测试中发现，分析 hprof 文件的占用内存和 hprof 中记录对象实例数量是成正比的，

测试时遇到的最大问题就是分析进程自身经常会发生OOM，导致分析失败。为了弄清楚分析进程为什么会占用这么大内存，我们做了两个对比实验：

在一个最大可用内存256MB的手机上，让一个成员变量申请特别大的一块内存200多MB，人造OOM，Dump内存，分析，内存快照文件达到250多MB，分析进程占用内存并不大，为70MB左右。
在一个最大可用内存256MB的手机上，添加200万个小对象（72字节），人造OOM，Dump内存，分析，内存快照文件达到250多MB，分析进程占用内存增长很快，在解析时就发生OOM了。

实验说明，分析进程占用内存与HPROF文件中的Instance数量是正相关的，在将HPROF文件映射到内存中解析时，如果Instance的数量太大，就会导致OOM。

计数压缩逻辑：如果存在重复的相同实例，则增加它的计数，不保存它的实例。Probe 定义超过实例超过 8000，则不会再继续保持它的实例对象。
hprof 文件分析
除了文件裁剪，Probe 还优化分析泄露对象的链路，因为 Probe 相对于其他方案，理论上它是支持所有对象的内存泄露检测的（排除了原始类型等），而 LeakCanary 和 Matrix 只支持 Activity 和 Fragment 等对象，这个优化是基于 RetainSize 越大的对象对内存的影响也越大，是最有可能造成OOM的“元凶” 这一原则。
首先，在 dump 出所有对象后，创建一个 TOP N 的小根堆，根据 RetainSize 排序，初始 N 默认是 5，这里有个小细节，就是 Probe 会处理 ByteArray 类型的实例：
if (isByteArray(var6)) {
    var6.parent.addRetainedSize(var1.getHeapIndex(var4), var6.getTotalRetainedSize());
    var15.add(var6.parent);
}
复制代码将 ByteArray 的 RetainSize 加到它的父节点，同时将它的父节点加到堆中。
在初始化小根堆后，继续遍历做动态调整，将大于文件大小 5% 的对象直接加到堆中，同时增大 TOP N 的值，否则，跟堆顶元素做比较。在遍历对象的同时，如果是相同类的不同实例，则只会保存一份实例，同时将它们的 RetainSize 和 Num 计算进去：
if (var3.containsKey(var9)) {                            
    InstanceExtra var17 = (InstanceExtra)var3.get(var9); 
    var17.retainSize += var6.getTotalRetainedSize();     
    ++var17.num;
}
复制代码还会将在 计数压缩逻辑 中 RetainSize 补回来：
if (var10 != null) {                                                                                  
    var16.retainSize += var10.getSize();                                                              
    long var11 = var16.num;                                                                           
    var16.num = (long)var10.getCount() + var11;                                                       
    ClassCountInfo var18 = (ClassCountInfo)AbandonedInstanceManager.getInstance().countMap.get(var9); 
    if (var18 != null && var18.classId > 0L) {                                                        
        var16.id = var18.classId;                                                                     
    }                                                                                                 
}                                                                                                     
复制代码实时监控
Probe 对监控 Activity 对象的处理方法和 Matrix 的 DumpMode 类似，都是分级处理，Probe 线上监控到 Activity 泄露后，不会进行 dump 操作，只是对 Activity 类名等进行上报处理（这些都是可以配置的）。
除了 Activity 以为，Probe 还会通过开启一个内存监控器，每隔 1s 查看当前应用可用内存，当剩余内存低于 10% （触底）时，会进行内存分析。
当发生 OOM 时，也会进行内存分析。
之前的配置都是通过接口下发的，这也能保证最大程度的灵活性。
小结
从上面三种方案中，我们可以得出一些结论：LeakCanary 虽然只会在测试环境使用，且只提供 Activity 和 Fragment 等对象的监控，但它提供监控的思路和 hprof 的分析，Matrix 和 Probe 或多或少都是基于它进行二次开发和优化。Matrix 主要是优化泄露对象的误判和 hprof 文件的裁剪，Probe 则是优化了在设备上进行 hprof 文件分析的内存占用，同时支持检测内存中所有对象（RetainSize 大的对象）。
可惜的是，上面几种方案都没有涉及如何优化 heap dump 操作，这个留着后续研究。
关于 OOM
Out of memory (OOM) 当系统无法满足申请的内存大小时，就会抛出 OOM 错误。导致 OOM 的原因，除了我们上面说讲的内存泄露以外，可能还会是线程创建超过限制，可以通过 /proc/sys/kernel/threads-max 获取：
cat /proc/sys/kernel/threads-max
26418
复制代码还有一种可能是 FD（File descriptor）文件描述符超过限制，可以通过 /proc/pid/limits 获取，其中 Max open files 就是可创建的文件描述符数量：
Limit                     Soft Limit           Hard Limit           Units
Max open files            4096                 4096                 files

