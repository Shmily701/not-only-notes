SogouApm项目介绍
系统的监控组件
1. 业务监控
2. 网络监控
3. 


性能优化
    糖猫手表app的启动时间的优化 
    方案：TraceView
    在关键节点打点，记录流程耗时，并使用Android Studio的Profiler加载查看导出trace文件，通过可视化的执行时间与调用栈来分析定位耗时节点。
    成功：
    将糖猫手表多款app的启动时间从2000ms优化到1000ms以内。
    启动时长降低50%左右。


    SogouApm
    针对
    内存优化  profiler
    ui优化  布局层级
    SogouApm使用到的技术 监控闭环处理 TODO 大头需要吃透


项目流程优化 20min
    为提升问题定位的速度与研发效率，浏览器包含隐藏功能：开发者模式。
    功能介绍：
        通过特殊路径，触发开发者模式。开发者模块提供功能
        1. 本地模拟云控数据下发，覆盖下发数据。
        2. 本地开启内核，系统核与x86内核切换
        3. 本地开启webview调试功能

        SogouApm组件
        1.业务监控
        2.网络监控 
        3.内存监控
            LeakCanary+KOOM的结合
        4. 
    针对
    内存优化  profiler
    ui优化  布局层级
    对SogouApm项目的学习与了解
    SogouApm使用到的技术 监控闭环处理 TODO 大头需要吃透


=======
    1. 基础组件 监控模块组件化 SogouAPM
    工作：梳理业务的重点与痛点，针对关注的模块做监控，
    对基础组件SogouAPM的业务监控的上层封装。

*********************
可用


架构设计
    方案
    基于搜狗浏览器的项目特性（较少的场景跳转，密集的方法调用）采用基于总线型组件化方案。结合接口下沉来组织各组件对外提供的能力，实现双向驱动通信模型。//并采用AOP思想，编译时通过编译字节码实现事件接口向总线的注册。以此来解决依赖反转的问题。

    搜狗浏览器的项目结构    
    主module
    业务组件
    功能组件
    基础组件

主要工作基础模块组件化
  
    1. 基础组件 上报模块组件化statistics  DONE
    原始状况：项目中分散各个业务场景的数据构建与数据发送，数据构建与发送耦合，且针对不同类别的上报后处理方式也不同，代码冗长。

    工作：梳理散落在项目中的上报代码块，拆分数据的构建与发送，将数据的发送剥离业务层，提取下沉到基础组件module_base层，进行模块封装，并提供接口外观供上层组件使用；针对上报后处理通过依赖倒置方式，发送组件接收抽象参数，由业务实现接口定制。
    而数据的构建，则提取数据管理类，统一参数列表，上报触发仍保留在原有的业务模块，数据构建统一调用数据管理类。
    
    组件内部包含：
    贴图
    组件内部的数据发送时机由空间占用||时间持久度阈值来判断，其中阈值作为参数可云控下发。

    发送策略: 直接发送.若发送失败,数据入库.待发送时机,通过发送本地数据库数据再次发送


----------------------------------------------------------------------

技术分享与执行落地 DONE
	1. Android单元测试方案调研与落地使用 （https://www.jianshu.com/p/264b24dce143）
    通过对业界各类单元测试框架进行调研 Java单测框架、Android单测框架、AndroidUI测试框架 从运行平台、功能支持、运行耗时等维度进行对比，
    最终采用JUnit + Robolectric + Mockito的方案进行单元测试 
    + PowerMock  弥补Mockito的局限性 
PowerMock 弥补了Mockito的不足 支持private方法的调用 static方法的mock final类的mock 以及提供了WhiteBox工具箱功能。
    
    成果：推广组件化模块的单测开发，推动组内成员，针对新的模块进行单测覆盖，
    使得新增模块的单测覆盖率达到40%以上。
    
    2. 动画方案的调用与逻辑使用
    对比业内广泛使用的动画框架，SVGA与Lottie，
    从CPU耗能，图片资源大小，应用场景与社区活跃度几个维度，臻选适合项目使用的动画框架Lottie，并结合源码与官网API来分析Lottie动画的解析流程、动画播放原理、绘制原理、并分享在使用过程中遇到的问题与解决方案。
    封装一套供浏览器项目使用的接口框架。
   https://www.jianshu.com/p/b7ededae9011
   成果：替换项目中的gif图，减少apk大小3M以上。


   	② ui卡顿的监控的调研 
	方案1 利用UI线程的Looper的Printer匹配，借助Looper提供的setMessageLogging(@Nullable Printer printer) 方法来实现定制化监控
	场景：适用于开发与灰度阶段

	方案2 利用Choreographer的sync的信号量的监控，借助Choreographer$postFrameCallback() API 在doFrame(long frameTimeNanos)回调中，即可计算Choreographer.doFrame()的执行时间
	场景：适用于线上环境监控

	-------------------------------------------




