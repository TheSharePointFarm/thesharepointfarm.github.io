---
layout: post
title: SharePoint Server 2016 Content Database Corruption Checks
tags: [sp2016]
---

SharePoint Server 2016 includes new Content Database corruption check methods that allow you to target corruptions based on parameters to a Site Collection or database-wide, depending on the method. Using the SharePoint Management Shell, the following options are available:

```powershell
$db = Get-SPContentDatabase DbName
$db.CorruptionChecker.CheckForDuplicateContentTypes(SPSite.Id, $true|$false) # Pass the GUID of the Site Collection and specify true or false to repair identified issues
$db.CorruptionChecker.CheckForMissingSecurityScopes($true|$false) # Checks the database for missing security scopes. Pass $true or $false to repair identified issues.
$db.Corruptionchecker.CheckForOrphanRoleAssignments($true|$false) # Checks the database for orphan role assignments. Pass $true or $false to repair identified issues.
$db.CorruptionChecker.CheckForOrphanWebs(SPSite.Id) # Pass the GUID of the Site Collection containing suspected orphan Webs.
$db.CorruptionChecker.CheckForSpecificOrphanWeb(SPSite.Id, webUrl, $true|$false) # Pass the GUID of the Site Collection containing suspected orphan Web, along with the web URL. Pass $true or $false to repair identified issues.
```

They're fairly self-explanatory. If you don't know how to get the GUID of a Site Collection, you use PowerShell for that:

```powershell
$site = Get-SPSite http://siteUrl
$site.Id
```