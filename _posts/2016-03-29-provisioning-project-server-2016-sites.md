---
layout: post
title: Provisioning Project Server 2016 Sites
tags: [sp2016]
---

Provisioning Project Server 2016 sites has been a bit more confusing in SharePoint Server 2016 than it was in past versions. There are some mandatory PowerShell steps that one must follow as there is no web-based UI, but none the less, it's pretty easy.

Project Server 2016 sites currently must be provisioned with the PWA#0 Site Template. The necessary resources are not provisioned correctly on other Site Templates, despite what the [Project Server 2016 documentation says](https://msdn.microsoft.com/en-us/library/ee662105%28v=office.16%29.aspx) (EDIT: this documentation has been updated and no longer reflects this statement). For example, under PWA Site Settings, these pages generate a 404:

* /MyJobs.aspx
* /Project Detail Pages/Forms/AllItems.aspx
* /Resources.aspx

"Enterprise Project Types" will also throw:

![EnterpriseProjectTypes](/assets/images/2016/03/EnterpriseProjectTypes.png)

Here are the steps I would recommend:

Create a Project Server Service Application. When selecting an Application Pool, I would recommend using the same Application Pool that you use for your other Service Applications (you're only [using one]({% post_url 2014-08-25-expense-application-pools %}), right?). You'll notice when creating the Service Application that it doesn't have you create a database. This is because the tables for Project Server are now in the Content Database(s).

Start the Project Server service, or if using the WebFrontEnd and/or Application MinRoles, it will automatically start there. One thing you may consider, although it is completely optional, would be to create a wildcard Managed Path on your Web Application to host PWA instances. In the below example, I will be using /pwa/ as the Managed Path.

Using the SharePoint Management Shell, create your Site Collection, then enable the PWASite feature.

```powershell
New-SPSite -Template PWA#0 -Url https://sharepoint.nauplius.local/pwa/myproject -OwnerAlias "nauplius\trevor"
Enable-SPFeature PWASite -Url https://sharepoint.nauplius.local/pwa/myproject
```

That will provision the PWA site correctly, preventing errors in the PWA Site Settings. On my test VM, this process in total takes about 1 minute and 50 seconds.