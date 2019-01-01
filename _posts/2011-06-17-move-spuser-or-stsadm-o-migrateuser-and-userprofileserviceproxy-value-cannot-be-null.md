---
layout: post
title: Move-SPUser (or stsadm -o migrateuser) and userProfileServiceProxy value cannot be null
tags: [sp2010]
---

Using the February 2011 CU on SharePoint 2010, I discovered when performing a Move-SPUser that I was getting the following error message:

```powershell
Move-SPUser : Value cannot be null.
Parameter name: userProfileApplicationProxy At line:1 char:12
+ Move-SPUser <<<< -Identity $user -NewAlias "DOMAINB\User" -IgnoreSID
+ CategoryInfo : InvalidData: (Microsoft.Share...PCmdletMoveUser:
SPCmdletMoveUser) [Move-SPUser], ArgumentNullException
+ FullyQualifiedErrorId : Microsoft.SharePoint.PowerShell.SPCmdletMoveUser
```

Even though the user account was a Farm Administrator, SharePoint Shell Administrator, and Local Administrator across the farm, the error persisted.

It turned out that the account was getting Access is denied on the Profile WCF endpoint at http://Microsoft.Office.Server.UserProfiles/GetProfileProperties.

The only account that functioned for this process was the Farm Administrator account.  With the Farm Administrator account, no error was produced and profiles were properly migrated between accounts.