# IdentityServer4

#### Swagger认证

需要RedirectUris一致。可在ClientRedirectUris表中看到。
```cs
     RedirectUris = {
         $"{clientUrlDict["ListApi"]}/swagger/oauth2-redirect.html",
         ReplaceToLocalhost($"{clientUrlDict["ListApi"]}/swagger/oauth2-redirect.html"),
     },
     PostLogoutRedirectUris = {
         $"{clientUrlDict["ListApi"]}/swagger/",
         ReplaceToLocalhost($"{clientUrlDict["ListApi"]}/swagger/"),
     },

```

用户账号在 AspNetUsers 表

# 术语

## OAuth2.0

涉及名词定义：
- **Third-party application OR Client**：第三方应用程序，前端或客户端。
- **HTTP service**：HTTP服务提供商，也即后端服务或目标服务器。
- **Resource Owner**：资源所有者，一般是指某个用户。
- **User Agent**：用户代理，本文中就是指浏览器。
- **Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器。
- **Resource server**：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。


[What is OAuth 2.0 and what does it do for you? - Auth0](https://auth0.com/intro-to-iam/what-is-oauth-2)
[理解OAuth 2.0 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
OAuth 2.0 is an authorization protocol and NOT an authentication protocol. As such, it is designed primarily as a means of granting access to a set of resources, for example, remote APIs or user data.

OAuth 2.0 uses Access Tokens. An **Access Token** is a piece of data that represents the authorization to access resources on behalf of the end-user. OAuth 2.0 doesn’t define a specific format for Access Tokens. However, in some contexts, the JSON Web Token (JWT) format is often used. This enables token issuers to include data in the token itself. Also, for security reasons, Access Tokens may have an expiration date.

**简单说，OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用(也可以理解是前端)进入系统（后端），获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。**

它规定四种方式获取令牌：
- 授权码（authorization-code）
- 隐藏式（implicit）
- 密码式（password）
- 客户端凭证（client credentials）

Token的生成方式或类型：Bearer Token
## Bearer Token

一个加密字符串，客户端拿到Token后，存储起来。在下次请求接口的时候，在请求头中包含`Authorization: Bearer <token>`。服务端收到请求后解析Token，验证Token合法性。

#### JWT
对`Bearer Token`的实现。
[JSON Web Token 入门教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

JWT 的原理是，服务器认证以后，生成一个 JSON 对象，发回给用户，就像下面这样。
```js
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```
以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名（详见后文）。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

## OpenID
[OpenID Connect 是什麼？ | HENNGE Taiwan 部落格](https://hennge.com/tw/blog/what-is-openid-connect.html)
基于`OAuth2.0`的单点登录技术。
以下两个术语有了新的名称：
- **Third-party application OR Client**：第三方应用程序，前端或客户端。现在称为 **Relying Party**
- **Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器。现在称为 **OpenID Provider** 除了可以放置 AccessToken 之外，也可以存放`ID Token`

### ID Token
使用 JWS （JSON Web Signature, [RFC 7515](https://tools.ietf.org/html/rfc7515)）的规范定义。

> BASE64URL(UTF8(JWS Protected Header)) || '.' ||  
> BASE64URL(JWS Payload) || '.' ||  
> BASE64URL(JWS Signature)

#### Header 
![](../../attachments/Pasted%20image%2020240707180409.png)

#### Data
![](../../attachments/Pasted%20image%2020240707180418.png)

#### Signature 
二进制数据，用来验证Header与Payload内容的完整性。

# Authentication(身份认证)

> the process of determining a user's identity.

In ASP.NET Core, authentication is handled by the authentication service, [IAuthenticationService](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.iauthenticationservice), which is used by authentication [middleware](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0). The authentication service uses registered authentication handlers to complete authentication-related actions. Examples of authentication-related actions include:

- Authenticating a user.
- Responding when an unauthenticated user tries to access a restricted resource.

## 术语
### Schemes(方案)

> The registered authentication handlers and their configuration options

such as [AddJwtBearer](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer) or [AddCookie](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.cookieextensions.addcookie). These extension methods use [AuthenticationBuilder.AddScheme](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationbuilder.addscheme) to register schemes with appropriate settings.

"JwtBearer" refers to a type of authentication scheme in ASP.NET Core. It is used for authenticating users based on JSON Web Tokens (JWT).


> OIDC stands for OpenID Connect, which is a standard protocol for authentication. It is used to verify the identity of users based on authentication performed by an authorization server.


## 实现

### AuthenticationManager 认证管理器

位于HTTPContext中，身份认证中间件。

```cs
public abstract class AuthenticationManager
{

    //AuthenticateContext包含了需要认证的上下文，里面就有CurrentUser，也就是ClaimsPrincipal
    public abstract Task AuthenticateAsync(AuthenticateContext context);
    
    //握手
    public abstract Task ChallengeAsync(string authenticationScheme, AuthenticationProperties properties, ChallengeBehavior behavior);
    
    //登入
    public abstract Task SignInAsync(string authenticationScheme, ClaimsPrincipal principal, AuthenticationProperties properties);
    
    //登出
    public abstract Task SignOutAsync(string authenticationScheme, AuthenticationProperties properties);
}

```

其实现有CookieAuthentication等，详见：[ASP.NET Core 之 Identity 入门（二） - Savorboard - 博客园 (cnblogs.com)](https://www.cnblogs.com/savorboard/p/aspnetcore-identity2.html)

### AuthenticationHttpContextExtensions

AuthenticationHttpContextExtensions 类是对 HttpContext 认证相关的扩展，它提供了如下扩展方法：

- **SignInAsync** 用户登录成功后颁发一个证书（加密的用户凭证），用来标识用户的身份。
    
- **SignOutAsync** 退出登录，如清除Coookie等。
    
- **AuthenticateAsync** 验证在 `SignInAsync` 中颁发的证书，并返回一个 `AuthenticateResult` 对象，表示用户的身份。
    
- **ChallengeAsync** 返回一个需要认证的标识来提示用户登录，通常会返回一个 `401` 状态码。
    
- **ForbidAsync** 禁上访问，表示用户权限不足，通常会返回一个 `403` 状态码。
    
- **GetTokenAsync** 用来获取 `AuthenticationProperties` 中保存的额外信息。


```cs
public static class AuthenticationHttpContextExtensions
{
    public static Task<AuthenticateResult> AuthenticateAsync(this HttpContext context, string scheme) =>
        context.RequestServices.GetRequiredService<IAuthenticationService>().AuthenticateAsync(context, scheme);

    public static Task ChallengeAsync(this HttpContext context, string scheme, AuthenticationProperties properties) { }
    public static Task ForbidAsync(this HttpContext context, string scheme, AuthenticationProperties properties) { }
    public static Task SignInAsync(this HttpContext context, string scheme, ClaimsPrincipal principal, AuthenticationProperties properties) {}
    public static Task SignOutAsync(this HttpContext context, string scheme, AuthenticationProperties properties) { }
    public static Task<string> GetTokenAsync(this HttpContext context, string scheme, string tokenName) { }
}

```


### AddAuthenticationCore

**AddAuthenticationCore** 中注册了认证系统的三大核心对象：`IAuthenticationSchemeProvider`，`IAuthenticationHandlerProvider` 和 `IAuthenticationService`，以及一个对Claim进行转换的 IClaimsTransformation(不常用)

```cs
public static class AuthenticationCoreServiceCollectionExtensions
{
    public static IServiceCollection AddAuthenticationCore(this IServiceCollection services)
    {
        services.TryAddScoped<IAuthenticationService, AuthenticationService>();
        services.TryAddSingleton<IClaimsTransformation, NoopClaimsTransformation>(); // Can be replaced with scoped ones that use DbContext
        services.TryAddScoped<IAuthenticationHandlerProvider, AuthenticationHandlerProvider>();
        services.TryAddSingleton<IAuthenticationSchemeProvider, AuthenticationSchemeProvider>();
        return services;
    }
}

```


- `IAuthenticationSchemeProvider` 用来提供对Scheme的注册和查询。Scheme 用来标识使用的是哪种认证方式（如cookie, bearer, oauth, openid 等等）
- 

# Authorization(授权)

## 术语
### Cliams
"Claims" in the context of ASP.NET Core authentication typically refers to the assertions about a user that a system receives and uses to make decisions about access or behavior. These claims can include information like the user's name, email, roles, or any other relevant data that can be used for authorization purposes within the application. In ASP.NET Core, these claims are used during the authentication process to determine the user's identity and authorization levels.

[ASP.NET Core 认证与授权 初识认证 - 雨夜朦胧 - 博客园 (cnblogs.com)](https://www.cnblogs.com/RainingNight/p/introduce-basic-authentication-in-asp-net-core.html)

[ASP.NET Core 之 Identity 入门（一） - Savorboard - 博客园 (cnblogs.com)](https://www.cnblogs.com/savorboard/p/aspnetcore-identity.html)

简单理解抽象定义：
- 证件单元（Claims） 
- 证件（ClaimsIdentity）
- 证件当事人（ClaimsPrincipal）

 
```cs
// 定义证件对象的基本功能。
public interface IIdentity
{
    //证件名称
    string Name { get; }
    
    // 用于标识证件的载体类型。
    string AuthenticationType { get; }
    
    //是否是合法的证件。
    bool IsAuthenticated { get; }
}

public class ClaimsIdentity : IIdentity
{
    //......
}

public class ClaimsPrincipal 
{
    //当事人可以拥有多个证件
    public ClaimsPrincipal(IEnumerable<ClaimsIdentity> identities){}
    
    //当事人的主证件，即身份证
    public virtual IIdentity Identity { get; }
    
    public virtual IEnumerable<ClaimsIdentity> Identities { get; }
    
    public virtual void AddIdentity(ClaimsIdentity identity);

}
```


![](../../attachments/Pasted%20image%2020240627160448.png)

