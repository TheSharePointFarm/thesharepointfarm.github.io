---
layout: post
title: MS13-052 Causes issues with SharePoint
tags: [sp2010]
---

Update 7/25/2013: [KB2872441](http://support.microsoft.com/kb/2872441) will resolve this issue.

Thanks to [@kelvin_hoyle](https://twitter.com/kelvin_hoyle) for pointing this one out, the .NET Security patch [MS13-052](http://support.microsoft.com/kb/2844286) is causing issues with SharePoint.  For now, it is recommended to not deploy MS13-052 to SharePoint servers.  See this [TechNet thread](http://social.technet.microsoft.com/Forums/sharepoint/en-US/cc9a557b-93cd-40d5-965c-e0a2f107624d/unable-to-display-this-web-part-error-message-after-patch-kb2844286) for further details.