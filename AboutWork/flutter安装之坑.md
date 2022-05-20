

  - flutter 的安装  
在安装flutter时，error：Android license status unknown 按照提示 Run `flutter doctor --android-licenses` 
得到一个更奇葩的提示
ERROR: JAVA_HOME is set to an invalid directory: /Applications/Android Studio.app/Contents/jre/jdk/Contents/Home

通过寻找JAVA_HOME的配置
open .bash_profile 
修改AVA_HOME的值 为 $(/usr/libexec/java_home) 
这种方式可以直接动态获取java_home的值 很灵活 在mac 10 之后的版本可以使用。

通过命令 `/usr/libexec/java_home -V`
可以打印出java_home的位置 可以去真实的路径下去核实  正确。

重新运行命令`flutter doctor --android-licenses` 

发现报错依然，可是JAVA_HOME路径的确更新了。
难道是缓存？难道是没有`source  .bash_profile `
几个回合下来，我猜想，没改对地方，应该还有个地方设置了java_home。 而flutter的安装正是读取的那处的java_home. 

基于之前的记录，某些mac系统的环境变量配置在 /.zshrc 下
执行 `open .zshrc` 果然查到了配置java_home错误的那个幺蛾子，修正后，重新run flutter相关命令，果然没有报错了。



也可以通过命令
`echo $JAVA_HOME` 获取到java_home的值。