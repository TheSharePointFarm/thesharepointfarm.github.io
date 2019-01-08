---
layout: post
title: SharePoint’s Support for URL Rewrite
tags: [sp2007,sp2010,sp2013]
---

Recently I was ask about URL Rewrite support for SharePoint, given the article I previously posted about the Application Request Routing module, which leverages URL Rewrite.  Microsoft has released a KB article on what is, and is not supported for URL Rewrite (and ARR).

<http://support.microsoft.com/kb/2818415>

The supported scenarios are 301, 302 redirects, as well as symmetrical rewrites (e.g. `http://sharepoint/sites/sitename/` to `http://intranet/sites/sitename/`).

Asymmetrical rewrites (e.g. `http://sharepoint/sites/sitename/` to `http://intranet/sitename/`) are not supported.

Note that in a redirect or symmetrical rewrite, the Alternate Access Mapping must reflect the protocol, domain, and port of the target redirect or rewrite rule.

Rewriting of the path (`/sites/...`) is not supported!