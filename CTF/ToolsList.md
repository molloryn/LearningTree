# MISC

## 压缩包

### 暴力破解


| 软件名              | 场景                                                                 | 备注                                                                                                                                      |
|:------------------- |:-------------------------------------------------------------------- |:----------------------------------------------------------------------------------------------------------------------------------------- |
| ARCHPR              | 可以用于明文攻击，即已有被加密过的压缩包内的部分文件，进行逆向破解。 | [压缩 (lanzoui.com)](https://hzgzs.lanzoui.com/s/yasuopj)                                                                                 |
| ziperello           |                                                                      | [主流格式压缩包密码破解方法+字典枚举 整理+工具](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1478639&highlight=%D1%B9%CB%F5%B0%FC) |
| 字典破解2.0         |                                                                      |                                                                                                                                           |
| PasswareKitForensic | 除明文攻击外，基本支持暴力破解相关所有功能                           | 已保存在百度网盘                                                                                                                          |
| bkcrack             | 可用于明文攻击                                                       |[Release Release v1.5.0 · kimci86/bkcrack (github.com)](https://github.com/kimci86/bkcrack/releases/tag/v1.5.0) [bkcrack：一款基于已知明文攻击的传统zip加密破解工具](https://cloud.tencent.com/developer/article/2215202)                                                                                                                                          |

注意：明文文件路径内zip文件头需要和加密压缩包文件头一样（同一压缩工具压缩），相关工具内列出了常用压缩工具文件头和文件头识别工具。
明文攻击是先把里面存在的文件用同样的压缩工具进行压缩。
明文攻击文件和加密zip内的文件CRC校验要一样。

## 图片

### 加解密隐写

|软件名 | 场景                                                                     | 备注                                                |
|:--------- |:------------------------------------------------------------------------ |:--------------------------------------------------- |
| oursecret |有些提示“我们的秘密”这种可能涉及该工具隐写加密，需要密码，可进行图片+文件加密隐写|  |
| zsteg     |zsteg可以检测PNG和BMP图片里的隐写数据，LSB部分支持 | [zed-0xff/zsteg](https://github.com/zed-0xff/zsteg) |
|LSB-Steganography|00000开头特征的LSB隐写 |  |
|stegpy |01双通道隐写|  |
|cloacked-pixel |带密码的LSB隐写|  |
|stegsolve|图片隐写综合分析工具|  |
|steghide|能够隐藏各种图像和音频文件中的数据。颜色方面的样本频率不会更改，因此使嵌入抗一阶统计检验。|`apt-get install stegdetect` |
|jphs(jphide) |使用stegdetect出现的提示的工具，用于有密码的jpg隐写文件，目前网上能下载到的带界面的为`jphswin`|  |

## 文件

| 软件名 | 场景                             | 备注                                                                                                                 |
|:------ |:-------------------------------- |:-------------------------------------------------------------------------------------------------------------------- |
| TrID   | File Identifier 用于识别文件类型 | [官网](https://www.mark0.net/soft-trid-e.html)   [教程](https://blog.csdn.net/qq_45699846/article/details/123514058) |
|  |  |                                                                                                                      |
|        |                                  |                                                                                                                      |
|        |                                  |                                                                                                                      |
|        |                                  |                                                                                                                      |
|        |                                  |                                                                                                                      |
## 编码/加密

| 软件名    | 场景                 | 备注                                                           |
|:--------- |:-------------------- |:-------------------------------------------------------------- |
| Ciphey    | 自动加解密   | [Ciphey/Ciphey (github.com)](https://github.com/Ciphey/Ciphey) |
| Cyberchef | 烹饪化加解密/编码工具         |                                                                |
| Ares      | 自动加解密，是Ciphey下一代 | [bee-san/Ares (github.com)](https://github.com/bee-san/Ares)                                                               |



# WEB

## 目录扫描

目录扫描：[kali linux web目录扫描工具汇总 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv18085514/)

## 漏洞扫描

| 软件名 | 场景 | 备注 |
|:------ |:---- |:---- |
| nmap   |      |      |
| zenmap |      |      |
|        |      |      |


## 抓包

| 软件名    | 场景 | 备注 |
|:--------- |:---- |:---- |
| BurpSuite |      | [BurpSuite Pro 2022.1.1汉化破解（无cmd框）](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1585838&highlight=burpsuite)     |
|           |      |      |
|           |      |      |





## SSH/SFTP/FTP

| 软件名    | 场景    | 备注 |
|:--------- |:------- |:---- |
| tabby     | 终端       | [A terminal for a more modern age](https://github.com/Eugeny/tabby)     |
| WindTerm  |         | [kingToolbox/WindTerm](https://github.com/kingToolbox/WindTerm)     |
| MobaXterm |         |      |
| WinSCP    | FTP工具 |      |
|           |         |      |


# PWN 

| 软件名        | 场景                      | 备注 |
|:------------- |:------------------------- |:---- |
| VolatilityPro | 自动化处理内存取证，含GUI | [Tokeii0/VolatilityPro](https://github.com/Tokeii0/VolatilityPro)     |
|               |                           |      |
|               |                           |      |



# 逆向 (Reverse)
## 反汇编

| 软件名       | 场景                        | 备注                                                           |
|:------------ |:--------------------------- |:-------------------------------------------------------------- |
| Xdbg         | 动态调试工具，Ollydbg替代品 | [x64dbg/x64dbg (github.com)](https://github.com/x64dbg/x64dbg) |
| IDA Pro      | 静态调试工具                | 可以硬代码反汇编，还可以配合Il2CppDumper进行源码分析           |
| Ollydbg      |                             |                                                                |
| Cheat Engine | 游戏修改/内存修改器         | 开源                                                           |
|              |                             |                                                                |

## 反编译

| 软件名       | 场景                              | 备注                                                                                                   |
|:------------ |:--------------------------------- |:------------------------------------------------------------------------------------------------------ |
| Javassist    | Java bytecode engineering toolkit | [jboss-javassist/javassist (github.com)](https://github.com/jboss-javassist/javassist)                 |
| JD-GUI       | Java Decompiler                   | [java-decompiler/jd-gui at v1.6.6 (github.com)](https://github.com/java-decompiler/jd-gui/tree/v1.6.6) |
| Jadx         |  Dex to Java decompiler                                 | [skylot/jadx: Dex to Java decompiler (github.com)](https://github.com/skylot/jadx)                     |
| dnspy        | C#反编译                          |                                                                                                        |
| ILspy        | C#反编译                          |                                                                                                        |
| Il2CppDumper | unity反编译                       |                                                                                                        |



# 中间件管理

| 软件名                        | 场景                  | 备注                                                                                                       |
|:----------------------------- |:--------------------- |:---------------------------------------------------------------------------------------------------------- |
| RedisInsight                  | Redis官方出品管理工具 | [RedisInsight/RedisInsight: Redis GUI by Redis (github.com)](https://github.com/RedisInsight/RedisInsight) |
| Another Redis Desktop Manager | Redis管理工具         | [qishibo/AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager)                |
|                               |                       |                                                                                                            |


# Linux效率

| 软件名        | 场景                       | 备注                                                                                        |
|:------------- |:-------------------------- |:------------------------------------------------------------------------------------------- |
| thefuck       | 辅助linux修复命令          | [nvbn/thefuck](https://github.com/nvbn/thefuck)                                             |
| powerlevel10k | zsh主题                    | [romkatv/powerlevel10k: A Zsh theme (github.com)](https://github.com/romkatv/powerlevel10k) |
| ohmyzsh       | 管理 linux shell zsh       | [ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)                                       |
| Dust          | Du+rust,增强disk usage命令 | [bootandy/dust](https://github.com/bootandy/dust)                                                                                            |




# 其他

| 软件名                     | 场景                                 | 备注                                                                                                   |
|:-------------------------- |:------------------------------------ |:------------------------------------------------------------------------------------------------------ |
| RustDesk                   | 远程桌面                             | 开源                                                                                                   |
| Parsec                     | 远程桌面                             | 支持移动端，高画质                                                                                     |
| Proxifier                  | 设置各种规则，使得本地软件走特定代理 |                                                                                                        |
| ContextMenuManager.NET.4.0 | Windows右键管理工具                  |                                                                                                        |
| Optimizer                  | Windows优化工具                      | [hellzerg/optimizer: The finest Windows Optimizer (github.com)](https://github.com/hellzerg/optimizer) |
| sysinternals               | Windows调试工具等，火绒剑替代        |                                                                                                        |
| BFG-repo-cleaner           | Git Secret blob 移除工具                  | [rtyley/bfg-repo-cleaner](https://github.com/rtyley/bfg-repo-cleaner)                                                                                                       |


# 效率

| 软件名       | 场景                                             | 备注                                                          |
|:------------ |:------------------------------------------------ |:------------------------------------------------------------- |
| Snipaste     | 截图贴图工具                                     |                                                               |
| GoldenDict   | 支持MDict的字典工具                              |                                                               |
| FluentSearch | 快捷搜索栏，大量功能                             |                                                               |
| Quicker      | 鼠标中键插件扩展、类Workflow编程、含有快捷搜索栏 | 收费版可翻页                                                  |
| utools       | 类似Quicker，但支持markdown笔记搜索              | [插件离线安装](https://blog.csdn.net/m0_32156988/article/details/126415067)                                                              |
| DevToys      | 如格式化JSON，JSON转YAML，比较文本，测试RegExp   | [veler/DevToys](https://github.com/veler/DevToys)             |
| PowerToys    | 微软开源功能增强工具，包括批量重命名、图像压缩等 | [microsoft/PowerToys](https://github.com/microsoft/PowerToys) |

