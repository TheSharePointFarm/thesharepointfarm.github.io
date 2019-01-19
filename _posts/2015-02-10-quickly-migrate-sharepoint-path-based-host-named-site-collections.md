---
layout: post
title: Quickly Migrate SharePoint Path-based to Host-named Site Collections
tags: [sp2013]
---

The February 2015 Public Update introduced a change to quickly allow you to migrate a Path-based Site Collection to a Host-named Site Collection. Here is how you would use this feature.

```powershell
$uri = New-Object System.Uri("http://host.example.com")
$site = Get-SPSite http://siteUrl
$site.Rename($uri)
```

And that's it! You've migrated http://siteUrl to http://host.example.com.