---
layout: post
title: User cannot be found when creating sub-site, Document Library, etc.
tags: [sp2010]
---

Had an issue where the user who enabled the Publishing feature on a site had been migrated from one domain to another, and `stsadm -o migrateuser` had been performed on their account without errors. After the migration, when someone attempted to create a Document Library, sub-site, etc., SharePoint would generate a "User cannot be not found" error message (although the item would be created).

It turned out that the Publishing workflows (as well as other site-wide workflows) were listed with the old account.  Opening and simply resaving the workflows as a user account that had been migrated fixed the issue.