---
layout: post
title: Clarification on Always On Availability Groups and SharePoint Server
tags: [sp2013,sp2016,sql]
---

This post is a clarification on Always On Availability Groups and SharePoint Server. When using a single SQL Server with your SharePoint farm, the long standing advice is to use a SQL Alias (via `cliconfg.exe`). This is great advice and provides exactly one benefit: Database Mobility.

Database mobility is simply a concept where by you can move your databases, such as the Configuration and Administration databases (as you cannot change those databases' connection strings) to another SQL Server. All you have to do is take down your farm, move the SQL databases to a new server/instance, and update your SQL Alias. And that's awesome!

However, with Always On Availability Groups, do not use SQL Aliases. Instead, use the Always On Availability Group Listener. Always On Availability Groups have the concept of database mobility built into them. You can:

* Move databases between SQL Servers participating in the AOAG; this includes adding a server to the AOAG to retire an older SQL Server
* Upgrade the underlying Windows OS (requires Windows Server 2012 R2 at a minimum; this is called a rolling upgrade)
* Major version upgrade (e.g. SQL Server 2016 to SQL Server 2017, again called a rolling upgrade)

With these points in mind, when performing a SharePoint upgrade or implementing a new SharePoint farm where you have the opportunity to use highly available SQL Server with AOAG Listeners, point SharePoint at the AOAG listener.

If you haven't tried or had the opportunity to upgrade SQL and/or Windows participating in an AOAG, you can find documentation on how to do this at [Upgrading Always On Availability Group Replica Instances](https://docs.microsoft.com/en-us/sql/database-engine/availability-groups/windows/upgrading-always-on-availability-group-replica-instances). For Windows, see [Cluster operating system rolling upgrade](https://docs.microsoft.com/en-us/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade).

Lastly, the Availability Group cmdlets in SharePoint are absolutely safe. They perform the same work as SSMS, executing T-SQL commands, with some additional logic to add/manage databases to/in an AOAG. For more information on their usage, [I covered them]({% post_url 2014-05-09-sharepoint-database-availability-group-cmdlets %}) when they were added to SharePoint.