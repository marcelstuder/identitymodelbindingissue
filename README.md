# identitymodelbindingissue

This application demonstrates the issue reported at: https://github.com/IdentityModel/IdentityModel2/issues/154    

## Issue details

Using IdentityServer4.AccessTokenValidation works perfectly fine with IdentityModel Version 3.9.0 but after upgrading to IdentityModel 3.10.1 the
call to `AddIdentityServerAuthentication` in `Startup.cs` causes the following exception:

```
Could not load file or assembly 'IdentityModel, Version=3.6.0.0, Culture=neutral, PublicKeyToken=null' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference. (Exception from HRESULT: 0x80131040).
``` 

* ASP.NET Core Web Application targeting .NET Framework 4.7.2 causes an exception due to a missing bindingredirect
* AutoGenerateBindingRedirects is set to `true` and binding redirects are being created (after building the solution there is a `IdentityModelBindingIssue.exe.config` file in the `bin\debug\net472` folder
that contains a lot of binding redirects)
* IdentityServer4.AccessTokenValidation package depends on IdentityModel.AspNetCore.OAuth2Introspection which depends on IdentityModel 3.8.0
* No binding redirect gets created for the transient dependency IdentityModel 3.8.0 (see commit 15f80e0e4a72b5653e6821372cb14f0081a49b5e)

## Attempts to fix

### Attempt 1

* called `sn -T identitymodel.dll` to get the public key token: `e7877f4675df049f`
* manually added a web.config file and specified a binding redirect for IdentityModel (see latest commit)

#### Result

* BindingRedirect from web config gets merged into automatically generated binding redirects in `IdentityModelBindingIssue.exe.config` but the problem remains

### Attempt 2

* disabled automatic binding redirects

#### Results 1

* started program, some other assembly-load-exceptions occured (as expected)
* added a web.config and put all bindingredirects in there - they were ignored

#### Results 2

* removed web.config and added an app.config with bindingredirects 
* app started up to the call to `AddIdentityServerAuthentication` and then the addembly load exception for the IdentityModel.dll reappeared


