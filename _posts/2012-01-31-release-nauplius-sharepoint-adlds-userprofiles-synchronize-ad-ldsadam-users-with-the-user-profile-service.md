---
layout: post
title: Release of Nauplius.SharePoint.ADLDS.UserProfiles – Synchronize AD LDS/ADAM Users with the User Profile Service
tags: [sp2010]
---

I’m please to announce the Beta 1 release of [Nauplius.SharePoint.ADLDS.UserProfiles](http://sharepointadlds.codeplex.com/releases/)!  This application will allow you to synchronize one or more AD LDS/ADAM Application Directory partitions with the SharePoint 2010 User Profile Service.

To get started, modify the Nauplius.SharePoint.ADLDS.UserProfiles.exe.config.  Change the sample partition by supplying your own AD LDS/ADAM server name, port AD LDS/ADAM is listening on, whether or not it is using SSL, the distinguished name where user’s reside (or parent container), which Web Application the AD LDS/ADAM users are logging on to, and finally what attribute they’re using to log on to SharePoint with.

Validate that the user running the application has at least Reader level access to the AD LDS instance, as well as full permissions to the UPA the Web Application is consuming.

Run `Nauplius.SharePoint.ADLDS.UserProfiles.exe` with an Administrator token and profiles will begin to import.

In the future, this will application will come with an installer as well as fully run as a Windows Service without the need to use the command line.  It will also provide a logging facility to help troubleshoot any errors.

This application is licensed under the GPLv2.

Please contact me if you run into any errors.

Thanks!