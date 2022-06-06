简述jvm运行时数据区

方法区 虚拟机栈 本地方法栈（native method stack）
堆            程序计数器 

方法区 常量 静态变量 运行时常量池

直接内存 引入的一种基于通道的缓冲区的IO方式，使用Native函数库直接分配对外内存。

所有的对象实例以及数组都要在堆上分配，这就是为什么要从这里入手

内存泄漏的核心：
长生命周期持有短生命周期对象的引用，使短周期对象不得已释放。

工具 IDE profile
AndroidStudio自带对堆转储（Heap Dump）文件进行分析，并且会把内存泄漏点明确标出来。GC按钮
点击GC按钮，再点击捕获堆转储文件按钮。AndroidStudio会自动跳转到以下界面： 

内存抖动就是内存在短暂时间频繁的GC和分配内存，导致内存不稳定。


private fun initHandler() {
        handler = Handler(Looper.getMainLooper(), Handler.Callback {
            for (index in 0..1000) {
                val asc = IntArray(100000) { 0 }
                Log.d("YI", "$asc")
            }
            handler?.sendEmptyMessageDelayed(0, 300)
            true
        })

        handler?.sendEmptyMessageDelayed(0, 300)
    }

这里有一个内存抖动的例子，由于在’方法内部‘创建对象，局部变量，在虚拟机栈中（类型的引用），所以在方法执行结束时，对象被释放了，
而所有对象实例是在java堆中分配的，所以这里出现大量的对象创建与销毁。


造成内存泄漏的原因：
1. 单例造成的内存泄漏  避免方式：单例持有上下文不要使用Activity 而需要使用application 
2. 非静态内部类持有外部类的引用 （内部类的构造函数中会传入外部类的实例）// 解决方案: 静态内部类
3. Handler引发的内存泄漏，在Message中持有mhandler的应用，msg的target   而mhandler是Activity的非静态内部类的实例，持有Activity的引用，这里间接导致了内存泄漏，同2  // 解决方案：静态内部类+弱引用
4. 异步任务导致的内存泄露  
5. 资源未关闭或者未释放造成的内存泄漏 io File cursor 
6. 注册，但未反注册
7. 集合类的泄露 // 静态全局final
8. webView内存泄漏 // 
让onDetachedFromWindow先走，在主动调用destroy()之前，把webview从它的parent上面移除掉。
9. HashMap 引发的内存泄漏，当想要使用对象作为HashMap的key时，可以考虑使用不可变对象作为HashMap的key，如常用的String类型，或者确保使用不可变的成员属性来生成hashcode 因为put时，只有key相同，才会覆盖

10. Bitmap 统一图片库 Bitmap.createBitmap与BitmapFactory相关接口 收拢 采样率 压缩 图片格式
更改Bitmap.Config格式。  通过BitmapFactory.Options来缩放图片，主要是用到了它的inSampleSize参数，即采样率
统一监控  图片超出屏幕 Canvas.clip 剪裁 等


ViewParent parent = mWebView.getParent();
if (parent != null) {
    ((ViewGroup) parent).removeView(mWebView);
}
mWebView.stopLoading();
// 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
mWebView.getSettings().setJavaScriptEnabled(false);
mWebView.clearHistory();
mWebView.clearView();
mWebView.removeAllViews();
mWebView.destroy();


强引用就是对象被强引用后，无论如何都不会被回收。
弱引用就是在垃圾回收时，如果这个对象只被弱引用关联（没有任何强引用关联他），那么这个对象就会被回收。
软引用就是在系统将发生内存溢出的时候，回进行回收。


debug 开发者模式，打开一个悬浮窗，显示内存 cpu的情况
内存监控API runtime.memory 相关 
GC 监控API debug.global


Today
#### 内存优化方案 从以下两个角度
1. 内存创建与释放的时机


2. 减少不必要对象创建方向


