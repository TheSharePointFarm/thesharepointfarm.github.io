---
layout: post
title: Using a CDN for Static Resources on SharePoint 2016
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 includes a feature to use a CDN for static resources, such as JavaScript. And what better CDN to leverage than SharePoint Online? Configuration is fairly simple. Enable CDN, configure it to point to `static.sharepointonline.com`, and set up a Side by Side token which matches your farm build number.

The PowerShell:

```powershell
$farm = Get-SPFarm
$wa = Get-SPWebApplication http://webAppUrl
$wa.WebService.SideBySideToken = $farm.BuildVersion.ToString()
$wa.WebService.CdnPrefix = "static.sharepointonline/bld"
$wa.WebService.EnableCdn = $true
$wa.WebService.Update()
```

Using Fiddler, browsing to a home page, you should now see requests to http://static.sharepointonline.com/bld/_layouts/15/16.0.4316.1217/init.js, as an example.

You can also configure your own personal CDN using the same method. As long as the `CdnPrefix` does not contain "static", you do not need to set a `SideBySideToken` value.