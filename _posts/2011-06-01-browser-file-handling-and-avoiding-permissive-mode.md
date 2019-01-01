---
layout: post
title: Browser File Handling and avoiding Permissive mode
tags: [sp2010]
---

A change in behavior from SharePoint 2007 to SharePoint 2010 is that only file types marked as "safe" are allowed to be opened in the browser (think PDF) or opened on a download request (instead of just saved to disk).

One way to work around this is to set the Browser File Handling policy to Permissive, from Strict, on the Web Application.  This has a negative impact on security as clients may open harmful content from SharePoint without being prompted to do so.

The preferred way of handling this is to leave the Browser File Handling on the Strict mode, then from the SharePoint Command Shell, run:

```powershell
$webApp = Get-SPWebApplication("http://webAppUrl")
$webApp.AllowedInlineDownloadedMimeTypes.Add("application/pdf")
$webApp.Update()
```

This example is to add the MIME type for PDF to be allowed to open within the browser window instead of forcing the user to save the file to disk before opening it.