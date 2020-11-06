---
layout: post
title: Overriding Sender for Sharing Emails
tags: [sp2016,sp2019]
---

In SharePoint Server 2013 and above, when a user sends an invite to another user, it sends the invite _as that user_. This can lead to SMTP servers rejecting mail from a SharePoint server due to lack of send on behalf rights.

Microsoft introduced a fix for SharePoint Server 2016 (and available in 2019 RTM) in November of 2017. [KB4011244](https://support.microsoft.com/help/4011244/) allows a farm admin to disable the send on behalf behavior, reverting to the SharePoint Server 2010 and below behavior of sending as the email address configured in the Outgoing Email settings of the farm or Web Application.

This is set via PowerShell on a per-Web Application basis.

```powershell
$wa = Get-SPWebApplication https://wa
$wa.OutboundMailOverrideEnvelopeSender = $true
$wa.Update()
```
