---
layout: post
title: Broken PowerPoint Service Application
tags: [sp2010]
---

When users would visit a PowerPoint file using the PowerPoint Viewer in SharePoint 2010, they would receive a generic error and a follow up message asking if they would like to open the file in PowerPoint (client).

In Central Administration, when attempting to manage the PowerPoint Service Application, an error of "The given key was not found in the dictionary" would be thrown.  Looking at the ULS logs, the error was:

```csharp
System.Collections.Generic.KeyNotFoundException: The given key was not present in the dictionary.
at System.ThrowHelper.ThrowKeyNotFoundException() 
at System.Collections.Generic.Dictionary`2.get_Item(TKey key) 
at Microsoft.Office.Server.PowerPoint.SharePoint.Administration.PowerPointWebServiceApplication.get_EnableViewingOpenDocumentFormats() 
at Microsoft.Office.Server.Powerpoint.SharePoint.Web.UI.PowerPointServiceManagePage.OnLoad(EventArgs e) 
at System.Web.UI.Control.LoadRecursive() 
at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
```

Using PowerShell to get the PowerPoint Service Application, I noticed that the `EnableViewingOpenXmlFormats` property was blank; it did not return true nor false.

Unfortunately it looks like the properties held within the PowerPoint SA were wiped out at some point.  The only way to recreate the properties is to recreate the Service Application.  Luckily for the PowerPoint Service Application, there is no database.  Simply deleting the Service Application, then creating a new one is all that is required.  There is no interruption to other services during this operation.

The PowerPoint properties are held within the Configuration database under the object `Microsoft.Office.Server.PowerPoint.SharePoint.Administration.PowerPointWebServiceApplication, Microsoft.Office.Server.Powerpoint.Web.MOSSHost, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c`.