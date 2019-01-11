---
layout: post
title: Maximum Distance for Stretched SharePoint Farms
tags: [sp2010,sp2013]
---

SharePoint 2010, and after a revision, SharePoint 2013 support stretched farms.  Microsoft terms a ‘stretched’ farm by being a farm not contained within the same data center. There are some serious limitations on the performance of stretched farms, primarily with [network latency] distance and network speed.  I would not recommend implementing a stretched farm.  It requires careful planning between the SharePoint Admin and network admin (and possibly teleco).  It may also require some fairly expensive equipment for proper implementation.  Microsoft also does not recommend this, but will support it.

Stretched farms require <= 1 ms one-way response time over an average of 10 minutes, and 1Gbps connectivity between the SharePoint servers and SQL Server(s).  This is primarily due to certain service applications, e.g. the User Profile Service, not using the proxy to make calls to SQL Servers.

Now, if you live in a vacuum and your network equipment introduces absolutely no latency, your maximum stretched farm distance is 186.3 miles, or 299.8 km.  From personal experience, I haven’t seen an even moderate distance WAN (MPLS) provide 1ms latency over a period of 10 minutes.

If the goal is content replication, look into 3rd party products like [Metalogix Replicator](http://www.metalogix.com/Products/Replicator/Replicator-for-SharePoint.aspx) or [AvePoint Replicator](http://www.avepoint.com/sharepoint-replication-docave/) (at least I’m not alone in lacking the ability to come up with marketing-based naming).  With the Metalogix product, it replicates the content, but nothing below, for example Web Application, Farm settings, or Farm solutions so you will need to maintain those manually.

As always, test, test, test, and avoid stretched farms.