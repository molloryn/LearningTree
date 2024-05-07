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
