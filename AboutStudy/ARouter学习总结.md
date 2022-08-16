一、ARouter的使用遇到的问题
ARouter在使用时，与三方sdk不同之处
1. .java 的配置与 .kt 在gradle中的配置不同
2. 基于1: 跳转的两个Activity绝不可以一个是.java 一个是.kt，否则跳转失效，亲测如此
3. 真正的配置很简单，以kotlin配置为示例
    
    #### ①在需要交互的module的gradle文件中配置如下
    ```
    kapt {
        arguments {
            arg("AROUTER_MODULE_NAME", project.getName())
        }
    } // 与android同级

    ```
    #### ②在需要交互的module的gradle文件中依赖如下
    ```
    implementation 'com.alibaba:arouter-api:1.5.2'
    kapt 'com.alibaba:arouter-compiler:1.5.2'

    kapt 需要引入：
    plugins {
        id 'com.android.library'
        id 'kotlin-android'
        id 'kotlin-kapt'
        id 'kotlin-parcelize'
    }
    
    ```
    #### ③在项目的gradle文件配置依赖如下
    ```
    classpath 'com.alibaba:arouter-register:1.0.2'
    ```

### build编译包
上述配置完成后，可以在对应的module路径下
```
build\generated\source\kapt\debug\com\alibaba\android\arouter\routes
```
看到生成的文件ARouter$$Group$$account 
如果这个文件未生成或不符合预期，那回头看配置是否哦正确。

二、ARouter做到了组件的业务跳转解耦，简单易用，功能强大
功能理解与罗列TODO


三、学习ARouter的源码TODO    框架方向
四、学习LeakCanary的源码TODO 性能方向
