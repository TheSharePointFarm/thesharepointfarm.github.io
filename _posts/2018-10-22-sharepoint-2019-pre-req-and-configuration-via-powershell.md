---
layout: post
title: SharePoint 2019 Pre-Req and Configuration via PowerShell
tags: [sp2019]
---

SharePoint Server 2019 has some small changes to note about the prerequisites as well as the configuration of a farm via PowerShell. For the prerequisites, you'll want to download the following.

[Microsoft Sync Framework Runtime v1.0 SP1](http://go.microsoft.com/fwlink/?LinkID=224449)

[Microsoft SQL Server 2012 SP4 Native Client](https://go.microsoft.com/fwlink/?LinkId=867003)

[Windows Server AppFabric](http://go.microsoft.com/fwlink/?LinkId=235496)

[Microsoft Identity Extensions](http://go.microsoft.com/fwlink/?LinkID=252368)

[WCF Data Services 5.6 Tools](http://go.microsoft.com/fwlink/?LinkId=320724)

[Cumulative Update 7 for Microsoft AppFabric 1.1 for Windows Server](http://go.microsoft.com/fwlink/?LinkId=627257)

[Active Directory Rights Management Services Client 2.1](http://go.microsoft.com/fwlink/?LinkID=544913)

[Microsoft .NET Framework 4.7](https://go.microsoft.com/fwlink/?linkid=871868)

[Microsoft Visual C++ 2012 Redistributable (x64)](http://go.microsoft.com/fwlink/?LinkId=627156)

[Microsoft Visual C++ 2017 Redistributable (x64)](https://go.microsoft.com/fwlink/?LinkId=848299)

To install the prerequisites via PowerShell, copy them to a single folder. You'll want to fix-up the paths in this script, but you'll need to call the prerequisites installer executable on the SharePoint 2019 ISO. In this particular case, I've copied the ISO contents to C:\SharePoint2019 and the prerequisites above to a subfolder called PrerequisiteInstallerFiles.

```powershell
Start-Process "C:\SharePoint2019\PrerequisiteInstaller.exe" -ArgumentList "/SQLNCli:`"C:\SharePoint2019\PrerequisiteInstallerFiles\sqlncli.msi`" `
/Sync:`"C:\SharePoint2019\PrerequisiteInstallerFiles\Synchronization.msi`" `
/AppFabric:`"C:\SharePoint2019\PrerequisiteInstallerFiles\WindowsServerAppFabricSetup_x64.exe`" `
/IDFX11:`"C:\SharePoint2019\PrerequisiteInstallerFiles\MicrosoftIdentityExtensions-64.msi`" `
/MSIPCClient:`"C:\SharePoint2019\PrerequisiteInstallerFiles\setup_msipc_x64.exe`" `
/KB3092423:`"C:\SharePoint2019\PrerequisiteInstallerFiles\AppFabric-KB3092423-x64-ENU.exe`" `
/WCFDataServices56:`"C:\SharePoint2019\PrerequisiteInstallerFiles\WcfDataServices.exe`" `
/DotNet472:`"C:\SharePoint2019\PrerequisiteInstallerFiles\NDP472-KB4054531-Web.exe`" `
/MSVCRT11:`"C:\SharePoint2019\PrerequisiteInstallerFiles\vcredist_x64.exe`" `
/MSVCRT141:`"C:\SharePoint2019\PrerequisiteInstallerFiles\VC_redist.x64.exe`"" 
```

One other thing to note is as SharePoint Custom Help is no longer available, we no longer need to run `Install-SPHelpCollection` while creating or adding a SharePoint server to the farm. The complete set of commands to create a farm are now:

```powershell
#New Farm
#New-SPConfigurationDatabase -DatabaseName Config -DatabaseServer SPSQL -AdministrationContentDatabase Admin -FarmCredentials (Get-Credential) -Passphrase (ConvertTo-SecureString "Password1" -AsPlainText -Force) -LocalServerRole Custom
#Join Farm
#Connect-SPConfigurationDatabase -DatabaseName Config -DatabaseServer SPSQL -Passphrase (ConvertTo-SecureString "Password1" -AsPlainText -Force) -LocalServerRole Custom

Initialize-SPResourceSecurity
#New Farm
#New-SPCentralAdministration -Port 8080 -WindowsAuthProvider NTLM
Install-SPService
Install-SPFeature -AllExistingFeatures
Install-SPApplicationContent
```

The above is not the most secure way to set up Central Admin, or does it necessarily include the correct switches for creating or joining your particular farm. Adjust accordingly!