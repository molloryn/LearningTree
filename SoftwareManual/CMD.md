# CMD

## PowerShell

### 安装SSH Server以支持SSH连接

```powershell
# 使用PowerShell以管理员权限执行，查看安装状态
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
# 安装Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 初始化ssh服务器
net start sshd
# 或
Start-Service sshd

# 设为开机自启
Set-Service -Name sshd -StartupType 'Automatic'
# This is a PowerShell command that retrieves all firewall rules that have "ssh" in their name.
Get-NetFirewallRule -Name *ssh*

# 卸载
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 添加防火墙规则
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH SSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 -Program "C:\Windows\System32\OpenSSH\sshd.exe"

# ssh客户端生成ssh公钥与私钥，以使用无密码连接，将生成的公钥放到ssh服务端
ssh-keygen -q -b 2048 -P "" -f <hostname>_rsa -t rsa
# 具体操作： https://stackoverflow.com/a/69970152
```

仍然无效的可以使用如下方法安装
[Manually install OpenSSH in Windows Server](https://www.saotn.org/manually-install-openssh-in-windows-server/)

### 获取CPU温度

```powershell
$data = Get-WMIObject -Query "SELECT * FROM Win32_PerfFormattedData_Counters_ThermalZoneInformation" -Namespace "root/CIMV2"

@($data)[0].HighPrecisionTemperature
```

### 临时无限制模式

`pwsh -ExecutePolicy Unrestricted`

在当前会话有效

### 获取电池寿命报告

`powercfg batteryreport output "D:\\battery_report.html"`

### 计算文件sha256

`Get-FileHash filePath`

`CertUtil -hashfile {PATH AND FILE NAME} {SHA256|MD5}`

 ### 同时使用WIFI与有线网络

首先使用`route print`查看路由表。
路由表有顺序优先级，越上面的优先级越高。
`0.0.0.0`指代所有目标？
![](../attachments/Pasted%20image%2020240616173732.png)
其中`192`部分为`WIFI`网关，`188`为有线网关（内网）。此时所有流量先走有线网，导致上不了外网。
而这种原因可能是配置了`DHCP`导致的，因此如果进行手动配置，填写好掩码、网关，则可以将优先级下降。
另外还有一种方式可以增加：
`route add 188.22.76.0 mask 255.255.255.0 188.22.77.1 -p`

> 将访问 188.22.76.0-188.22.76.255 网段的所有数据，通过 188.22.77.1 这个网关进行转发，其中`-p`是永久生效，默认是拔网线重启后就失效。

### 端口转发

`netsh interface portproxy add v4tov4 listenaddress=10.0.10.21 listenport=8081 connectaddress=192.168.33.111 connectport=8081`

`netsh interface portproxy` 表示端口映射列表

`add v4tov4` 表示添加的是`IPV4`到`IPV4`的端口

`listenaddress` 表示侦听的ip地址，填的是映射方

`listenport` 侦听的端口，可以与被映射的端口设置成不一样

`connectaddress` 被映射方（连接方）的ip地址

`connectport` 被映射方的端口

`netsh interface portproxy delete v4tov4 listenaddress=10.0.10.21 listenport=8081`

`netsh interface portproxy show all` 查看存在的转发

注意，浏览器出现`ERR_UNSAFE_PORT`错误不是因为服务器那边，而是浏览器本身定义了内置端口，这些端口是无法被使用的：

```text
1, // tcpmux
7, // echo
9, // discard
11, // systat
13, // daytime
15, // netstat
17, // qotd
19, // chargen
20, // ftp data
21, // ftp access
22, // ssh
23, // telnet
25, // smtp
37, // time
42, // name
43, // nicname
53, // domain
77, // priv-rjs
79, // finger
87, // ttylink
95, // supdup
101, // hostriame
102, // iso-tsap
103, // gppitnp
104, // acr-nema
109, // pop2
110, // pop3
111, // sunrpc
113, // auth
115, // sftp
117, // uucp-path
119, // nntp
123, // NTP
135, // loc-srv /epmap
139, // netbios
143, // imap2
179, // BGP
389, // ldap
465, // smtp+ssl
512, // print / exec
513, // login
514, // shell
515, // printer
526, // tempo
530, // courier
531, // chat
532, // netnews
540, // uucp
556, // remotefs
563, // nntp+ssl
587, // stmp?
601, // ??
636, // ldap+ssl
993, // ldap+ssl
995, // pop3+ssl
2049, // nfs
3659, // apple-sasl / PasswordServer
4045, // lockd
6000, // X11
6665, // Alternate IRC
6666, // Alternate IRC
6667, // Standard IRC
6668, // Alternate IRC
6669, // Alternate IRC
```

## Windows

### 反复异常蓝屏

可能是更新驱动导致系统文件损坏，可通过命令尝试修复

```powershell
sfc /scannow
# sfc是检查所有系统文件并将组件存储（Component Store，存放在WinSxS文件夹下）里的已签名文件替换掉被破坏的系统文件。
```

或使用

```powershell
Dism /Online /Cleanup-Image /ScanHealth
Dism /Online /Cleanup-Image /CheckHealth
Dism /Online /Cleanup-Image /RestoreHealth
```

### 解除文件关联

```cmd
assoc.mp4 # 获取mp4关联的文件类型
# .mp4=WMP11.AssocFile.MP4
ftype | findstr "MP4" # 查找文件类型关联的打开程序
# PotPlayerMini64.MP4="D:\Program Files\DAUM\PotPlayer\PotPlayerMini64.exe" "%1"
# 通过下面两条置空
assoc.mp4=
ftype PotPlayerMini64.MP4=
```

### 入域

计算机名联系部门相关人员，应是一人一计算机名。
确认入域时，会提醒输入域账号密码验证，这时候如果这个计算机名没有关联到域账号，那会显示找不到路径之类的错误提示，这时候要联系域管理员关联上，才能使用域账号密码登陆。

![](../attachments/Pasted%20image%2020230726170452.png)

### 虚拟机桥接联网

有线一根，连接后虚拟机上连接该有线网卡，然后在虚拟机适配器中开启连接共享（虚拟机设置NAT模式即可），如果之前已经开了共享，那需要**重新开启共享**，不然有线网卡无法加入。
如果偶尔会掉线，则需要通过ping 命令一直ping对端PC。刷新arp路由表。


### 游戏强制全屏的窗口化

`Alt+Enter`

#### 分屏窗口移动
使用键盘快捷键（通用但可能因软件限制而效果不同）
Alt + Space（打开窗口菜单）然后 M（移动）
首先按下Alt + Space组合键，这会打开当前活动窗口的系统菜单。这个菜单包含了诸如还原、最小化、最大化等操作选项。
接着按下M键，此时鼠标指针会变成一个四向箭头，表示进入了窗口移动模式。
然后你可以使用方向键（上、下、左、右）来移动窗口。按一下方向键，窗口就会朝相应的方向移动一个像素单位。如果一直按住方向键，窗口会持续移动。
当你把窗口移动到可以看到的屏幕区域后，再使用鼠标来精确调整窗口位置。

Windows 键 + 左 / 右方向键
如果你的双屏是水平排列的，使用Windows + 左方向键可以将窗口移动到左边的屏幕（如果窗口在右边屏幕），Windows + 右方向键可以将窗口移动到右边的屏幕（如果窗口在左边屏幕）。不过这种方法在另一个屏幕无法正常显示的情况下可能无法准确操作，因为系统可能会根据屏幕状态做出不同反应。但在一些情况下，它可以帮助你快速切换窗口所在的屏幕位置，你可以多尝试几次看看是否能将窗口移动到可见的屏幕区域。

#### 应用自启动最小化

`start /min D:\Euynac\Dictionary\GoldenDict\GoldenDict.exe`

保存为`.bat`

cmd运行`shell:startup`打开自定义自启动列表放入该bat文件

`start "" /B /D "D:\Desktop\bongo_cat_mver_0.1.6_64" "Bongo Cat Mver.exe"`

`start`的参数可以看帮助文件学习, 主要的坑就是第一个参数. 空引号是用来指定新窗口的标题。在 `start` 命令里，第一个引号内的内容会被当作新窗口的标题。若省略这部分，后续带引号的程序路径会被错误识别为窗口标题。所以使用空引号能避免这种情况，让命令正确识别后续的程序路径。

#### win11启动组策略gpedit.msc

```powershell
@echo off

pushd "%~dp0"

dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt

dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt

for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"

pause
```

#### 解决UTF8编码问题

`CHCP 65001`

解决python乱码问题：python等项目多采用使用`setenv.bat`文件去临时创建一个运行环境，在其中添加：

```sh
export PYTHONUTF8=1  # linux / macOS
set PYTHONUTF8=1  # windows
```

千万注意`bat`文件中使用`set xxxx=xxxx`时，结尾不要含有空格，否则也会当作传入的值。

#### 关闭/启用默认使用管理员权限打开程序

程序所在的exe右键属性中，可以控制是否默认以管理员权限打开。

#### 修复系统损坏文件

管理员模式运行

`sfc /scannow`

#### 换回win10右键

切换到旧版右键菜单：

``` sh
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```

恢复回Win11右键菜单：

```sh
reg delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}" /f
```

重启Windows资源管理器生效：

`taskkill /f /im explorer.exe & start explorer.exe`

#### 自启动程序

文件位置打开后，按 Windows 徽标键 + R，键入`shell:startup`，然后选择"确定"。这将打开"启动"文件夹。

将该应用的快捷方式从文件位置复制并粘贴到"启动"文件夹中。

#### WIFI密码

`netsh wlan show profiles` 查看所有连接过的WiFi配置

`netsh wlan show profiles "TP-LINK_5G_4926" key=clear`  查看配置文件

#### 查看文件md5等值

Certutil.exe作为证书服务的一部分安装。您可以使用Certutil.exe转储和显示证书颁发机构（CA）配置信息，配置证书服务，备份和还原CA组件，以及验证证书，密钥对和证书链。

```sh
1.  certutil -hashfile filename MD5
2.  certutil -hashfile filename SHA1
3.  certutil -hashfile filename SHA256
```

#### 快速切换到指定文件夹

文件资源管理器直接打开那个文件夹然后在url地址栏填上cmd打开

#### 强行删除权限不足的文件（需要管理员权限）

```bash
set path="XX"

takeown /F %path% /r /d y

cacls %path% /t /e /g Administrators:F

rd /s /q $path$
```

#### 打开cmd当前目录

```sh
1.  explorer .
2.  explorer %cd%
3.  start .
4.  start %cd%
```

#### Windows键失效

尝试`FN+Windows`键一起连续按多几次（十几次），解锁win键

#### 不重启使环境变量修改生效

以修改环境变量"PATH"为例，修改完成后，进入DOS命令提示符，输入：`set PATH=C:` ，关闭DOS窗口。再次打开DOS窗口，输入：`echo %PATH%` ，可以发现PATH 值已经生效。

DOS窗口必须重启后环境变量才有效。

#### 以树形形式展示目录结构

`tree`

#### 判断程序是否以管理员运行

任务管理器中，详细信息栏，选择列，选择开启"特权"一栏。

#### Windows服务启动类型无法修改，为灰色

可以尝试使用命令修改：`sc config "Muse Hub Background Service(服务属性中的服务名称)" start=demand`
如果报错拒绝访问，可能是因为权限不足。即使是管理员权限，也无法修改。其实还有一个电脑的最高特殊权限，所以还可以通过这个软件：`M2-Team NSudo Launcher`[GitHub - M2TeamArchived/NSudo: [Deprecated, work in progress alternative: https://github.com/M2Team/NanaRun] Series of System Administration Tools](https://github.com/M2TeamArchived/NSudo) （不过需要下载Preview版本，8.2版本会token生成错误）来启动 `services.msc`来修改。

如果还是为灰色，那就说明这是个流氓软件，只能通过注册表修改了：
参考路径：`计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\<服务名称>`
里面注册表项`Start`即为启动类型：`1 - 自动延时启动 2 - 自动 3 - 手动 4 - 禁用`。修改后重启电脑即可。
还有个方案是可以将注册表中的`Security`（有这个说明控制了权限）随便改个名字，让它无效，这样就可以手动控制启动类型了。

### 端口占用问题

Hyper-V 会保留部分tcp端口，开始到结束范围内的端口不可用, 使用如下命令查看保留的端口：

`netsh interface ipv4 show excludedportrange protocol=tcp`

保留的端口是随机的，每次重启电脑都会改变，因此可以通过重启电脑来解决。

除了重启电脑，也可以运行`net stop winnat`停止 winnat 服务，然后再运行`net start winnat`启动 winnat 服务。

让Hyper-V再随机初始化一些端口保留，如果正好没随机到要用的端口，那一次成功。如果还是随机到了要用的端口，那就只能多来几次。

也可以永久排除保留端口：

①在运行 Docker 之前，以管理员身份运行 powershell

②使用以下命令永久排除6379作为保留端口(如果端口被占用需要重启一次电脑)

`netsh int ipv4 add excludedportrange protocol=tcp startport=6379 numberofports=1 store=persistent`

提示：关键在于`store=persistent`参数表示持久化信息

**上面的命令可以通过修改numberofports参数保留startport开始的多个端口**

③再次运行 `netsh interface ipv4 show excludedportrange protocol=tcp` 命令可以看到6379端口已被排除(带有*号标记)

还有一种办法是重新设置一下「TCP 动态端口范围」，让Hyper-V只在我们设定的范围内保留端口即可。可以以管理员权限运行下面的命令，将「TCP 动态端口范围」重新设定为`49152-65535`。如果你觉得这个范围太大，还可以改小一点。

```sh
netsh int ipv4 set dynamic tcp start=49152 num=16384
netsh int ipv6 set dynamic tcp start=49152 num=16384
```

然后重启电脑即可。

重启电脑后，再运行命令`netsh int ipv4 show dynamicport tcp`查看动态端口范围，发现确实已经修改为了`49152-65535`。

### 网络问题

#### 对于受限网络WIFI会自动断开

To disable the Windows Network Connectivity Status Indicator (NCSI) active test, use the Group Policy Editor (`gpedit.msc`) to enable the "Turn off Windows Network Connectivity Status Indicator active tests" setting. Navigate to `Computer Configuration` > `Administrative Templates` > `System` > `Internet Communication Management` > `Internet Communication settings` and change the policy to Enabled. After making the change, restart your computer or run `gpupdate /force` to apply it.

#### 使用代理

- **Windows Settings Proxy**: The proxy you configure in _Settings → Network & Internet → Proxy_ applies to apps using the WinINET API (like Edge, Internet Explorer, or apps that rely on system networking).
- **CMD & Console Tools**: Many command-line tools (like `curl`, `git`, `npm`) rely on **WinHTTP** or their own proxy settings, not WinINET. That’s why CMD doesn’t automatically use the proxy you set in Windows Settings.
- Some tools respect environment variables: (like `curl`)
```sh
set http_proxy=http://proxyserver:port
set https_proxy=http://proxyserver:port
```



## 设置Alias

[Doskey | Microsoft Learn](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-xp/bb490894(v=technet.10)?redirectedfrom=MSDN)

`doskey npd=notepad $2 $1`

即`npd aaa bbb = notepad bbb aaa`

`doskey prt=echo $*`

即 `prt xxx xxx xx = echo xxx xxx xx`

`doskey newline=echo $1 $T echo $2`

其中`$t`或`$T`是新的一行，即多执行下一行。

`doskey /macros` 显示设置的宏定义

`doskey macroname=` 删除宏定义（将已设置的宏设置为null）

但是注意，它是临时的，需要做如下操作变成永久：

[command line - Create permanent DOSKEY in Windows cmd - Super User](https://superuser.com/questions/1134368/create-permanent-doskey-in-windows-cmd)

## 文件操作

注意批处理`*.bat` 读取文件是用`ANSI`编码的，所以如果是用的`UTF-8`编码之类的，读取中文之类会乱码。因此需要先转`ANSI`编码，具体可以使用文本编辑器里的编码选项中进行转换文件的编码。

`cd`命令只能在当前盘符使用。切换硬盘直接输入`d:`、`e:`等。

## 网络

| 命令                                  | 参数                                                                                                                                       | 简介                                                                                                                                                    |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| netstat                             | -a 显示一个所有的有效连接信息列表，包括已建立的连接（ESTABLISHED），也包括监听连接请求（LISTENING）的那些连接。 -b 显示在创建每个连接或侦听端口时涉及的可执行程序。 -n 以数字形式显示地址和端口号。 -o 显示拥有的与每个连接关联的进程 ID。 | net status 显示网络连接、路由表和网络接口信息，可以让用户得知有哪些网络连接正在运作。 不带参数则是显示活动的TCP连接 netstat -aon\|findstr "80"（以80端口为例） 其他用例： 查看某个端口具体被那个应用占用 tasklist \| findstr "PID" |
| set http_proxy set https_proxy      | =socks5://127.0.0.1:1080 或http也可以                                                                                                        | CMD设置临时代理（永久需要在环境变量中设置）可以用curl测试，不能使用ping。                                                                                                            |
| netsh interface ipv4 show neighbors |                                                                                                                                          | 查看局域网邻居ip                                                                                                                                             |
| tracert                             |                                                                                                                                          | 路由跟踪，指示到目标地址经过的路由IP                                                                                                                                   |
|                                     |                                                                                                                                          |                                                                                                                                                       |
|                                     |                                                                                                                                          |                                                                                                                                                       |

### 调整网络优先级

通过 PowerShell 命令调整网络适配器优先级

1. 查看网络适配器信息
  `Get-NetIPInterface | Where-Object { $_.InterfaceAlias -match 'Wi-Fi|以太网' } | Select-Object InterfaceAlias, InterfaceIndex, AddressFamily, InterfaceMetric`
  
  
2. 调整接口优先级
    - 找到无线网络和有线网络的接口索引（InterfaceIndex）
    - 降低无线网络的接口跃点数（值越小优先级越高）

  设置无线网络优先级（假设接口索引为12）
  `Set-NetIPInterface -InterfaceIndex 12 -InterfaceMetric 10`

  设置有线网络优先级（假设接口索引为15，值要大于无线网络）
 `` Set-NetIPInterface -InterfaceIndex 15 -InterfaceMetric 20`


# 注册表

#### 控制启动服务

有些服务`services.msc`中调整启动类型的选项是灰色的，可以通过修改注册表强行更改。

注册表位置：`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\`

找到服务，其中`Start`值就是目标。

**值=0**，默认开机bios启动而启动

**值=1**，跟随操作系统启动

**值=2**，自动启动

**值=3**，手动启动

**值=4**，禁用

#### 设备管理器注册表

设备管理器中，右键属性-详细信息可以查看驱动或硬件的注册表值，可以通过其中的`驱动程序关键字`（英文`Driver Key`）去搜索值找到对应驱动注册表的位置。

#### 驱动文件更换（如TypeC耳机驱动问题）

以H180 Plus这款TypeC耳机为例：
![](../attachments/Pasted%20image%2020230803190424.png)
上图是已经更换了旧驱动，原本新驱动无法使用，只能识别为麦克风，更换后正常。

更换驱动需要对应的`*.inf`文件

查看该设备对应的驱动：
![](../attachments/Pasted%20image%2020230803191354.png)

旧驱动取自一台windows10电脑。
使用powershell命令`Get-WindowsDriver -Online -Driver "wdma_usb.inf"` 查看驱动信息。

```txt
Driver              : wdma_usb.inf
OriginalFileName    : C:\Windows\System32\DriverStore\FileRepository\wdma_usb.inf_amd64_63b279bcecfa8b38\wdma_usb.inf
Inbox               : True
ClassName           : MEDIA
ClassDescription    : 声音、视频和游戏控制器
ClassGuid           : {4D36E96C-E325-11CE-BFC1-08002BE10318}
BootCritical        : False
ProviderName        : Microsoft
Date                : 7/6/2023 12:00:00 AM
Version             : 10.0.22621.1992
ManufacturerName    : Yamaha
HardwareDescription : Yamaha USB MIDI
Architecture        : x64
HardwareId          : USB\VID_0499&PID_1FFF
ServiceName         : usbaudio
CompatibleIds       :
ExcludeIds
```

这里可以得到驱动文件所在的地址
`OriginalFileName    : C:\Windows\System32\DriverStore\FileRepository\wdma_usb.inf_amd64_63b279bcecfa8b38\wdma_usb.inf`

将文件夹从旧电脑拷贝出来，放到本机电脑，然后对指定设备替换驱动。
`更新驱动程序`->`浏览我的电脑以查找`->`让我从计算机可用驱动列表选取`->`从磁盘安装`->选择对应inf文件。
这里可能会出现该文件没有经过数字签名，需要临时禁用这个安全机制。
`Shift`+开始菜单的重启按钮，进入疑难解答界面，接着`高级选项`->`启动设置`->`重启`，然后可以选择`禁用驱动程序强制签名`，电脑这回重启后可以安装未签名的驱动了，按上面步骤安装驱动后重启后，耳机就可以用了。

### 硬件ID

![](../attachments/Pasted%20image%2020240309124642.png)
在各种属性信息中，硬件 Id 是最为关键的一组信息，我们可以看到鼠标这个设备的 VID 为 0000，PID 为 3825。其中，VID 是指 Vendor ID，即：供应商识别码；PID 是指 Product ID，即：产品识别码。事实上，所有的 USB 设备都有 VID 和 PID，VID 由供应商向 USB-IF 申请获得，而 PID 则由供应商自行指定，计算机正是 VID、PID 以及设备的版本号来决定加载或者安装相应的驱动程序。

> 注意他们是十六进制。
# 批处理基础

## 注释

```bash
:: 注释
rem 注释
```

## 输入与输出

#### \>与\>\>

```sh
ping sz.tencent.com > a.txt
ping sz1.tencent.com >> a.txt
```

将前面命令的内容输入到a.txt文件中。

区别是`>`会覆盖文本中的内容，`>>`是追加写入到文件中。

`<`，输入重定向命令，从文件中读入命令输入，而不是从键盘中读入。

`>&`，将一个句柄的输出写入到另一个句柄的输入中。

`<&`，刚好和`>&`相反，从一个句柄读取输入并将其写入到另一个句柄输出中。

## 标识符

DOS不支持长文件名，所以就出现了`Tempor~1`

一般可以在`cmd`使用`help xx`来看帮助

| 标识符               | 作用                                                                                                                                                                                                                                                                                                                                                               |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| @echo off            | echo off 可以关闭命令执行一行时显示本身的效果，即只会输出命令执行的结果。 比如如果有echo hello这行命令，执行时候会是 \>\>echo hello hello 如果使用了echo off ，那么echo hello也没有了 @echo off则连同echo off该命令也不显示。 常与\>、\>\>使用，输出echo结果到某文件中 在命令后跟  1\>nul 不显示命令运行的正确提示 2\>nul 不显示错误的提示 \>nul 屏蔽命令输出内容  |
| ::                   | **注释**，不会执行                                                                                                                                                                                                                                                                                                                                                 |
| pause                | 暂停。是让当前程序进程暂停一下，并显示一行信息：请按任意键继续. . .                                                                                                                                                                                                                                                                                                |
| goto xxx 与 :xxx     | goto是个跳转命令，:xxx是一个标签。当程序运行到goto xxx时，将自动跳转到:xxx定义的部分去执行了                                                                                                                                                                                                                                                                       |
| %n(数字)             | 批参数 可以用于获取执行批处理时用户输入的参数 比如echo %2 %1 %3，保存为a.bat，执行a.bat 1 2 3，将会输出2 1 3 而%0是输出调用bat时候的前缀，比如上面a.bat 1 2 3，会输出a.bat 批脚本里的 %\* 是指出所有的参数(如 %1 %2 %3 %4 %5 ...) 批参数(%n)的替代已被增强 %\~1 - 删除引号(")，扩展 %1 %\~f1 - 将 %1 扩展到一个完全合格的路径名 %\~d1 - 仅将 %1 扩展到一个驱动器号 |
| call                 | 从批处理程序调用另一个批处理程序。                                                                                                                                                                                                                                                                                                                                 |
| type filename        | 显示文本文件的内容。                                                                                                                                                                                                                                                                                                                                               |
| \|                   | 管道符。让前一命令的输出当做后一命令的输入。                                                                                                                                                                                                                                                                                                                       |
| more                 | 逐屏显示输出。                                                                                                                                                                                                                                                                                                                                                     |
| command & command    | 用来连接n个DOS命令，并把这些命令按顺序执行，而不管是否有命令执行失败。copy a.txt b.txt /y & del a.txt 其实这句和move a.txt b.txt的效果是一样的                                                                                                                                                                                                                     |
| command && command   | （且）把它前后两个命令组合起来当一个命令来用，与&命令不同之处在于，它在从前往后依次执行被它连接的几个命令时会自动判断是否有某个命令执行出错，一旦发现出错后将不继续执行后面剩下的命令。                                                                                                                                                                            |
| command \|\| command | （或）前一个命令执行成功便不执行后面的命令 连续使用&&和\|\|，优先级&&比\|\|高                                                                                                                                                                                                                                                                                      |
| %var%                | 获取变量var的值，使用set var=xxx设定的局部变量，等号两边不能有空格                                                                                                                                                                                                                                                                                                 |

## IF 条件判断

```bash
IF [NOT] ERRORLEVEL number do command
IF [NOT] string1==string2 do command
IF [NOT] EXIST filename do command
```

cmd中输入`help if`可以看到详细解释

`ERRORLEVEL number` ：如果最后运行的程序返回一个等于或大于指定数字的退出代码，指定条件为 `true`。(环境变量`errorlevel`的初始值为0，表示命令执行成功。当一些命令执行不成功，就会返回一个数值，如：1 ，2，需要具体查命令失败返回的数字是什么意思)

#### ELSE

ELSE 子句必须出现在同一行上的 IF 之后。例如:

```bash
IF EXIST filename. (
del filename.
) ELSE (
echo filename. missing.
)

```

#### 比较数值

```bash
set /a num1=20
set /a num2=15
if %num1% gtr %num2% echo %num1%大于%num2%
if %num1% EQU %num2% echo %num1%等于%num2%
if %num1% LSS %num2% echo %num1%小于%num2%
```

比较运算符一览：
EQU - 等于
NEQ - 不等于
LSS - 小于
LEQ - 小于或等于
GTR - 大于
GEQ - 大于或等于

## FOR 循环语句

在cmd窗口中：`for %I in (command1) do command2`
在批处理文件中：`for %%I in (command1) do command2`

`for`, `in`, `(command1)`, `do`  不可省略；`command1`表示字符串或变量，`command2`表示字符串、变量或命令语句

在cmd窗口中，`for`之后的形式变量I必须使用单百分号引用，即`%I`；而在批处理文件中，引用形式变量I必须使用双百分号，即`%%I`

#### for中的形式变量i

`for`语句的形式变量I，可以换成26个字母中的任意一个，这些字母会区分大小写，也就是说，`%%I`和`%%i`会被认为不是同一个变量；形式变量I还可以换成其他的字符，但是，为了不与批处理中的`%0`～`%9`这10个形式变量发生冲突，请不要随意把`%%I`替换为`%%0` ～`%%9`中的任意一个

#### command1（in）

`in`和`do`之间的`command1`表示的字符串或变量可以是一个，也可以是多个，每一个字符串或变量，我们称之为一个元素，每个元素之间，用空格键、跳格键、逗号、分号或等号分隔；（即一个list中的元素）

`for`语句依次提取`command1`中的每一个元素，把它的值赋予形式变量I,带到do后的`command2`中参与命令的执行；其实就是类似`C#`中的`foreach var i in command1`

### for /f （设计用于解析文本）

`for /f %%i in (test.txt) do echo %%i & pause`

`for /f` 语句是**以行为单位**处理文本文件的

#### delims= （切分字符串）

`for /f "delims=，" %%i in (test.txt) do echo %%i`

将会获取test.txt中每行被`，`分割的前面的字符串作为in中的list元素（`，`及`，`后面的内容被省略），`delims`可以设定多个分隔符，但`for /f` 只会提取分隔符分割每行字符串后的第一组元素

需要注意的是：如果没有指定`"delims=符号列表"`这个开关，那么，`for /f` 语句默认以空格键或跳格键作为分隔符号

#### tokens= （定点提取）

因为`delims`分割后提取的总是第一组字符串内容，如果要提取其他组的字符串，需要使用`tokens`。

`tokens=` 后面一般跟的是数字，如 `tokens=2`，也可以跟多个，但是每个数字之间用逗号分隔，如 `tokens=3,5,8`，它们的含义分别是：提取第2节字符串、提取第3、第5和第8节字符串。注意，这里所说的"节"，是由 `delims=` 这一开关划分的，它的内容并不是一成不变的。

`tokens=1-5`，提取1-5节的内容，需要用5个变量接收

`tokens=1,*` 仅切分一次字符串成两组（因为只提取第一组内容），剩下的作为整体（且没有被切分），需要两个变量接收

##### 需要接收多个组的情况

`for /f "delims=， tokens=2,5" %%i in (test.txt) do echo %%i %%j`

如果单独使用`%%i`，会发现只是得到第二组的字符串，第五组的没有。

`for /f`对该情况做了规定，如果变量是`%%i`接收的第一个数字指代的内容，那么用`%%j`接收第二个数字、`%%k`接收第三个数字。比如如果是用的`%%x`，则`%%y`就是第二个数字。

#### skip=n （跳过n行内容）

很多时候，有用的信息并不是贯穿文本内容的始终，而是位于第N行之后的行内，为了提高文本处理的效率，或者不受多余信息的干扰，`for /f` 允许你跳过这些无用的行，直接从第N+1行开始处理，这个时候，就需要使用参数 `skip=n`，其中，n是一个正整数，表示要跳过的行数。

`for /f "skip=2" %%i in (test.txt) do echo %%i`

这段代码将跳过头两行内容，从第3行起显示test.txt中的信息。

#### eol= （忽略以给定单个字符开头的行不处理）

`for /f "eol=;" %%i in (test.txt) do echo %%i`

忽略`;`开头的行

`eol`本意应该是end of line，本应该是忽略某字符结尾，可能是微软bug

`for /f` 语句是**默认忽略以分号打头的行**内容的，正如它默认以空格键或跳格键作为字符串的切分字符一样。

如果要取消这个默认设置，可选择的办法是：

1、为`eol=`指定另外一个字符；

2、使用 `for /f "eol="` 语句，也就是说，强制指定字符为空，就像对付`delims=`一样。

#### usebackq

一般来说`for /f` 的用法就是1、2、3种，1就是使用文件作为输入的时候，2就是命令运行结果作为输入的，3是字符串作为输入。

但如果第一种中，文件名是有空格或&等特殊符号的时候，使用第一种写法就会出错。这时候就使用`usebackq`来拓展第一种文件输入方式变为""的，5、6相应拓展2、3

```sh
1、for /f %%i in (文件名) do (……)
2、for /f %%i in ('命令语句') do (……)
3、for /f %%i in ("字符串") do (……)
4、for /f "usebackq" %%i in ("文件名") do (……)
5、for /f "usebackq" %%i in (`命令语句`) do (……)
6、for /f "usebackq" %%i in ('字符串') do (……)
```

### 变量延迟

```sh
@echo off
set num=0
for /f %%i in ('dir /a-d /b *.exe') do (
    set /a num+=1
    echo num 当前的值是 %num%
)
```

批处理中的一般使用的变量和编程语言的变量不太一样。批处理是自上而下逐条执行，而且所谓的一条也不是指一行，比如`if…else`、`for`、管道符、连接符（`&`、`&&`、`||`）中一块代码整体才被当做一条。

批处理的预处理机制也是一条一条处理的，分析完第一条`set num = 0`后，然后分析整体的`for`循环语句，检测语法错误等，其中`%num%`在num已经有赋值，所以可以被替换。而`num+=1`虽然确实有用，但num值已经被替换掉了，所以结果一直都是0。

这种替换在一条内部就确定变量的值了，所以如果让这种特性"延迟"，那么num变量就能变化了（即暂不替换变量），因此有了延迟变量。有两种用法：

① 使用 `setlocal enabledelayedexpansion` 语句：在获取变化的变量值语句之前使用`setlocal enabledelayedexpansion`，并把原本使用百分号对闭合的变量引用改为使用感叹号对来闭合；

```
set num=0
setlocal enabledelayedexpansion
for /f %%i in ('dir /a-d /b *.exe') do (
    set /a num+=1
    echo num 当前的值是 !num!
)
```

② 使用 `call` 语句：在原来命令的前部加上 `call` 命令，并把变量引用的单层百分号对改为双层。

```
for /f %%i in ('dir /a-d /b *.exe') do (
    set /a num+=1
    call echo num 当前的值是 %%num%%
)
```

### for /r （遍历文件夹）

其中`r`是recursive，递归的

`for /r [目录] %%i in (元素集合) do 命令语句集合`

1. 列举"目录"及该目录路径下所有子目录，并把列举出来的目录路径和元素集合中的每一个元素拼接成形如"目录路径\\元素"格式的新字符串，然后，对每一条这样的新字符串执行"命令语句集合"中的每一条命令；
    特别的是：当"元素集合"带以点号分隔的通配符?或*的时候，把"元素集合"视为文件（不视为文件夹），整条语句的作用是匹配"目录"所指文件夹及其所有子文件夹下匹配的文件；若不以点号分隔，则把"元素集合"视为文件夹（不视为文件）；
2. 当省略掉"目录"时，则针对当前目录；
3. 当元素集合中仅仅是一个点号的时候，将只列举目录路径；

```sh
@echo off
for /r d:\test %%i in (.) do echo %%i
pause
```

执行的结果如下所示：

```log
d:\test\.
d:\test\1\.
d:\test\2\.
d:\test\3\.
```

效果就是显示 `d:\test` 目录及其之下是所有子目录的路径，其效果与 `dir /ad /b /s d:\test` 类似。若要说到两者的区别，可以归纳出3点：

1、`for /r` 列举出来的路径最后都带有斜杠和点号，而 `dir` 语句则没有，会对获取到的路径进行进一步加工产生影响；

2、`for /r` 不能列举带隐藏属性的目录，而 `dir` 语句则可以通过指定 `/a` 后面紧跟的参数来获取带指定属性的目录，更加灵活；

3、若要对获取到的路径进行进一步处理，则需要把 `dir` 语句放入 `for /f` 语句中进行分析，写成 `for /f %%i in ('dir /ad /b /s') do ……` 的形式；由于 `for /r` 语句是边列举路径边进行处理，所以，在处理大量路径的时候，前期不会感到有停顿，而 `for /f` 语句则需要等到 `dir /ad /b /s` 语句把所有路径都列举完之后，再读入内存进行处理，所以，在处理大量路径的时候，前期会感到有明显的停顿。

第2点差别很容易被大家忽视，导致用 `for /r` 列举路径的时候会造成遗漏；而第3点则会让大家有更直观的感受，很容易感觉到两者之间的差别。

要是"元素集合"不是点号呢？

```
@echo off
for /r d:\test %%i in (a b c) do echo %%i
pause
```

运行的结果是：

```
D:\test\1\a
D:\test\1\b
D:\test\1\c
D:\test\2\a
D:\test\2\b
D:\test\2\c
D:\test\3\a
D:\test\3\b
D:\test\3\c
```

原来，它的含义是：列举 d:\test 及其所有的子目录，对所有的目录路径都分别添加a、b、c之后再显示出来。

```
@echo off
for /r d:\test %%i in (*.txt) do echo %%i
pause
```

运行结果是：

```
D:\test\test.txt
D:\test\1\1.txt
D:\test\1\2.txt
D:\test\2\a.txt
D:\test\2\b.txt
D:\test\3\1.txt
```

这段代码的含义是：列举 `d:\test` 及其所有子目录下的txt文本文件（以.txt结尾的文件夹不会被列出来）。
