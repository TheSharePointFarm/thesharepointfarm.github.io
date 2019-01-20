---
layout: post
title: Enabling the Office 365 (SuiteNav) in SharePoint 2013
tags: [sp2013]
---

Office 365 and SharePoint Server 2016 have what is called the "SuiteNav". This is the navigation element at the top of the page that contains the affectionately-named "waffle" on the upper left.

![SuiteNav2016](/assets/images/2016/07/SuiteNav2016.png)

Well, now this feature has come to SharePoint 2013 with the [SharePoint Server 2013 July 2016 Cumulative Update](https://support.microsoft.com/en-us/kb/3115293) and more specifically, the [SharePoint Server 2013 hotfix KB3115286](https://support.microsoft.com/en-us/kb/3115286). [Instructions are provided](https://technet.microsoft.com/library/mt740142.aspx) on how to enable this feature, but if you want to enable the feature everywhere, you'll need to do a bit more scripting. Two scripts are provided below, one to enable it everywhere except for Central Administration, and one to enable it everywhere including Central Administration (similar to SharePoint Server 2016).

Without Central Administration:

```powershell
Add-PSSnapIn Microsoft.SharePoint.PowerShell -EA 0

Install-SPFeature SuiteNav -Force

foreach($wa in Get-SPWebApplication)
{
    $sites = Get-SPSite -WebApplication $wa -Limit ALL | ?{$_.CompatibilityLevel -ne '14'}
    foreach($site in $sites)
    {
        Enable-SPFeature -Identity SuiteNav -Url $site.Url -Force
    }
}
```

With Central Administration:

```powershell
Add-PSSnapIn Microsoft.SharePoint.PowerShell -EA 0

Install-SPFeature SuiteNav -Force

foreach($wa in (Get-SPWebApplication -IncludeCentralAdministration))
{
    $sites = Get-SPSite -WebApplication $wa -Limit ALL | ?{$_.CompatibilityLevel -ne '14'}
    foreach($site in $sites)
    {
        Enable-SPFeature -Identity SuiteNav -Url $site.Url -Force
    }
}
```

The SharePoint Server 2013, like the SharePoint Server 2016 SuiteNav, does not require any form of hybrid functionality with Office 365. You can enable the SuiteNav on any farm as long as one of the two previously mentioned patches are deployed to the farm.

Once the feature is enabled, the site will have a SuiteNav bar like this:

![SuiteNav2013](/assets/images/2016/07/SuiteNav2013.png)