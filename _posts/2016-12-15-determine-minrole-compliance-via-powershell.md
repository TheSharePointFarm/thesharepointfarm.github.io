---
layout: post
title: Determine MinRole Compliance via PowerShell
tags: [sp2016]
---

MinRole compliance is available via Central Administration, but what if you needed to determine MinRole compliance via PowerShell? There's no simple property via `Get-SPServer` or any other similar PowerShell methods. Instead, we must use reflection. Reflection is generally unsupported by Microsoft, so do keep that in mind. This script is simple, use the SharePoint Management Shell (so we don't have to manually load the assembly) and just make sure to pass the server name into the first line.

```powershell
$server = Get-SPServer 'ServerName'
$type = [System.Type]::GetType("Microsoft.SharePoint.Administration.SPServerRoleManager,Microsoft.SharePoint, Version=16.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c")
$flags = [Reflection.BindingFlags] "NonPublic,Static"
$method = $type.GetMethod("IsServerCompliantWithRole",$flags)
$method.Invoke($null,$server)
```

It will output a true or false statement. If the value returns true, the server is in compliance with the assigned MinRole, and conversely, if the value is false, the server is not in compliance with the assigned MinRole.

If the server is using the Custom role, there will be no output. Instead, ULS will log this message.

```
SharePoint Foundation	Topology	a75h7	Medium	The role assigned to this server is 'Custom', which is not managed by MinRole. Skipping server role compliance check.
```

And that's it. Hopefully in the future Microsoft will provide an out of the box, fully supported method to get this particular detail.