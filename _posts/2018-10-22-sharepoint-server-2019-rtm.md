---
layout: post
title: SharePoint Server 2019 RTM!
tags: [sp2019]
---

SharePoint Server 2019 has RTM'ed today! This is a big release bringing many of the modern components found in SharePoint Online to SharePoint on-premesis. While there is a good laundry list of features, some of the big highlights are:

* Modern Team sites and Communication sites
* SharePoint Home
* \# and % characters supported in file names
* 400 character path limit
* 15GB maximum file size support
* OneDrive Next-Gen Sync Client support for OneDrive sites and document libraries
* Newly styled App Launcher
* SMTP Authentication
* Distributed Cache set up the way it should be!
    * backgroundGC=true
    * CU7 pre-installed
* SPFx 1.4.1 including Extensions and Webhooks

There are also a four new PowerShell cmdlets and one new switch to an existing cmdlet.

`Get-SPContentDatabaseOrphanedData` replaces `stsadm.exe -o enumallwebs` for retrieving orphaned content.

`Set-SPApplicationCredentialKey` and `Remove-SPApplicationCredentialKey` replace `stsadm.exe -o setapppassword`.

`Clear-SPContentDatabaseSyncData` and `Update-SPProfileSync` replace `stsadm.exe -o sync`.

Finally, `Get-SPContentDatabase` has a new switch, `-DaysSinceLastProfileSync` which also replaces `stsadm.exe -o sync`.

There are also two new health analyzer rules for the PeoplePicker and SMTP authentication. When the People Picker is using a username and password to access another domain or SMTP Authentication is configured, the Application Credential Key must be identical on all farm members. This rule checks for this and will throw an error if the key is not set properly on all farm members.

I hope you enjoy some of the big improvements. Finally, congratulations to the SharePoint and OneDrive teams, as well as all the testers and TAP participants who gave feedback!