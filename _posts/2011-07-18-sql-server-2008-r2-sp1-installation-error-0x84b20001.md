---
layout: post
title: SQL Server 2008 R2 SP1 Installation Error 0x84B20001
tags: [sql]
---

With SharePoint 2010 SP1 and SQL Server 2008 R2 (CU6 at the time) with Database Engine, SQLServer Reporting Services, and PowerPivot, I ran into the following error:

![clip_image00263](/assets/images/2011/07/clip_image00263.png)

Clicking OK would exit the installation process.  Modifying the following value fixed the issue:
`HKLM\Software\Microsoft\Microsoft SQL Server\100\ConfigurationState` and change `Analysis_Server_SPI` to “1” to fix the “[Analysis_Server_SPI,]” error.  To fix the PowerPivot error, modify `HKLM\Software\Microsoft\Microsoft SQL Server\MSAS10_50.POWERPIVOT\ConfigurationState` and change “Analysis_Server_Full” to “1”.  In my case, the data was “2”.