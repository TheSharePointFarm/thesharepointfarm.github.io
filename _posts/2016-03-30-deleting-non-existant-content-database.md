---
layout: post
title: Delete a Phantom Content Database
tags: [sp2016]
---

I've encountered this a few times, where a Content Database has gone missing, thus you need to delete a phantom Content Database. When encountering this scenario, typically when creating a Site Collection you'll receive an error, and in the ULS you'll find that SharePoint isn't able to figure out what database to put it in. To determine if the cause is a non-existent database, use the SharePoint Management Shell and run:

```powershell
$wa = Get-SPWebApplication http://spwebapp1
$wa.ContentDatabases
```

If the output looks similar to the following, you have a phantom database.

```powershell
Id               : 9883a602-4218-4d73-aec7-9087d4761477
Name             : SharePoint_CDB1
WebApplication   : SPWebApplication Name=SharePoint
Server           : SPSQL
CurrentSiteCount : 6

Id               : 626c208d-0f3b-4d3f-b57e-5a72d1433e55
Name             : 
WebApplication   : SPWebApplication Name=SharePoint
Server           : SPSQL
CurrentSiteCount :
```

One might think that simply running Remove-SPContentDatabase -Identity or $db = Get-SPDatabase | ?{$_.Id -eq ""};$db.Delete() would do the trick. Nope! The problem with those methods is they instatinate the database object... which doesn't exist! Instead, we have to take an alternate approach using the SPWebApplication object via PowerShell.

```powershell
$wa.ContentDatabases.Delete("626c208d-0f3b-4d3f-b57e-5a72d1433e55")
```

That will cleanly remove the Content Database object from the Web Application.