---
layout: post
title: Azure SQL PaaS for SharePoint Content Databases
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM. This functionality is highly likely to not exist in the RTM product and is shown here as an academic exercise.]

SharePoint 2016 includes some very interesting features that allow us to take a look at what is being used within SharePoint Online (“the service”). Leveraging Azure SQL PaaS for SharePoint Content Databases allows you to fore-go storing content on a traditional SQL Server. This process is significantly easier than leveraging Azure Blob Storage for content, but like ABS, is not supported by the SharePoint Product Group.  If support for this is interesting for you or your customers, make sure to let the SharePoint Product Group know via [UserVoice](http://sharepoint.uservoice.com/) or an RFC through standard Microsoft support offerings. If this process was ever supported, the assumption would be that it will only be supported for SharePoint farms running in Azure Virtual Machines and where the farm is in the same Azure region as the primary Azure SQL PaaS account.

To get started, create a SQL PaaS database in Azure. The only requirement on the SharePoint side is that the database must use the Latin1_General_CI_AS_KS_WS collation. You may use a Basic or S0 database for demonstration purposes, and yes, it will be slow! Record the administrative credentials for the SQL PaaS instance. Next, using SQL Server Management Studio, connect to your SQL PaaS instance and create a new query on the master database to add an additional user account. This is the user that SharePoint will be using to read and write to the Content Database.

```sql
CREATE LOGIN 'username' WITH PASSWORD=N'passWord';
qqq

Add this user to the `loginmanager` role.

```sql
EXEC sp_addrolemember 'loginmanager', 'username';
```

Next, add this new user to the database that was created in Azure and add it to the `db_owner` role, again using SSMS.

```sql
CREATE USER [username] FOR LOGIN [username] WITH DEFAULT_SCHEMA = db_owner;
EXEC sp_addrolemember 'db_owner','username';
```

On the SharePoint Farm, create a new Content Database using the standard cmdlet. `DatabaseCredentials` will be the account you created for the Azure SQL Database.

```powershell
New-SPContentDatabase -Name AzureDbName -WebApplication (Get-SPWebApplication http://webAppUrl) -DatabaseServer AzureDbServerName.database.windows.net -DatabaseCredentials (Get-Credentials)
```

This process will take quite some time! Using a Basic instance, roughly 20 minutes while the on-premises SharePoint server resided within the same region as the SQL PaaS instance.

Once the database has been created, using SSMS, you may view the tables, stored procedures, and views. It will look similar, although not identical, to the on-premises Content Databases.

You can also check the `SPDatabase.IsSqlAzure` property is set to true for this database by running:

```powershell
(Get-SPContentDatabase AzureDbName).IsSqlAzure
```

This will, of course, return true! At this point, create Site Collections per the normal process and there are no further steps required.