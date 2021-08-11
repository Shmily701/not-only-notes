本文使用测试工程[AndroidTestJNI](AndroidTestJNI.zip)及x86_64模拟器。只能使用CPU架构为x86_64的机器.

修改`hello-jni.c`文件内容如下：

```c
#include <string.h>
#include <jni.h>

void testCrash() {
    int* p = NULL;
    *p = 1;         //line 6
}

JNIEXPORT jstring JNICALL
Java_org_lwh_test_HelloJNI_stringFromJNI( JNIEnv* env,
                                                  jclass clazz )
{
    testCrash();
    return (*env)->NewStringUTF(env, "Hello from JNI !");
}
```

在工程根目录下执行命令`./nb.sh`，观察到测试应用已安装运行，但因为代码有bug，所以运行后崩溃退出。执行命令
`
adb logcat -c  
adb logcat -d > 1.txt
`
获取到logcat文件。接下来开始分析native崩溃日志。

native崩溃日志是以固定字符串`*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***`开始的，从上述`1.txt`中截取部分内容如下：
native崩溃日志是以固定字符串`Tombstone written to: /data/tombstones/tombstone`结束的.

```s
02-23 14:32:14.727  2822  2822 F DEBUG   : *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
02-23 14:32:14.727  2822  2822 F DEBUG   : Build fingerprint: 'Android/sdk_x86_64/generic_x86_64:10/QP1A.190711.019/eng.lwh.20210125.182755:eng/test-keys'
02-23 14:32:14.727  2822  2822 F DEBUG   : Revision: '0'
02-23 14:32:14.727  2728  2995 D EGL_emulation: eglMakeCurrent: 0x7fc7e4e63360: ver 3 0 (tinfo 0x7fc7e4ed0740)
02-23 14:32:14.727  2822  2822 F DEBUG   : ABI: 'x86_64'
02-23 14:32:14.728  2822  2822 F DEBUG   : Timestamp: 2021-02-23 14:32:14+0800
02-23 14:32:14.728  2822  2822 F DEBUG   : pid: 2778, tid: 2778, name: org.lwh.test  >>> org.lwh.test <<<
02-23 14:32:14.728  2822  2822 F DEBUG   : uid: 10113
02-23 14:32:14.728  2822  2822 F DEBUG   : signal 4 (SIGILL), code 2 (ILL_ILLOPN), fault addr 0x7fc799844600 (*pc=0x90660b0f)
02-23 14:32:14.729  2822  2822 F DEBUG   :     rax 00007fc799844600  rbx 00007fc8747fcc00  rcx 00007ffc42824160  rdx 00007ffc4282416c
02-23 14:32:14.729  2822  2822 F DEBUG   :     r8  16bf071000000001  r9  747fcc0000000001  r10 fffffffffffff2c6  r11 0000000000000000
02-23 14:32:14.729  2822  2822 F DEBUG   :     r12 00007ffc42824200  r13 00007fc8749be7f1  r14 0000000000000000  r15 00007ffc42825828
02-23 14:32:14.729  2822  2822 F DEBUG   :     rdi 00007fc8748b46c0  rsi 00007ffc42823f74
02-23 14:32:14.729  2822  2822 F DEBUG   :     rbp 00007ffc42823f70  rsp 00007ffc42823f48  rip 00007fc799844600
02-23 14:32:14.782  2728  2995 I chatty  : uid=10088(com.android.systemui) RenderThread identical 6 lines
02-23 14:32:14.788  2728  2995 D EGL_emulation: eglMakeCurrent: 0x7fc7e4e63360: ver 3 0 (tinfo 0x7fc7e4ed0740)
02-23 14:32:14.823  2822  2822 F DEBUG   : 
02-23 14:32:14.823  2822  2822 F DEBUG   : backtrace:
02-23 14:32:14.823  2822  2822 F DEBUG   :       #00 pc 0000000000000600  /data/app/org.lwh.test-q5kNUvKLvEHi5miTFh659w==/lib/x86_64/libhello-jni.so (Java_org_lwh_test_HelloJNI_stringFromJNI) (BuildId: f9c81b2c2a617a6524d5eace3e6500a553882999)
02-23 14:32:14.823  2822  2822 F DEBUG   :       #01 pc 0000000000174641  /apex/com.android.runtime/lib64/libart.so (art_quick_generic_jni_trampoline+209) (BuildId: 05ec7204fd5b2d5a21d6e4cf8dddc1ee)
...
02-23 14:32:14.939  1780  1780 E /system/bin/tombstoned: Tombstone written to: /data/tombstones/tombstone_01
```

> 观察上述日志的最后一行，当发生native崩溃时，会在目录`/data/tombstones/`下生成tombstone文件，文件中也会包含此处logcat中输出的native崩溃信息。

上文中编译后我们注意到生成了两个so文件，一个是`~app/src/main/java/libs/x86_64/libhello-jni.so`，另一个是`~app/src/main/java/obj/local/x86_64/libhello-jni.so`，
第一个so不带symbols信息，我们可以对比这两个so文件的大小，发现不带symbols信息的so明显较小。我们这里要用到的是带symbols信息的so。
symbols信息就是用来解析堆栈使用的.

下面将使用如下工具对堆栈进行分析：
- addr2line
- ndk-stack
- objdump

### addr2line
用来分析单个pc地址对应的源码行数，需要使用和so文件相同的架构:
    如本文编译的so为x86_64架构，则需要使用`x86_64-linux-android-addr2line`（在我的电脑上的路径为`C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\toolchains\x86_64-4.9\prebuilt\windows-x86_64\bin`）。
    从上文日志中找到崩溃处的pc地址为`0000000000000680`，执行如下命令：

```
cd C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\toolchains\x86_64-4.9\prebuilt\windows-x86_64\bin
执行命令  x86_64-linux-android-addr2line -e D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64\libhello-jni.so 0000000000000680
------------
如下:
C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\toolchains\x86_64-4.9\prebuilt\windows-x86_64\bin>x86_64-linux-android-addr2line -e D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64\libhello-jni.so 0000000000000680
```

输出结果为：

```
D:/shmily/workSpace/AndroidTestJNI/app/src/main/java/jni/hello-jni.c:6
```

查看`hello-jni.c`文件，发现正确定位到了问题位置。 

### ndk-stack
用来把log信息全部翻译成更加详细的带源码行数信息的log，相当于在整个crash堆栈信息都执行addr2line命令。
log放置路径: 带symbols信息的so文件的地址

```
cd C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\prebuilt\windows-x86_64\bin 
执行命令  ndk-stack -sym D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64 -dump 2.txt > 6.txt
(两个文件2.txt 6.txt 都需放置在这个目录下,其中2.txt为log信息文件)
C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\prebuilt\windows-x86_64\bin>ndk-stack -sym D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64 -dump 2.txt > 6.txt

```

输出结果为 :
```
********** Crash dump: **********
Build fingerprint: 'generic_x86_64/sdk_phone_x86_64/generic_x86_64:5.0.2/LSY66K/4174711:eng/test-keys'
#00 0x0000000000000680 /data/app/org.lwh.test-1/lib/x86_64/libhello-jni.so (Java_org_lwh_test_HelloJNI_stringFromJNI)
                                                                            testCrash
                                                                            D:/shmily/workSpace/AndroidTestJNI/app/src/main/java/jni\hello-jni.c:6:8
                                                                            Java_org_lwh_test_HelloJNI_stringFromJNI
                                                                            D:/shmily/workSpace/AndroidTestJNI/app/src/main/java/jni\hello-jni.c:13:5
Crash dump is completed

```
有代码行数与函数信息.


### objdump
使用addr2line命令我们已经定位到代码出错的位置，但是这种方法只能获取到代码行数，不能显示出函数信息。可使用`objdump`获取到函数信息。执行如下命令：

```
cd C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\toolchains\x86_64-4.9\prebuilt\windows-x86_64\bin
执行命令 x86_64-linux-android-objdump -S D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64\libhello-jni.so > ./hello.asm
/Users/apple/Library/Android/sdk/ndk-bundle/toolchains/x86_64-4.9/prebuilt/darwin-x86_64/bin/x86_64-linux-android-objdump -S /Users/apple/Documents/Space/workspace/AndroidTestJNI/app/src/main/java/obj/local/x86_64/libhello-jni.so > ~/Desktop/hello.asm
如下:
C:\Users\Administrator\AppData\Local\Android\Sdk\ndk\22.1.7171670\toolchains\x86_64-4.9\prebuilt\windows-x86_64\bin>x86_64-linux-android-objdump -S D:\shmily\workSpace\AndroidTestJNI\app\src\main\java\obj\local\x86_64\libhello-jni.so > ./hello.asm

```

`hello.asm`文件内容节选如下：(请参考文件hello.asm)

```
00000000000005f0 <testCrash>:
#include <string.h>
#include <jni.h>

void testCrash() {
    int* p = NULL;
    *p = 1;
 5f0:    0f 0b                    ud2    
 5f2:    66 66 66 66 66 2e 0f     data16 data16 data16 data16 nopw %cs:0x0(%rax,%rax,1)
 5f9:    1f 84 00 00 00 00 00 

0000000000000600 <Java_org_lwh_test_HelloJNI_stringFromJNI>:
 600:    0f 0b                    ud2    
 602:    66 90                    xchg   %ax,%ax
```

在其中按pc值`0000000000000600`可搜索到异常位置。

### 参考
- [Debugging Native Android Platform Code](https://source.android.com/devices/tech/debug)
- [Diagnosing Native Crashes](https://source.android.com/devices/tech/debug/native-crash)


