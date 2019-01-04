---
layout: post
title: Installing SharePoint 2010 and SQL Server 2012 on Windows Server 2012 Release Preview
tags: [sp2010,sql]
---

As an update to my previous blog post, this installation guide will cover installing SharePoint 2010 with Service Pack 1 and SQL Server 2012 on Windows Server 2012 Release Preview.

As with Windows Server 8 Beta, you will need the following in order to install SharePoint:

[ServerManagerCmd.exe](http://blog.hand-net.com/sharepoint/2010-06-10-error-lors-de-linstallation-des-office-web-apps-2010-sur-windows-7.htm)

Slipstreamed install of SP1 for SharePoint 2010 (http://blogs.msdn.com/b/ronalg/archive/2011/07/11/slipstream-sharepoint-2010-sp1-and-language-packs-w-sp1-into-rtm.aspx and http://toddklindt.com/blog/Lists/Posts/Post.aspx?List=56f96349-3bb6-4087-94f4-7f95ff4ca81f&ID;=295&Web;=48e6fdd1-17db-4543-b2f9-6fc7185484fc)

[Microsoft Chart Controls for Microsoft .NET Framework 3.5](http://www.microsoft.com/en-us/download/details.aspx?id=14422)

[Microsoft Filter Pack 2.0](http://download.microsoft.com/download/0/A/2/0A28BBFA-CBFA-4C03-A739-30CCA5E21659/FilterPack64bit.exe) (also available in PrerequisiteInstallerFilesFilterPack)

[Microsoft SQL Server 2008 Native Client](http://go.microsoft.com/fwlink/?LinkId=123718&clcid=0x409)

[Microsoft SQL Server 2008 R2 ADOMD.NET](http://go.microsoft.com/fwlink/?LinkId=130652&clcid=0x409)

[Microsoft Sync Framework 1.0](http://www.microsoft.com/en-us/download/details.aspx?id=15391)

From Server Manager, go to Manage –> Add Roles and Features.

Add the Application Server and Web Server (IIS) roles:

![image53](/assets/images/2012/06/image53.png)

Next, add the Windows Identity Foundation 3.5 feature:

![image54](/assets/images/2012/06/image54.png)

Install the following Web Server (IIS) role features:

![image55](/assets/images/2012/06/image55.png)

![image56](/assets/images/2012/06/image56.png)

![image57](/assets/images/2012/06/image57.png)

![image58](/assets/images/2012/06/image58.png)

Install the following Application Server role features:

![image59](/assets/images/2012/06/image59.png)

First, installing SQL 2012:

Fortunately, with SQL 2012, there is nothing more that needs to be done for the SQL Database Engine, Integration Services, and SQL Management Studio.  During installation, it will enable .NET 3.5.1 which is also required for SharePoint 2010.

Next, slipstream SP1 into SharePoint 2010 if you haven’t done so already.  Copy ServerManagerCmd.exe from the above download into C:\Windows\System32.

Proceed with the SharePoint 2010 installation using setup.exe from the extracted SharePoint installation.

Run the following command from an elevated PowerShell command prompt:

```powershell
Set-WebConfigurationProperty “/system.applicationHost/applicationPools/applicationPoolDefaults” –Name managedRuntimeVersion –Value “v2.0” –PSPath IIS:\
```

When SharePoint creates application pools, they will be set to v2.0 which is currently required for SharePoint 2010.

After the installation is complete, run the SharePoint Configuration Wizard.  If you get the below error, you did not correctly run the above command to set the default Application Pool creation to v2.0.  Instead, you will need to go to IIS and set the SharePoint-related Application Pools from v4.0 to v2.0, then go and run the above PowerShell command, otherwise you’ll continue to face these types of errors.

![image40](/assets/images/2012/06/image40.png)

SharePoint 2010 PowerShell Management Shell:

In Windows Server 2012, PowerShell is version 3 by default, using the .NET 4 Framework.  This will not work for the SharePoint .NET assemblies.

In the Start screen, right click the SharePoint 2010 Management Shell link, then click Open File Location at the bottom of the screen:

![image51](/assets/images/2012/06/image51.png)

Go to Properties on the Management Shell and on the Shortcut tab add “-version 2.0” after PowerShell.exe, for example, the Target should look like:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\PowerShell.exe  -version 2.0 -NoExit  " & ' C:\Program Files\Common Files\Microsoft Shared\Web Server Extensions\14\CONFIG\POWERSHELL\Registration\\sharepoint.ps1 ' "
```

The Management Shell will now function as expected.

And yes, the User Profile Synchronization Service still functions as expected!