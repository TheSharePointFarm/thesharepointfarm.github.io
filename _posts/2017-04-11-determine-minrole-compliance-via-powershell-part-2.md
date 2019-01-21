---
layout: post
title: 
tags: [sp2016]
---

With the release of the April 2017 Public Update, it is now possible to determine MinRole Compliance via PowerShell without any reflection! The process is simple, and [as before]({ % post_url 2016-12-15-determine-minrole-compliance-via-powershell %}), it returns a boolean value of true if compliant, false if not, and no value is returned when using the Custom role.

```powershell
$server = Get-SPServer serverName
$server.CompliantWithMinRole
```

To check on all servers in the farm, use:

``powershell
Get-SPServer | ?{$_.Role -ne 'Invalid'} | Select Name,Role,CompliantWithMinRole
```

The output will be similar to:

```powershell
Name                              Role CompliantWithMinRole
----                              ---- --------------------
CASP01 WebFrontEndWithDistributedCache                 True
CASP02 WebFrontEndWithDistributedCache                 True
CASP03           ApplicationWithSearch                 True
CASP04           ApplicationWithSearch                 True
```

And that's it!