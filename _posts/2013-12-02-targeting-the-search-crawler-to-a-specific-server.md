---
layout: post
title: Targeting the Search Crawler to a Specific Server
tags: [sp2010,sp2013]
---

In many medium to large SharePoint deployments, search is a critical piece of the farm architecture. An underperforming search instance can lead to severe discoverability issues as well as potentially missing critical business information. SharePoint Administrators can improve search crawl performance by dedicating a SharePoint server as a crawl target for one or more Web Applications. By creating this target, the SharePoint Administrator can greatly reduce the load on SharePoint servers that users interact with and potentially reduce crawl times.  Much of the advise on how to target a crawler is around editing a servers host file to point the DNS entry of a Web Application at a specific server. A hosts file entry was not how Microsoft intend a crawl target to be set up! Microsoft created a new property with SharePoint 2010 named [SiteDataServers](http://msdn.microsoft.com/en-us/library/office/microsoft.sharepoint.administration.spwebapplication.sitedataservers.aspx). It allows an Administrator to target crawls at one or more SharePoint servers on a Web Application and Web Application Zone basis.

In the SharePoint 2007 days, it was possible to target a specific server for crawling via Central Administration; in SharePoint 2010 and 2013, PowerShell must be used. In this example farm, there are three SharePoint servers:

SPWFE

SPSRC

SPCRW

SPWFE and SPCRW both run the Microsoft SharePoint Foundation Web service, but only SPWFE has DNS records pointed at it, and is the only server directly used by users. In the farm there is a Web Application of http://teamsites.example.com with a Default Zone.

To target the crawler on SPSRC at SPCRW for http://teamsites.example.com, run the following PowerShell commands in the SharePoint Management Shell:

```powershell
$wa = Get-SPWebApplication http://teamsites.example.com
$target = New-Object System.Uri("http://SPCRW") #Specify the machine name of the crawl target using a URL format
$uri = New-Object System.Collections.Generic.List[System.Uri](1) 
$uri.Add($target) 
$zone = [Microsoft.SharePoint.Administration.SPUrlZone]'Default' #Specify zone name
$wa.SiteDataServers.Add($zone, $uri)
$wa.Update()
```

Since a dedicated crawl target has been specified, it is likely that removing the HTTP throttling can be disabled. This can be done on the crawl target via PowerShell:

```powershell
$web=[Microsoft.SharePoint.Administration.SPWebServiceInstance]::LocalContent
$web.DisableLocalHttpThrottling=$true
$web.Update()
```

To remove a crawl target, simply run:

```powershell
$zone = [Microsoft.SharePoint.Administration.SPUrlZone]'Default' #Specify zone name
$wa = Get-SPWebApplication http://teamsites.example.com
$wa.SiteDataServers.Remove($zone)
$wa.Update()
```

From now on, there should be no reason to edit a host file on a SharePoint search server in order to crawl a specific crawl target! This process can be done on a per-Web Application Zone basis.  An Administrator can also specify multiple crawl targets, even per Zone!