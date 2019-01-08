---
layout: post
title: SharePoint DocKit 3.2.1 Review
tags: [review,sp2010,sp2013]
---

Full disclosure: As part of the MVP Program, [Acceleratio](http://acceleratio.net/) has offered MVPs a Consultant Premium license.  This license offers the full features on an unlimited number of farms.  However, I've been using a purchased farm license since the 1.0 version.

If you've ever had to document a SharePoint farm, perhaps even a single server installation, you know that the amount of documentation required can be enormous.  One farm I worked on was a 200+ page book, screenshots included, to document a medium sized farm.  This level of documentation was a requirement for disaster recovery purposes and frankly, nearly impossible to keep up-to-date with all of the changes to the farm (luckily those are well noted in helpdesk and change requests!).  With the [Documentation Toolkit for SharePoint](http://acceleratio.net/products/documentation-toolkit-for-sharepoint/) (SPDocKit), this process is automated for you.  Originally, SPDocKit just compiled the most relevant information and provided the output to you in Word or PDF format.  The tool has now grown into much more, providing far more settings that are documented for you, best practices, log review, permissions review, etc.

This overview covers version 3.2.1 on SharePoint 2013.  SPDocKit is also available for SharePoint 2010.

After installing the application, creating the SPDocKit database, and enabling the snapshot service to track farm changes, the first task is to load the farm data.  This will parse through the farm -- Web Applications, Service Applications, Services Instances, servers, hardware, and various other settings.  When the load is complete, we're left with a tree view of (almost) all of the settings you could want.  Here is my boring farm topology:

![SPDocKit2](/assets/images/2013/06/SPDocKit2.png)

This version has a Site Explorer feature, which allows you to view not only the Web Application settings related to sites, but also deployed solutions to both the Web Application and Site Collections.  One feature I like is the ability to display the People Picker settings.  This level of detail is essential to successful documentation for recovery purposes!

![SPDocKit3](/assets/images/2013/06/SPDocKit3.png)

Another feature I leverage on a weekly basis is the snapshot functionality.  These can be scheduled by way of the snapshot service (and emailed automatically), or taken manually.  In this example, I've taken a snapshot automatically (on initial load), changed a peoplepicker-searchadforests property on http://sp2013webapp1, then manually took another snapshot.  By comparing the snapshots, you can see how easily it is to narrow down changes to the farm:

![SPDocKit5](/assets/images/2013/06/SPDocKit5.png)

Not only is this invaluable for settings changes, but also for changes to security of the farm, such as additions or removal of Farm Administrators.  You can compare different farms (this would be valuable for comparing pre-production to production, for example), Web Applications, or Site permissions.

A relatively new feature is the Best Practices.  This shows some overall best practices for SharePoint, and whether or not the farm is configured according to those best practices (either as defined by Acceleratio, or custom defined best practices).  A helpful dashboard is included with an overall status, showing errors, warnings, and passes).

![SPDocKit7](/assets/images/2013/06/SPDocKit7.png)

Not all 'best practices' defined here will fit every farm deployment, but in general the values are appropriate.  This allows you to easily target and resolve issues that may otherwise go undetected, such as database autogrowth settings.

Back to the original purpose of SPDocKit, farm documentation, the output it generates is very well laid out and easy to read.  You can customize the style and company logo, but otherwise SPDocKit will insert a Table of Contents as well as great looking tables for the farm settings.  The output is customizable.  You can configure if you just want the overall farm topology, or even documentation down to the configuration of each Site Collection.

![SPDocKit9](/assets/images/2013/06/SPDocKit9.png)

The resulting output is similar to this:

![SPDocKit8](/assets/images/2013/06/SPDocKit8.png)

Each table is collapsible, which makes reading the Word document significantly easier.  Another option that is probably better suited for consultants or small businesses rather than enterprises is a SkyDrive or Dropbox synchronization feature.  This function allows you to automatically or manually synchronize farm settings, snapshots, and documentation to 'the cloud'.  I can see this as being useful for monitoring farms, but I can also see this as being an issue for security-conscious organizations allowing core documentation to reside off site in a location that could potentially be compromised.

A couple of negative points, the first being just a limitation of the SharePoint platform: SPDocKit must be run on a SharePoint server that is a member of the farm in order to load changes.  You can run SPDocKit on a machine without SharePoint to open a saved farm and perform a review of the existing settings, but you cannot update SPDocKit with changes to a farm.  The second function that I've had issues with is the Monitoring function, specifically the Event viewer built into SPDocKit.  Be careful when using this function as the memory for the SPDocKit just balloons when viewing the ULS log.  For example, loading 242K rows of ULS resulted in a 400MB increase in the SPDocKit process memory usage.  That said, it is very useful for diagnostic history.  It allows you to sort and filter log events, as well as export to Excel and PDF.  It is significantly easier and faster to parse through than using the [ULS Viewer](http://archive.msdn.microsoft.com/ULSViewer), providing Excel-like filtering and sorting.  This is not a replacement for the ULS Viewer as it does not monitor the log file in near-real time.

![SPDocKit6](/assets/images/2013/06/SPDocKit6.png)

A property that I actually need, but is not part of the feature set, is a record of IIS MIME Type additions as well as the `SPWebApplication.AllowedInlineDownloadedMimeTypes`.  Since all of my Web Applications are set to Strict, over time I've had to add various media files to this particular property.

Overall I feel this is a must-have product for SharePoint Administrators.  It has saved me many hours of documentation updates (I only wish it existed prior to building my farm!) and allows me to track changes to the farm easily and efficiently; changes I may otherwise be unaware of.  I'd encourage SharePoint Administrators to evaluate it on a test platform to see if it meets their organizations needs.