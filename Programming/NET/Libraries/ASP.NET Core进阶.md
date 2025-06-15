# ASP.NET Core 进阶

## ApplicationPartManager

在 ASP.NET Core 中，`ApplicationPartManager` 的核心作用是通过 ​**​Features（特性）​**​ 实现应用程序资源的动态发现和加载。这些 Feature 是用于组织应用程序元数据的​**​逻辑分组​**​，它们由 `ApplicationPartManager` 自动生成，开发者可以通过其 API 访问这些元数据。

### Features 的核心作用

1. ​**​运行时发现资源​**​  
    动态扫描 `ApplicationPart`（程序集、项目等）中的 MVC 相关组件（如控制器、视图组件、Tag Helpers）。
2. ​**​支持插件式架构​**​  
    允许动态加载外部程序集并集成其 MVC 组件。
3. ​**​优化启动性能​**​  
    避免在每次请求时反射扫描程序集，而是在启动时一次性缓存元数据。

---

### 常见的 Features 类型及其作用

以下是 `ApplicationPartManager` 管理的核心 Features：

| 特性类型 (Feature)             | 包含内容                            | 作用                                                    |
| -------------------------- | ------------------------------- | ----------------------------------------------------- |
| `ControllerFeature`        | 所有控制器的 `Type` 信息                | 注册控制器到路由系统，使框架识别请求的 Controller 类。                     |
| `TagHelperFeature`         | 所有 Tag Helpers 的 `Type` 信息      | 使 Razor 视图能识别和使用自定义 Tag Helpers（如 `<my-custom-tag>`）。 |
| `ViewComponentFeature`     | 所有视图组件（View Components）的 `Type` | 允许在视图中通过 `@await Component.InvokeAsync(...)` 调用组件。    |


---

### 工作机制

1. ​**​填充 Feature​**​  
    通过 `ApplicationPartManager.PopulateFeature<TFeature>()` 方法获取特定类型的 Feature 数据：
    
    ```cs
    var controllerFeature = new ControllerFeature();
    applicationPartManager.PopulateFeature(controllerFeature);
    // controllerFeature.Controllers 包含所有控制器类型
    ```
    
2. ​**​与 MVC 集成​**​  
    MVC 框架在启动时自动使用 `ApplicationPartManager` 发现并注册控制器、Tag Helpers 等组件：
    
    ```cs
    services.AddControllersWithViews()
        .ConfigureApplicationPartManager(apm =>
        {
            apm.ApplicationParts.Add(new AssemblyPart(pluginAssembly)); // 添加外部程序集
            apm.FeatureProviders.Add(new CustomFeatureProvider());       // 自定义发现逻辑
        });
    ```


## Application Model

代表了 MVC 应用程序的组件结构，通过灵活的定制机制控制 MVC 元素的行为。
[Work with the application model in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/application-model?view=aspnetcore-9.0)

---

#### ​**​核心概念​**​

1. ​**​模型结构​**​：
    
    ```
    ApplicationModel
    ├─ Controllers (ControllerModel)
    │    ├─ Actions (ActionModel)
    │    │    └─ Parameters (ParameterModel)
    │    └─ Properties (全局属性)
    └─ Properties (应用级属性)
    ```
    
    - 每层都有 `Properties` 集合，支持继承和覆盖
    - 属性最终存储在 `ActionDescriptor.Properties` 中
2. ​**​提供者模式​**​：
    
    - 框架通过 `IApplicationModelProvider` 加载模型
    - 内置提供者：
        - `DefaultApplicationModelProvider`（-1000）：基础行为
        - `AuthorizationApplicationModelProvider`（-990）：授权处理
        - `CorsApplicationModelProvider`（-990）：CORS 处理
3. ​**​约定(Conventions)​**​：
    
    - 替代提供者的推荐方式
    - 接口：
        - `IApplicationModelConvention`
        - `IControllerModelConvention`
        - `IActionModelConvention`
        - `IParameterModelConvention`

---

#### ​**​实用场景与代码示例​**​

1. ​**​全局属性添加​**​
    
    ```cs
    public class AppDescriptionConvention : IApplicationModelConvention
    {
        public void Apply(ApplicationModel application)
        {
            application.Properties["description"] = "订单管理系统";
        }
    }
    
    // Startup.cs
    services.AddMvc(options => 
        options.Conventions.Add(new AppDescriptionConvention()));
    ```
    
2. ​**​控制器级定制​**​
    
    ```cs
    [ControllerDescription("用户管理模块")]
    public class UserController : Controller
    {
        public IActionResult Index() 
            => Content($"模块描述: {ControllerContext.ActionDescriptor.Properties["description"]}");
    }
    
    // 特性实现
    public class ControllerDescriptionAttribute : Attribute, IControllerModelConvention
    {
        private readonly string _desc;
        public void Apply(ControllerModel model) 
            => model.Properties["description"] = _desc;
    }
    ```
    
3. ​**​路由命名空间集成​**​
    
    ```cs
    public class NamespaceRouteConvention : IApplicationModelConvention
    {
        public void Apply(ApplicationModel application)
        {
            foreach (var controller in application.Controllers)
            {
                if (controller.ControllerName.EndsWith("Api"))
                {
                    controller.Selectors[0].AttributeRouteModel = new AttributeRouteModel
                    {
                        Template = controller.ControllerType.Namespace?
                            .Replace('.', '/') + "/[controller]/[action]"
                    };
                }
            }
        }
    }
    ```
    
    - 将 `Admin.UserApi` 命名空间转换为 `/Admin/UserApi/Index` 路由
4. ​**​参数绑定控制​**​
    
    ```cs
    public class RouteParamOnlyAttribute : Attribute, IParameterModelConvention
    {
        public void Apply(ParameterModel model)
        {
            model.BindingInfo ??= new BindingInfo();
            model.BindingInfo.BindingSource = BindingSource.Path;
        }
    }
    
    // 使用示例
    public IActionResult Get([RouteParamOnly] int id) { ... }
    ```
    
5. ​**​API 文档控制​**​
    
    ```cs
    public class ApiVisibilityConvention : IApplicationModelConvention
    {
        public void Apply(ApplicationModel application)
        {
            foreach (var controller in application.Controllers)
            {
                // 仅暴露名称包含"PublicApi"的控制器
                controller.ApiExplorer.IsVisible = 
                    controller.ControllerName.Contains("PublicApi");
            }
        }
    }
    ```
    

---

#### ​**​关键要点​**​

1. ​**​启动注册​**​：
    
    ```cs
    services.AddMvc(options => 
    {
        options.Conventions.Add(new CustomConvention());
        options.Conventions.Add(new AnotherConvention());
    });
    ```

