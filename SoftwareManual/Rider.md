# Rider


自动换行： Soft Wrap

## Keymap

- Select File in Project View (In Other - Bookmarks)


## Verison Control

开启 `Enable Staging area` 

## Theme

### HighContrast

#### 统一色调

- Color Scheme -> General -> Text -> Default text  代码背景色
- Color Scheme -> Console colors -> Console -> Background  Console 背景色
- Color Scheme -> General -> Background in read-only files 只读文件（如代码生成器代码）背景色
- Version Control -> File Status Colors   Commit窗口中的文件颜色



## 调试 SourceGenerator

应用程序入口点加入类似如下的配置：`launchSetting.json` 即可在 `SourceGenerator` 的代码中增加断点，注意要切换为该配置进行Debug。

```json
"Generators": {  
  "commandName": "DebugRoslynComponent",  
  "targetProject": "./StatisticsService.API.csproj"  
}

```

有个缺陷是如果SourceGenerator会导致编译不通过，那还是只能回退到使用 `Visual Studio` 用 `Debug.Launch` 的方式调试。[Can not debug code generator when target project does not build : RIDER-118936](https://youtrack.jetbrains.com/issue/RIDER-118936/Can-not-debug-code-generator-when-target-project-does-not-build)