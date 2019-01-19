---
layout: post
title: SharePoint Server 2016 Zero Downtime - The Real World
tags: [sp2016]
---

So what is SharePoint Server 2016 Zero Downtime in the real world? With the release of the first SharePoint Server 2016 patch, we can officially say -- it's a bit less than zero. Zero Downtime means exactly one thing:

The farm is never fully taken offline.

What does that mean for individual farm members? They require service restarts and/or reboots. They'll be offline.

![ZeroDowntime](/assets/images/2016/04/ZeroDowntime.png)

![ZeroDowntime2](/assets/images/2016/04/ZeroDowntime2.png)

This is why Zero Downtime _requires_ High Availability of all services [you want to keep available]. This is certainly an improvement over previous versions of SharePoint where you had the inevitable downtime when running the Config Wizard as the database schema are updated, but for those not running highly available farms, expect downtime for your services and plan accordingly.

![ZeroDowntime3](/assets/images/2016/04/ZeroDowntime3.png)

I would continue to encourage you to look at [Russ Maxwell's patch script](https://blogs.msdn.microsoft.com/russmax/2013/04/01/why-sharepoint-2013-cumulative-update-takes-5-hours-to-install/), replacing all instances of "OSearch15" with "OSearch16". Like SharePoint Server 2013, your patching process will be rather slow without a similar script in place.

You can test "Zero Downtime" by installing [KB2920721](https://www.microsoft.com/en-us/download/details.aspx?id=51701) today!