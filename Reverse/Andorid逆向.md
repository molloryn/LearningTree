
安卓手机基本是ARM架构CPU
Arm汇编与x86汇编的区别



`classes.dex`经过java的dxv虚拟机解析。
java调C语言（.so），托管语言容易反编译？
### Dalvik虚拟机

安卓系统里的Java虚拟机（Dalvik）与传统意义上的Java虚拟机不兼容。

Dalvik 是 google 专门为 Android 操作系统设计的一个虚拟机，经过深度优化，有以下特点：

1.专用的DEX可执行文件格式，体积更小，执行速度更快。 

2.基于寄存器架构，并拥有一套完整的指令系统。

3.提供了对象生命周期管理、堆栈管理、线程管理、安全和异常管理以及垃圾回收等重要功能。

4.所有的Android程序都运行在Android系统进程里，每个进程对应着一个Dalvik虚拟机实例。

### Smail代码
Dalvik虚拟机拥有专属的DEX可执行文件格式和指令集代码。Smali和baksmali则是针对DEX执行文件格式的汇编器和反汇编器，反汇编后DEX文件会产生.smali后缀的代码文件，Smali代码拥有特定的格式与语法，Smali语言是对Dalvik虚拟机字节码的一种解释，它实现了DEX格式所有功能（注解、调试信息、行号等）。

Smali提供反汇编功能的同时，也提供了打包反汇编代码重新生成DEX的功能，因此Smali被广泛地用于APP广告注入、汉化和破解、ROM定制等方面。

### DEX
Dex是Dalvik虚拟机的可执行文件，Dex文件与标准的Class文件在结构设计上有着本质上的区别。标准的Java程序在经过编译后，还需要通过dx工具将在编译过程中所生成的数个Class文件整合成一个Dex文件。Dex文件是传统Jar文件大小的50%左右。并且每个apk文件中包含一个classes.dex，是Dalvik可执行文件。




### ADB
连上安卓系统，就相当于ssh连上linux一样。
ADB 全称 Android Debug Bridge，译作Android 调试桥。 ADB是一种功能多样的命令行工具，可让您与设备进行通信。ADB 命令可用于执行各种设备操作（例如安装和调试应用），并提供对 Unix shell（可用来在设备上运行各种命令）的访问权限。它是一种客户端-服务器程序，对我们以后进行安卓开发与设备系统安装方面有极大的用处。


```sh
adb connect host:port # 连接安卓模拟器
adb device # 列出已连接的设备
adb -s 设备号 shell # 连上shell
```


# 软件
## 脱壳

和PE文件脱壳不一样，PE文件的壳运行完后就解密了整个文件到内存，EOP？
然后dump出来。
安卓壳是加载整个到内存，卡好时间点dump。

### BlackDex
开源软件，脱壳失败可能是安卓版本过高。

### MT管理器

安卓中的app，可以对apk进行重打包、签名。

## Hook
对某个方法或参数进行hook，支持使用frida或xposed进行。
比如可以将某个方法实现变为自己传入的代码片段。
[凡墙总是门 (kevinspider.github.io)](https://kevinspider.github.io/)

### Frida
Frida是个轻量级别的hook框架，是Python API，但JavaScript调试逻辑


## 反编译
### Jadx

反编译apk回java代码，可以以更自然的形式观看代码。
新版本搜索文本可以在资源和注解里面搜索。但是新版本某些apk却无法反编译（jadx问题）。

如果apk没壳，理论上都能反汇编成功。
还有Jeb、GDA，这两个不仅支持反编译java，也支持动态调试。

对于Java调C语言层里面的方法，反编译回的java代码只有方法定义（有`native`关键字），没有具体实现，需要去找到相应的so包进行IDA反汇编（IDA的64位和32位是指反汇编的目标架构）。

![](../attachments/Pasted%20image%2020230904112652.png)

比如这里就是需要去apk压缩包中的lib文件找到相应的so文件（动态链接库）。注意不同架构的so文件可能不太一样。

IDA 还原JNI函数方法名，用于显示调用的具体函数名称
[IDA 还原JNI函数方法名 的三种方法_ida jni_暗夜枭熊的博客-CSDN博客](https://blog.csdn.net/yb493071294/article/details/80378730)




### Andorid Killer

反编译apk回Smali代码，通过Smali语法对apk文件在汇编层面进行更改，修改完成后还可以重新编译、签名回apk文件。
![](../attachments/Pasted%20image%2020230904101145.png)
如果编译报错，比如某png文件报错，可以把那个文件删了（资源文件不影响）

# 代码

入口类（通常是MainActivity）中有OnCreate方法（相当于init）
也可以通过`AndroidManifest.xml`查看。
![](../attachments/Pasted%20image%2020230904095851.png)