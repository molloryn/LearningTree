# IsolationCSS

## 问题排查

`Razor` 需要使用 `publish` 去构建项目，才会打包静态资源进去，直接使用 `dotnet build -c Release` 是没用的，不会把 `wwwroot` 等静态资源生成到构建产物内。

如果发现诸如`_content/Monica.AI.UI/Monica.AI.UI.qup0fznz4t.bundle.scp.css` css没有更新，那可能就是 IDE 出了bug，没有自动重新编译，没有更新项目目录下的 `staticwebassets.development.json`

这时候需要手动修改一下 `css` 文件（不能只加空白字符，可以调换下顺序，有时候改CSS不会触发重新编译，这时候可以试着改一下`.cs`文件），然后重新编译一下对应的服务，看看是否有更新。