---
layout: post
title: Disable Teams Creation Prompt in SharePoint Online
tags: [spo]
---

Have you seen this lately? It appears to any Office 365 Group owner where a Team has not been associated with a SharePoint Online site.

![CreateTeamInSPO](/assets/images/2019/04/CreateTeamInSPO.PNG)

There's a simple method to disable it. Using the SharePoint Online PnP cmdlets, run the following cmdlet.

```powershell
$tenant = "https://tenant-admin.sharepoint.com"
$web = "https://tenant.sharepoint.com/sites/ModernTeam"

Connect-PnPOnline -Url $tenant -SPOManagementShell
$site = Get-PnPTenantSite -Detailed -Url $web
if ($site.DenyAddAndCustomizePages -ne 'Disabled') {
    $site.DenyAddAndCustomizePages = 'Disabled'
    $site.Update()
    $site.Context.ExecuteQuery()
}

Set-PnPPropertyBagValue -Key 'TeamifyHidden' -Value 'True'
```

The first portion of the above script verifies that `DenyAndAddCustomizePages` is disabled on the site. This enables us to set a property bag value of `TeamifyHidden` to `true`. If you refresh the homepage after setting the value, the dialog box to create Teams should no longer appear.