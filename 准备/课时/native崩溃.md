#### Android的两种崩溃
1. java

2. native - 访问非法地址 
          - 地址对齐出了问题 
          - 程序主动abort
    这些都会产生响应的signal信用，导致程序异常退出
 
      异常捕获机制的了解？？？
   
   Android 平台 Native 代码的崩溃捕获机制及实现  https://mp.weixin.qq.com/s/g-WzYF3wWAljok1XjPoo7w?      



   Native 层的崩溃捕获机制及实现 
   https://juejin.cn/post/6931939584548798471

   Native Crash捕获原理与实践（一）
   https://blog.csdn.net/sailuoatm/article/details/118463339
