---
layout: post
title: SharePoint Server 2016 Project Server Public Update Issue
tags: [sp2016]
---

Starting with the December 2019 Public Update for SharePoint Server 2016, if your content database has Project Server tables (i.e. you've enabled Project Server on your farm), you may run into an error upgrading one or more content databases due to a Deny ACL on Project Server tables to the PSDataAccess role. This issue only occurs when you have provisioned your content database via PowerShell under an account other than the farm admin service account (the account running `owstimer.exe`). If you have provisioned your content databases through Central Administration or PowerShell under the context of the farm admin service account, you should not see this issue during execution of the Config Wizard.

When running the Config Wizard or `psconfig`, the ULS and Upgrade logs will reflect an error similar to the below.

```text
The SELECT permission was denied on the object 'MSP_PROJECTS', database 'ContentDatabaseName', schema 'pjpub'.
```

To resolve this, in the database users, verify the user mapped to the `dbo` user is the farm admin service account, similar to the below screenshot. This may also impact content databases that were upgraded from previous farms if the previous farm used a different service account for the farm admin service account.

![dbo-farm-admin-mapping](/assets/images/2020/01/dbo-farm-admin-mapping.PNG)

Once you have remapped `dbo` to the farm admin service account, re-run the Config Wizard to complete the upgrade.

This also impacts the January 2020 Public Update if you skipped the December 2019 Public Update.

There is an [TechNet thread](https://social.technet.microsoft.com/Forums/office/en-US/0aaf7ee5-f6b2-4c57-ae8b-3f593a0602a8/) if you would like to follow along for a potential product-based resolution.
