---
layout: post
title: Updated 5 Hour SharePoint Patch Script
tags: [sp2013,sp2016]
---

Russ Maxwell [created a SharePoint 2013 patch script](https://blogs.msdn.microsoft.com/russmax/2013/04/01/why-sharepoint-2013-cumulative-update-takes-5-hours-to-install/) that prevented the binary patching process from taking quite a few hours to complete. This has come to be affectionately known as the '5 hour SharePoint patch script'.

It's a great script and I've used it many times with many customers. However, it does not handle SharePoint Server 2016 and I wanted to make a few minor adjustments. So I updated the script and put it on GitHub! The updated script handles both SharePoint 2013 as well as SharePoint 2016. Another improvement is handling multiple patches located in the same directory (obviously this does not mean multiple Cumulative Updates for SharePoint 2013 due to the cab files). This will allow you to apply multiple SharePoint 2013 patches (e.g. security patches or hotfixes, or a mix thereof) as well as allowing you to place both the sts and wssloc executables for SharePoint Server 2016 in the same directory and have the patch process complete for both without having to run the script twice.

As with Russ Maxwell's original script, drop the executables (and .cab files if it is a SharePoint 2013 Cumulative Update) into the same folder as the SharePointPatchScript.ps1. Run Powershell (or the SharePoint Management Shell) 'As Administrator' and it will ask you if you want to pause or stop the Search Service (this will affect all Search Service Applications on the farm). From there, it will perform a silent installation.

You can go grab it on GitHub at <https://github.com/Nauplius/SharePoint-Patch-Script/>. Feel free to pass on suggestions for this local patch script or even make pull requests to improve upon it. But more improvements from my side are to come. The current list is:

* Do not resume Search
* Make into a PowerShell Module - this would allow for parameter passing in a friendly manner
* Specify the patch path, rather than having to place the patch in the same directory
* Better informational messages about what the script is doing

Let me know if you have any other thoughts!