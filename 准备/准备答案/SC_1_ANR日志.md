ANR  ANR的监测机制是system_server进程的AMS线程  而这个进程pid与线程tid在下面的日志中可以看到。
AMS.appNotResponding() 首先会打印event log，然后再打印system log
在event  log中搜索am_anr 能看到发生ANR的进程
在system log中搜索ANR in  能看到发生ANR的原因

通过上述搜索到的进程与原因
在查看trace文件，通过block或sleep的信息，查到lock的held thread编号，再查看thread编号
对应的堆栈，即可得到实际信息。


思考一个面试题，一个 Service 运行在独立的进程里，在这个 Service 的 onCreate 方法里执行耗时操作会造成 ANR 吗？

盘点ANR 的四种场景：

Service TimeOut:  service 未在规定时间执行完成：前台服务 20s，后台 200s
BroadCastQueue TimeOut: 未在规定时间内未处理完广播：前台广播 10s 内, 后台 60s 内
ContentProvider TimeOut:  publish 在 10s 内没有完成
Input Dispatching timeout:  5s 内未响应键盘输入、触摸屏幕等事件
ANR 的根本原因是：应用未在规定的时间内处理 AMS 指定的任务才会 ANR。

所以 Service 未在指定时间内执行完成而且非主进程的 Service 仍然需要通过 AMS 进行通信。
这也能说明一定会产生 ANR 的。

ANR_SHOW_BACKGROUND 可以在「开发者选项」里配置显示所有“应用程序无响应”，开启后即使是该场景也会有 ANR 弹窗
未开始上述配置情况下，如果是后台进程，则不显示 ANR 弹窗

回到开头的面试题，虽然是非主进程，也不能胡作非为。


