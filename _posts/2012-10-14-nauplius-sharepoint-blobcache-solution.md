---
layout: post
title: Nauplius.SharePoint.BlobCache Solution
tags: [sp2010]
---

This is a solution used to manage the BlobCache element for SharePoint Web Applications.  This release allows the Farm Administrator to modify all available elements of the BlobCache: location, path, maxSize, max-age, and enabled.

This prevents the need to modify web.configs across SharePoint Servers running the Foundation Web Application service as it leverages the WebConfigModification class.

Any comments, criticisms, or requests are [welcome](http://blobcache.codeplex.com/).  You can download the solution from <http://blobcache.codeplex.com>.