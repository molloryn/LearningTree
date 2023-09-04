
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



# 
## Jadx

反编译apk回java代码，可以以更自然的形式观看代码。
新版本搜索文本可以在资源和注解里面搜索。

### Hook
对某个方法或参数进行hook，支持使用frida或xposed进行。
比如可以将某个方法实现变为自己传入的代码片段。
[凡墙总是门 (kevinspider.github.io)](https://kevinspider.github.io/)


## Andorid Killer

反编译apk回Smali代码，通过Smali语法对apk文件在汇编层面进行更改，修改完成后还可以重新编译、签名回apk文件。
![](../attachments/Pasted%20image%2020230904101145.png)
如果编译报错，比如某png文件报错，可以把那个文件删了（资源文件不影响）
### MT管理器

安卓中的app，可以对apk进行重打包、签名。


# 代码

入口类（通常是MainActivity）中有OnCreate方法（相当于init）
也可以通过`AndroidManifest.xml`查看。
![](../attachments/Pasted%20image%2020230904095851.png)