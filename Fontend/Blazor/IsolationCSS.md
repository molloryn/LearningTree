# IsolationCSS

## 问题排查

`Razor` 需要使用 `publish` 去构建项目，才会打包静态资源进去，直接使用 `dotnet build -c Release` 是没用的，不会把 `wwwroot` 等静态资源生成到构建产物内。

如果发现诸如`_content/Monica.AI.UI/Monica.AI.UI.qup0fznz4t.bundle.scp.css` css没有更新，那可能就是 IDE 出了bug，没有自动重新编译，没有更新项目目录下的 `staticwebassets.development.json`