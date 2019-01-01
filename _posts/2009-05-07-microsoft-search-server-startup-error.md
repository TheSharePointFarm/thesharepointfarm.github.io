---
layout: post
title: Microsoft Search Server (Express) Startup Error (HOSTS file)
tags: [mss, sp2007]
---

You may encounter this error in the Application Event Log after installing MSS(E) in your farm:

```text
Event Type:        Error
Event Source:    Office SharePoint Server
Event Category:                Office Server Shared Services
Event ID:              6482
Date:                     5/4/2009
Time:                     11:00:49 AM
User:                     N/A
Computer:          SEARCHSERVER

Description:
Application Server Administration job failed for service instance Microsoft.Office.Server.Search.Administration.SearchServiceInstance (b2ef3361-3f34-41e4-8e2a-56cbd5fc78f4).
Reason: Access to the path 'C:\WINDOWS\system32\drivers\etc\HOSTS' is denied.

Technical Support Details:
System.UnauthorizedAccessException: Access to the path 'C:\WINDOWS\system32\drivers\etc\HOSTS' is denied.
```

For what ever reason, Microsoft wanted Search Server to edit the HOSTS file on the MSS(E) server.  In order to resolve this error, provide the WPG_ADMIN group with Modify NTFS permissions to the `%windir%\System32\drivers\etc` directory.  If you restart the Search Server service, the error should not appear after making that modification.