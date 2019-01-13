---
layout: post
title: SharePoint 2013 SP1 and Heartbleed
tags: [sp2013]
---

As many have noted, SharePoint 2013 Service Pack 1 downloads were pulled. This is due to a potential issue with upgrading from Service Pack 1 to any future update. This does not impact day-to-day operations and should not be a cause of concern for those currently running a farm upgraded to Service Pack 1. Microsoft will release a fix to address this particular issue so administrators may update to post-Service Pack 1.

The ISOs that include Service Pack 1 ("SharePoint 2013 with Service Pack 1") are not impacted by this particular issue. Those ISOs can be found on the MSDN Download Center and the Volume License website.

Now as for Heartbleed, because Microsoft does not use the OpenSSL crypto library, Microsoft products and properties (with the exception of Yammer) will not be vulnerable to this particular issue. This, by nature, extends to SharePoint. _However_, if the same SSL certificate was used with an IIS site as well as a site using the vulnerable OpenSSL crypto library, the certificate will still need to be revoked and reissued, and of course any user associated with the site should change their password(s).

Unfortunately this particular bug is very bad for users. They cannot know if a web server is vulnerable, and cannot know if and when the vulnerability is resolved. As always, it is best to regularly rotate passwords and use unique passwords on a site-by-site basis. I'd suggest using an application like LastPass, so you don't necessarily have to "remember" each password.

Patch Tuesday has passed, and as you may have noted, Patch Tuesday typically means Cumulative Updates for SharePoint. We're still waiting for the SharePoint 2013 CUs to be released. Hopefully the SP1 update issue will also be addressed with this release.

On another note, when the April 2014 CU is released for SharePoint 2013, we'll have official support for SQL Server 2014 RTM! The blocking issue was due to Forefront Identity Manager (when is FIM not the blocking issue?).