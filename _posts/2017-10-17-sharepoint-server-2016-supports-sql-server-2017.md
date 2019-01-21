---
layout: post
title: SharePoint Server 2016 Supports SQL Server 2017
tags: [sp2016]
---

The [software requirements for SharePoint Server 2016](https://technet.microsoft.com/en-us/library/4d88c402-24f2-449b-86a6-6e7afcfec0cd(v=office.16)#section4) were updated with Microsoft SQL Server 2017 support. Two features I'm excited about:

* SQL Server 2017 now supports Always On Availability Groups without an underlying Windows Cluster
* SQL Server 2017 now supports RedHat, SUSE, and Ubuntu Linux distributions

More information can be found in [What's new in SQL Server 2017](https://docs.microsoft.com/en-us/sql/sql-server/what-s-new-in-sql-server-2017).

Another consideration is that [SQL Server 2017 no longer includes SQL Server Reporting Services integration with SharePoint](https://docs.microsoft.com/en-us/sql/reporting-services/install-windows/supported-combinations-of-sharepoint-and-reporting-services-server). This means that you must stick with SQL Server 2016 for SSRS integration with SharePoint Server 2016 or move to SSRS Native, however like past versions of SSRS, you can host downlevel SSRS database on an uplevel version of SQL Server (e.g., SSRS 2016 databases on SQL Server 2017).