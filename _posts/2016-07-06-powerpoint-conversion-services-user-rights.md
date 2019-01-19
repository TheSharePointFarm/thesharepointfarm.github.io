---
layout: post
title: PowerPoint Conversion Services User Rights
tags: [sp2016]
---

n SharePoint Server 2013, PowerPoint Conversion Services was required to be run as a Farm Administrator as users in the WSS_WPG local group, where non-Farm Administrator service accounts are added, did not have the ability to create folders in C:\ProgramData\Microsoft\SharePoint\. For PowerPoint Conversion Services, a folder is created by the user running the IIS Application Pool where the Service Application resides named 'PowerPointConversion'.

In SharePoint Server 2016, WSS_WPG now has the ability to create the PowerPointConversion folder as WSS_WPG has Full Control rights on C:\ProgramData\Microsoft\SharePoint\.

One additional note to make is there is no binding redirection for Microsoft.Office.Server.PowerPoint.dll, which means your solution must rebind this DLL prior to deployment for SharePoint Server 2016. If the solution is deployed without the rebinding of Microsoft.Office.Server.PowerPoint.dll, during the conversion process you will see a 'File Not Found' error in the UI (if you're raising errors in your UI, should you have one), with the more specific error about the v15 binary not being found.