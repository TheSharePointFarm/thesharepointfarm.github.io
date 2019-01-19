---
layout: post
title: SharePoint Support for Disabling SSL 3.0 and TLS 1.0
tags: [sp2010,sp2013]
---

Update 7/1/2016: TLS 1.1 and TLS 1.2 are now supported for SharePoint 2010 and 2013.

SharePoint 2010 2013 relies on an old default: SSL 3.0 and TLS 1.0 for secure communication. While you can disable SSL 3.0 on SharePoint servers, you cannot disable TLS 1.0.

A [.NET hotfix](https://technet.microsoft.com/library/security/2960358) was add support for TLS 1.1 and TLS 1.2 in the [.NET 4.5 Framework](https://msdn.microsoft.com/en-us/library/system.security.authentication.sslprotocols(v=vs.110).aspx), but this requires rebuilding the application that relies on the .NET framework in order to use the new protocols -- not something that will happen with SharePoint 2013.

Current versions of SQL Server also have the same limitation when using encrypted connections (which you should be).

So, disable SSL 3.0 on your SharePoint servers, but leave TLS 1.0 enabled. I created a [Group Policy ADMX](https://github.com/Nauplius/SchannelConfiguration) file to help with this in mass-deployments.