---
layout: post
title: After applying KB2677070, SPTimerV4 and Claims to Windows Token Service Fail
tags: [sp2010]
---

After applying [KB2677070](http://support.microsoft.com/kb/2677070), the SharePoint Timer Service (SPTimerV4) and Claims to Windows Token Service fail to start.  This will occur if the SharePoint Server(s) do not have Internet access (tcp/443) in order to update the root server certificates from Microsoft.

In this case, manually download and install the new root certificates on the SharePoint Server(s):

<http://ctldl.windowsupdate.com/msdownload/update/v3/static/trustedr/en/disallowedcertstl.cab>

<http://ctldl.windowsupdate.com/msdownload/update/v3/static/trustedr/en/authrootstl.cab>