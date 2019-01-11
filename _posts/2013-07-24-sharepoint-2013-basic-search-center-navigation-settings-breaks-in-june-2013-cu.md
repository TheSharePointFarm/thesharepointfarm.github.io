---
layout: post
title: SharePoint 2013 Basic Search Center Navigation Settings Breaks in June 2013 CU
tags: [sp2013]
---

In the June 2013 Cumulative Update for SharePoint 2013, if you modify the navigation settings of a Basic Search Center (`/_layouts/15/AreaNavigationSettings.aspx`), the OK button generates the following error:

```csharp
System.NullReferenceException: Object reference not set to an instance of an object.   
 at Microsoft.SharePoint.Publishing.Internal.CodeBehind.AreaNavigationSettingsPage.OKButton_Click(Object sender, EventArgs e)    
 at System.Web.UI.WebControls.Button.RaisePostBackEvent(String eventArgument)    
 at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)

Getting Error Message for Exception System.Web.HttpUnhandledException (0x80004005): Exception of type 'System.Web.HttpUnhandledException' was thrown. ---> System.NullReferenceException: Object reference not set to an instance of an object.    
 at Microsoft.SharePoint.Publishing.Internal.CodeBehind.AreaNavigationSettingsPage.OKButton_Click(Object sender, EventArgs e)    
 at System.Web.UI.WebControls.Button.RaisePostBackEvent(String eventArgument)    
 at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)    
 at System.Web.UI.Page.HandleError(Exception e)    
 at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)    
 at System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)    
 at System.Web.UI.Page.ProcessRequest()    
 at System.Web.UI.Page.ProcessRequest(HttpContext context)    
 at System.Web.HttpApplication.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute()    
 at System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously)
```

To work around this, on the Navigation Settings page, you can create a node under "Structural Navigation: Editing and Sorting".  By placing a node underneath the Global or Current Navigation nodes, the error is bypassed.