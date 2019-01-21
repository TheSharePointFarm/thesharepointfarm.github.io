---
layout: post
title: SharePoint 2016 July 2018 PU Configuration Failure
tags: [sp2016]
---

The [July 2018 PU](https://support.microsoft.com/en-us/help/4022228/description-of-the-security-update-for-sharepoint-server-2016-july-10) for SharePoint 2016 has this fix in it which comes from a [long standing bug](https://www.techmikael.com/2014/10/caution-if-you-have-used.html):

> When a farm administrator creates a search service application by using PowerShell, the database owner of the search service application database is the farm administrator and not the farm account that is used to run the timer service. The database owner of all databases should be the farm account. This update changes the owner of search-related databases to be the farm account.

The fix is self explanatory, except it doesn't work. It will cause this failure when running the Config Wizard after installation, provided you have indeed created the Search Service Application via PowerShell with a user other than the timer service account, which of course is best practice.

```csharp
An exception of type Microsoft.SharePoint.PostSetupConfiguration.PostSetupConfigurationTaskException was thrown.  Additional exception information: 
Upgrade [SearchAdminDatabase Name=PROD_Search] failed.	(EventID:an59t)

Exception: The database principal owns a database role and cannot be dropped.The proposed new database owner is already a user or aliased in the database.	(EventID:an59t)

Upgrade Timer job is exiting due to exception: System.Data.SqlClient.SqlException (0x80131904): The database principal owns a database role and cannot be dropped.The proposed new database owner is already a user or aliased in the database.
```

Ultimately this will not impact Search functionality and can be ignored. All Search components should be in a healthy state post-Config Wizard run. Due to the failure of the Config Wizard, make sure to run `iisreset /start` on all of your SharePoint servers post-update as the IIS sites will likely be in a stopped state.

This will hopefully be resolved in a future update.