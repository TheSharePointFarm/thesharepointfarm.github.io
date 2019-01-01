---
layout: post
title: Iisreset Not Required After Starting User Profile Sync Service
tags: [sp2010, sp2013]
---

Instead of running an `iisreset` after starting (or re-starting) the User Profile Synchronization Service when Central Administration is running on the same SharePoint Server as the UPSS, you can simply recycle the Central Administration Application Pool.  To do that through PowerShell, use:

```powershell
$appPool = gwmi -Namespace "root\MicrosoftIISv2" -class "IIsApplicationPool" | where {$_.Name -eq "W3SVC/APPPOOLS/SharePoint Central Administration v4"}
$appPool.Recycle()
```

Once you do this, you can manage the Synchronization Connections as well as initiate an Incremental/Full Sync.  You’ll also notice that the Profile Synchronization Status is Idle and not “unprovisioned”.