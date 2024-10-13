# Linux易忽略点
## 虚拟设备

### \/dev\/tcp

> 它其实是一个 bash 的 feature，或者说接口，由于是 bash的 feature，因此在别的 shell下就不能生效，所以需要注意使用shell类型。简单来说，设计如此。

`/dev/tcp`  打开这个文件就类似于发出了一个socket调用，建立一个socket连接，读写这个文件就相当于在这个socket连接中传输数据。
我们可以通过重定向实现基于tcp/udp协议的软件通讯，`/dev/tcp/host/port`  只要读取或者写入这个文件，相当于系统会尝试连接`host`这台机器，对应`port`端口。
如果主机以及端口存在，就建立一个socket 连接，将在 `/proc/self/fd` 目录下面，有对应的文件出现。
## 命令的参数顺序

#### 有内容参数
比如 `tar`，你会发现，大家总是将 `f`参数放在最后，因为 `-f <archive file>` 这个参数是必须要接收内容的。
`tar -cvfp xxx xx` 这种顺序就会导致出错，错误的将`p`也当作参数内容。
而需要将 "bool" 型参数放在前面
`tar -cpvf xxx xxx`

## 特殊语法
#### 反引号命令执行 \`\` 
在Linux中，反引号（\`）用于将其中的命令执行结果以参数形式（非字符串）展开（或者说是将结果替换到命令），因此它是最先执行的，展开后才轮到bash去处理整段命令。
```sh
echo `echo 123` # 执行完里面的内容，直接以参数展开，然后变为了echo 123，这时才轮到babsh执行这段命令
echo $(ls) # 除了反引号，也可以使用$()来实现相同的效果，不过不是所有版本linux都支持
```


#### &
放在命令最后是指在后台运行命令，这里也可以看到一个命令就会被当作一个 `process`，可以用 `ps`查看当前执行的指令
#### ;
`echo 123; echo 456 ` 分别执行命令，并一起输出（用`\n`）分隔
`bash`本身在一个会话之中也可以写脚本，类似于python，所以`;`显然就是当作了两个语句。
那么还可以定义函数：`function_name() { ... }`，在当前会话中有效。

**Fork Bomb** `:(){ :|:& };:`也就好理解了，重复创建自身进程耗尽linux的进程资源使电脑死机无法响应，可以通过限制 `ulimit` 限制资源
#### ${}

```sh
a=123 # shell中定义变量，=之间不能有空格
echo $a # $a读取变量(先宏替换再执行命令)
echo aa${a}bb # 其实也类似于$a读取变量，不过在与其他字符串合并的时候可以边界明确

# 还有字符处理的功能
# #是去掉左边（键盘上#在 $ 的左边）  
# %是去掉右边（键盘上% 在$ 的右边）  
# 单一符号#，%是最小匹配；两个符号##，%%是最大匹配
path=/etc/sysconfig/network
echo ${path#*/} # 去掉path变量左边 第一次出现 */ 子串 *代表匹配任意0或0个以上字符 结果：etc/sysconfig/network
echo ${path##*/} # 最大匹配 结果：network
echo ${path%/*}  # 去掉path变量右边匹配的子串 /etc/sysconfig
# 截取
echo ${path:0:5} # 结果：/etc/
echo ${path:3:7} # 结果：c/sysco

# 替换
echo ${path/et/op} # 第一个出现的et替换成op /opc/sysconfig/network 
echo ${path//et/op} # 所有出现的et替换成op /opc/sysconfig/nopwork
```


#### `'`与 `"`的区别


```sh
echo "$PWD"
echo '$PWD'
echo '`echo 123`'
```
可以发现，`'`可以将内部直接转义为纯字符串。


#### {}
可以在terminal中多行执行命令，而无需编写shell脚本

```sh
{
  command1
  command2
  command3
}

# 也可使用
command1 && \
command2 && \
command3

```


## 文件描述符 (File Descriptor)

[Linux反弹shell（一）文件描述符与重定向 | K0rz3n's Blog](https://www.k0rz3n.com/2018/08/05/Linux%E5%8F%8D%E5%BC%B9shell%EF%BC%88%E4%B8%80%EF%BC%89%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6%E4%B8%8E%E9%87%8D%E5%AE%9A%E5%90%91/)

> Linux中一切皆文件（我更希望理解为对象），因此下面所讲述的文件可以指资源、文件、Socket连接、设备等等
> 可以理解为linux跟踪打开文件，而分配的一个数字，这个数字有点类似c语言操作文件时候的句柄（Handle），通过句柄就可以实现文件的读写操作。

当Linux启动的时候会默认打开三个文件描述符，分别是：

标准输入standard input 0 （默认设备键盘）  
标准输出standard output 1（默认设备显示器）  
错误输出error output 2（默认设备显示器）
默认指向到`/dev/tty0`

默认一个会话下只有三个已定义的文件描述符，可以通过`exec`将某个文件添加为新的文件描述符。

文件所有输入输出都是由打开的文件描述符控制的。

一条命令执行以前先会按照默认的情况进行绑定（也就是上面所说的 0,1,2），如果我们有时候需要让输出不显示在显示器上，而是输出到文件或者其他设备，那我们就需要重定向。

[Linux系统：/dev/tty、/dev/tty0 和 /dev/console之间的区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/632099551)
#### 重定向符号

输入重定向 `<` `<<`
输出重定向 `>` `>>`

1. bash 在执行一条指令的时候，首先会检查命令中存不存在重定向的符号，如果存在那么首先将文件描述符重定向（之前说过了，输入输出操作都是依赖文件描述符实现的，重定向输入输出本质上就是重定向文件描述符），然后再把重定向去掉（意思是先解析完所有的重定向指令），再执行指令
2. 如果指令中存在多个重定向，那么不要随便改变顺序，因为重定向是从左向右解析的，改变顺序可能会带来完全不同的结果
3. `<` 是对标准输入 0 重定向 ，`>` 是对标准输出 1 重定向
4. `>`和`<`与文件描述符之间不能有空格，但是可以和普通文件有空格。而且右边的文件描述符为了与文件区分，会与 `&`结合，比如`&1`。
5. 使用了重定向符号的shell命令，只对那一条命令有效。


##### n> file 和 n>&m
将指定文件描述符`n`重定向到指定文件，`n`可以为空，默认是相应的标准输入输出文件描述符。
`>`就是将左边输出重定向到指定地点输出（即以写方式打开文件）
`<`相应的是将左边输入重定向到从指定地点输入（即以读方式打开文件）
`n>&m` 将文件描述符`n`输出重定向给文件描述符`m`

##### &> 和 >&
将标准输出与标准错误输出都定向到指定文件（以写的方式打开），两种格式意义完全相同。
这种格式等价于 `>file 2>&1` 
`2>&1` 是将标准错误输出复制到标准输出（此时的标准输出已经重定向到file了），`&`是为了区分文件1和文件描述符1的。

At first, `2>1` may look like a good way to redirect `stderr` to `stdout`. However, it will actually be interpreted as "redirect `stderr` to a file named `1`".

`&` indicates that what follows and precedes is a _file descriptor_, and not a filename. Thus, we use `2>&1`. Consider `>&` to be a redirect merger operator.

https://stackoverflow.com/a/40319372

##### 绑定文件描述符到文件
`exec 3<>file` 绑定文件描述符3到file
那么后续的操作，如 `echo 123 >&3` 则会定向到file。



##### 找到进程拥有的文件描述符
[Investigating Linux Process File Descriptors for Incident Response and Forensics (linkedin.com)](https://www.linkedin.com/pulse/investigating-linux-process-file-descriptors-incident-craig-rowland#:~:text=A%20file%20descriptor%20is%20a,a%20pointer%20to%20a%20file.)
可以使用`ls -al /proc/<PID>/fd`找到。其中`fd`就是File Descriptor
![](../../attachments/Pasted%20image%2020230907154606.png)

##### 示例
- `>`：将命令的输出`覆盖写入`到指定文件，如果文件不存在则创建文件。
- `>>`：将命令的输出`追加写入`到指定文件，如果文件不存在则创建文件。
- `2>`：将命令的错误输出覆盖写入到指定文件。
- `2>>`：将命令的错误输出追加写入到指定文件。
- `&>` 或 `&>>`：将命令的标准输出和错误输出合并后写入到指定文件。
```sh
echo "How are you?" >> output.txt # 命令的输出追加到文件末尾
echo "Error occurred." 2> error.txt # 错误输出写入文件
```

