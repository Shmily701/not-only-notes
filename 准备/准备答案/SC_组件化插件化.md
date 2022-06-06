
组件化与模块化的区别
模块化是业务导向，组件化是功能导向。
[模块] 和 [组件] 间最明显的区别就是模块的粒度更大
每个组件都可以以一个单独的 module 开发，并且可以单独抽出来作为 SDK 对外发布使用。
二者共同目标：代码的重用和业务解耦
组件化
1. app壳工程  负责管理各个业务组件和打包apk 无具体业务和功能。
2. 集成模式   所有业务组件被"app壳功能"依赖，组成完整的app
3. 每一个组件都是一个app 独立工程 

单独运行调试
    组件的build.gradle文件中 动态配置appid  libraryid  apply plugin 
    动态配置组件的 ApplicationId 和 AndroidManifest

组件间的通信
    添加base模块，base模块被所有组件依赖，在这个模块中，定义了组件对外提供访问自身数据[导出]的接口，「每个组件需要实现这个接口，并提供对外的数据访问」
    在base模块提供了一个ServiceFactory， 而组件实现类对象添加到ServiceFactory中。这样不同的组件可以通过ServiceFactory获取调用组件的接口实现，然后调用其中的方法。
    view Fragment 也可以这么搞 

界面跳转
    url注解方式 作为路由跳转

代码解耦如何实现 
runtimeOnly  依赖项仅在运行时对模块及其消费者可用，编译期间依赖项的代码对其消费者时完全隔离的。

资源隔离
在每个组件的 build.gradle 中添加 resourcePrefix 配置来固定这个组件中的资源前缀。
不过 resourcePrefix 配置只能限定 res 中 xml 文件中定义的资源，并不能限定图片资源，所以我们在往组件中添加图片资源时要手动限制资源前缀。

#### 插件化

插件化优点
插件化让 Apk 中的代码（主要是指 Android 组件）能够免安装运行，这样能够带来很多收益：

    减少安装Apk的体积、按需下载模块
    动态更新插件
    宿主和插件分开编译，提升开发效率
    解决方法数超过65535的问题

// 打包的时候将宿主Apk和插件Apk分开打包  动态下发

插件化的几个难题
1. 四大组件是需要在系统中注册，宿主动态加载执行插件Apk中 Android 四大组件是难点
2. 资源冲突
3. 插件代码如何加载


借助ClassLoader 
java中的 
BootstrapClassLoader
负责加载 JVM 运行时的核心类，比如 JAVA_HOME/lib/rt.jar 等等

ExtensionClassLoader
负责加载 JVM 的扩展类，比如 JAVA_HOME/lib/ext 下面的 jar 包

AppClassLoader
负责加载 classpath 里的 jar 包和目录

Android中的ClassLoader 
PathClassLoader 用来加载系统类和应用程序类，可以加载已经安装的 apk 目录下的 dex 文件

DexClassLoader 用来加载 dex 文件，可以从存储空间加载 dex 文件。
插件化中一般使用的是 DexClassLoader。

### 双亲委派机制 
每一个 ClassLoader 中都有一个 parent 对象，代表的是父类加载器，在加载一个类的时候，会先使用父类加载器去加载，如果在父类加载器中没有找到，自己再进行加载，如果 parent 为空，那么就用系统类加载器来加载。通过这样的机制可以保证系统类都是由系统类加载器加载的。

动态加载技术按照加载的可执行文件的不同大致可以分为两种：

动态加载 .so 库
动态加载 dex/jar/apk文件


框架
VirtualAPK 是滴滴开源的一套插件化框架
1. 四大组件无需注册 自定义view 
2. 兼容性好
3. Android特性 
4. 插件构建使用Gradle插件构建


Shadow 
1. 四大组件
2. Fragment 


##### Today 组件化问题回答时
先对组件化的模块进行介绍  不同的模块方案不同
1. 技术组件  纵向
2. 业务组件  横向










