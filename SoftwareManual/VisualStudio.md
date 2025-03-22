
功能增强参考：[2.14 Visual Studio 高效率 | Furion (baiqian.ltd)](http://furion.baiqian.ltd/docs/vsfast/)
### 文件编码与代码规范

文件编码可以在这里检查：
![](../attachments/Pasted%20image%2020230802171032.png)
![](../attachments/Pasted%20image%2020230802171059.png)

如果没发现这个选项，需要进行自定义：
![](../attachments/Pasted%20image%2020230802174653.png)

VS2022之后，可以设置编辑器的默认encoding。在Solution或Project下添加`.editorconfig`文件
![](../attachments/Pasted%20image%2020230802171236.png)
然后在其中添加
```editorconfig
[*]
charset = utf-8
```

添加后，后续增加的新文件应都是UTF-8格式的了。

`.editorconfig`文件还可以定义代码规范，和resharper的规范共存：
![](../attachments/Pasted%20image%2020230802175133.png)

### 离线下载
##### 教程
[Create an offline installation - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio?view=vs-2022)

##### 下载bootstrapper vs_enterprise.exe
[Create a network-based installation - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/install/create-a-network-installation-of-visual-studio?view=vs-2022#download-the-visual-studio-bootstrapper-to-create-the-layout)
```shell
# 使用 bootstrapper进行运行命令 create local layout
vs_enterprise.exe --layout D:\VS2022\vslayout --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.NetWeb --add Microsoft.VisualStudio.Component.Git --includeRecommended --lang en-US zh-CN

# 更新layout到最新版本
vs_enterprise.exe --layout D:\VS2022\vslayout

# --lang在安装时无效
# --fix可以检验vslayout是否有包是损坏的


# 到内网机器下，一定要使用同一条命令安装，而且启动bootstrapper换成了vslayout里面的
D:\VS2022\vslayout\vs_enterprise.exe --noweb --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.NetWeb --add Microsoft.VisualStudio.Component.Git --includeRecommended
# 指定路径安装：--installPath d:\VS2021
```



### CLI

Windows下设置dotnet cli工具语言为英文：`set DOTNET_CLI_UI_LANGUAGE=en`

Linux下设置系统语言为英文即可同步变为英文：`export LANG=en_US.UTF-8`

### 插件推荐

`Dracula official` 主题

`Clean Bin and Obj` 快速清理obj和bin文件夹，以解决改变项目名带来的多项目间依赖缓存问题。不过这个插件清空解决方案不好使，可以单独对项目清理。
[Clean Bin and Obj - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=dobrynin.cleanbinandobj&ssr=false#overview)

`Debug Single Thread `多线程时单线程调试

`Multiline Search and Replace` 多行替换、搜索

[File Path On Footer - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ShemeerNS.FilePathOnFooter)
可以在打开的文件下面显示文件所在地址，快速打开相关文件夹等
#### Folder To Solution Folder
直接添加物理文件夹成为相应的Solution Folder，不用再一个一个对比。
#### SwitchStartupProject

能够保存不同的Startup Project的设置。
[SwitchStartupProject for VS 2022 - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=vs-publisher-141975.SwitchStartupProjectForVS2022)

#### Better Comments VS2022

将指定格式的注释不同颜色高亮

#### Visual Commander

Visual Commander 可以用C\#编码设定强大的功能（类似于Word、PPT的宏）
[vlasovstudio.com/visual-commander/](https://vlasovstudio.com/visual-commander/)

复制当前行的行数及方法名到剪切板：
```cs
EnvDTE.TextSelection ts = DTE.ActiveWindow.Selection as EnvDTE.TextSelection;

if (ts == null)
  return;

EnvDTE.CodeFunction func = ts.ActivePoint.CodeElement[vsCMElement.vsCMElementFunction]
            as EnvDTE.CodeFunction;

if (func == null)
  return;

string result = DTE.ActiveWindow.Document.FullName + System.Environment.NewLine +
  "Line " + ts.CurrentLine + System.Environment.NewLine +
  func.FullName;

System.Windows.Clipboard.SetText(result);
```

### Stack Trace Explorer（堆栈信息高亮）

[Stack Trace Explorer 2022 - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SamirBoulema.StackTraceExplorer2022)

可以将粘贴进的Stack Trace高亮并启动路由功能
其实来源于ReSharper，默认快捷键为 `Ctrl+E,T`
当前插件的快捷键去这里看View - Other Windows - Stack Trace Explorer

实测不如ReSharper的。

### 注释高亮

以BUG开头的注释会高亮

另外还可以安装Better Comments插件

### 语法高亮

如果语法高亮出现问题，方法等仍然是白色的，可以尝试：
This happens when you set Color Theme: Use system setting
Set Color Theme: Dark - or Color Theme: Blue

#### 正则

三种方式让正则表达式高亮
```cs
// language=regex

var str = @"[A-Z]\d+";

MyMethod(/* language=regex */ @"[A-Z]\d+");


```

不过目前不支持\$”{}”，只能是纯string

.NET 7新增方法上的正则高亮支持

https://devblogs.microsoft.com/dotnet/regular-expression-improvements-in-dotnet-7

```cs
void MyMethod([StringSyntax(StringSyntaxAttribute.Regex)] string regex);
```


### Enable and disable package restore in Visual Studio

[NuGet Package Restore | Microsoft Learn](https://learn.microsoft.com/en-us/nuget/consume-packages/package-restore#enable-and-disable-package-restore-in-visual-studio)

### Code Intelligence

[Visual Studio keyboard shortcut to display IntelliSense - Stack Overflow](https://stackoverflow.com/questions/3640544/visual-studio-keyboard-shortcut-to-display-intellisense)

### StackTrace

When you instantiate the StackTrace, use the following:

`StackTrace callStack = new StackTrace(1, true);`

The "true" parameter is telling it to include such things as LineNumbers, etc.

PDB can be generated for Release as well as for Debug.

Project → Properties → Build → Advanced → Debug Info

### Multiple startup project

In Solution Explorer, select the solution (the top node).

Choose the solution node's context (right-click) menu and then choose Properties. The Solution Property Pages dialog box appears.

![Graphical user interface, table Description automatically generated](../attachments/0b5138a97665abbeb9547d2e24e0c722.png)

Solution Property Pages

Expand the Common Properties node, and choose Startup Project.

Choose the Multiple Startup Projects option and set the appropriate actions.

### 反编译

ILSpy

针对VS自带反编译方案的不足,这里推荐使用ILSpy,通过VS扩展市场[Extensions for Visual Studio family of products | Visual Studio Marketplace](https://marketplace.visualstudio.com/vs)离线下载,安装成功后在VS`工具->ILSpy`打开,然后通过菜单栏定位到要查看的程序集,打开后可查看程序集的结构,命名空间,类等所有对象的定义与源码

如果想阻止别人反编译你的源码,可为你的程序集加上SuppresslIdasmAttribute属性



# 项目配置

## 解决方案统一配置项目

project的xml中可以使用 `Import` 方式导入一个统一的xml文件：

`common.xml`:
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	<PropertyGroup>
		<TargetFramework>net7.0</TargetFramework>
		<SatelliteResourceLanguages>en;zh-Hans</SatelliteResourceLanguages>
	</PropertyGroup>
</Project>
```

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

	<Import Project="..\..\..\..\common.xml"></Import>
    <PropertyGroup>
        
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
        <UserSecretsId>575a2d7c-be72-40ea-bf82-2ece95e5ad68</UserSecretsId>
        <DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>
        <DockerfileContext>..\..\..\..</DockerfileContext>
        <DockerComposeProjectPath>..\..\..\..\docker-compose.dcproj</DockerComposeProjectPath>
        <GenerateDocumentationFile>true</GenerateDocumentationFile>
        <NoWarn>1591</NoWarn>
    </PropertyGroup>
```

## ASP.NET Core项目编译下的多语言文件夹

通过 `SatelliteResourceLanguages` 可以指定需要的。不指定会全部都有。。。
```xml
<SatelliteResourceLanguages>en;zh-Hans</SatelliteResourceLanguages>
```

## CopyToOutputDirectory但复制到特定目录下

使用`<Link>`可以实现复制到特定输出目录
```xml
<ItemGroup>
	<Content Include="config.json">
		<Link>Config\config.json</Link>
		<CopyToOutputDirectory>Always</CopyToOutputDirectory>
	</Content>
</ItemGroup>

```



# 发布

容器发布

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
https://go.microsoft.com/fwlink/?LinkID=208121.
-->
<Project>
  <PropertyGroup>
    <WebPublishMethod>Container</WebPublishMethod>
    <ContainerPublishMethod>NetSdk</ContainerPublishMethod>
    <ContainerRepository>username/test</ContainerRepository>
    <RegistryUrl>index.docker.io</RegistryUrl>
    <UserName>9kit</UserName>
    <PublishImageTag>latest</PublishImageTag>
    <PublishProvider>ContainerRegistry</PublishProvider>
    <LastUsedBuildConfiguration>Release</LastUsedBuildConfiguration>
    <LastUsedPlatform>Any CPU</LastUsedPlatform>
    <RuntimeIdentifier>linux-x64</RuntimeIdentifier>
    <ProjectGuid>9f221ded-210c-4907-be8a-7d0d489f9f70</ProjectGuid>
    <_IsDockerfilePresent>true</_IsDockerfilePresent>
    <_TargetId>NetSdkDockerContainerRegistry</_TargetId>
  </PropertyGroup>
</Project>

```

如果发布失败或没反应，直接拷出output中的命令自己执行看看报错


# 模板

## 项目模板

[dotnet new uninstall - .NET CLI | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new-uninstall)

显示拥有的项目模板：
`dotnet new uninstall` 
它也顺便告诉你如何卸载拥有的项目模板。
还有使用插件形式安装的是在 `Extensions`中进行卸载，需要以管理员权限启动VS
![](../attachments/Pasted%20image%2020231113101056.png)

创建和导入：
[Create multi-project templates - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/ide/how-to-create-multi-project-templates?view=vs-2022)

导入自定义的模板目前要手动找到相应目录，并拷贝`.zip`进去。
目录默认在（设置的文档目录）：
`D:\Documents\Visual Studio 2022\Templates\ProjectTemplates`
但也可能读取的是：
`C:\Users\<Name>\Documents\Visual Studio 2022\Templates\ProjectTemplates`
具体可以自己生成一个，勾上自动导入，然后通过 `everything`等工具找到实际目录。

模板xml文件的属性：[Visual Studio Template Schema Reference - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/extensibility/visual-studio-template-schema-reference?view=vs-2022)

> 有个bug，用windows自带的zip压缩的文件VS无法识别。


# 调试

### 设置

`Ctrl`+`ALT`+`E`打开`Exception` `Setting`，这里设置`Debug`时是否捕捉异常并停止提示。

比如一些异步的方法、`Task`中引发的异常，默认是不捕获异常的，需要将`Common` `Language` `Runtime` `Exception`勾上。

**多进程调试**

In Visual Studio you can set multiple startup projects to run/debug two projects at once:

Right click on the solution in the Solution Explorer and select Properties.

Under Common Properties\\Startup Project select Multiple startup projects and set the Action to be Start for the two projects that you want to debug.

解决方案属性-\>多个启动项目

![电脑萤幕的截图 描述已自动生成](../attachments/e65d34b03eae060ab5c231e2780469d1.png)
### 断点

行号上`Ctrl`+`Mouse Left Click`可以快速插入临时断点。 

### 即时调试（Debug.Immediate）

-   即时调试窗口:命中断点后`Ctrl`+`ALT`+`I`

    可输入参数,实例化对象,常量,也可执行方法或函数

    ![电子设备的屏幕 描述已自动生成](../attachments/1bffabf753c0392c3520a8248c0ce74a.gif)

-   特殊参数
    -   \$retrunValue:方法返回值
    -   \$exception:未捕获的异常

        ![电脑萤幕画面 描述已自动生成](../attachments/3d9d66c3d38b5e5891805466c6bf4816.gif)

### 多线程调试

安装Debug Single Thread插件仅调试单个线程。

-   三个调试操作窗口
    -   线程[Ctrl+Shift+H]
    -   并行堆栈
    -   并行监视[Ctrl+Shift+D]
-   三种操作方法
    -   标记
        标记线程后,在并行监视窗口中显示标记线程,其他线程如果无断点则不会跳进

    -   冻结
        冻结线程后停止工作,此方法不适用于有多线程同步操作的

    -   添加断点
        -   条件表达式:参数
        -   筛选器:线程名称或者线程Id

            ![截图里有图片 描述已自动生成](../attachments/6fab81425b78a47bfbcfff7aa846bf09.png)

### 远程测试调试

测试-\>测试资源管理器

-   Docker Desktop:基于WSL/WSL2
    -   Windows
    -   Linux
-   WSL/WSL2
    -   Ubuntu
    -   其他Linux发行版
-   SSH
-   远程Windows
-   远程Linux

    ![图形用户界面, 文本, 应用程序 描述已自动生成](../attachments/806b136d6916f36720ab524d59906f68.png)


### 调试外部dll

[Presentation options for compiled code | ReSharper Documentation (jetbrains.com)](https://www.jetbrains.com/help/resharper/Code_Presentation_Options.html)
[Debug modules that have no debug information (PDB) | ReSharper Documentation (jetbrains.com)](https://www.jetbrains.com/help/resharper/Debugging_Without_Source_Code.html)

- In the debug mode, open the Visual Studio's Modules window (Debug | Windows | Modules), select one of several modules, right-click the selection, and choose Load Symbols with ReSharper Decompiler.

关闭勾选Just My Code。加载所有dll之后，打断点会提示代码对不上。这时设置断点可以调试对不上的代码。

##### 拥有pdb文件时
可使用原生VisualStudio加载符号，但需要手动创建一个目录，使用everything复制所有pdb文件（一般就在nuget目录下）到该文件夹，在Options->Debugging->Symbols->Symbol file locations 中添加该目录


### 调试打印类信息

默认情况下，如果有重写`ToString()`，则使用该方法在Debugger中显示。
还可以通过 `[DebuggerDisplay]` 进行配置
[Display custom info using DebuggerDisplay - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/debugger/using-the-debuggerdisplay-attribute?view=vs-2022)

### 应用程序处于中断模式（The application is in break mode）
下了断点在自己代码里但发现中断模式，在通过dapr插件启动微服务调试时遇到，这种情况可能是那个无法命中断点的程序的dll较旧，需要重新build一下。


# 快捷操作

所有快捷键均可在工具-\>选项-\>键盘中自定义

注意，它还区分优先级，即，Global优先级最小，其次是各个子窗口。如果子窗口有相应的设置，则Global的可能导致无效，注意检查。

代码窗口是Text Editor 而不是C\# Editor

### 个人定义

Alt + C Collapse all (在Solution Explore界面)

![Title: fig:](../attachments/bae11f3d5036aabc8e6911587026bfff.png)
`SoulutionExplorer.SyncWithActiveDocument`
默认快捷键 `Ctrl+[,S` （个人建议改成Ctrl+W，Ctrl+W，表示Where）

#### Resharper相关

Ctrl + R , C Change Signatrue (TextEditor)

Ctrl + Arrow Down ReSharper_GotoPrevMember (TextEditor)

### 代码操作

Ctrl+F12 Go to Implementation（转到接口实现）.

## 切换

Ctrl+Tab 快速在已打开的文件中切换

## W类窗口

Window类所以先按W

打开错误列表 Ctrl+W, E (ErrorList)

打开解决方案窗口Ctrl+W, S (Solution)

打开输出窗口Ctrl+W, O (Output)

Git Changes窗口建议改为Ctrl+W, G (Git)

Package Manage Console建议改为Ctrl+W, C (Console) `View.PackageManagerConsole`

Bookmark窗口 Ctrl+W, B (Bookmark)

## K类代码

### Intelligence

列出当前可选成员 Ctrl+K, L (List)

当前快速信息（比如出现红线、蓝线等有提示，一般要用鼠标移上去的） Ctrl+K, I (Info)

### 注释

注释当前行/选中行 Ctrl+ K, C (Cancel)

取消注释Ctrl+K, U (Undo cancel)

### 格式化

格式化当前行/选中行：Ctrl + K, F (Format)

格式化当前文档：Ctrl + K, D (Document)

### 书签

书签设置在行号上，而不是代码上。 如果修改代码，书签会保留在行号上，不会随代码移动。

当前行设置/删除书签 Ctrl+K, K

转到下一个书签 Ctrl + K, N (Next)

## 其他

### 全文搜索

[![](../attachments/3a4da5d726a60914579c20b555b267df.png)] 默认快捷键 Ctrl+T(Go To) 可根据文件名称,类名称,方法名称全搜索

![电子设备的屏幕 描述已自动生成](../attachments/ac8a1ef40d84035bb50ed78443b77ee7.gif)

### 快速定位活动文件

[![](../attachments/36206c5edebb7bcbce3d264bab464531.png)] 默认快捷键 Ctrl+[,S （个人建议改成Ctrl+W，Ctrl+W，表示Where）

![电脑的屏幕截图 描述已自动生成](../attachments/3f16e5f798883588e52433d2202c0017.gif)

### 多行编辑

-   Alt+鼠标拖选
    -   Alt+Shift+方向键

        ![文本 描述已自动生成](../attachments/8de332850a989919e180a6b5e6729632.gif)

### 多光标同时编辑

Ctrl+Alt+鼠标点击要编辑的位置

![文本 描述已自动生成](../attachments/0fa9cbfac79bca665079eb4f704a6dad.gif)

### 整行上下移动

Alt+上下方向健

![文本 描述已自动生成](../attachments/caf9cc12bacfc56d1d637869e5b3b7c9.gif)

整个方法块折叠以后也可以上下移动

### 历史粘贴

Ctrl+Shift+V 打开粘贴板历史内容,点击粘贴

![图形用户界面, 文本, 应用程序 描述已自动生成](../attachments/5052e72105bf683d6650d3de22438dd5.png)

# ReSharper

### 问题

#### prop template
`prop`有非常奇怪的问题，转而使用VisualStudio的比较好。

> 其实原因发现居然是VS自己的IntelliCode，关闭C# suggestioins.
> 又发现一个解决方案，可以开启IntelliCode，但是选择 `Apply whole line completions on right arrow`


ReSharper > Tools > Template Explorer
![](../attachments/Pasted%20image%2020230829170428.png)

### 配置

Shortcut Scheme中可以开启Shortcut Browser，按三下Ctrl快速弹出当前支持的快捷键。

如果发现对不上快捷键，需要去Keyboard配置中重新覆盖一下配置。


### File Layout
其中有功能可以排序属性

应用File Layout需要使用
![](../attachments/Pasted%20image%2020231114103910.png)
这个右键文件使用 `Cleanup Code…` 功能，注意不是VS的CleanupCode

按字母顺序排序属性：
![](../attachments/Pasted%20image%2020231114104833.png)

### 快捷

#### Alt+\` Navigate To

##### To-do explorer
需要在写了 `TODO` `BUG`等描述的地方进入
[Navigate To: To-do Explorer | ReSharper Documentation (jetbrains.com)](https://www.jetbrains.com/help/resharper/Navigation_and_Search__Navigate_from_Here__Todo_Explorer.html)

#### Alt+Insert Generate Code
[Generate type members | ReSharper Documentation (jetbrains.com)](https://www.jetbrains.com/help/resharper/2023.2/Generating_Type_Members.html)

#### Ctrl+Shift+Alt+方向键 快速移动参数/行顺序

MoveUp和MoveDown可能要重新设置一下

#### Alt+Enter 集大成

**Alt+Enter** 快速看看当前代码支持的功能，不仅是重构，还可以是直接转向接口实现等

如果直接输入word，还可以查找所有Action

![Navigating to action](../attachments/fc4c8e56e190ad09b005012c03b28fd0.png)

比如输入

Nearby，启动转到文件附近文件功能

Regular，启动validate regular expression 功能

### Navigation

![图形用户界面, 应用程序 描述已自动生成](../attachments/ec5910620828f74c2ecaa8836b9df46c.png)

一定要设置的是TextEditor

ReSharper_GotoPrevMember

#### Quick Documentation （Ctrl+Shift+F1）

可以快速转到MSDN

![Quick Documentation pop-up](../attachments/2ff040f06d057f130c1faf0cb63e3640.png)

## Code Snippets

ReSharper \| Tools \| Templates Explorer.

[jetbrains.com/resharper/features/code_templates.html](https://www.jetbrains.com/resharper/features/code_templates.html)

## Convenient Functions

### Complete Statement 快速加括号

This feature inserts necessary syntax elements (braces, semicolons etc.) and sets you in position to start the next statement, saving you from excessive juggling with the caret. As you work, keep in mind the default shortcut for this feature: **Ctrl+Shift+Enter**.

Complete Statement (also known as Smart Enter) comes to rescue in numerous scenarios, including auto-closing parentheses, adding semicolons, completing if, while and for statements, and so on.

### Namespace

By default, ReSharper assumes that the namespace each class appears in matches its location in the project. The 'root' namespace for the project is defined in the project properties:

![Text Description automatically generated](../attachments/1c6a5a06abe576207723de4a8b38645c.png)

项目内部的才会自动按照文件夹分namespace

![](../attachments/84cd8787ed543b696bff31bd1430a952.png)
