---
layout: post
title: Domain Migration, Stale DNs, the User Profile Service, and the Sync Database
tags: [sp2010]
---

We were performing a domain migration for end users.  We're using ADMT 3.2 to perform the migration between two 2003-functional level forests.  Due to the complexity of the farm, and laziness, we were using `stsadm -o migrateuser -oldlogin domAuser -newlogin domBuser`, instead of Move-SPUser (due to requiring a bind and knowing where a user is on the farm).  The User Profile Application was set up with 2 Synchronization Connections, one to the old forest and one to the new forest.

After users were migrated, their DN (Distinguished Name, or where their object was located in Active Directory) would be updated correctly in Manage User Profiles (it would show CN=John Doe,DC=domB,DC=com).  However, when the user came in the following day, they would receive an Access is denied error!  Overnight, SharePoint was performing a User Profile Incremental Sync with all Synchronization Connections.  It turned out that SharePoint was actually performing the user migration backwards!  We could see in the ULS logs that SharePoint was performing the migration of domBuser to domAuser.  After the user was migrated, the Profile service is responsible for pushing down permissions from the User Profile Service back to the UserInfo table in each Content Database.

In working with Microsoft, they had us look at `dbo.Profile.DNLookup`. When performing the query, `select * from DNLookups where DN like '%user%'`, the DN of our user was only ever the DN from the old forest.  That explained why the user was being migrated backwards overnight.  It was finding the DN of the user and making sure it matched a user in Active Directory.

Microsoft had us follow the [procedure](http://technet.microsoft.com/en-us/library/ff681014.aspx) to re-provision the Sync database, and then asked we only created the User Profile Synchronization Connect to the destination domain for users.  After performing a Full Sync with Active Directory, all DNs have been correct and no further issues with permissions have been identified.