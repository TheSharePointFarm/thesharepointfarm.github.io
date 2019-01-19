---
layout: post
title: SharePoint 2016 New Response Header
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 includes a new response header. Named X-SP-SERVERSTATE, this one is very simple. It just tells you if the site is Read Only or not.

The header is returned on calls to the API endpoint, such as "/_api/search/searchcenterurl", and calls to "/_vti_bin/".

The value if the site is marked as `SPSite.ReadOnly = true` is `X-SP-SERVERSTATE: ReadOnly=1`. Otherwise, it will read `X-SP-SERVERSTATE: ReadOnly=0`.