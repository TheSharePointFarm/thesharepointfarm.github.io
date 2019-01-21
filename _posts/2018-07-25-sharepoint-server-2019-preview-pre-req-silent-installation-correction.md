---
layout: post
title: SharePoint Server 2019 Preview Pre-Req Silent Installation Correction
tags: [sp2016]
---

The instructions for installing the pre-reqs aren't correct if you look at the PrerequisiteInstaller.exe /? help. Instead, use the switches below.

```powershell
PrerequisiteInstaller.exe /SQLNCli:<path> /Sync:<path> /AppFabric:<path> /IDFX11:<path> /MSIPCClient:<path> /KB3092423:<Path> /WCFDataServices56:<path> /DotNot472:<path> /MSVCRT141:<path>
```

There is one addition, MSVCRT11 that isn't a valid switch, but the file is still required.

Lastly, here are the links for the files required.

Microsoft Sync Framework Runtime v1.0 SP1 (x64) (Synchronization.msi): <http://go.microsoft.com/fwlink/?LinkID=224449>
Microsoft SQL Server 2012 SP4 Native Client (sqlncli.msi): <https://go.microsoft.com/fwlink/?LinkId=867003>
Windows Server AppFabric (WindowsServerAppFabricSetup_x64.exe): <http://go.microsoft.com/fwlink/?LinkId=235496>
Microsoft Identity Extensions (MicrosoftIdentityExtensions-64.msi): <http://go.microsoft.com/fwlink/?LinkID=252368>
WCF Data Services 5.6 Tools (WcfDataServices.exe): <http://go.microsoft.com/fwlink/?LinkId=320724>
Cumulative Update 7 for Microsoft AppFabric 1.1 for Windows Server (AppFabric-KB3092423-x64-ENU.exe): <http://go.microsoft.com/fwlink/?LinkId=627257>
Active Directory Rights Management Services Client 2.1 (setup_msipc_x64.exe): <http://go.microsoft.com/fwlink/?LinkID=544913>
Microsoft .NET Framework 4.7 (NDP472-KB4054531-Web.exe): <https://go.microsoft.com/fwlink/?linkid=871868>
Microsoft Visual C++ 2012 Redistributable (x64) (vcredist_x64.exe): <http://go.microsoft.com/fwlink/?LinkId=627156>
Microsoft Visual C++ 2017 Redistributable (x64) (VC_redist.x64.exe): <https://go.microsoft.com/fwlink/?LinkId=848299>