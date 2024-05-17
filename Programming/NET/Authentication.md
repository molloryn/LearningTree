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


# Authentication(身份认证)

> the process of determining a user's identity.

In ASP.NET Core, authentication is handled by the authentication service, [IAuthenticationService](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.iauthenticationservice), which is used by authentication [middleware](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0). The authentication service uses registered authentication handlers to complete authentication-related actions. Examples of authentication-related actions include:

- Authenticating a user.
- Responding when an unauthenticated user tries to access a restricted resource.

## Schemes(方案)

> The registered authentication handlers and their configuration options

such as [AddJwtBearer](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer) or [AddCookie](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.cookieextensions.addcookie). These extension methods use [AuthenticationBuilder.AddScheme](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authentication.authenticationbuilder.addscheme) to register schemes with appropriate settings.

"JwtBearer" refers to a type of authentication scheme in ASP.NET Core. It is used for authenticating users based on JSON Web Tokens (JWT).

# Authorization(授权)


## Cliams
"Claims" in the context of ASP.NET Core authentication typically refers to the assertions about a user that a system receives and uses to make decisions about access or behavior. These claims can include information like the user's name, email, roles, or any other relevant data that can be used for authorization purposes within the application. In ASP.NET Core, these claims are used during the authentication process to determine the user's identity and authorization levels.
