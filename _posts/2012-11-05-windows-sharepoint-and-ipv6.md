---
layout: post
title: Windows, SharePoint, and IPv6
tags: [ps2010,sp2010,sp2013,sql]
---

I often see talk about administrators disabling IPv6 for one reason or another.  A lot of that comes down to “I don’t use it, so I don’t leave it enabled”.  Always makes sense from an administrator perspective… except when it comes to IPv6.

Microsoft has made sure, since Windows Vista and Server 2008, that IPv6 was a first-class citizen in Windows.  In fact, so first-class that Windows will leverage IPv6 before attempting to leverage IPv4.  If there is an IPv6 path available, IPv6 will be used.

Microsoft has an [FAQ on IPv6](http://technet.microsoft.com/en-us/network/cc987595.aspx) which specifically states while they understand that some administrators do disable IPv6 in Windows, IPv6 is a mandatory part of Windows and do not test disabling IPv6 during the development and testing process for Windows (this means leave IPv6 enabled!).

[SharePoint 2010 fully supports IPv6](http://technet.microsoft.com/en-us/library/cc748826.aspx) in an IPv6-only or a mixed IPv4 and IPv6 environment.  All supported editions of SQL Server for SharePoint 2010 also support IPv6.  The only restriction with IPv6 for SharePoint is that you cannot browse directly to the IP address over the HTTP protocol (e.g. browsing to <http://[fe80::bc95:be32:cc03:845b%2511]> would not work) and all DNS records must be using quad-A (AAAA) records for IPv6 addressing.  Any farm element that otherwise can take an IP address will accept an IPv6 address, given it is enclosed in brackets “[IPv6-Address]”.

[The world is out of IPv4 address](http://en.wikipedia.org/wiki/IPv4_address_exhaustion).  It is time to start testing that IPv6 rollout!