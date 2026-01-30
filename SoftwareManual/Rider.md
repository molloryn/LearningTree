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


### SourceGenerator控制生成目录
默认设置代码生成路径会在 `C:\Users\{Username}\AppData\Local\Temp\SourceGeneratedDocuments\1415ABEDF6AED9378BC5BC80\Monica.Generators.AutoController\Monica.Generators.AutoController.HttpApiControllerSourceGenerator` 诸如此类的路径

通过设置 `.csproj` 文件：
```xml
<PropertyGroup>  
    <EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>  
    <CompilerGeneratedFilesOutputPath>Generated</CompilerGeneratedFilesOutputPath>  
</PropertyGroup>  
  
<ItemGroup>  
    <!-- Exclude the output of source generators from the compilation -->  
    <Compile Remove="$(CompilerGeneratedFilesOutputPath)/**/*.cs" />  
</ItemGroup>
```
可以生成到设置的项目目录下。

如果不设置 `CompilerGeneratedFilesOutputPath`，默认是在 `{BaseIntermediateOutpath}/generated/{Assembly}/{SourceGeneratorName}/{GeneratedFile}`

设置了就是在 
`{CompilerGeneratedFilesOutputPath}/{Assembly}/{SourceGeneratorName}/{GeneratedFile}`
[Always Developing - Customize source generator output location](https://www.alwaysdeveloping.net/dailydrop/2022/02/22-emit-source-generator-files/)