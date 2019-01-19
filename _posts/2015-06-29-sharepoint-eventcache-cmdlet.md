---
layout: post
title: SharePoint EventCache Cmdlet
tags: [development,news,sp2010,sp2013]
---

The [EventCache table](https://msdn.microsoft.com/en-us/library/office/bb861918(v=office.14).aspx) stores a lot of valuable information about what has occurred in a SharePoint content database, including information that is either difficult or not possible to find within the ULS log.

To facilitate this, I've created the [SharePoint EventCache Cmdlet](https://github.com/Nauplius/EventCache/releases), which can be found on my [Github repro](https://github.com/Nauplius). This cmdlet will let you perform a query of the table with parameters without needing SharePoint installed on the local machine.

Setup is made easy via a standard MSI installer.

Let me know what you think!