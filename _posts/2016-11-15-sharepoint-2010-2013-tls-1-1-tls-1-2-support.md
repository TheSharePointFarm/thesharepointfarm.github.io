---
layout: post
title: SharePoint 2010 and 2013 TLS 1.1 and TLS 1.2 Support
tags: [sp2010,sp2013]
---

Microsoft now officially supports TLS 1.1 and TLS 1.2 with [SharePoint 2010](https://technet.microsoft.com/en-us/library/mt773992(v=office.14).aspx) and [SharePoint 2013](https://technet.microsoft.com/en-us/library/mt773991.aspx) (Foundation, Server, and Project Server).

The steps to force-enable TLS 1.1 or TLS 1.2 are the same as in [SharePoint Server 2016]({% post_url 2016-04-04-enabling-tls-1-2-support-sharepoint-server-2016 %}), beyond the additional applications you need to update per the previous links. Remember there are a few caveats to forcing TLS 1.1 and 1.2, namely with other server software that must communicate back (such as Workflow Manager) and non-Windows 10 Windows Explorer (WebDAV; Open with Explorer) client connections (just don't use them!). Make sure you test all functionality in a non-production environment before configuring production.

Office Online Server has [instructions](https://technet.microsoft.com/library/mt791311%28v=office.16%29.aspx) to support TLS 1.1 and TLS 1.2.