---
layout: post
title: SharePoint 2016 Site Masters
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 has new functionality for Site Template deployment called "Site Masters". These are master Site Templates, configured by a SharePoint Administrator, that allow quick deployment of a Site Template when a new SharePoint Site is requested.

Creating Site Masters is simple, using the SharePoint Management Shell, input the WebTemplateId and the Content Database to store it in, like this:

```powershell
New-SPSiteMaster -ContentDatabase "DatabaseName" -Template "Template#ID"
```

Site Master information is stored in the Content Database dbo.SiteMaster table.

To enable a WebTemplate to be used as a Site Master, you can run:

```powershell
Enable-SPWebTemplateForSiteMaster -Template "STS#0"
```

In this example, we're using the standard Team Site template. To deploy a site from the Site Master using New-SPSite, simply add a new switch:

```powershell
New-SPSite -Name "Team Site" -Url http://siteUrl -Template "STS#0" -OwnerAlias "domain\username" -CreateFromSiteMaster
```

In my lab, creating a new site takes approximately 100 seconds. Creating a site from a Site Master takes 77 seconds. While not a huge difference, this is of course a lab environment and running pre-RTM code.