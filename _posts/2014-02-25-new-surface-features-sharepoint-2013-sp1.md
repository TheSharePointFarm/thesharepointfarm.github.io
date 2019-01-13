---
layout: post
title: SharePoint 2013 and Office 365 with Yammer Integration
tags: [sp2013]
---

Service Packs typically contain little-to-no features. SharePoint 2013 SP1 is no different, however there are a couple of things I wanted to point out.

First, both Yammer and OneDrive are apparently a critical SharePoint Central Administrator error:

![YammerOneDriveAvailabile](/assets/images/2014/02/YammerOneDriveAvailabile.png)

You can now configure Yammer and OneDrive for Business on Office 365 with SharePoint On-Premises integration. This can be done via Central Administration:

![ConfigureYammerOneDriveO365](/assets/images/2014/02/ConfigureYammerOneDriveO365.png)

OneDrive now replaces SkyDrive in the top link bar. Included with this is Yammer, if you activate the Yammer feature on-premises via Central Administration.

![TopLinkBar](/assets/images/2014/02/TopLinkBar.png)

When you click on the Yammer link, you'll be directed to /_layouts/15/Yammer.aspx and be asked to log in. That's about it...

![YammerWelcome](/assets/images/2014/02/YammerWelcome.png)

Once Yammer is activated, the SharePoint "newsfeed" will now display this message when you visit your MySite.

![YammerActivatedNewsFeed](/assets/images/2014/02/YammerActivatedNewsFeed.png)

For OneDrive for Business on Office 365 integration, again in Central Administration -> Office 365 -> Configure OneDrive and Sites Link, you'll simply input your OneDrive host in Office 365 (<https://<tenant>-my.sharepoint.com>) and the top link bar will redirect you there when you click on OneDrive.

![OneDriveForBusinessO365](/assets/images/2014/02/OneDriveForBusinessO365.png)

That is it for 'on the surface' new features!

Another note I wanted to point out is use [Russ Max's installation script](http://blogs.msdn.com/b/russmax/archive/2013/04/01/why-sharepoint-2013-cumulative-update-takes-5-hours-to-install.aspx). It will save you hours of deployment on Service Pack 1. In addition, a reboot will not necessarily be required.