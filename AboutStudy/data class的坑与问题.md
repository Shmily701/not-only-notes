
11-24 即将上线的前一天，公司领导要求不许使用data class 说膨胀引发许多问题。于是我用java的方式copy别人的写法，非常痛苦，花了许多时间。。。贻笑大方了。。

比起data class 和 class 它们之间的差异在哪里？
首先在kotlin中，想要写一个javaBean类，非常简单。

@SuppressLint("ParcelCreator")
@Parcelize
class StagingConfirmOrderBean(
    // 	数量
    var count: Int?
) : Parcelable

（如果不需要序列化，相关Parcelable的注解都可以删除。）将其编译为Java代码：在as中Tools-Kotlin-Show Kotlin Bytecode-Decompile
可以看到Kotlin将 数据类中的get、set、Parcelable等方法都做了，无需想javaBean那样的大量胶水代码。

将上述代码改为data class 即仅在class 前面加一个data关键字
编译为java代码看下
可以看到新增了几个方法
copy、hashCode、equals、toString等方法 即帮你做了对象比较功能的支持。
（支持对象的比较，需要重写hashcode，equals等方法）。
so 如果你不需要做对象比较，或将对象的类型作为一个key值，那么无需重写这些方法，使得类不要那么臃肿，当然在kotlin层面只是有无data关键字的差异，而对于占用包大小，内存编译，也有几十K的差别叭~

