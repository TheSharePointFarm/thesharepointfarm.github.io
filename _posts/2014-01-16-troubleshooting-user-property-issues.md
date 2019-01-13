---
layout: post
title: Troubleshooting User Property Issues
tags: [sp2010]
---

Let's ignore SharePoint Foundation for a bit, because that doesn't automatically sync with Active Directory anyways (besides the initial add). Typically, with Standard and Enterprise, SharePoint leverages the User Profile Sync Service (or AD Import in 2013) to pull in user information from Active Directory to the User Profile Service Application. This is what stores the Display Name, First Name, Last Name, Telephone Number, and all of that stuff.

This post takes the assumption that the User Profile Import is functioning, and under the User Profile Service Application -> Manage User Profiles, the data is up-to-date for the particular profile. If that isn't working, that is another issue altogether!

The situation is that you have a user, or a collection of users where you've changed a property, such as their Display Name in Active Directory and it shows as valid in the UPA, but on one or more Site Collections in one or more Web Applications, the Display Name hasn't updated for more than 1 hour after it was updated in the UPA. The fix is often very simple, so let's get to that part.

First, if you want to validate "things aren't working right", then simply run:

```powershell
stsadm.exe -o sync -listolddatabases 0
```

What this does is get the last time all of the Content Databases were synchronized by the appropriate timer job (descriptions below). This information is stored in each Content Database under `dbo.DatabaseInformation`.  You can also view this with PowerShell by running `foreach($db in Get-SPContentDatabase){$db.Name+" - "+$db.LastProfileSyncTime}`. If this date is older than one day, the timer job is likely not functioning correctly for the particular Content Database, thus all Site Collections within that database. To resolve it, run:

```powershell
stsadm.exe -o sync -deleteolddatabases 0
```

Now, a parameter that says `DeleteOldDatabases` sounds really bad. And usually it would be. Except in this case, all it is doing is clearing the applicable information out of the DatabaseInformation table where the date stored in the value `_System_LastProfileSyncTime` is older than today. This is done via a stored procedure in each Content Database named `proc_SetDatabaseInformation`.

Once this process has been completed, the next step to deploy changes from the UPA to the Site Collections is to either wait, or manually run the timer job User Profile Service Application - User Profile to SharePoint Full Synchronization (internal name is `ProfSync`). This job will 'push' out the User Profile Properties from the UPA to each Site Collection, making sure all Site Collections are up-to-date. This job will also make sure Memberships are up-to-date. Specifically, User Adds and Updates, as well as Group Adds, Deletes, Group Membership Adds, Deletes, and Updates. Finally, it also includes items that have been Restored. By default, this job runs every hour on the hour.

There is another job, similar to the above job, named User Profile Service Application - User Profile to SharePoint Quick Synchronization (internal name is `SweepSync`). Unlike the User Profile to SharePoint Full Synchronization timer job, this job only synchronizes User Adds and Updates. By default, this job runs every five minutes.

Note that the name of the job may be different as it pulls in the name of the User Profile Service Application. Queries for profile changes are done via a standard [`SPChangeQuery`](http://msdn.microsoft.com/en-us/library/Microsoft.SharePoint.SPChangeQuery.aspx). In addition to the above processed changes, if any users are found to have a new sAMAccountName in the User Profile Service Application, but the old `sAMAccountName` is reflected on the Site Collection, SharePoint will execute the standard Migration process for that user (the same code that [`Move-SPUser`](http://technet.microsoft.com/en-us/library/ff607729.aspx) executes). The comparison is done between the User Profile AccountName property and the LoginName property on the Site Collection.

Hopefully this helps you have further understanding in what the legacy `stsadm -o sync` commands do and how to resolve this somewhat common issue of the UPA Profiles being unable to synchronize with the Site Collection User Information List.

And to round things off, here is now my favorite Obsolete decoration:

```powershell
[Obsolete("This method is spelled wrong. Use SynchronizeContentDatabase instead")]
public static void SynchronizeContentDatabse(SPContentDatabase cdb)
{
    SynchronizeContentDatabase(cdb);
}
```