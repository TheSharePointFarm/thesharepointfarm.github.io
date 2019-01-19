---
layout: post
title: Unable to provision the Claims to Windows Token Service
tags: [sp2010,sp2013]
---

The Claims to Windows Token Service is stopped by default. Typically, starting this service is easy. In fact, I've never come across a situation where it didn't "just start", until now! In [this thread](http://social.technet.microsoft.com/Forums/office/en-US/1ea61b72-97ca-4df3-8440-ff1b58cfe46a) on the SharePoint TechNet forums, a user was unable to provision the Claims to Windows Token Service due to the error "Illegal characters in path". With no further information, it is difficult to identify _which_ path this error is referring to. However, we have .NET Reflector! So what we do is simply take a look at the full stack trace of the error. Within the stack trace on the thread, you can see `Illegal characters in path. at System.IO.Path.GetFileName`, so clearly we're dealing with the system attempting to retrieve a file name. Down further in the stack trace, we come across the first mention of a SharePoint related method, `Microsoft.SharePoint.Administration.Claims.SPWindowsTokenServiceInstance.Provision()`. This is the method we'll reflect.

When looking at the `Provision()` method, it is easily identifiable as to what path we're looking for.

```csharp
public override void Provision()
{
    string filename = (string) Registry.GetValue(@"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\c2wts", "ImagePath", null);
...
}
```

Clearly we need to look at the registry entry to make sure it is valid. In this case, `HKLM\SYSTEM\CurrentControlSet\services\c2tws` should have an `ImagePath` value of `%ProgramFiles%\Windows Identity Foundation\v3.5\c2wtshost.exe`. In the case of the poster on TechNet, the value was `%ProgramFiles%\"Windows Identity Foundtation"\v3.5\c2wtshost.exe`. Note the quotes within the path.

While I'm unsure as to how the value ended up with quotes, removing the quotes allowed the service to start successfully.