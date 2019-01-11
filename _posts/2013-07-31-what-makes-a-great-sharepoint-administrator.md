---
layout: post
title: What Makes a Great SharePoint Administrator?
tags: [news]
---

So what makes a great SharePoint Administrator?  These are my thoughts on things to look for in a great SharePoint Administrator; someone who will be responsible for one of the companies’ most important collaboration and information platforms.

Before we get to the technologies, basic troubleshooting skills are a must.  Without the basics, a SharePoint Administrator may not have the appropriate knowledge to troubleshoot the appropriate technology.  As much as I hate to say it, SharePoint farms will have problems.  From mild, to irritating, to severe.  Without core troubleshooting abilities of Windows systems and the various first and third party tools involved in troubleshooting, identifying the root cause and potential solution can be a significant challenge.

I’m presenting these technologies in what I feel are order of importance, from least to most important.

* DNS

DNS is a requirement to understand how the hostnames for the Web Applications are looked up.  Troubleshooting with nslookup can also help a SharePoint Administrator diagnose various end user accessibility issues.

* Active Directory

A basic understanding of Active Directory is important for the SharePoint Administrator.  If a SharePoint Administrators manages the User Profile Service, works with users across Domain Trusts, handles Incoming Email, or works with Kerberos, a more in-depth Active Directory knowledge base is required.

* Virtualization

If virtualization is being used for the SharePoint and/or SQL Servers, the SharePoint Administrator should be able to identify and provide information on how virtualization can affect SharePoint and the restrictions Microsoft has in place for SharePoint (e.g. no dynamic memory, time synchronization, snapshotting, and so forth).

* SQL Server

I usually put this one ‘low’ on the need-to-understand list, but SQL is important.  The good thing about being a SharePoint Administrator is you do not have to understand the internal structure of SharePoint databases; they should be treated as a ‘blackbox’ (and if you have DBAs, let them know this, too!).  This means the focus is on server administration.  Making sure the SQL instance is configured correctly for SharePoint, maintenance plans are in place, and performance of the instance is monitored.  If using SQL high availability, follow Microsoft SharePoint guidelines surrounding high availability methods.

* IIS, ASP.NET, and the .NET Framework

Next in line for skills would be an in depth understanding of IIS and it’s relationship with the .NET Framework and ASP.NET.  A SharePoint Administrator should have a throughough understanding of IIS Sites, Application Pools, configuration files (machine, applicationHost, and web.configs).  In addition, an understanding of how IIS handles user authentication (AuthN) is a must.

* SharePoint

Of course, knowing SharePoint is one of the most important parts of being a SharePoint Administrator.  They need to understand an ever growing amount of core concepts behind managing and maintaining a SharePoint farm, often expanding into Microsoft’s BI stack, Office Web Apps, and Project Server.

* PowerShell (SharePoint Management Shell)

PowerShell skills are very high up on the ‘must have’ for a good SharePoint Administrator.  Certain things in SharePoint can only be done via PowerShell or are significantly more efficient to do in PowerShell.  Other things, such as creating a repeatable environment, can also only be done via PowerShell.  Every SharePoint Administrator should know, understand, love, and enjoy using PowerShell day in and day out.

And here is where I believe a good SharePoint Administrator evolves into a great SharePoint Administrator.  Blurring that line between Administration and Development, a SharePoint Administrator should have limited knowledge of C# and process debugging.  Many issues that are encountered in SharePoint are often internal to the SharePoint code itself.  Knowing how to read C# and being able to use tools like .NET Reflector to decompile and run SharePoint code through the Visual Studio debugger will provide a SharePoint Administrator with that final bit of troubleshooting that may allow them to avoid the bug in the first place, identify a work around, work with PSS, or keep an eye out for future resolutions to the particular bug.  Unfortunately, as with a lot of Microsoft software, stack traces or error messages themselves do not provide a complete view of the issue at hand.  It isn’t until looking under the covers does the error become apparent.

I would strongly urge any SharePoint Administrator who hasn’t taken that last step to try it out.  Find a known issue in a SharePoint Cumulative Update and try to identify the bug itself in the Visual Studio debugger.  A new world of troubleshooting will open up, and it can be quite rewarding.  In addition, I believe you’ll also discover “how” things work in SharePoint.

This is what I believe makes a great SharePoint Administrator.

What is your take on what makes a great SharePoint Administrator?  I’d love to hear your thoughts!  Please feel free to post a comment.