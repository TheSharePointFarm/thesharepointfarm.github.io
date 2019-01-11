---
layout: post
title: Using PowerShell to Manage SharePoint Information Rights Management Settings
tags: [sp2010,sp2013]
---

Information Rights Management (IRM) allows users to restrict how documents are handled.  With SharePoint, IRM settings are applied at the List/Library level.  When a document is added to an IRM-enabled Library, the IRM is stripped from the document.  When that document is downloaded from the Library, the document has the IRM settings from the Library applied to it.  This allows SharePoint to crawl the content.

Configuring IRM can be accomplished via PowerShell. This allows you to turn it on using the default server specified in Active Directory (which is set via the AD RMS Service Connection Point) or specify an existing server.

To enable IRM using the AD RMS SCP:

```powershell
$ws = [Microsoft.SharePoint.Administration.SPWebService]::ContentService
$ws.IrmSettings.IrmRMSEnabled = $true
$ws.IrmSettings.IrmRMSUseAD = $true
$ws.Update()
```

Or to enable IRM using a specified AD RMS server:

```powershell
$ws = [Microsoft.SharePoint.Administration.SPWebService]::ContentService
$ws.IrmSettings.IrmRMSEnabled = $true
$ws.IsrmSettings.IrmRMSUseAD = $false
$ws.IrmSettings.IrmRMSCertServer = "https://adrms.company.com"
$ws.Update()
```

And finally, to disable IRM:

```powershell
$webSvc = [Microsoft.SharePoint.Administration.SPWebService]::ContentService
$webSvc.IrmSettings.IrmRMSEnabled = $false
$webSvc.Update()
```

You can use PowerShell to manage IRM settings for each Library, and it is straightforward.  Properties that use the InformationRightsManagementSettings are available in SharePoint 2013 only.

First, bind to the Web and the List:

```powershell
$web = Get-SPWeb http://webUrl
$list = $web.GetList("LibraryTitle")
```

When making a change to a list property, make sure to call the Update() method, for example:

```powershell
$list.IrmEnable = $true
$list.Update()
```

Here are the various settings you can apply.  I'll be translating from the SharePoint UI to the PowerShell property.

![IRMLibrary](/assets/images/2013/07/IRMLibrary.jpg)

Restrict permissions on this library on download

```powershell
$list.IrmEnable = $true
```

Create a permission policy title

```powershell
$list.InformationRightsManagementSettings.PolicyTitle = "string_value"
```

Add a permission policy description:

```powershell
$list.InformationRightsManagementSettings.PolicyDescription = "string_value"
```

Do not allow users to upload documents that do not support IRM

```powershell
$list.IrmReject = $true
```

Stop restricting access to the library at

```powershell
$list.IrmExpire = $true
$list.InformationRightsManagementSettings.DocumentLibraryProtectionExpireDate = "10/15/2015" #valid date in MM/dd/yyyy format
```

Prevent opening documents in the browser for this Document Library

```powershell
$list.InformationRightsManagementSettings.DisableDocumentBrowserView = $true
```

Allow viewers to print 

```powershell
$list.InformationRightsManagementSettings.AllowPrint = $false
```

Allow viewers to run script and screen reader to function on downloaded documents

```powershell
$list.InformationRightsManagementSettings.AllowScript = $true
```

Allow viewers to write on a copy of the downloaded document

```powershell
$list.InformationRightsManagementSettings.AllowWriteCopy = $true
```

After download, document access rights will expire after these number of days (1-365)

```powershell
$list.InformationRightsManagementSettings.EnableDocumentAccessExpire = $true
$list.InformationRightsManagementSettings.DocumentAccessExpireDays = "nn" #value in days between 1 and 365
```

Users must verify their credentials using this interval (days)

```powershell
$list.InformationRightsManagementSettings.EnableLicenseCacheExpire = $true
$list.InformationRightsManagementSettings.LicenseCacheExpireDays = "nn" #value in days
```

Allow group protection. Default group:

```powershell
$list.InformationRightsManagementSettings.EnableGroupProtection = $true
$list.InformationRightsManagementSettings.GroupName = "string_value" #e.g. group@domain.com
```

As noted, those properties in `InformationRightsManagementSettings` are not available in SharePoint 2010.  However, you can manipulate the properties directly.  Again, get the list object into a variable.

![IRMLibrary2](/assets/images/2013/07/IRMLibrary2.jpg)


Permission policy title:

```powershell
$list.RootFolder.Properties.Add("vti_irm_IrmTitle", "string_value")
$list.RootFolder.Update()
```

Permission policy description:

```powershell
$list.RootFolder.Properties.Add("vti_irm_IrmDescription", "string_value")
$list.RootFolder.Update()
```

Allow users to print documents

```powershell
$list.RootFolder.Properties.Add("vti_irm_IrmPrint", 2) #2 = true, 0 = false
$list.RootFolder.Update()
```

Allow users to access content programmatically

```powershell
$list.RootFolder.Properties.Add("vti_irm_IrmVBA", 2) #2 = true, 0 = false
$list.RootFolder.Update()
```

Users must verify their credentials every:

```powershell
$list.RootFolder.Properties.Add("vti_irm_IrmOffline", 2) #2 = true, 0 = false
$list.RootFolder.Properties.Add("vti_irm_IrmOfflineDays", 45) #value in days
$list.RootFolder.Update()
```

Stop restricting permission to documents in this library on:

```powershell
$list.IrmExpire = $true
$list.RootFolder.Properties.Add("vti_irm_IrmExpireDate", "10/15/2015") #valid date in MM/dd/yyyy format
$list.RootFolder.Update()
```