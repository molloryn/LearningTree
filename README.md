# Markdown 

## 配置

#### 不限制页面宽度
`Editor` -> `Display` -> `Readable line length`，取消勾选。

## Markdown语法

[Basic formatting syntax - Obsidian Help](https://help.obsidian.md/Editing+and+formatting/Basic+formatting+syntax)

- 现在已经设置自动将粘贴的图片放在attachments文件夹下 

| 语法   | 作用                                                                           |
|:------ |:------------------------------------------------------------------------------ |
| `[[]]` | 内部链接，链接到当前文档项目任何文档或图片等。已配置自动转换为Markdwon规范链接 |
| `[]()` | Markdown规范链接                                                               |
| `> `   | 引用                                                                               |

[Basic Syntax | Markdown Guide](https://www.markdownguide.org/basic-syntax/)
[Extended Syntax | Markdown Guide](https://www.markdownguide.org/extended-syntax/)
## Typora
### 快捷键

| 快捷键      | 作用     |
| ----------- | -------- |
| Shift+Enter | 表格换行 |
|             |          |
|             |          |

#### 自定义配置

`Advanced Setting`可以打开配置文件夹，其中`conf.user.json`中定义：

```json
 "keyBinding": {
    // for example:
    // "Always on Top": "Ctrl+Shift+P"
    // All other options are the menu items 'text label' displayed from each typora menu
    "Code":"Alt+1",
    "Code Fences":"Alt+3"
  },
```



## Obsidian快捷键

目前收集的和自定义的常用快捷键

| 快捷键                    | 作用                     |
| ---------------------- | ---------------------- |
| Ctrl+1...6             | 设置为Heading1-6          |
| Ctrl+T                 | 快捷增加模板（Template）       |
| Ctrl+B                 | 加粗（Bold）               |
| Ctrl+P                 | 命令面板（Panel）            |
| Ctrl+Q                 | 表格高级编辑（需要先整体选中表格）      |
| 表格最后一行Enter            | 添加下一行                  |
| 表格Tab                  | 同行切换，最后一列后开始添加下一列      |
| Ctrl+D                 | 删除当前段落（Delete）         |
| Ctrl+E                 | 切换为编辑模式进行预览            |
| Ctrl+Shift+I           | 打开DevTools查看Obsidian报错 |
| Alt+1                  | 添加代码行                  |
| Alt+3                  | 添加代码块                  |
| Alt+4                  | 设置来源字体颜色插件颜色           |
| Alt+5                  | 切换颜色块                  |
| Alt+6                  | 自定义当前字体颜色设置            |
| Alt+Q                  | Quiet Outline插件的目录     |
| 文件面板Ctrl+左键单击          | 使用新tab打开文档             |
| 文件面板Alt+左键单击           | 多选文档                   |
| 使用Alt功能选择文档后松开，拖拽文件到文档 | 建立双向链接                 |
| Ctrl+O                 | 快速定向到指定文件              |

> 模板在attachments/templates文件夹下，可以自行增加模板

## 其他功能

#### 将搜索结果作为双向链接
![](attachments/Pasted%20image%2020230823084310.png)

## Obsidian 命令Alias

`/`即可出现命令面板

| 缩写 | 全称          | 作用         |
|:---- |:------------- |:------------ |
| dr   | delete raw    | 删除当前行   |
| dc   | delete column | 删除当前列   | 
| sd   | swap down     | 与下一行互换 |
| su   | swap up       | 与上一行互换 |


### AI提示词

```txt
将给定的Markdown文件进行修改：  
1. 将文件标题作为Heading1，如果文件内部存在这个标题，则跳过。  
2. 除了文件标题H1之外，其余地方不允许再出现H1。请调整每个标题级别符合要求。  
3. 对其中涉及的专有英文名词，使用反引号"`"包裹。  
4. 对于其中没有使用代码块"```"语法包裹的，请使用合适的代码块语言包裹  
5. 尽量不要对内容进行改动，仅修复格式、语法错误或事实错误。  
6. 若有事实错误修正的，最后总结改动项。  
7. 标号也统一使用markdown列表序号，如①等需要进行修改。  
8. 如果存在代码段，或命令，请换用反引号包裹，如: .HasKey(c =\> c.Id) => `.HasKey(c => c.Id) `  
9. 如果存在外部地址形如`<https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/recommended-tags>`格式的，替换为可阅读的格式：`[Recommended XML documentation tags - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/recommended-tags)`
10. 如果存在英文原文的一大段话，无需翻译成中文，仅需调整格式。
```


```
将给定Markdown文件进行拆分，按照大章节目录拆为多个小文件，根据当前文件层级来拆分，如按照H1拆分，如拆分文件仍过大，则按照H2拆分，以此类推。拆分后的文件名与当前章节有关，如果是英文则保持英文标题。  
要以当前文件名建立新文件夹，并把拆分后的文件放入到新文件夹下。  
对每个拆分后的文件进行修改：  
1. 将文件标题作为Heading1，如果文件内部存在这个标题，则跳过。  
2. 除了文件标题H1之外，其余地方不允许再出现H1。请调整每个标题级别符合要求。  
3. 对其中涉及的专有英文名词，使用反引号"`"包裹。  
4. 对于其中没有使用代码块"```"语法包裹的，请使用合适的代码块语言包裹  
5. 尽量不要对内容进行改动，仅修复格式、语法错误或事实错误。  
6. 若有事实错误修正的，最后总结改动项。  
7. 标号也统一使用markdown列表序号，如①等需要进行修改。  
8. 如果存在代码段，或命令，请换用反引号包裹，如: .HasKey(c =\> c.Id) => `.HasKey(c => c.Id) `  
9. 如果存在外部地址形如`<https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/recommended-tags>`格式的，替换为可阅读的格式：`[Recommended XML documentation tags - C# reference | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/recommended-tags)`
10. 如果存在英文原文的一大段话，无需翻译成中文，仅需调整格式。
```
