---
layout: post
title: SharePoint 2016 Large List Auto Indexing
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 includes a new large list auto indexing timer job named job-list-automatic-index-management. This job is designed to automatically create list indexes when the List is less than twice the List View Threshold (so a list < 10,000 items by default) on a daily basis.

Automatic List Index functionality can be enabled and disabled on a list-by-list basis (enabled by default). To disable auto indexing using PowerShell, run:

```powershell
$web = Get-SPWeb http://webUrl
$list = $web.Lists["List Name"]
$list.EnableManagedIndexes = $false
$list.Update()
```

Beyond the requirements of the list item count being < 2 * the LVT (maximum of 10,000 items by default), the list must have indexable fields and a View with a Filter in place. Fields that are indexable can also be found via PowerShell:

```powershell
$list.Fields | Select Title,Indexable
```

If a custom View does not have a Filter against an Indexable field, a Managed Index will not be created by the timer job. Once a Managed Index has been created, it will appear similar to this on the Indexed Columns page:

![ManagedIndex](/assets/images/2015/07/ManagedIndex.png)