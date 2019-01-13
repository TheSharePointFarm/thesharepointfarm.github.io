---
layout: post
title: Announcing the Release of Nauplius.SharePoint.BlobCache 1.8
tags: [development,news,sp2010,sp2013]
---

This is a major "behind the scenes" release, with some new additions for SharePoint 2010 and SharePoint 2013. In the short-lived 1.6.1 release, a new setting for SharePoint 2013 was added, debugMode. Included in this release as well, this function adds a "hit-count" in the response headers, which can be inspected with a tool such as Fiddler. For example, with debugMode set to true, when using Fiddler with an asset that has been cached in the BLOB cache, you'll see the following in the response header:

![debugModeHitCount](/assets/images/2014/05/debugModeHitCount.png)

For both SharePoint 2010 and 2013, there is now a counter displayed for how many times the BLOB cache has been flushed on the Web Application. In addition, both versions now have a "Restore Defaults" button. This button removes all settings made through the BLOB cache tool, or manually and reverts them to the out-of-the-box settings.

You can [download](https://blobcache.codeplex.com/releases) the solution and find the [documentation](https://blobcache.codeplex.com/documentation) at the [project site](https://blobcache.codeplex.com/).