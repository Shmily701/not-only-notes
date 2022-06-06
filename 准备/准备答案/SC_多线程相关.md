题目：
多线程，线程池，
asyncTask使用场景与优缺点   
handlerThread原理
数据同步，锁，锁的性能与特点 使用场景与优缺点  

AsyncTask: 轻量级异步任务机制，适用于需要立即启动，但是异步执行的生命周期短暂的使用场景。原理：通过Handler机制

    @MainThread
    protected void onPreExecute() {
    }
    
    @MainThread
    protected void onPostExecute(Result result) {
    }

    @MainThread
    protected void onProgressUpdate(Progress... values) {
    }

    @WorkerThread
    protected abstract Result doInBackground(Params... params);

缺点：多个异步操作都需要进行Ui变更时,就变得复杂.
HandlerThread: 为某些回调方法或者等待某些任务的执行设置一个专属的线程，并提供线程任务的调度机制。   继承Thread类 & 封装Handler类
借助Thread  借助Handler 


浏览器中使用的线程池

/**
 * ThreadPoolExecutor(  int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit,
 * BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler
 * handler);
 *
 *
 * ThreadPoolExecutor参数 
 1.当池子中线程数  <  corePoolSize，就创建新线程，任务直接提交给线程执行。 
 2.当池子中线程数  >= corePoolSize,会把请求放到workQueue中，而不是新添加线程。 
 3.如果继续添加任务，已经无法向workQueue中添加任务了，那么就会创建新的线程。
 4.继续第三步，如果创建的线程超过了maximumPoolSize，那么任务就直接被拒绝，执行RejectedExecutionHandler的逻辑.
 *
 *
 * 队列---BlockingQueue： |--SynChronousQueue---      本身是无界的，能存无限的任务，但是具有同步的的特性，【在某次添加任务后必须等到其他线程取走任务后才能继续添加】----
 * 注意上面的第3条 |--LinkedBlockingQueue      本身是无界的，也能无限提交任务---永远不可能出现上面的第3条,也就是说线程池如果有这个queue永远不触发新线程的创建，corePoolSize会干完所有的活。
 * |--ArrayBlockingQueue       有界的队列.按正常的逻辑能走完上面的四步。
 *
 *
 目前主要线程池的情况: 
 a.Executors.newSingleThreadExecutor------1，1，LinkedBlockingQueue ----一个线程干完所有的活(sSingleThreadExecutorForSharePreference) 
 
 b.Executors.newFixedThreadPool -----n, n,LinkedBlockingQueue  ----n个线程干完所有的活（mIoBoundExecutor） 
 
 c.Executors.newCachedThreadPool -----0,0,SynChronousQueue      ---来一个任务就创建一个线程来处理(如果上一个任务在之前的60s内完成会复用前一个线程)（mTimeoutExecutor）
 
 d.new ThreadPoolExecutor    -----1,n,LinkedBlockingQueue ----一个线程干完所有的活(mCpuBoundExecutor,mDbBoundExecutor)


 应用场景 
 为浏览器提供全局/缓存线程池 针对本地文件读写（文件或文件夹读写）操作，
 提供2个固定线程 针对纯运算操作（例如加解密，编解码等），
 提供与CPU内核数一致的线程池，针对数据库（读写）操作
 提供单一线程的线程池 针对系统跨进程API调用，提供超时调用机制
 */

  ThreadPoolExecutor(  int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler);
  解析线程池的构造方法  
  核心线程个数 corePoolSize
  最大线程个数 maximumPoolSize
  工作队列 workQueue


keepAliveTime  “闲置的” '非核心线程' 超时时长
如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么“闲置的” '核心线程'
也会受此超时时长的限制 

 优先级： corePoolSize > workQueue > maximumPoolSize > RejectedExecutionHandler

队列---BlockingQueue： |--SynChronousQueue---  本身是无界的，能存无限的任务，但是具有’同步‘ 阻塞 的的特性，【在某次添加任务后必须等到其他线程取走任务后才能继续添加】

-LinkedBlockingQueue      本身是无界的，也能无限提交任务---永远不可能出现上面的第3条,也就是说线程池如果有这个queue永远不触发新线程的创建，corePoolSize会干完所有的活。
-ArrayBlockingQueue       有界的队列.按正常的逻辑能走完上面的四步。

线程池优点
1. 重用，避免创建与销毁所带来的性能开销
2. 有效控制并发数量，避免大量线程直接因互相抢占系统资源而导致的阻塞现象
3. 管理线程，定制功能：周期，定时等各种任务。

一 定长线程池 只有核心线程且不会被回收，则意味着它能够更快速的响应外界的请求。
（场景：2个固定长度加解密，与cpu内核个数一致的核心进行数据库读写）
// 不超时，且任务队列无大小限制

二 缓存线程池 只有非核心线程 Integer.MAX_VALUE  2的31次方-1 需要就创建，线程有超时 
（场景：文件读写）
三 周期线程池 周期性任务
四 单例线程池 （系统跨进程API调用）
场景 

#### 数据同步

关于线程同步  从安全的数据结构涉及，锁的技术就会提到，线程封闭处理等。

使用安全的类/数据结构 使得线程安全简单化
构建线程安全的类，或使用java.util.concurrent类库
ConcurrentHashMap

不可变对象，

基础数据类型使用volatile来声明已足够，或者不用volatile，而用锁。
volatile 是比较弱的同步机制。

Vector 线程安全，但是锁粒度比较大，使用监视器模式，Vector的机制，同步代码块的方式。
好处：锁的扩展
缺点：锁力度大
在Vector和同步封装器类的文档中指出，使用内置锁来支持客户端加锁，客户端可以通过获取内置锁的方式来实现独占访问。
人家声明承认了，未声明承认的，还是别这样做，有风险。

对现有类的扩展而产生的线程安全的问题？查看源码，看下原有类的同步的设计方式，
并与之兼容，但是扩展是很有风险的，风险是原有类的变化。在设计一个类的时候，怎么设计同步策略。

如果不能扩展，或者文档未声明，怎么办？
可以考虑通过委托的方式：通过自身的内置锁增加了一层额外的加锁，不关心底层实现，而是提供一套一致的枷锁机制，保证线程安全性，虽然额外的同步可能会导致轻微的性能损失，但比扩展更健壮。


线程安全的数据结构
Java监视器模式：将对象的所有可变状态都封装起来，并由对象自己的内置锁来保护（Vector和Hashtable）

ArrayList -> Collections.synchronizedList

### 问题：安全的数据结构，安全的处理此数据结构的方法，eg：方法加锁，就一定是数据安全的吗？
这里是陷阱问题，答案，需要是同一个锁。
数据同步

CopyOnWriteArrayList  Copy-on-write，写入时复制：每次修改时，都会创建并重新发布一个新的容器副本，从而实现可变性
“写入时复制”容器的迭代器保留一个指向底层基础数组的引用
仅当迭代操作远远多于修改操作时，才应该使用“写入时复制”容器，因总是copy是有性能开销的。

ConcurrentHashMap  锁分段技术 
优势：在并发访问环境下将实现更高的吞吐量，而在单线程环境中只损失非常小的性能
劣势：ConcurrentHashMap 无法通过获得锁来实现独占访问，这也是它的劣势。
<对于ConcurrentHashMap而言，要获取内置锁的一个集合，唯一方式是递归>
在ConcurrentHashMap的实现中使用了一个包含16个锁的数组，每个锁保护所有散列桶的1/16，其中第N个散列桶由第（N mod 16）个锁来保护。

只有当应用程序需要加锁Map以进行独占访问时，才应该放弃使用ConcurrentHashMap。


内置锁 Vector的示例  
对象锁
线程封闭


再说各种锁
锁/线程的重入性，如果某个线程视图获取已经由它持有的锁，那么请求能够成功，获取锁的粒度是线程。重入性的实现原理，为每个锁关联一个计数器与线程标识。
重入性提升了加锁行为的封装性。
只有同一个锁，不同线程获取，才会有互斥行为。

数据同步，锁，锁的性能与特点 使用场景与优缺点  

悲观锁适合'写操作多'的场景，先加锁可以保证写操作时数据正确。
乐观锁适合'读操作多'的场景，不加锁的特点能够使其读操作的性能大幅提升。

乐观锁 
CAS全称 Compare And Swap（比较与交换）无锁的情况下实现数据同步。
java.util.concurrent包中的原子类 是使用了CAS的机制
1. ABA的问题 jdk1.5 通过修改的版本标记的思想修复此问题
2. 自旋时间长开销大
3. 只能保证一个共享变量的原子操作 jdk1.5修复 多个对象的CAS机制

关于锁请关注java主流锁
公平性 阻塞性 重入性 锁住资源 无锁重试
1. 是否锁住同步资源 分为悲观与乐观
2. 线程竞争锁排队否？排队是公平锁  插队再排队 非公平锁
3. 一个线程中的多个流程是否获得同一不把锁，能够获得，可重入锁，不能获得，非重入锁
4. 多个线程竞争资源，未获得资源的线程 自旋等待锁释放的是轻量级锁，阻塞等待的是重量级锁 
5. 同一个线程执行同步资源时，自动获得锁，---偏向锁
6. 无锁，不锁住资源，多个线程，只有一个能修改成功，其他的只能重试
