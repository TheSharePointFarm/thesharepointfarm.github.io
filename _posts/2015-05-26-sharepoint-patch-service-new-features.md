---
layout: post
title: SharePoint Patch Service New Features
tags: [development,news,sp2010,sp2013]
---

The SharePoint Patch Service is a service to help you discover available SharePoint patches and community-sourced known issues with those patches. The first release of the service was on 3/24/2015, and has undergone some major changes with a significant amount of features added to help developers and users alike make it easier to use and provide more valuable information.

In addition to the service being improved, a set of PowerShell cmdlets that leverage the service have been created. These [PowerShell cmdlets](https://sharepointupdates.com/PowerShell) let you easily review, download, extract, and install SharePoint patches to SharePoint 2010 and SharePoint 2013. Each PowerShell cmdlet has help and examples provided. The list of PowerShell cmdlets are:

* `Get-SPPatchInfo`  - Retrieves information about SharePoint patches from this web service.
* `Get-SPPatch`  - Downloads patches based on KB# from Microsoft.
* `Extract-SPPatch`  - Extracts patches from Microsoft's 'legacy' patch service. This does not support patches from the Microsoft Download Center.
* `Install-SPPatch`  - Installs a SharePoint patch. Included are the options to stop SharePoint services as well as pause SharePoint Search, if applicable, helping to [remediate long installation times](http://blogs.msdn.com/b/russmax/archive/2013/04/01/why-sharepoint-2013-cumulative-update-takes-5-hours-to-install.aspx). This cmdlet requires SharePoint to be installed locally where the cmdlet is run. In addition, the correct version of SharePoint must be selected at installation time.

The service also now has a UI! For those who do not want to leverage the API, you can now search SharePoint Builds and Known Issues. A Contact page is also available -- if you have found an issue, or I made a mistake in the data provided.

If there are any additions or changes you would like to see, or have an issue you want to report, feel free to reach out and let me know!