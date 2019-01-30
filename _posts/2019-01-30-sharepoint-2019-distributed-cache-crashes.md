---
layout: post
title: SharePoint 2019 Distributed Cache Crashes
tags: [sp2019]
---

If your Distributed Cache is crashing repeatedly in your brand new SharePoint 2019 farm, there is a fairly simple fix you can run. This script was provided by PSS -- it has worked for myself and others who have used it. Review the script -- there are _manual steps_ as notated in the comments. These steps must be executed on a server hosting Distributed Cache.

```ps
Use-CacheCluster

# Stop the Caching Services on all cache hosts in the cluster.
Stop-CacheCluster

# Export existing cache cluster configuration
Export-CacheClusterConfig -Path C:\temp\appfabconfig.txt

# make a copy of "appfabconfig.txt" and name it "appfabconfig2.txt"
# Edit appfabconfig2.txt
# Change <caches partitionCount="256" to "128"

# Import the changes.
Import-CacheClusterConfig -Path c:\temp\appfabconfig2.txt

# Start the Caching Services on all cache hosts in the cluster.
Start-CacheCluster

# Stop the Caching Services on all cache hosts in the cluster.
Stop-CacheCluster

# Import the original settings
Import-CacheClusterConfig -Path c:\temp\appfabconfig.txt

# Start the Caching Services on all cache hosts in the cluster.
Start-CacheCluster
```