---
layout: post
title: SharePoint ASP.NET Compilation Errors (CS0016)
tags: [sp2013]
---

I encountered a rather frustrating issue with SharePoint 2013. Like most farms, this farm has a specific Domain User running the IIS Application Pool handling Web Applications. These users had a Windows Profile created (this is done by elevating the user temporarily to Administrator, then running an application such as cmd.exe or notepad.exe as that user). "Out of the blue", when users would navigate to certain built-in pages (e.g. List Settings, or Version Settings underneath List Settings)  they would generate errors similar to the below in the Application Event Log.

```
Log Name:      Application
Source:        Microsoft-SharePoint Products-SharePoint Foundation
Date:          7/30/2014 12:00:00 AM
Event ID:      7043
Task Category: Web Controls
Level:         Error
Keywords:      
User:          NT AUTHORITY\IUSR
Computer:      SHAREPOINT
Description:
Load control template file /_controltemplates/15/DetailsControl.ascx failed: (0): error CS0016: Could not write to output file 'c:\Windows\Microsoft.NET\Framework64\v4.0.30319\Temporary ASP.NET Files\root\0853d323\d847efaf\App_Web_detailscontrol.ascx.b16d5c94.pbri1kbd.dll' -- 'The directory name is invalid. '
```

In addition, when attempting to publish SharePoint Designer Workflows in SharePoint 2010 mode, SharePoint Designer would present the message:

![CS0016WorkflowError](/assets/images/2014/07/CS0016WorkflowError.png)

Another form of error the end user received was:

![CS0016Sorry](/assets/images/2014/07/CS0016Sorry.png)

As other suggestions on the Internet indicated, rebooting the server did work, for a little while. But eventually the errors would return (don't ask me why they wouldn't present themselves immediately, I don't know!). Eventually, [ProcessMonitor](http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx) showed that files could not be created in the user's Temp directory running the IIS Application Pool! Lo and behold, when navigating to `C:\Users\serviceaccount\AppData\Local\`, the Temp directory was missing!

Simply creating the Temp directory, so `C:\Users\serviceaccount\AppData\Local\Temp` resolved the issue immediately, no iisreset or restart required. The key here is that the user performing the compilation of ASP.NET pages needs to have a valid `%Temp%` path, where ever that location may be -- specifically, the error message "The directory name is invalid" refers to this %Temp% location.