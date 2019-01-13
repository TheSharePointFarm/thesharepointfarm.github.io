---
layout: post
title: Quickly Identifying Reporting Services Subscriptions in SharePoint
tags: [ssrs]
---

With Reporting Services integrated into SharePoint, it is often not easy to identify where certain Reporting Services features are used, for example, where users have set up Subscriptions.

We can instead leverage the native Reporting Services Web Service and PowerShell to quickly identify all subscriptions within a particular Site Collection.

```csharp
$siteUri = "http://spwebapp1"
$proxy = New-WebServiceProxy -Uri "$siteUri/_vti_bin/ReportServer/ReportService2010.asmx" -UseDefaultCredential -Namespace "SSRS"
$proxy.ListSubscriptions($siteUri)
```

And again, using the Web Service, we can easily delete all subscriptions in a particular Site Collection:

```csharp
foreach($sub in $proxy.ListSubscriptions($siteUri))
{
    $proxy.DeleteSubscription($sub.SubscriptionID)
}
```

There is a lot of functionality exposed via the Reporting Services Web Service, which you can discover by running `$proxy | gm` after connecting to the Web Service. Have fun!