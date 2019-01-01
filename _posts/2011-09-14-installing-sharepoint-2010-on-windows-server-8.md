---
layout: post
title: Installing SharePoint 2010 on Windows Server 8
tags: [sp2010]
---

Installing SharePoint 2010 on Windows Server 8 isn't quite as straight forward as it is on Server 2008 R2.  The SQL Server 2008 R2 Database Engine, Full Text Search, and Management Components install without any compatibility notice.

![image2](/assets/images/2011/09/14/images2.png)

The SharePoint 2010 Products Preparation Tool, however, will not run properly.  It will fail when attempting to configure IIS.  You'll need to install the following manually:

![image17](/assets/images/2011/09/14/images17.png)

![image20](/assets/images/2011/09/14/images20.png)

![image23](/assets/images/2011/09/14/images23.png)

![image26](/assets/images/2011/09/14/images26.png)

![image29](/assets/images/2011/09/14/images29.png)

A manual installation of ADOMD 2008 (SQLSERVER2008_ASADOMD10.msi), MS Chart for .NET 3.5 SP1 (MSChart.exe), Microsoft Sync Framework (Synchronization.msi) and the Microsoft Filter Pack 2.0 (FilterPack.msi – can be found on the SharePoint DVD) are required.

Create a SharePoint 2010 SP1 Slipstreamed installation point (http://blogs.msdn.com/b/ronalg/archive/2011/07/11/slipstream-sharepoint-2010-sp1-and-language-packs-w-sp1-into-rtm.aspx).  Run Setup.exe.  Attempting to install SharePoint 2010 will yield this error:

![image31](/assets/images/2011/09/14/images31.png)

The reason for this is that the SharePoint Setup attempts to execute a script passed to ServerManagerCmd.exe.  This is no longer present in Windows Server 8, and is instead replaced by PowerShell scripts.

To get around this, use the `ServerManagerCmd.exe` from http://blog.hand-net.com/sharepoint/2010-06-10-error-lors-de-linstallation-des-office-web-apps-2010-sur-windows-7.htm, or compile the code into a C# console application with the name `ServerManagerCmd.exe` and drop it in `\Windows\System32`.  Install SharePoint as you normally would.

The Config Wizard failed when attempting to provision the Central Administration site with a few errors about the web.config.

![image38](/assets/images/2011/09/14/images38.png)

The issue here is that the Application Pools created during the SharePoint installation are all v4.0.  SharePoint only supports Pools running v2.0 (ASP.NET 3.5).  Change the SecurityTokenService, SharePoint Web Services, the Pool hosting the Topology service (will be a randomly string of characters/numbers), and SharePoint Central Administration:

![image41](/assets/images/2011/09/14/images41.png)

You should now be able to log into Central Administration.  I have not tested any Service Applications.  For new Web Applications, you’ll want to pre-stage your Application Pool with .NET 2.0 in Integrated mode.  Changing the .NET Framework Version on the Server node of IIS does not appear to change the version new Application Pools are created as.

But yes, the User Profile Synchronization Service does run :-)