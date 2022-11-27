1. 安装比较新版的as时,buid工程报错奇怪

无法创建任务`':app:minifyReleaseWithR8'`  搜索一圈,各种关于R8的配置,都无法解决问题
以为是as版本的问题,于是下载了最新as版本依然无法解决.
期间发现jdk版本配置问题,顺手修复了

stackoverflow有一篇对此介绍
`https://stackoverflow.com/questions/64459937/could-not-create-task-appminifyreleasewithr8-cannot-query-the-value-of-this`
只当时是对应sdk版本下载的问题,于是卸载了sdk 30 版本,又重装了一次,但是无果.
但是当时显然没完全看懂


只好通过`通过gradlew 命令执行, gradlew assembleDebug --info`得知真正报错的原因是与buildToolsVersion: "30.0.3",有关, 于是问题缩小为buildToolsVersion小版本的问题,
切换到SDK Tools 下,勾选show Package Details 能够发现,果然Version为
30.0.3的版本未安装,下载并安装,解决此问题.  

结论: 问题如下:
1. 遇到问题,无脑搜索,没有真正看明白问题所在.
2. 明知道`gradlew assembleDebug --info `命令好用之处,却没有形成习惯.
3. 明白buildToolsVersion的小版本的含义,大版本下实际小版本
