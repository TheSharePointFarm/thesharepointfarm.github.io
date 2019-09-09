---
layout: post
title: Farm Deployment Experiment - Farm Launch
tags: [sp2019]
---

In SharePoint Server 2016, I saw a reference to a class named `SPFarmInitialization`. That piqued my interest and I have [previously blogged]({% post_url 2016-03-14-unattended-configuration-for-sharepoint-server-2016 %}) about it. I decided to take that project one step further and build a PoC on what could be done with this. Turns out, it would be a neat way to build a farm, if you could build a complete farm.

This method uses an [unattended.txt](https://github.com/Nauplius/FarmLaunch/blob/master/FarmLaunch/ConfigFile/unattend.txt) file to build a farm. However, there was more to the _creation_ of the farm than simply dropping in a file and running the Config Wizard. Instead, using a [NamedDataSlot](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.getnameddataslot?view=netframework-4.8) one could run an application to initialize the build of the farm. Presumably this is done to pull secrets from a centralized location that itself is secure.

[Farm Launch](https://github.com/Nauplius/FarmLaunch) is a project I created to execute this process. You'll note that it uses hardcoded usernames, passwords, and other values in both the code as well as the supplied unattended.txt file that you will need to adjust should you want to try this on a test server. Because of this, I'm not providing a binary  -- you will have to edit and compile it yourself.

This solution does not support adding additional farm members and the `SPFarmInitialization` class does not support provisioning _most_ services, making it's current implementation only useful in scenarios where a new farm would be consuming services from another farm for Managed Metadata, Search, and the User Profile Service.

Ultimately, this is a thought experiment, but something I can only hope that the product group investigates for future versions of SharePoint Server.
