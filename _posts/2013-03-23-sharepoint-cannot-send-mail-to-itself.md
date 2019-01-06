---
layout: post
title: SharePoint Cannot Send Mail to Itself
tags: [sp2010]
---

When using the built-in mechanism to send email from SharePoint (or [SPUtility.SendEmail](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.utilities.sputility.sendemail.aspx)), SharePoint stamps a `X-Mailer` header of:

```text
Microsoft SharePoint Foundation 2010
```

If SharePoint is configured to send to the same SMTP service it is set to pick mail up from, SharePoint will drop any message with the above X-Mailer header indicating a possible message loop.