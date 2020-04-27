---
layout: post
title: SharePoint Designer Custom Forms Deprecated
tags: [spo]
---

Today, Microsoft announced the deprecation of SharePoint Designer custom forms in SharePoint Online, directing individuals to instead use PowerApps for custom form scenarios.

The deprecation was immediate with no notice. There also isn't any information as to why, but I would suspect an immediate removal would be due to a potential security issue, similar to the SharePoint 2010 workflow remote code execution vulnerability which allows an application installed on a SharePoint Server to be run in the context of the IIS Application Pool user -- in other words, it is likely a significant security issue.

If there is a patch or global setting to remove SharePoint custom forms in SharePoint Server within the next two months, that will tell us :-)

As always, I recommend those venturing into SharePoint Online to avoid "legacy" SharePoint functionality, such as SharePoint 2010 or 2013 workflows, InfoPath, classic sites, and so on.

Here is the complete notice.

> **Awareness: SharePoint Designer feature deprecation**
>
> MC210713, Stay Informed, Published date: Apr 25, 2020
>
> We've identified an issue affecting SharePoint Designer
>
> We've identified an issue affecting SharePoint Designer functionality for [creating and editing custom Forms](https://support.microsoft.com/en-us/office/create-a-custom-list-form-using-sharepoint-designer-917d8fdb-ee00-4441-adb3-a94612d1d105?ui=en-us&rs=en-us&ad=us#bm2) within SharePoint Online. After careful examination, we've determined that there is no known fix for this issue and have elected to disable the feature effective as of 3:00 AM UTC on Saturday, April 25, 2020.
>
> Users who have previously leveraged SharePoint Designer to create custom Forms are instead able to utilize [PowerApps](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/customize-list-form) for this purpose.
>
> PowerApps is an easy and powerful tool that allows users operating in the SharePoint Online Modern experience to create and edit custom Forms for SharePoint lists and document libraries right from a browser window. PowerApps does not require traditional coding knowledge or any additional app downloads such as InfoPath.
>
> *Note: SharePoint Online Classic users will need to temporarily switch to the Modern experience to access and utilize PowerApps; though, all custom Forms created in PowerApps are accessible by SharePoint Online Classic experience users. 
>
>[Additional information](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/customize-list-form)