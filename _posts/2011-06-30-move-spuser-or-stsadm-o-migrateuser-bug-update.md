---
layout: post
title: Move-SPUser (or stsadm -o migrateuser) bug update
tags: [sp2010]
---

Before migrating users, there is a bug in the process.  The Profile service does not get updated properly until you apply Service Pack 1 and the June 2011 Cumulative Update.  Within the [June CU](http://support.microsoft.com/kb/2536599) is this note:

```text
Profile synchronization does not support domain migration in Microsoft SharePoint Server 2010.
```

This was a bug found internally at Microsoft which causes the Profile database to not be updated when a user is migrated between domains.  If you're unable to apply SP1, there is the post-April hotfix package [KB2534417](http://support.microsoft.com/kb/2534417) that is also available.