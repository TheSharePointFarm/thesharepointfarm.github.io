---
layout: post
title: Find the Workflow Manager Address from SharePoint
tags: [sp2013,sp2016,sp2019,wfm]
---

If you're not sure what address your SharePoint farm is consuming Workflow Manager from, PowerShell can identify the full URI to the Workflow Manager farm that was registered with SharePoint.

```powershell
$site = Get-SPSite https://sharepoint.cobaltatom.com
$proxy = Get-SPServiceApplicationProxy | ?{$_.TypeName -eq 'Workflow Service Application Proxy'}
$proxy.GetHostname($site)
$proxy.GetWorkflowScopeName($site)
```

The output of that command will be the full URI to the Workflow Manager farm, including port and protocol. Additionally we are retrieving the ScopeName used to register Workflow Manager. This is useful if we have multiple scopes in use, for example if you have multiple SharePoint farms registered with Workflow Manager.

The $site object may be any Site Collection on SharePoint, we just need a single reference from the farm.