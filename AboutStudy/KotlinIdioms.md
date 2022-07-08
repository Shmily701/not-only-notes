[TOC]

### 小技巧

#### 代码审查

代码完成后，使用"Analyze → Inspect Code"不仅可以帮助检查代码中的问题，还可以提供对代码的优化建议(如消除不简洁的写法)
如果出现编译错误，直接看输出的log是看不懂的。。。这里的Inspce code帮了大忙，直接指定编译错误所在行。

### 属性

#### 常量值

直接在Kotlin文件中创建顶级常量值

#### 变量赋值

可直接使用 `if`，`when`，`try-catch` 表达式的值为变量赋值：

```kotlin
val y = if (x == 1) 2 else 3
```

#### 属性访问

```kotlin
val height = mDecorView.getContext().getResources().getDisplayMetrics().heightPixels
```

可简写为：

```kotlin
val height = mDecorView.context.resources.displayMetrics.heightPixels
```

#### 属性设置

```kotlin
mBtn.setText("text")
```

可简写为：

```kotlin
mBtn.text = "text"
```

#### 属性访问(仅提供get方法)

```kotlin
class Rectangle(var width: Int, var height: Int) {
    val isSquare
        get() = width == height
}
```

#### 扩展属性(提供set方法示例)

```kotlin
fun main() {
    val stringBuilder = StringBuilder("Kotlin?")
    stringBuilder.lastChar = '!'
    println(stringBuilder) // Kotlin!
}

var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value) = setCharAt(length - 1, value)
```

### 方法

#### 参数默认值

使用 `参数默认值` 以及 `命名参数` 等方式代替方法重载

```kotlin
fun foo(a: Int = 0, b: String = "") {}
```

#### 单表达式方法

单表达式方法建议不要使用花括号，直接写在一行上。

```kotlin
fun theAnswer() = 42
```

单表达式写法不写返回值可以避免与Java交互时的一个问题。例如有如下Java代码：

```java
public class Test {
    public static String get() {
        return null;
    }
}
```

从Kotlin中调用 `Test#get` ，代码如下：

```kotlin
fun main() {
    get()
}

fun get(): String = Test.get()
```

运行后程序将崩溃。

将Kotlin代码修改为如下形式，程序正常运行：

```kotlin
fun main() {
    get()
}

fun get() = Test.get()
```

#### 静态功能方法

不需要像Java中创建一个静态功能方法类，直接使用顶级方法

#### 构造方法

```kotlin
class MyView(ctx: Context, attrs: AttributeSet? = null) : View(ctx, attrs) {}
```

#### 嵌套函数

```kotlin
fun getFormatTime(timeStamp: Long): String {

    fun getTime(format: String) = SimpleDateFormat(format, LOCALE).format(Date(timeStamp))
    fun getDayTime() = getTime("HH:mm")
    fun getMonthTime() = getTime("MM-dd HH:mm")
    fun getYearTime() = getTime("yyyy-MM-dd HH:mm")

    ...
}
```

#### 扩展方法

为现有类(例如标准库中的 `String` 类)添加扩展方法

```kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

#### 扩展方法不会重写

首先看一个基类 `View` 和 继承类 `Button`：

```kotlin
open class View {
    open fun click() = println("View clicked")
}

class Button : View() {
    override fun click() = println("Button clicked")
}
```

执行如下代码：

```kotlin
val view: View = Button()
view.click() // Button clicked
```

可看到会执行 `Button` 的 `click()` 实现。

此时为 `View` 和 `Button` 均添加扩展方法 `showOff()`：

```kotlin
fun View.showOff() = println("I'm a View")
fun Button.showOff() = println("I'm a Button")

val view: View = Button()
view.showOff() // I'm a View
```

当然，如果不指定 `view` 的类型为 `View`， 上述代码将会执行`Button#showOff()`：

```kotlin
val view = Button()
view.showOff() // I'm a Button
```

### 类

#### 改变类的名称

假设Kotlin文件名称为"Test.kt"，默认生成的类名为"TestKt"，如果不想使用默认值，可以在kotlin文件顶部添加注解，编译后的类名将使用注解中提供的名称

```kotlin
@file:JvmName("Test")
```

#### 获取对象类

```kotlin
val list = arrayListOf(1, 2, 3)
println(list.javaClass) // class java.util.ArrayList
```

`javaClass` 等同于java中的 `getClass()`。

#### 单例类

```kotlin
object Singleton {
    val name = "Name"
}
```

#### 数据对象类

```kotlin
data class Customer(val name: String, val email: String)
```

创建的 `Custom` 类默认提供如下功能：

- 为每个属性提供 `getter` 方法 (如果需要提供 `setter` 方法使用 `var`)
- `equals()`
- `hashCode()`
- `toString()`
- `copy`

#### Sealed class

```kotlin
interface Expr
class Num : Expr
class Sum : Expr

fun eval(e: Expr) =
    when (e) {
        is Num -> 1
        is Sum -> 2
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

假如 `Expr` 只有两个继承类 `Num` 和 `Sum` ，但在 `eval` 方法中必须提供一个 `else` 分支，否则无法通过编译。

如果将 `Expr` 声明为 `sealed class`，就不必提供 `else` 分支，使得代码更加简洁。

```kotlin
sealed class Expr {
    class Num : Expr()
    class Sum : Expr()
}

fun eval(e: Expr) =
    when (e) {
        is Expr.Num -> 1
        is Expr.Sum -> 2
    }
```

### 表达式

#### when表达式

多重 `if-else` 逻辑用 `when` 实现

```kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

#### 将赋值语句提取出 `when`

```kotlin
when (x) {
    1 -> y = ...
    2 -> y = ...
    else -> y = ...
}
```

可简化为：

```kotlin
y = when (x) {
    1 -> ...
    2 -> ...
    else -> ...
}
```

#### `if-else` 简化

```kotlin
if (boolean) {
    mBtn.text = str1
} else {
    mbtn.text = str2
}
```

可简化为：

```kotlin
mBtn.text = if (boolean) str1 else str2
```

### 返回值

#### 将表达式的值作为返回值

```kotlin
fun getFileType(): String {
    when (mFileType) {
        FILE_TYPE_IMAGE -> return "imagefolder"
        FILE_TYPE_APK -> return "apk"
        FILE_TYPE_MUSIC -> return "audio"
        FILE_TYPE_FOLDER, FILE_TYPE_SDCARD -> return "storage"
        FILE_TYPE_DOC -> return "doc"
        FILE_TYPE_OTHER -> return "other"
        FILE_TYPE_ZIP -> return "zip"
    }
    return ""
}
```

可简化为：

```kotlin
fun getFileType() =
        when (mFileType) {
            FILE_TYPE_IMAGE -> "imagefolder"
            FILE_TYPE_APK -> "apk"
            FILE_TYPE_MUSIC -> "audio"
            FILE_TYPE_FOLDER, FILE_TYPE_SDCARD -> "storage"
            FILE_TYPE_DOC -> "doc"
            FILE_TYPE_OTHER -> "other"
            FILE_TYPE_ZIP -> "zip"
            else -> ""
        }
```

#### 返回 `when` 表达式的值

```kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

更简洁的形式：

```kotlin
fun transform(color: String) =
    when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
```

#### `try-catch` 返回值

```kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // 使用 result
}
```

#### 返回多个值

```kotlin
fun main() {
    val (x, y) = getResult()
    println(x) // 1
    println(y) // 2
}

data class Result(val x: Int, val y: Int)

fun getResult() = Result(1, 2)
```

### 对象方法调用

#### 使用 `with` 调用一个对象的多个方法

```kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for (i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

#### 使用 `apply` 为对象的属性赋值

```kotlin
val myRectangle = Rectangle().apply {
    length = 4
    breadth = 5
    color = 0xFAFAFA
}
```

### 判空

#### 如果值不为 `null` 简写

```kotlin
val files = File("Test").listFiles()
println(files?.size)
```

#### 如果值不为 `null` else 简写

```kotlin
val files = File("Test").listFiles()
println(files?.size ?: "empty")
```

#### 如果值不为 `null` 执行一段逻辑

```kotlin
val value = ...

value?.let {
    ... // execute this block if not null
}
```

#### 多重判 `null`

普通的多重判 `null`：

```kotlin
val school: School = null

if (school != null && school.students != null) {
    doSomething(school.students.get("billy"))
}
```

简洁的写法如下：

```kotlin
school?.students?.let { doSomething(it.get("billy")) }
```

### 集合

#### map遍历

```kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

`k`，`v` 变量名可以为任意值

#### 范围

```kotlin
for (i in 1..100) { ... }  // 闭区间，包括100
for (i in 1 until 100) { ... } // 半开区间，不包括100
for (x in 2..10 step 2) { ... } // 遍历时使用步进2
for (x in 10 downTo 1) { ... } // 反向遍历
if (x in 1..10) { ... }
```

#### 只读列表

```kotlin
val list = listOf("a", "b", "c")
```

#### 只读map

```kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

#### map访问

```kotlin
println(map["key"])
map["key"] = value
```

#### 列表过滤

```kotlin
val list = arrayListOf(-1, 2, -3, 4)
val positives = list.filter { x -> x > 0 }
```

或使用更简洁的 `it` 方式：

```kotlin
val list = arrayListOf(-1, 2, -3, 4)
val positives = list.filter { it > 0 }
```

#### 判断集合中的元素

```kotlin
val list = arrayListOf("a", "b", "c", "d")
if ("a" in list) { }
if ("d" !in list) { }
```

#### 从可能为空的集合中获取第一个元素

```kotlin
val emails: ArrayList<String>? = null // null or empty
val mainEmail = emails?.firstOrNull() ?: "no main"
```

如果集合为 `null` 或者 `元素个数为0`，取值 `no main`，否则获取其第一个元素。

#### 使用集合内置函数

```kotlin
val list = arrayListOf(1, 2, 3)
println(list.last()) // 3
println(list.max()) // 3
```

#### 改变集合的打印形式

```kotlin
val list = listOf(1, 2, 3)
println(list) // [1, 2, 3]
```

如果想更改 `list` 的打印形式，可使用 `joinToString()` 方法：

```kotlin
val list = listOf(1, 2, 3)
println(list.joinToString("; ", "(", ")")) // (1; 2; 3)
```

### lazy

#### 懒加载属性

```kotlin
class Test {
    public val p: String by lazy {
        // compute the string
        "a"
    }
}
```

在 `Test` 类对象访问 `p` 之前，`p` 不会初始化，状态为：Lazy value not initialized yet.

#### 晚初始化

使用 `lateinit` 关键字修饰变量，处理无法在构造函数中初始化的变量，避免后续使用变量时的判空操作(如：loginBtn?.text=...)

```kotlin
class MyActivity : Activity() {
    private lateinit var loginBtn: Button
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        loginBtn = findViewById(R.id.login_button)
    }
}
```

### 自动释放资源

#### Java 7资源 `try-with` 形式

```kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```

### lambda表达式与高阶函数

#### lambda表达式

lambda表达式是指可以传入函数中的一段代码。

```kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1, 2)) // 3
```

为 `mBtn` 设置点击事件：

```kotlin
mBtn.setOnClickListener(object : View.OnClickListener {
            override fun onClick(v: View?) {
                Log.e("zzz", "btn clicked")
            }
        })
```

简洁写法：

```kotlin
mBtn.setOnClickListener { v -> Log.e("zzz", "btn clicked") }
```

如果变量 `v` 在点击事件代码中未使用，可以用 `_` 代替：

```kotlin
mBtn.setOnClickListener { _ -> Log.e("zzz", "btn clicked") }
```

另一个例子：

```kotlin
mListView?.onItemClickListener = AdapterView.OnItemClickListener { _, _, position, _ ->
    Log.e("zzz", "pos=$position")
}
```

#### SAM转换(单一抽象方法转换)

单一抽象方法可用匿名函数来表示，此过程称为单一抽象方法转换或SAM转换(**S**ingle **A**bstract **M**ethod)，SAM转换可使代码更简洁

```kotlin
mBtn.setOnClickListener { Log.e("zzz", "btn clicked") }
```

#### 高阶函数

高阶函数是指 `以函数作为参数` 或 `返回值为函数` 的函数。在Kotlin中，函数可以用lambda表达式或函数引用来表示，因此任何以`lambda`或者函数引用作为参数的函数，或者返回值为`lambda`或者函数引用的函数，或者两者都满足的函数是高阶函数。

```kotlin
val strLength = stringMapper("Android") { input -> input.length }

fun stringMapper(str: String, mapper: (String) -> Int) = mapper(str)
```

```kotlin
fun main() {
    val calculator = getCostCalculator(Delivery.STANDARD)
    println(calculator(Order(20)))
}

enum class Delivery {
    STANDARD, EXPEDITED
}

class Order(val goodCount: Int)

fun getCostCalculator(delivery: Delivery): (Order) -> Int =
    when (delivery) {
        Delivery.STANDARD -> { order -> order.goodCount * 10 + 10 }
        Delivery.EXPEDITED -> { order -> order.goodCount * 25 + 20 }
    }
```

### 其他

#### 数据引用

```kotlin
println("Name $name")
```

#### 使用可空的 `Boolean`

```kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` is false or null
}
```

#### 交换两个变量值

```kotlin
var a = 1
var b = 2
a = b.also { b = a }
```