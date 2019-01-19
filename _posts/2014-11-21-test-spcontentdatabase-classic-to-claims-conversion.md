---
layout: post
title: Test-SPContentDatabase Classic to Claims Conversion
tags: [sp2010,sp2013]
---

Attaching a Content Database from a Classic Web Application to a Claims-enabled Web Application does not automatically convert the users contained within that Content Database from Classic to Claims. When you run `Test-SPContentDatabase`, it performs quite a few tests against the target Content Database, including a test to check if users within the UserInfo table are Claims-enabled on a Claims-enabled Web Application (and vice versa).

```
Category        : Configuration
Error           : False
UpgradeBlocking : False
Message         : The [SharePoint] web application is configured with claims
                  authentication mode however the content database you are
                  trying to attach is intended to be used against a windows
                  classic authentication mode.
Remedy          : There is an inconsistency between the authentication mode of
                  target web application and the source web application.
                  Ensure that the authentication mode setting in upgraded web
                  application is the same as what you had in previous
                  SharePoint 2010 web application. Refer to the link
                  "http://go.microsoft.com/fwlink/?LinkId=236865" for more
                  information.
Locations       :
```

When a `Test-SPContentDatabase` is executed (without additional switches), it runs the following T-SQL against the UserInfo table of the Content Database being tested:

```sql
SELECT TOP 1 [tp_SiteID],[tp_Login] FROM [UserInfo] WITH (NOLOCK) WHERE tp_IsActive = 1 AND tp_SiteAdmin = 1 AND tp_Deleted = 0 and tp_Login not LIKE 'i:%'
```

If it returns any results, the message above will be displayed. So what is the solution, here? Just migrate the users...

```powershell
$wa = Get-SPWebApplication http://webAppUrl
$wa.MigrateUsers($true)
```

This will migrate any valid users within the database from Classic to Claims. Running Test-SPContentDatabase should now succeed without any warning message, unless...

Any Site Collection Administrator has been deleted from Active Directory and cannot be converted! In that case, identify all Site Collection Administrators in the Content Database (e.g. `$web = Get-SPWeb http://webAppUrl/sites/rootWebUrl; $web.SiteAdministrators`), remove them from the Site Collection Administrators and as an optional step, delete them from the Site Collection. This way, they will no longer be marked as `tp_SiteAdmin = 1` and will not fulfill the requirements of the `Test-SPContentDatabase` query on the `UserInfo` table.