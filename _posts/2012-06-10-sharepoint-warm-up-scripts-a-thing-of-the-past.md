---
layout: post
title: SharePoint Warm-Up Scripts - A Thing of the Past
tags: [sp2010]
---

Enter a new feature to IIS 8: Application Initialization.  This module will allow you to set Application Pools to always run, regardless if an end user request has been made to them or not.  IIS will instead send a request when the Application Pool is enabled (e.g. server startup, iisreset, and so forth).

To install the Application Initialization module, in Server Manager, go to Manage –> Add Roles and Features Wizard.  Expand the Web Server (IIS) role, and under Web Server –> Application Development, check Application Initialization.  Complete the installation.

![Application-Initialization3](/assets/images/2012/06/Application-Initialization3.png)

Next, find your Web Application application pools.  Right click each one and go to Advanced Settings.  Under (General), change the Start Mode from OnDemand to AlwaysRunning:

![image4](/assets/images/2012/06/image4.png)

Click OK.  If the pool did not already have a w3wp.exe process, one will immediately start.  The next step is to have IIS pre-load the application, helping alleviate JIT lag.

Open a PowerShell command prompt as Administrator.  Run the following command, replacing “SharePoint – 80” with the name of your site (not application pool):

```powershell
Set-WebConfigurationProperty '/system.applicationHost/sites/site[@name="SharePoint - 80"]/application' -Name "preloadEnabled" -Value "true" -PSPath IIS:\
```

When the Application Pool starts, IIS will send a fake request to the site, forcing a JIT compile to take place. This will speed up the first request an end user makes.

This module is also available for IIS 7.5 [here](http://www.iis.net/download/ApplicationInitialization).  Further information and settings about the IIS 8.0 Application Initialization module can be found at [learn.iis.net](http://learn.iis.net/page.aspx/1089/iis-80-application-initialization/).