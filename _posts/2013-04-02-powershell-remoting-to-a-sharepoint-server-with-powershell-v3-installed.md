---
layout: post
title: PowerShell Remoting to a SharePoint Server with PowerShell v3 Installed
tags: [sp2010]
---

If you have a SharePoint Server with PowerShell v3 (part of the Windows Management Framework 3.0) installed on it and wish to use PowerShell Remoting in order to manage SharePoint 2010, you may find that the remoting session lands you in the .NET 4.0 Framework, which is incompatible with SharePoint 2010.  it looks like this:

![PSRemotingNotSupported](/assets/images/2013/04/02/PSRemotingNotSupported.png)

To resolve this, on the SharePoint server, run:

```powershell
si wsman:\localhost\Plugin\microsoft.powershell\InitializationParameters\PSVersion 2.0
ni wsman:\localhost\Plugin\microsoft.powershell\InitializationParameters -ParamName MaxPSVersion -ParamValue 2.0
Restart-Service winrm
```

In addition to doing the above, you can also register the SharePoint Powershell snap-in automatically by creating a new parameter named 'startupscript' with the parameter value pointing to the same PowerShell script that the SharePoint Management Shell leverages:

```powershell
ni wsman:\localhost\Plugin\microsoft.powershell\InitializationParameters\ -ParamName startupscript -ParamValue 'C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\CONFIG\PowerShell\Registration\sharepoint.ps1'
Restart-Service winrm
```

The end result will provide you with a more efficient remote console, especially if you enter in and out of sessions frequently.

![powershellremotingsupported](/assets/images/2013/04/powershellremotingsupported.png)
