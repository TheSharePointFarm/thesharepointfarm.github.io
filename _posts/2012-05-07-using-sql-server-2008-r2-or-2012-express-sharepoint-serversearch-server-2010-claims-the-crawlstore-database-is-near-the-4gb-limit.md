---
layout: post
title: Using SQL Server 2008 R2 or 2012 Express, SharePoint Server/Search Server 2010 claims the CrawlStore database is near the 4GB limit
tags: [sp2010, sql]
---

SharePoint Server 2010 and Search Server 2010 ship with SQL Server 2008 Express by default.  However, you can leverage SQL Server 2008 R2 or 2012 Express as well which have 10GB MDF limits.  SharePoint/Search Server are unaware of this, however, and simply detects the edition of the SQL Database Engine – if it identifies it as Express, they believe the max MDF is 4GB.  A warning is logged in ULS if the database is near that 4GB limit, and an error is logged if the database has exceeded that 4GB limit.

However, Microsoft provided an undocumented way of modifying the logic for detecting the max size.  In Microsoft.Office.Server.Search.Administration.IndexingScheduleJobDefinition, the method `ReadMaxDBSizeForSqlExpress()` has a registry key we can set:

```csharp
private static float ReadMaxDBSizeForSqlExpress()
{
    float num = 3686.4f;
    using (RegistryKey key = Registry.LocalMachine.OpenSubKey(@"Software\Microsoft\Office Server\14.0\Search\Global"))
    {
        object obj2 = key.GetValue("MaxSearchDBSizeMB");
        if (obj2 is int)
        {
            num = (int) obj2;
        }
    }
    return num;
}
```

By creating a new DWORD of `MaxSearchDBSizeMB` with a value of 10240 in `HKLM\Software\Microsoft\Office Server\14.0\Search\Global\`, you can adjust the rule to not apply until it is near or at the 10GB limit of SQL 2008 R2 or 2012 Express.