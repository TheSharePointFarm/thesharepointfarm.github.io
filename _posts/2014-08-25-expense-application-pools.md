---
layout: post
title: The Expense of Application Pools
tags: [sp2010,sp2013,sp2016,sp2019]
---

Microsoft's current recommendation for SharePoint is to leverage a single Web Application and as few Application Pools as possible. This is due to the overhead of not only each Web Application (additional timer jobs), but IIS Application Pools as well. This article will cover the memory expenses of Web Applications running on separate IIS Application Pools, a fairly common configuration practice.

This SharePoint 2013 SP1 farm runs on Windows Server 2012 R2. SQL Server 2014 is hosted on another virtual machine running Windows Server 2012 R2 Core.

The farm has the default Service Instances running. In addition, no Service Applications have been created. There are four Web Applications: SP04-A, SP04-B, SP04-C, and SP04-D. SP04-A and SP04-D share the same IIS Application Pool, SP04-A. The other Web Applications have an IIS Application Pool named the same as the Web Application itself. In addition, each Web Application has a single root Site Collection, using the Team Site template.

Using [VMMap](http://technet.microsoft.com/en-us/sysinternals/dd535533.aspx), we'll be paying close attention to the processes' Private Working Set. To give you some background, the Private Working Set is a region of the _allocated_ memory of the process that is not shared. This region does not include memory-mapped files (files loaded from disk into memory), but may include memory _allocated_ to those memory-mapped files. One thing you'll notice is that a good portion of the Working Set is allocated to the [Managed Heap](http://msdn.microsoft.com/en-us/library/f144e03t(v=vs.110).aspx). This is a contiguous location of reserved memory for the application, where .NET code is loaded.

What we want to identify is the impact of loading each IIS Application Pool (SP04-A, SP04-B, SP04-C), and what is the impact of loading additional Web Applications into a shared IIS Application Pool (SP04-D into SP04-A). In order to monitor memory usage, from a client machine we'll simply be going to http://hostname for each Web Application. This will spin up the Application Pool (not applicable for SP04-D).

Starting with SP04-A, after an Iisreset, we can see the Private Working Set using roughly 467 MB virtual memory, with most of that being assigned to the Managed Heap (or .NET code).

![SP04-A](/assets/images/2014/08/SP04-A.png)

Adding SP04-B, the Private Working Set is roughly 379 MB:

![SP04-B](/assets/images/2014/08/SP04-B.png)

Now, SP04-C, the third Web Application with its own Application Pool. Roughly 411 MB for its Private Working Set.

![SP04-C](/assets/images/2014/08/SP04-C.png)

Here, we're adding SP04-D. SP04-D is a separate Web Application, but uses the same Application Pool that SP04-A uses.

![SP04-A-D](/assets/images/2014/08/SP04-A-D.png)

Notice the Private Working Set has increased from 467 MB to about 526 MB, or a difference of 59 MB. That is significantly better than an entire new Application Pool running at 350 MB+!

Clearly, given the choice, a single Application Pool for the Web Application is a more economical choice. In addition, each Web Application that starts up after the first Web Application has spun up the Application Pool responds to the end user in a shorter period of time when using a single IIS Application Pool.

So, what about Microsoft's recommendation of using Host-Named Site Collections? Some in the SharePoint community, and rightly so, believe that the HNSC recommendation is simply to make it 'easier' to migrate an on-premises installation to SharePoint Online. While clearly that is Microsoft's end-goal, how about we take a look at it from a performance perspective?

There is a new Host-Named Site Collection Web Application, SP04-MT, assigned an Application Pool of the same name. The Root Site Collection is http://sp04-mt.nauplius.local, with two other Site Collections: http://sp04-mt2.nauplius.local and http://sp04-mt3.nauplius.local, all using the standard Team Site template.

Here is the memory usage after allowing http://sp04-mt.nauplius.local to fully load.

![SP04-MT-1](/assets/images/2014/08/SP04-MT-1.png)

So we have a Private Working Set of 535,420 KB. Now, let's load the second Site Collection, http://sp04-mt2.nauplius.local.

![SP04-MT-2](/assets/images/2014/08/SP04-MT-2.png)

A Private Working Set of 537,604 KB! A difference of a little over 2 MB. In addition, the Site Collection is spun up instantly, thanks to our binaries having already been [JIT'ed](http://msdn.microsoft.com/en-us/magazine/cc163791.aspx). Now, the last Site Collection, http://sp04-mt3.nauplius.local.

![SP04-MT-3](/assets/images/2014/08/SP04-MT-3.png)

The Private Working Set is now at 538,484 KB. A difference of 880 KB from loading the second Site Collection, and just under a 3 MB difference from the original load of the Application Pool.

From a performance and memory utilization perspective, leveraging Host-Named Site Collections should be the goal of SharePoint Administrators, where the solution can be applied. If Host-Named Site Collections for some reason cannot be used, instead use the same IIS Application Pool for all of the Web Applications in the farm, with the exception of Central Administration.