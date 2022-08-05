1. 下载安装 https://www.charlesproxy.com/
2. pc端也需要安装证书  
    - 1.charles点击【Help】>>【SSL proxying】>> 【install charles root certificate】
    - 2.点击【安装证书】
    - 3.【本地计算机】>>【下一步】
    - 4.选择【将所有的证书都放人下列存储】>>【浏览】>>【受信任的根证书颁发机构】>>【下一步】>>【完成】
    - 5.重启Charles
3. 手机端也要安装证书 
4. 手机端设置代理 IP and port
以上最基本的抓包功能就有了

还有一些常用的功能-
找到对应的请求，右键Breakpoints，然后点击工具栏的铅笔 >> 点击右边的Execute 可以看到Edit Request 
### Edit Request

### Edit Response 亲测有效
工具栏 >> Proxy >> Breakpoints Settings >> Enable Breakpoints 选中请求双击 勾选 
Request && Response 
或者勾选其中一项  在请求返回来时会断点住，可以看到右边的 Edit Request 点击 Execute 可以看到 Edit Response 并切换到下方的返回json 直接更改返回值即可。

### 模拟弱网--很好用的 亲测有效
- 点亮工具栏的小乌龟并设置
- Proxy >> Throttle Settings >> Enable Throttle
- 设置Utilistation(利用百分比)、
Round-trip(往返延迟)
- 如果自定义，记得add-Preset 

### 基本强大的设置都在工具栏-Proxy下面

### 要免费todo


Registered Name: https://zhile.io
License Key: 48891cf209c6d32bf4

在线激活码计算器
https://www.zzzmode.com/mytools/charles/

可以参考的文章
https://zhuanlan.zhihu.com/p/407537666
https://zhuanlan.zhihu.com/p/340973096