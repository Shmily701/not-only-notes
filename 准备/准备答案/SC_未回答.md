Today
###### 互斥锁的问题 1
公平锁与非公平锁的区别
公平锁：多个线程按照申请锁的顺序排队，队列中第一个线程才能获得锁，其余的线程只能阻塞等待。新来的申请锁的线程排到等待队列中阻塞，需要唤醒的线程数+1

优点：等待的线程不会饿死。
缺点：整体吞吐效率较低，等待队列中的线程会阻塞，CPU唤醒阻塞的线程开销较大。
公平锁示例：排队打水

非公平锁：线程会尝试先获得锁，如果锁正好闲置，新来的线程就会优选获得锁，不需要在队列里排队，也不会阻塞，CPU不需要增加唤醒工作。如果插队失败，再去排队，那么CPU需要唤醒的线程个数同公平锁。

独享锁 VS 共享锁
独享锁（排它锁）：该锁一次只能被一个线程所持有，如果线程T对数据A加上排他锁后，其他线程不能再对A加任何锁。 独享锁 eg：synchronized Lock

共享锁：该锁可被多个线程所持有，如果线程T对数据A加上共享锁后，则其他线程只能对A再加共享锁，不能加排它锁。获得共享锁的线程只能读数据，不能修改数据。

ReentrantReadWriteLock 内部有两把锁，ReadLock和WriteLock。读写锁。
读锁与写锁的锁主题都是Sync，但是加锁方式不同。读锁是共享锁，写锁是独占锁。
读锁的共享锁可以保证并发读高些。而读写、写读、写写互斥，因为读写锁分离。所以ReentrantReadWriteLock并发性比一般互斥锁有很大提升。

```
ReentrantLock里面有一个内部类Sync，Sync继承AQS（AbstractQueuedSynchronizer），添加锁和释放锁的大部分操作实际上都是在Sync中实现的。它有公平锁FairSync和非公平锁NonfairSync两个子类。ReentrantLock默认使用非公平锁，也可以通过构造器来显示的指定使用公平锁。
```

重入锁- 占有锁的线程再次获取锁的场景


读写锁
读写锁维护了一对锁，一个读锁和一个写锁，通过分离读锁和写锁，使得并发性相比一般的排他锁有了很大提升
ReentrantLock  
ReentrantReadWriteLock


#### 设计模式 2  ing

#### Http 3 
客户端如何认证服务端？证书
服务端如何认证客户端？证书
http 1.0 2.0 3.0的区别  // 先不看


断点下载
客户端
范围请求（Range Request） 
1. 在请求头[Range Header]中，指定字节范围，告知服务端字节想要获取对象的哪部分。 Range:bytes=[1-1000]
2. 执行范围请求时，会用到首部字段Range来指定资源的byte范围
多重范围的范围请求，响应会在首部字段Content-Type标明multipart/byteranges后返回响应报文。

服务端
需要服务端的支持，且响应里也要带content-range字段，1024-2000/2100
我传递的是1024-2000的内容，而整个文件大小是2100字节
status code 206 
status code 416 表示客户端发送的range范围有误。

出现的异常情况
1. 不支持： 服务端返回的响应头部如果没有content-range字段，说明服务忽略了请求里的range字段。
2. 还是从头传。响应头部包含了content-range字段，但还是从0开始下载的。
这是服务端实现严格，考虑到了文件被修改的情况，需要标识下载资源的唯一性，
可以借助HTTP协议的两个字段 ETag（类似md5的唯一表示） 或 Last-Modified（最后一次修改的时间）

如果服务端比较严格，检查文件是否有变化，方案如下
1. 第一次下载需提供ETag 或 Last-Modified
2. 断点下载的请求，将If-Range && Range字段

服务端需要记住已下载的内容，这样下次可以跳过之前已下载的部分。


HTTP+加密+认证+完整性保护=HTTPS  
1. 加密
2. 认证 证书认证 可正免服务端或客户端的身份
3. 完整性保护


#### 最后看下简历里的监控 LeakCanary的原理  7
看简历
看源码原理

#### native崩溃栈的查看与处理   
1. 「拿到崩溃栈」tombstones文件，有native崩溃信息 
2. 「解析堆栈」带symbols信息的so文件，symbols用来解析堆栈 
3. 「分析堆栈」工具名称  addr2line ndk-stack 对堆栈进行分析

当发生native崩溃时，会在目录`/data/tombstones/`下生成tombstone文件，文件中也会包含此处logcat中输出的native崩溃信息。
带symbols信息的so
symbols信息就是用来解析堆栈使用的.

下面将使用如下工具对堆栈进行分析：
- addr2line
- ndk-stack
- objdump


#### 加解密，安全策略相关  待补充
两类
1. 前端： XSS CSRF 
2. 针对协议： HTTP DNS -> HTTPS  HTTPDNS
3. app端： 防导出 不注册非必要的组件


返回私密数据的mask，，前端加密，钓鱼，salt
前端跨域安全攻击有 XSS，CSRF
XSS（Cross Site Scripting）跨站脚本攻击，攻击者在web页面插入恶意的script代码，当用户浏览该页面之时，嵌入其中的script代码会被执行，从而达到恶意攻击用户的目的。

CSRF(Cross Site Request Forgery)跨站请求伪造，其原理是攻击者构造网站后台某个功能接口的请求地址，诱导用户去点击或者用特殊方法让该请求地址自动加载。

HTTPDNS是避免dns劫持的一种有效手段 WebViewClient$shouldInterceptRequest

内核回调WebViewClient$shouldInterceptRequest 

跨域安全

网络安全 HTTPS 
    一般情况下，私密数据的传输用post+body就好。但如果是HTTP请求，每个byte都会在网络上明文传播，不管是url还是body

app安全
防导出
不注册非必要的组件

 
#### HashMap的原理 
    1. 结构：底层是一个数组，数组的每一项又是一个链表。数组中存储的，是链表地址的引用，链表中的存储的数据结构是Entry对象 key value 通过next相连
    2. hash%length = index  链地址发
    3. 链表长度过长，会影响查询效率，超过负载因子（Entry集合填满程度）resize为原来的两倍大小
===========================



##### 组件化架构
先对组件化的模块进行介绍  不同的模块方案不同
1. 技术组件  纵向
2. 业务组件  横向
TODO

### kotlin携程与多线程的区别
线程是同步机制，而协程则是异步。 
线程是操作系统调度，
协程是应用系统自己调度
对线程的封装 
TODO


#### 内存优化方案 从以下两个角度
1. 内存创建与释放的时机
    未释放的
    细化表达

2. 减少不必要对象创建方向
   循环方法内部创建过多局部变量


#### Http相关问题阐述 四个角度：
    协议与相关协议
    传输过程
    安全性
    状态码


### 多线程锁相关 数据同步
1. 使用过的互斥锁
   sync

2. 数据结构中的锁
   Collections.synchronizedList
   Vector
   ConcurrentHashMap

3. 数据同步
   安全的数据结构
   线程封闭
   static voliate 弱同步    
   不可变量减少不必要的同步 
========================
 
4. 锁的分类
   读写锁 
   公平锁 悲观乐观 重入非重入 


### UI优化回答，三个角度
1. ui本身绘图优化
	减少层级
	viewStub
	复用ui
	局部刷新
	过渡绘制

2. 卡顿导致的问题
	viewGroup测量
	多个view绘制

3. 内存 绘图效率  平衡 取舍的做法
	列表复用
	ui缓存
	图片缓存
	页面缓存

==============================



===============================
遇到的问题 未回答好的

    组件化设计说下
    TODO 
    MVP与MVC  MVVM的区别
    TODO 
    内存监控框架的理解与区别
    TODO 
===============================
刷题

    flutter的理解


