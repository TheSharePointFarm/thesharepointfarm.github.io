---
layout: post
title: Reporting Services Feature Pack
tags: [sp2010,sp2013,ssrs]
---

In order to use SQL Server Reporting Services in a multi-tenant situation, follow the standard setup for SSRS. That is, install SSRS on a SharePoint server, start the Service Instance, create a Service Application and Proxy, and assign the Proxy to the Web Application hosting your multi-tenant sites.

For the Feature Pack, there are only two Features that are required to enable Reporting Services support. In this example, these Features are added to an existing Feature Pack.

```powershell
$fp = Get-SPSiteSubscriptionFeaturePack -Identity GUID
Add-SPSiteSubscriptionFeaturePackMember -Identity $fp -FeatureDefinition ReportServer
Add-SPSiteSubscriptionFeaturePackMember -Identity $fp -FeatureDefinition ReportServerItemSync
Set-SPSiteSubscription -Id $SubscriptionId -FeaturePack $fp
New-SPSite -Url http://siteUrl -OwnerAlias "domain\username" -Template "STS#0" -SiteSubscription $sub
iisreset
```

That's it. Now the Site Collection "Report Server Integration Feature" and Web "Report Server File Sync" features can be enabled and the SSRS Content Types will be available.