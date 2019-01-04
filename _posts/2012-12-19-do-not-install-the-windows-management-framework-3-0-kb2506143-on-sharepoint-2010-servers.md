---
layout: post
title: Do not install the Windows Management Framework 3.0 (KB2506143) on SharePoint 2010 Servers
tags: [sp2010]
---

The [Windows Management Framework 3.0 (KB2506143)](http://www.microsoft.com/en-us/download/details.aspx?id=34595) includes PowerShell 3.0.  Because SharePoint 2010 is based off of the .NET 3.5 Framework, it is incompatible with PowerShell 3.0, which is based off of the .NET 4.0 Framework.  Installing this Optional update from Microsoft Update or the Microsoft Download Center will cause the SharePoint Management Shell to throw an error:

```powershell
Add-PsSnapin : Incorrect Windows PowerShell version 3.0. Windows PowerShell ver
sion 2.0 is supported in the current console.
At C:\program files\common Files\microsoft shared\Web server extensions\14\conf
ig\powershell\registration\sharepoint.ps1:3 char:13
+ Add-PsSnapin <<<<  Microsoft.SharePoint.PowerShell
    + CategoryInfo          : InvalidArgument: (Microsoft.SharePoint.PowerShel
   l:String) [Add-PSSnapin], PSArgumentException
    + FullyQualifiedErrorId : AddPSSnapInRead,Microsoft.PowerShell.Commands.Ad
   dPSSnapinCommand
```

Uninstalling and rebooting the SharePoint Server will resolve this issue.

WMF 3.0 is also incompatible with Exchange 2007 and 2010, as well as some System Center products.

EDIT: The PowerShell team has put together a list of products with known compatibility issues as well as temporarily removed the WMF 3.0 from Windows Update:

[Windows Management Framework 3.0 Compatibility Update](http://blogs.msdn.com/b/powershell/archive/2012/12/20/windows-management-framework-3-0-compatibility-update.aspx)

EDIT2: And now a knowledge base article on this:

[SharePoint 2010 Management Shell does not load with Windows PowerShell 3.0](http://support.microsoft.com/kb/2796733)