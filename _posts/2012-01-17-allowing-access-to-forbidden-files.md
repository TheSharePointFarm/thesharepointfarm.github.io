---
layout: post
title: Allowing Access to ‘Forbidden’ Files
tags: [sp2010]
---

SharePoint, by default, [blocks a variety of file types](http://technet.microsoft.com/en-us/library/cc262496.aspx).  IIS also blocks any file type that does not have a defined MIME Type or is specified in the [HttpForbiddenHandler](http://msdn.microsoft.com/en-us/library/ms404282(v=vs.90).aspx).  The default list is:

.asax, .ascx, .master, .skin, .browser, .sitemap,  .config, .cs. .csproj, .vb, .vbproj, .webinfo, .licx, .resx, .resources, .mdb, .vjsproj, .java, .jsl, .ldb, .ad, .dd, .ldd, .sd, .cd, .adprototype, .lddprototype, .sdm, .sdmDocument, .mdf, .ldf, .exclude, .refresh

In order for SharePoint to serve up a file on the defined block list, navigate to Central Administration, go to Manage Web Applications, and highlight the target Web Application.  Click on Blocked File Types and remove the target file type.

In this example, we have:

![image6](/assets/images/2012/01/image6.png)

* ActiveDirectoryDates.vb
* DIIUnin.exe
* EventStreamService.cs
* twain.dll
* Web.config

.Vb, .exe, .dll, and .config were removed from the blocked file types for this particular Web Application.  .Cs is not a defined blocked file type in SharePoint.  The file types .dll and .exe can now be downloaded by highlighting them and selecting Download a Copy.  Clicking on any of the other files yields a 404 due to IIS blocking the file type as well.

To unblock .config, .cs, and .vb, open IIS Manager and find the IIS site that needs to be modified.  Under the Request Filtering feature on the File Name Extensions tab, unblock the desired file types by right clicking them and selecting Remove.  At this point, you will be able to download .cs and .vb via clicking on the files, and .config via Download a Copy.  In order to download a file named Web.config by clicking on it, under the Request Filtering feature, go to the Hidden Segments tab and remove “web.config”.  I do not recommend doing this as it could allow clients to download the web.config for the specific Web Application which may contain sensitive information!

To allow .dll and .exe files to be downloaded from SharePoint by clicking on them instead of having to use Download a Copy, another step must be taken.  This may not be supported by Microsoft.  You've been warned!  In the Feature View in the IIS Manager for the IIS site, go to Handler Mappings.  Find the Handler Mappings “CGI-exe” and “ISAPI-dll” and remove them.  You will now be able to download the files without having to use Download a Copy.

The last type of file IIS will block is one not defined in the IIS MIME Types.  To add a new MIME Type/extension, in IIS Manager, click on the server node.  Under MIME Types, add the new MIME type to the list.  Note that IIS will attempt to detect the file’s MIME Type, even if it doesn’t match an existing extension, so not every extension needs to be added.

This should cover the out of the box scenarios for file types that are blocked by SharePoint and/or IIS, and how to resolve them.