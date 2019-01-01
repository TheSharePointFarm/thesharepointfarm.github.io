---
layout: post
title: SharePoint – Out of Memory When Dealing with Large Files
tags: [sp2007]
---

I had a case where an approximately 300MB file was stored in a Document Library.  The WSS server was allocated 3GB of RAM and was running 32bit Server 2003 SP2.  When attempting to open said file, this error occurred:

![image3](/assets/images/2010/09/image3.png)

Microsoft has some good information when dealing with large files here.  Some highlights are:

_End-users should not use the Download a Copy item on the Send To sub menu on the Edit menu in the document libraries. The Download a Copy option opens the full file in the memory of the Web server._

Encourage end-users not to use Explorer view to view large document libraries. Instead, they should use the All Documents view. When you open a SharePoint document library in Explorer view, placing the pointer over any of the enumerated files requests metadata for all of the files in the folder that you are browsing. In some cases, this might request the whole file. In folders that contain lots of items, this process can take a long time and might affect the performance of the server farm.