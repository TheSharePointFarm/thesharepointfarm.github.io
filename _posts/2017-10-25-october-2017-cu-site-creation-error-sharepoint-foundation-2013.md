---
layout: post
title: October 2017 CU Site Creation Error for SharePoint Foundation 2013
tags: [sp2013]
---

EDIT 11/14/2017: This is resolved in the November 2017 CU, specifically [KB4011256](https://support.microsoft.com/help/4011256).

When attempting to create a Site Collection via Central Admin in SharePoint Foundation 2013 using the October 2017 Cumulative Update, you will encounter an error when attempting to load the page (/_admin/createsite.aspx).

![OCT2017CUSiteCreationError](/assets/images/2017/10/OCT2017CUSiteCreationError.png)

ULS will show the following.

```
Application error when access /_admin/createsite.aspx, Error=Common Language Runtime detected an invalid program.   at Microsoft.Office.Server.UserProfiles.ProfileLoader.GetProfileLoader()     at Microsoft.SharePoint.WebControls.HybridOneDriveDefaultToCloudCommon.GetHybridMySiteUrl()     at Microsoft.SharePoint.WebControls.TemplatePicker.RegisterWebTemplateClientScript()     at Microsoft.SharePoint.WebControls.TemplatePicker.OnPreRender(EventArgs e)     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
System.InvalidProgramException: Common Language Runtime detected an invalid program.    at Microsoft.Office.Server.UserProfiles.ProfileLoader.GetProfileLoader()     at Microsoft.SharePoint.WebControls.HybridOneDriveDefaultToCloudCommon.GetHybridMySiteUrl()     at Microsoft.SharePoint.WebControls.TemplatePicker.RegisterWebTemplateClientScript()     at Microsoft.SharePoint.WebControls.TemplatePicker.OnPreRender(EventArgs e)     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
Getting Error Message for Exception System.Web.HttpUnhandledException (0x80004005): Exception of type 'System.Web.HttpUnhandledException' was thrown. ---> System.InvalidProgramException: Common Language Runtime detected an invalid program.     at Microsoft.Office.Server.UserProfiles.ProfileLoader.GetProfileLoader()     at Microsoft.SharePoint.WebControls.HybridOneDriveDefaultToCloudCommon.GetHybridMySiteUrl()     at Microsoft.SharePoint.WebControls.TemplatePicker.RegisterWebTemplateClientScript()     at Microsoft.SharePoint.WebControls.TemplatePicker.OnPreRender(EventArgs e)     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Control.PreRenderRecursiveInternal()     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.HandleError(Exception e)     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.ProcessRequest()     at System.Web.UI.Page.ProcessRequest(HttpContext context)     at System.Web.HttpApplication.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute()     at System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously)
```

In addition, the Configure hybrid OneDrive and Sites features page (/_admin/cloudconfiguration.aspx) under Office 365 in Central Admin will also generate an error.

![OCT2017CUO365Error](/assets/images/2017/10/OCT2017CUO365Error.png)

And the ULS message associated with Hybrid configuration:

```
Application error when access /_admin/cloudconfiguration.aspx, Error=Object reference not set to an instance of an object.   at Microsoft.SharePoint.WebControls.SPPinnedSiteTile.OnInit(EventArgs e)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
System.NullReferenceException: Object reference not set to an instance of an object.    at Microsoft.SharePoint.WebControls.SPPinnedSiteTile.OnInit(EventArgs e)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)
Getting Error Message for Exception System.Web.HttpUnhandledException (0x80004005): Exception of type 'System.Web.HttpUnhandledException' was thrown. ---> System.NullReferenceException: Object reference not set to an instance of an object.     at Microsoft.SharePoint.WebControls.SPPinnedSiteTile.OnInit(EventArgs e)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Control.InitRecursive(Control namingContainer)     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.HandleError(Exception e)     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)     at System.Web.UI.Page.ProcessRequest()     at System.Web.UI.Page.ProcessRequest(HttpContext context)     at System.Web.HttpApplication.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute()     at System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously)
```

So why are we getting this error? Well, when we run the `Microsoft.Office.Server.UserProfiles.dll` through [PEVerify](https://docs.microsoft.com/en-us/dotnet/framework/tools/peverify-exe-peverify-tool), we end up with a plethora of errors like this:

```
[IL]: Error: [C:\windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.Office.Server.UserProfiles\v4.0_15.0.0.0__71e9bce111e9429c\Microsoft.Office.Server.UserProfiles.dll : Microsoft.Office.Server.UserProfiles.ProfileLoader::GetUserProfileManager][offset 0x00000000] Return value missing on the stack.
[IL]: Error: [C:\windows\Microsoft.NET\assembly\GAC_MSIL\Microsoft.Office.Server.UserProfiles\v4.0_15.0.0.0__71e9bce111e9429c\Microsoft.Office.Server.UserProfiles.dll : Microsoft.Office.Server.UserProfiles.ProfileLoader::GetUserProfile][offset 0x00000000] Return value missing on the stack.
```

Essentially, this means there are no return values possible from the method. And indeed, when we reflect the binary we can see why in the method `Microsoft.Office.Server.UserProfiles.ProfileLoader.GetUserProfile()`. Here is the September 2017 CU binary:

```csharp
[ClientCallable]
public UserProfile GetUserProfile() => 
    this.GetUserProfile(true);
```

Compare that to the broken October 2017 CU binary:

```csharp
public UserProfile GetUserProfile()
{
}
```

With the October 2017 CU binary, it appears most of the UserProfile (and other methods) have been blanked out in a similar fashion.

For Site Creation, it appears that it is still possible to create new sites via `New-SPSite`. It does not appear that it is possible to manage the Office 365 Hybrid OneDrive for Business and Sites configuration. This particular issue will have to wait for a fix from Microsoft.

A big thank you to [Todd Klindt](http://www.toddklindt.com/default.aspx) for raising this issue!