# kotlin学习总结
kotlin的好的书籍太少了，于是需要自己总结下。

别人的总结：https://www.cnblogs.com/yongfengnice/category/1462834.html
让app永不崩溃
https://www.cnblogs.com/yongfengnice/p/16278014.html 这个值得一试
https://blog.csdn.net/weixin_43901866/article/details/113735312

- 异常处理思想：对抛出的异常与返回的结果一视同仁。
- 取消响应：取消响应中的响应是很关键的一点，需要异步任务主动提供这样的能力，如果它不配合，那么外部也就没有办法干预。所以在构建异步程序的时候，需要程序设计者考虑到这点。

### 常用的理解



### 常用语法
let和apply类似，唯一不同的是返回值：apply返回的是原来的对象，而let返回的是闭包里面的值。let函数返回的是闭包的最后一行。

如果我们不仅仅只想判空，还想加入条件，这时let可能显得有点不足。让我们来看看takeIf
```
val result = student.takeIf { it.age >= 18 }.let { ... }

```
与takeIf相反的还有takeUnless，即接收器不满足特定条件才会执行。


## kotlin协程 



## Tips
ConcurrentHashMap中的value不能为null，所以我们可以使用EMPTY_BITMAP，它是一个空的Bitmap对象，用来充当空对象。


## 语法集锦

`block`
```
block 理解为一段代码块而已（匿名的参数||方法）。可以在调用的时候，把一段代码块实现作为参数而已。
//语法格式
块名:(参数:参数类型) -> 返回值类型
//kotlin不需要写return，最后一行则是返回值

block的使用有4中形式
1、无参数，无返回值 
```
testAdd{
    println("-----")
}

fun testAdd(block:()->Unit) {
    block()
}
```

2、无参数，有返回值
testAdd{
    "-----"
}

fun testAdd(block:()->String) {
    val str = block()
    println("return str： " + str)
}

3、有参数，有返回值
testAdd{
    x,y->x*y
}

fun testAdd(block:(x:Int, y:Int)->Int) {
    val result = block(5,6)
    println("return result:" + result)
}

4、有参数，无返回值，回调参数 
testAdd{
    x,y-> print("x="+x+"   y="+y)
}

fun testAdd(block:(x:Int, y:Int)->Unit){
    block(5,6)
}

Kotlin的Block，让代码更简洁，定义set方法的同时就定义了回调的入参和返回值，省略了冗余的接口定义，轻量的定义方式，并使用Lambda表达式减少内部类的冗余。


```
`Unit` 
```
This type corresponds to the `void` type in Java. 理解为void
```
`Any` 
```
 The root of the Kotlin class hierarchy. 理解为java的Object 
```

`Nothing`
```
Koltin中一切方法都是表达式，也就是都有返回值，那么正常方法返回 Unit ，无法正常返回的方法就返回 Nothing.


```



```
fun <T> BaseViewModel.request(
    block: suspend () -> BaseResponse<T>,
    success: (T) -> Unit,
    error: (AppException) -> Unit = {},
    isShowDialog: Boolean = false,
    loadingMessage: String = "请求网络中..."
)

```

T类型，是block函数返回值BaseResponse<T>的包裹类型，也是参数success【函数型参数】参数T的类型，返回值是Unit。
所以success接收的参数就是T