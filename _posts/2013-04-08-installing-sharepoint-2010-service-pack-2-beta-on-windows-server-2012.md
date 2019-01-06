---
layout: post
title: Installing SharePoint 2010 Service Pack 2 beta on Windows Server 2012
tags: [sp2010]
---

The SharePoint Server 2010 Service Pack 2 beta has been [released on Connect today](https://connect.microsoft.com/office/program7722).  With this public beta comes ability to install SharePoint 2010 (including Office Web Apps and Project Server 2010) on Windows Server 2012 (and Windows 8) in a supported manner.  Installing SharePoint Server 2010 SP2 (beta) must be done with newly released media as slipstreaming SP2 into a pre-SP2 installation of SharePoint media will not work.

Installation for SharePoint is similar to what every administrator is used to.  Run the pre-req installer, which will configure Windows Server roles and features, as well as install any necessary binaries for SharePoint 2010.

![prereq](/assets/images/2013/04/prereq.png)

Once completed, run setup.exe from the new media per the normal process, and install SharePoint.  This server will be installed in Farm Complete mode.

![SP2BetaInstallationProcess](/assets/images/2013/04/SP2BetaInstallationProcess.png)

Configuration has not changed, either...

![SP2BetaConfig](/assets/images/2013/04/SP2BetaConfig.png)

Our farm ends up at version 14.0.7011.1000.

PowerShell is still an issue.  The shortcut included with the installation package does not affix "-Version 2.0" to the command line, hence the SharePoint Management Shell shortcut must still be manually edited to support using the Management Shell on Windows Server 2012.

![SP2BetaPS](/assets/images/2013/04/SP2BetaPS.png)

Unlike installing pre-SP2 on Windows Server 2012, however, is that the IIS Application Pools are correctly configured to run under the .NET Framework 2.0.

![SP2BetaIISAppPools](/assets/images/2013/04/SP2BetaIISAppPools.png)

One new feature that IIS8 brings is SNI (Server Name Indicator) support.  With this support, you can leverage multiple SSL certificates against a single IP address.  For example, you can have sslhost1.domain.com and sslhost2.domain.com, using two separate SSL certificates on two separate IIS sites assigned to the same IP address.

![SPSSL1](/assets/images/2013/04/SPSSL1.png)

![SPSSL2](/assets/images/2013/04/SPSSL2.png)

This could potentially eliminate the use of Wildcard Certificates in some environments while eliminating multiple IP addresses for a farm in others.

I was unable to install SharePoint + Project Server with Office Web Apps due to an issue with the WAC install indicating that trial bits (which SharePoint and Project Server SP2 Beta are) with licensed editions (which WAC SP2 beta _should not_ be).

As I find more changes, I'll write them up here, however SP2 is meant as a cumulation of updates and security fixes that are, for the most part, already available via CUs/CODs and security hotfixes.