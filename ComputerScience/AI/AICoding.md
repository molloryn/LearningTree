
# AI  Coding



## 记忆

### Blazor

- 在Blazor中使用MudBlazor的Icon属性时，必须使用@开头引用，例如Icon="@Icons.Material.Filled.Info"，而不是Icon="Icons.Material.Filled.Info"。不使用@开头引用会导致Icon不生效。
- 在Blazor中，JavaScript互操作调用不能在OnInitializedAsync中进行，因为组件在静态渲染期间JavaScript互操作调用只能在OnAfterRenderAsync生命周期方法中执行。同时，大部分耗时的接口初始化工作也不应该放在OnInitializedAsync中，因为这会导致页面阻塞和空白等待。正确做法是在OnAfterRenderAsync(bool firstRender)中进行JavaScript互操作和耗时初始化，使用firstRender参数确保只在首次渲染时执行。



## 规则

- If you find that information is missing or you need help, please stop immediately and output your needs, instead of constantly trying messy methods to reduce code quality.
- Write reusable, low-coupling and high-cohesion implementations with multiple abstractions. Split files to avoid overly large single files. Instead of just fixing errors, use simplified thinking and refactor whenever possible.
- Always generate well-structured documentation comments for every necessary element when coding C#.