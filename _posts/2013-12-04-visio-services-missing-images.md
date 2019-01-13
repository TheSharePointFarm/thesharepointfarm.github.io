---
layout: post
title: Visio Services Missing Images
tags: [sp2013]
---

In SharePoint 2013, Visio Services may throw errors in the Application Event Log. This event appears to happen when you open the Web Part Options for the Visio Web Access web part:

```text
Event code: 3012 
Event message: An error occurred processing a web or script resource request. The requested resource 'fMicrosoft.Office.Visio.Server,15.0.0.0,,71e9bce111e9429c|Microsoft.Office.Visio.Server.WebControls15.Images.MessageBoxImage.png' does not exist or there was a problem loading it. 
Event time: 12/4/2013 7:18:11 AM 
Event time (UTC): 12/4/2013 3:18:11 PM 
Event ID: 0ec1bca456fb4f95ae860c13336e68b7 
Event sequence: 38 
Event occurrence: 1 
Event detail code: 0 

Application information: 
    Application domain: /LM/W3SVC/471498445/ROOT-1-130306424238033738 
    Trust level: Full 
    Application Virtual Path: / 
    Application Path: C:\inetpub\wwwroot\wss\VirtualDirectories\spwebapp180\ 
    Machine name: SP2013
```

This appears after loading a Visio web document in the web part:

```text
Event code: 3012 
Event message: An error occurred processing a web or script resource request. The requested resource 'fMicrosoft.Office.Visio.Server,15.0.0.0,,71e9bce111e9429c|Microsoft.Office.Visio.Server.WebControls15.Images.CloseButton.png' does not exist or there was a problem loading it. 
Event time: 12/4/2013 7:23:19 AM 
Event time (UTC): 12/4/2013 3:23:19 PM 
Event ID: f27090bc89eb47ebb415910753b96172 
Event sequence: 125 
Event occurrence: 11 
Event detail code: 0 

Application information: 
    Application domain: /LM/W3SVC/471498445/ROOT-1-130306424238033738 
    Trust level: Full 
    Application Virtual Path: / 
    Application Path: C:\inetpub\wwwroot\wss\VirtualDirectories\spwebapp180\ 
    Machine name: SP2013
```

Here we can see with the Visio Services DLL the use of these two embedded images:

```csharp
internal sealed class ProgressControl : Panel
{
    // Fields
    internal const string LoadingImage = "Microsoft.Office.Visio.Server.WebControls15.Images.Loading.gif";
    internal const string StoppedImage = "Microsoft.Office.Visio.Server.WebControls15.Images.CloseButton.png";
```

And then again, the other embedded resource contained within `VwaMessageBoxImage`:

```csharp
private HtmlControl CreateImageControl()
{
    string webResourceUrl = null;
    HtmlGenericControl control2;
    if ((this.Page != null) && (this.Page.ClientScript != null))
    {
        webResourceUrl = this.Page.ClientScript.GetWebResourceUrl(base.GetType(), "Microsoft.Office.Visio.Server.WebControls15.Images.AllImages.png");
    }
    string str2 = "VwaMessageBoxImage";
    switch (this.Icon)
    {
        case MessageBoxIcon.Warning:
            str2 = str2 + " VwaMessageBoxWarn";
            break;

        case MessageBoxIcon.Info:
            str2 = str2 + " VwaMessageBoxInfo";
            break;

        default:
            str2 = str2 + " VwaMessageBoxError";
            break;
    }
```

Here we can see all of the available resources in the `Microsoft.Office.Visio.Server.dll`, where we're missing the two PNGs referenced in code:

![VisioServicesResources](/assets/images/2013/12/VisioServicesResources.png)

This issue appears to have existed since SharePoint 2013 RTM.