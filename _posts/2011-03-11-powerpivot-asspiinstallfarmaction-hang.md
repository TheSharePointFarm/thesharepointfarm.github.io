---
layout: post
title: PowerPivot ASSPIInstallFarmAction Hang
tags: [sp2010, powerpivot]
---

3/23 - Update - Please see [this post]({% post_url 2011-03-23-powerpivot-asspiinstallfarmaction-hang-update %}) for a proper workaround

If PowerPivot is stuck on the ASSPIInstallFarmAction installation step, see if `STSADM.exe` is currently running.  Also check `C:\Program Files\Microsoft SQL Server\100\Setup Bootstrap\Log\<date>\Detail.txt`.  If the last line is:

```text
<date_time> AS: Running Script: & "C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\BIN\psconfig.exe" -cmd services -install
```

Manually deploy `powerpivotfarm.wsp` globally and `powerpivotwebapp.wsp` to the Central Administration site.  Terminate `STSADM` from Task Manager.  The PowerPivot installation will see that the solutions have already been deployed and will continue with the installation..  PowerPivot should finish successfully.