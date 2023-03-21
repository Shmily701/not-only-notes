
常用的构建协程的方式
1. CoroutineScope

2. async/await 二者配对使用，且必须在协程内使用
```
val job = Job()
CoroutineScope(job).launch {
	val result = async {
		delay(3000)
		"操作完成"
	}.await();	
	Log.d("zzz",result)
}

运行结果：3s后打印出"操作完成"
```
await方法会阻塞当前协程，直到获取执行结果。

3. withContext 一个挂起函数
怎么理解suspend函数：suspends until it completes，and return the result. 
suspend函数只能在协程作用域中调用，或者在suspend方法中调用。
withContext函数会将最后一行的执行结果，作为返回值

协程代码一定在子线程中吗？ nope！
eg：witchContext函数需要"线程参数" 这个线程参数决定了协程作用域的代码执行在哪个线程。
灵活优雅的借助协程来完成线程切换，将异步代码实现同步调用。

```
	fun test() {
		GlobalScope.launch(Dispatchers.Main) {
			val ret = loadData()
			updateView(ret)
			val ret1 = loadData()
			updateView(ret1)
		}
	}


	private suspend fun loadData() =
		withContext(Dispatchers.IO) {
			Log.e("zzz", "loadData " + Thread.currentThread());
			delay(3 * 1000)
			"delay 3 s"
		}

	private fun updateView(result:String) {
		Log.e("zzz","updateView " + Thread.currentThread() + " result " + result )
	}

```	
Retrofit与协程的结合
Retrofit自动为开发者做了这样一个处理：当接口声明为suspend类型时，Retrofit会自动切换到子线程中执行。

KTX 扩展提供的创建协程作用域的方法 
在UI页面创建协程
lifecycleScope.launch {
	
}

在ViewModel中创建协程
viewModelScope.launch(Dispathcers.IO) {
	
}
使用KTX扩展创建协程作用域的好处是：viewModelScope或lifecycleScope是与生命周期绑定的，所以当Activity销毁时，会自动取消协程操作，无须开发者进行额外的操作。


Kotlin的数据流
Flow 在协程中通过async或withContext挂起函数可以返回单个数据值，数据流（Flow）以协程为基础构建，可以按顺序返回多个值。
Flow冷流，需订阅后才开始执行发射数据流的代码，会返回数据。
使用collect方法对Flow进行订阅
filter操作符为开发者提供了对结果限制的功能。


协程的非阻塞式挂起是如何实现的？


已有协程了，为什么还要有线程？
线程切换成本高，线程的由操作系统调度，开销无法忽视，而协程是更轻量级的线程，协程的切换可以由程序自己控制，so使用协程降低了开销（线程包含协程，如内存足够，一个线程中可以有任意多个协程，多个协程分享线程中的计算机资源。）



关于携程的阻塞
```
import kotlinx.coroutines.experimental.*

fun main(args: Array<String>) = runBlocking { 
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    delay(2000L)
}

```
runBlocking创建的是主协程，而launch里创建的是子协程。可以在主协程中创建子协程，反之不可以。
通过launch构建的子协程，不会阻塞线程，而使用runBlocking创建的主协程会阻塞线程的。


delay方法只能在协程内部使用，它用于挂起协程，不会阻塞线程；
