---
layout: post
title: SharePoint 2013 April 2014 CU Claims Conversion Bug
tags: [sp2013]
---

6/11/2014 Update:

This bug is resolved in the [June 2014 Cumulative Update](2014-06-10-sharepoint-2013-june-2014-cumulative-updates).

-------------

6/6/2014 Update:

Microsoft Product Support Services has provided a workaround for this issue. Simply migrate each user and group post Web Application conversion. This can be done via `stsadm` or `Move-SPUser`. For example:

```powershell
stsadm -o migrateuser -oldlogin nauplius\trevor -newlogin "i:0#.w|nauplius\trevor" -ignoresidhistory
```

This will migrate the user across the entire farm. If you have other Web Applications that remain in Classic mode within the same farm with the same user accounts, this will migrate the user to Claims format within those databases. In this scenario, the workaround is not valid.

-------------

There is a fatal bug in the April 2014 Cumulative Update as well as MS14-022 for SharePoint 2013 when attempting to use the `Convert-SPWebApplication` cmdlet. With this cmdlet came a large code base change in order to add significant new functionality, but it unfortunately broke the Classic (Windows) to Claims migration functionality.

This cmdlet also comes with some new parameters, undocumented in either the cmdlet help (perhaps to be updated via Update-SPHelp in the future) or on TechNet. For reference, it now has a -From and -To switch:

* `-From` accepts the following string values: `Legacy`, `Claims-Windows`, `Claims-Trusted-Default`

* `-To` accepts the following string values: `Claims`, `Claims-Windows`, `Claims-Trusted-Default`, `Claims-SharePoint-Online`

For the purposes of this post, the values in use are -From Legacy -To Claims -RetainPermissions

When the migration process starts, the migrator loops through each Content Database, and for each online Content Database, each Site Collection, and finally each User/Group. In this example, we're taking `NT AUTHORITY\Authenticated Users` and converting to `c:0!.s|windows`. The major break appears to occur in `Microsoft.SharePoint.Administration.SPWebApplicationMigrator` in the method `GetEntityMigrationDataFromOldCallBack(SPMigrationEntity entity)`.

Near the top of the method, a value is set:

`SPMigrateEntityCallbackResult failure = SPMigrateEntityCallbackResult.Failure;`

This is a fair thing to set, assume we're going to fail the migration, and make sure we'll succeed for each user instead. Good idea, honestly. And for this user, we do actually succeed at making a valid claim, and I'll show you that.

In this same method, we pass through this portion where the conversion process from Classic to Claims is actually performed, with no errors:

```csharp
            try
            {
                ULS.SendTraceTag(0x301851, ULSCat.msoulscat_WSS_ClaimsAuthentication, ULSTraceLevel.VerboseEx, "{0}: Calling convert from old user. {1}", new object[] { this.LogPrefix(), entity.LogString() });
                str2 = this.m_MigrateUserCallback.ConvertFromOldUser(key, entity.AuthenticationType, entity.IsGroup);
                ULS.SendTraceTag(0x301852, ULSCat.msoulscat_WSS_ClaimsAuthentication, ULSTraceLevel.VerboseEx, "{0}: Calling convert from old user without exception. {1}", new object[] { this.LogPrefix(), entity.LogString() });
            }

Where the bug appears to be happening is after the conversion. That failure variable is never updated to Success.

            if ((str2 != null) && string.Equals("skipUser", str2, System.StringComparison.OrdinalIgnoreCase))
            {
                return SPMigrateEntityCallbackResult.AlreadyMigrated;
            }
            if (string.IsNullOrEmpty(str2))
            {
                ULS.SendTraceTag(0x301854, ULSCat.msoulscat_WSS_ClaimsAuthentication, ULSTraceLevel.High, "{0}: Could not get migration data for user. Check migrator for further logs. {1}", new object[] { this.LogPrefix(), entity.LogString() });
                return SPMigrateEntityCallbackResult.Failure;
            }
            entity.MigratedName = str2;
            return failure;
```

![ClaimsMigrationGroup](/assets/images/2014/05/ClaimsMigrationGroup.png)

Here we can see that the `str2` variable is not null and `skipUser` is also not set. `AlreadyMigrated` is also not set, but in addition, the failure variable has not been updated to Success either, hence the migration never is allowed to move forward.

Here is another view after having left the last method where the failure variable should have been updated to Success:

![ClaimsMigrationGroup2](/assets/images/2014/05/ClaimsMigrationGroup2.png)

In the end, what should happen is that we skip over a bunch of other validation methods to to make sure that `SPMigrateEntityCallbackResult` does not equal Success, and finally if the entity does equal Success, add it to the Migration Cache to be migrated, and finally execute the migration.

In this example, I have the following users in my UserInfo table. The first user you see in Claims format is placed there automatically by the conversion process in order to start the conversion process.

```sql
tp_id      tp_login
16         i:0#.w|nauplius\trevor
15         NAUPLIUS\s-sp2010sr
14         NAUPLIUS\s-sp2010su
1          NAUPLIUS\trevor
12         NT AUTHORITY\authenticated users
1073741823 SHAREPOINT\system
```

Next, running through each user, manually updating from Failure to Success after the internal conversion (again, the conversion in SQL has not happened yet) from Classic to Claims using the following method in the `SPMigrateEntityCallbackResult MigrateEntity(SPClaimMigrationContext context, SPMigrationEntity entity)` method:

![FailureToSuccess](/assets/images/2014/05/FailureToSuccess.png)

Each user is added to a new "migration cache". I didn't dive too deeply into this cache, but needless to say, the migration is successful. Examining the UserInfo table again, post manual variable change:

```sql
tp_id	  tp_login
12	     c:0!.s|windows
15	     i:0#.w|nauplius\s-sp2010sr
14	     i:0#.w|nauplius\s-sp2010su
4	      i:0#.w|nauplius\trevor
1	      i:0#.w|nauplius\trevor
16	     i:0#.w|nauplius\trevor
1073741823 SHAREPOINT\system
```

After the SQL query completes execution, from here I am able to log in as any specified user that has been migrated successfully. Otherwise, if the Web Application has been migrated to Claims but the users have not, the user will receive Access Denied in their Windows ("Classic") login format.

Unfortunately the old SharePoint 2010 `MigrateUsers($true)` method (deprecated) also runs through the same code, so it will produce the same results. In the end, we'll need to wait for a patch from Microsoft on this issue.