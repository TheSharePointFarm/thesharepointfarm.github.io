---
layout: post
title: SharePoint Server 2019 New PowerShell Cmdlets
tags: [sp2016]
---

SharePoint Server 2019 includes a handful of new PowerShell cmdlets They're fairly self explanatory, but I do want to highlight them as they replace important stsadm functionality.

First is `Set-SPApplicationCredentialKey` which replaces `stsadm -o setapppassword`. This would be used in a case where you need to use authenticated outgoing email, another new feature of SharePoint Server 2019, or setting a user to query a domain over a one-way trust. Note that you must run this cmdlet with the same values on all SharePoint servers in the farm. This value is stored in the registry at `HKLM\SOFTWARE\Microsoft\Shared Tools\Web Server Extensions\16.0\Secure` in a `REG_BINARY` named `AppCredentialKey` and is not replicated between farm members. There is a corresponding `Remove-SPApplicationCredentialKey`, as well.

The second is `Get-SPContentDatabaseOrphanedData` to replace `stsadm -o enumallwebs`. This only replaces a portion of the stsadm command and as the cmdlet implies, it looks for orphaned data in a content database.

The last set of cmdlets, and probably my favorite, are regarding User Profile Service Application to Site Collection User Information List sync. These replace `stsadm -o sync`.

`Clear-SPContentDatabaseSyncData [-DaysSinceLastProfileSync <Int32>]` does the same thing as `stsadm -o sync -deleteolddatabases <Int32>`. `Update-SPProfileSync` is a cmdlet that lets you update the UPSA sync jobs between the UPSA and Site Collection UILs. Lastly, `Get-SPContentDatabase` has a new switch, `-DaysSinceLastSync <Int32>` that replaces `stsadm -o sync -listolddatabases <Int32>`. This parameter allows you to see when the UPSA to Site Collection UIL jobs last updated the UIL.