---
layout: post
title: SQL Server - Let's Support Them All!
tags: [sp2019]
---

A major change is taking place with how the SharePoint product group supports SQL Server with SharePoint Server -- namely, support will be defined by the database compatibility levels rather than a specific version of SQL Server. This allows customers to deploy SharePoint Server 2019 with SQL Server 2016, 2017, 2019, and beyond as long as SQL Server supports the database compatibility level 130.

Likewise, for SharePoint Server 2016, you can use any version of SQL Server that supports a database compatibility level of 110. This would mean that SharePoint Server 2016 now supports SQL Server 2012 through SQL Server 2019 and any future version of SQL Server that supports the database compatibility level of 110.

This is a great improvement and allows customers the flexibility to maintain forward momentum to keep up with SQL Server versions rather than having to maintain long term support for an aging version.

This change does not extend to SharePoint 2013 or previous versions of SharePoint. Continue following the specific versions of SQL Server that are supported for SharePoint 2013 and below.

The [Hardware and software requirements](https://docs.microsoft.com/sharepoint/install/hardware-and-software-requirements-2019#minimum-requirements-for-a-database-server-in-a-farm) document has been updated with this new guidance.