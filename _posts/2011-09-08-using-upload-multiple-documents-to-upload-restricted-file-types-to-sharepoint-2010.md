---
layout: post
title: Using Upload Multiple Documents to upload restricted file types to SharePoint 2010
tags: [sp2010]
---

Certain file types are restricted by IIS by default.  These file types cannot be uploaded to SharePoint 2010 when you select “Upload Multiple”.

![image4](/assets/images/2011/09/image4.png)

![image13](/assets/images/2011/09/image13.png)

All other file types are allowed.  To make an exception to this list, go to the Web Site (Web Application) or Server level node in IIS Manager.  Double click on Request Filtering.  Highlight the file extension you want to change from restricted to allowed, and click Remove on the right hand side bar.  Then add the extension back in as explicitly allowed.  For example, here I've enabled .cs (C# source files) to the allowed list on a particular Web Application:

![image12](/assets/images/2011/09/image12.png)

You can check the IIS log for the web site and look for “404 7” status codes.  This indicates that the file extension was denied.