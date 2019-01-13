---
layout: post
title: Re-calculate Site Storage Metrics On Demand
tags: [sp2013]
---

With the release of SharePoint 2013 Service Pack 1, you can now re-calculate site storage metrics on demand. This is done via PowerShell.

```powershell
$site = Get-SPSite http://siteUrl
$site.RecalculateStorageMetrics()
```

This method executes `proc_RecalculateStorageMetricsForSite`, which can be found in each Content Database. The Content Database must be at the SP1 schema level.

Storage metrics can be found on each site by visiting http://siteUrl/_layouts/15/storman.aspx.