---
layout: post
title: Mapping Guids in IIS with Friendly Names
tags: [sp2010,sp2013]
---

To map the IIS Application Pools that appear as GUIDs (Service Application Pools) to their friendly names, run:

```powershell
$pools = Get-SPServiceApplicationPool
$AppPools = @{}
foreach($pool in $pools)
{$AppPools[$pool.DisplayName] = $pool.Id.ToString().Replace("-", "")}
$AppPools
```

This will output the friendly name of the Application Pool along with the ID of the Application Pool as you see it in IIS Manager.

For Service Applications that appear under the IIS Site commonly known as "SharePoint Web Services", run:

```powershell
$sas = Get-SPServiceApplication
$ServiceApps = @{}
foreach($sa in $sas)
{$ServiceApps[$sa.DisplayName] = $sa.IisVirtualDirectoryPath}
$ServiceApps
```

Hopefully this de-mystifies the GUIDs as shown in IIS.