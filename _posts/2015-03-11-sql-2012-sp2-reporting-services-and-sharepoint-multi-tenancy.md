---
layout: post
title: SQL 2012 SP2, Reporting Services, and SharePoint Multi-Tenancy
tags: [sp2010,sp2013,ssrs]
---

EDIT: [SSRS is officially not supported with multi-tenancy.](https://msdn.microsoft.com/en-us/library/jj219068(v=sql.110).aspx)

SharePoint has a mode known as multi-tenancy. This allows you to have multiple 'tenants', or isolated data sets for multiple customers. This mode is similar to how SharePoint Online operates. SQL Server Reporting Services does work in multi-tenant mode, until you install SSRS 2012 Sp2.

While there is an explicit "no support" statement for [SSRS 2014](https://msdn.microsoft.com/en-us/library/jj219068.aspx), this note does not (yet) exist for [SSRS 2012](https://msdn.microsoft.com/en-us/library/jj219068(v=sql.110).aspx) (or for the installing SSRS on [SharePoint 2010 SSRS 2012](https://msdn.microsoft.com/en-us/library/gg492276(v=sql.110).aspx) and [SSRS 2014](https://msdn.microsoft.com/en-us/library/gg492276(v=sql.120).aspx) articles). With SSRS 2012 pre-SP2, multi-tenancy functions as expected. After applying SP2 (tested up to CU6), the value of Globals.SPSiteSubscriptionId is a null value. This value must be equal to the SPSite.SiteSubscription value in order to function properly.

![SSRS2012Sp2MT](/assets/images/2015/03/SSRS2012Sp2MT.png)

After opening a case on this, this appears to be a non-tested, unsupported usage scenario for SQL Reporting Services. A bug report has been filed, but my assumption is that the documentation will simply be updated across the board to indicate that there is no support for using SQL Server Reporting Services in SharePoint multi-tenancy.

Update 3/26/2014: This issue appears to be resolved for SQL Server 2012 in [Service Pack 2 Cumulative Update 5](http://support.microsoft.com/en-us/kb/3037255). It is now possible to open RDL and RSDS files in SharePoint 2010.