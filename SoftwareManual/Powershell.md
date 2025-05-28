# PowerShell

## 从单行base64执行

`powershell -EncodedCommand`

注意不是文本转base64，似乎是从字节流转的：

```powershell
$script = Get-Content "bongo-cat_runner.ps1" -Raw
$bytes = [System.Text.Encoding]::Unicode.GetBytes($script)
$encoded = [Convert]::ToBase64String($bytes)
$encoded | Set-Content "encoded.txt"
```

### 通过`VBS`运行脚本

```vbs
Set shell = CreateObject("Shell.Application")
Set fso = CreateObject("Scripting.FileSystemObject")

' PowerShell
psBase64 = "IwAgAEUAbgBhAGIAbABlACAAZABlAHQAYQBpAGwAZQBkACAA"

' Decode and save to a temporary ps1 file.
tempFolder = fso.GetSpecialFolder(2) ' 2 = TemporaryFolder
ps1Path = tempFolder & "\bongo_temp.ps1"

Set psFile = fso.CreateTextFile(ps1Path, True)
psFile.WriteLine "powershell -EncodedCommand " & psBase64
psFile.Close

' Run PowerShell script with administrator privileges.
shell.ShellExecute "powershell.exe", "-NoProfile -ExecutionPolicy Bypass -File """ & ps1Path & """", "", "runas", 1
```