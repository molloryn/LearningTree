### 导出当前所有安装的插件的vsix文件以供离线安装

保存为`.ps1`文件运行即可。

```powershell
[CmdLetBinding()]
Param (
    $Path = "D:\Downloads\Extensions"
)

Function Get-VSMarketPlaceExtension {
    [CmdLetBinding()]
    Param(
        [Parameter(ValueFromPipeline = $true,Mandatory = $true)]
        [string[]]
        $extensionName
    )
    begin {
        $body=@{
            filters = ,@{
            criteria =,@{
                    filterType=7
                    value = $null
                }
            }
            flags = 1712
        }
    }
    process {
        foreach($Extension in $extensionName) {
            write-verbose "getting $($extensionName)"
            $response =  try {
                $body.filters[0].criteria[0].value = $Extension
                $Query =  $body|ConvertTo-JSON -Depth 4
                (Invoke-WebRequest -Uri "https://marketplace.visualstudio.com/_apis/public/gallery/extensionquery?api-version=6.0-preview" -ErrorAction Stop -Body $Query -Method Post -ContentType "application/json")
            } catch [System.Net.WebException] {
                Write-Verbose "An exception was caught: $($_.Exception.Message)"
                $_.Exception.Response
            }
            $statusCodeInt = [int]$response.StatusCode

            if ($statusCodeInt -ge 400) {
                Write-Warning "API Error :  $($response.StatusDescription)"
                return
            }
            $ObjResults = ($response.Content | ConvertFrom-Json).results

            If ($ObjResults.resultMetadata.metadataItems.count -ne 1) {
                Write-Warning "Extension not found"
                return
            }

            $extension = $ObjResults.extensions

            $publisher = $extension.publisher.publisherName
            $extensionName = $extension.extensionName
            $version = $extension.versions[0].version

            $uri = "$($extension.Versions[0].assetUri)/Microsoft.VisualStudio.Services.VSIXPackage"
            Invoke-WebRequest -uri $uri -OutFile (Join-Path $Path "$publisher.$extensionName.$version.VSIX")
        }
    }
}

Invoke-Expression "code --list-extensions" -OutVariable extensions | Out-Null
$extensions | Get-VSMarketPlaceExtension
```



### 将当前文件用新窗口打开(移到另一屏幕)

命令名（用Ctrl+Shift+P输入）：`Open Active File in New Window`

快捷键：`Ctrl+K, O`

### 将转码后的文本保存下来

点击下面那个切换编码格式的（比如GB-2312） → 会弹出框，选择Save with encoding, 选择UTF-8.

### 自动补全function括号(parentheses)

`Ctrl + Shift + P` =\> Settings.json 或 Ctrl+, 打开Preference Setting.

```json
"typescript.suggest.completeFunctionCalls": true,
"javascript.suggest.completeFunctionCalls": true,
"C_Cpp.autocompleteAddParentheses": true "python.analysis.completeFunctionParens": true,
```


## 快捷操作

### HTML

#### Emmet abbreviation

button =\> `<button><button\>`

button\#test =\> `<button id="test"><button\>`

button.test =\> `<button class="test"><button\>`


