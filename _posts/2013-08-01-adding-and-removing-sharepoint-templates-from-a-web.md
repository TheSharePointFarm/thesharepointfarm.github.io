---
layout: post
title: Adding and Removing SharePoint Templates from a Web
tags: [sp2010,sp2013]
---

In certain cases you may not want Site Collection Administrators or otherwise delegated users to use a certain type of Web template.  This can be achieved using 3rd party tools quite easily, or if Publishing is turned on at the Site Collection level.  However, in some cases neither of these options are available.  In this case, we can do it with PowerShell.  You will need your LCID ([Language ID](http://technet.microsoft.com/en-us/library/ff463597.aspx)), in this case, 1033, or English.

```powershell
$web = Get-SPWeb http://webUrl
$collection = @($web.GetAvailableWebTemplates(1033))
$collection2 = $collection | where {$_.Name -ne "COMMUNITY#0"}
$web.SetAvailableWebTemplates($collection2, 1033)
$web.Update()
```

To add a template back into the list, run:

```powershell
$web = Get-SPWeb http://webUrl
$collection = @($web.GetAvailableWebTemplates(1033))
$webtemplate = Get-SPWebTemplate | where {$_.Name -eq "COMMUNITY#0"}
$collection2 = $collection + $webtemplate
$web.SetAvailableWebTemplates($collection2, 1033)
$web.Update()
```

The change will appear when a user attempts to create a new Web of the selected Web.