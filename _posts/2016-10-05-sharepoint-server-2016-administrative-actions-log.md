---
layout: post
title: SharePoint Server 2016 Administrative Actions Log
tags: [sp2016]
---

One of the top feature requests from day one on [sharepoint.uservoice.com](https://sharepoint.uservoice.com/forums/330318-sharepoint-administration) is [Administration Auditing](https://sharepoint.uservoice.com/forums/330318-sharepoint-administration/suggestions/7074825-sharepoint-management-shell-central-administration). The idea behind administrator auditing was that actions administrators took against the farm, creating a new Web Application, deleting a site via Remove-SPSite, changing property bag values, and so on, would be logged by SharePoint. This would allow a team of SharePoint Administrators to know who is doing exactly what and when. I've managed quite a few farms where there multiple SharePoint Administrators, from "tier 2" to SharePoint Architects. There are quite a few instances where it would have been helpful to know who did what, exactly. The SharePoint Server 2016 Administrative Actions Log, formerly announced for release in the [November PU with Feature Pack 1](https://blogs.office.com/2016/09/26/announcing-feature-pack-1-for-sharepoint-server-2016-cloud-born-and-future-proof/), is a start in the right direction for my feature request. We'll see a couple examples of how you might use this.

To start with, to use this functionality you will need two things: The November PU and a Usage Service Application. Administrator Actions logging stores the data in the Usage database using the 31 day retention period. Like other data written to the Usage database, the Administrator Actions logging has 32 tables; dbo.AdministrativeActions_Partition0 through dbo.AdministrativeActions_Partition31. These tables can be queried directly or query the view AdministrativeActions directly, which provides a condensed view of all tables.

Let's move onto SharePoint. When an administrator action event occurs, it is first logged to the file system of that particular SharePoint server. This will be at the location where your Usage log files are under the AdministrativeActions folder. By default, this is located under C:\Program Files\Common Files\microsoft shared\Web Server Extensions\16\LOGS\. When actions take place, they're logged to a .usage log. For this example, SP05-20160926-1542-20160926-15420.usage is the name of the log. It is actually just a plain text file with human readable information. But this isn't the way we'll interact with it.

The SharePoint Management Shell offers a way to query and sort Administrative Actions via the `Merge-SPUsageLog` cmdlet. This is a generic cmdlet for querying Usage logging information, but is fairly easy to use. To query the Administrative Actions log, simply use:

```powershell
Merge-SPUsageLog -Identity "Administrative Actions"
```

From there, we can also pass in the `-StartTime`, `-EndTime`, or `-Servers` parameters. If you do not pass in the -StartTime parameter, the cmdlet will query data from the previous hour only. So let's look at an example of creating a new Web Application. Now, this process creates a lot of entries in the Administrative Actions log, so I won't show all of them, but this will give you a general idea of what the data looks like.

```powershell
ParentType         : Microsoft.SharePoint.Administration.SPAdministratorActionProvider
ActionName         : Administration.WebApplication.New
TargetName         : SharePoint - intrnaet.nauplius.local80
Details            : {"Service":"","ApplicationPool":"SharePoint"}
FarmId             : 1a4919ad-6960-41c0-a543-6c77ce5f4443
User               : NAUPLIUS\trevor
SiteSubscriptionId : 00000000-0000-0000-0000-000000000000
Timestamp          : 9/26/2016 8:42:33 AM
TimestampUtc       : 9/26/2016 3:42:33 PM
ParentTypeGuid     : 0e82641e-4c17-416d-ad88-6c1a37c0f567
CorrelationId      : 6395a79d-74f6-3044-d6b8-241dd37f5d09
MachineName        : SP05
ParentInstanceName :

ParentType         : Microsoft.SharePoint.Administration.SPAdministratorActionProvider
ActionName         : Administration.WebApplication.UserPolicy.New
TargetName         : NAUPLIUS\s-sp2016service
Details            : {"DisplayName":"Search Crawling Account"}
FarmId             : 1a4919ad-6960-41c0-a543-6c77ce5f4443
User               : NAUPLIUS\trevor
SiteSubscriptionId : 00000000-0000-0000-0000-000000000000
Timestamp          : 9/26/2016 8:45:28 AM
TimestampUtc       : 9/26/2016 3:45:28 PM
ParentTypeGuid     : 0e82641e-4c17-416d-ad88-6c1a37c0f567
CorrelationId      : 6395a79d-74f6-3044-d6b8-241dd37f5d09
MachineName        : SP05
ParentInstanceName :
```

Based on this, we can see the User who performed the action (me!). There are a lot of filters we could use here, such as `Merge-SPUsageLog -Identity "Administrative Actions" | ?{$_.User -match "trevor"} or Merge-SPUsageLog -Identity "Administrative Actions" -StartTime 9/20/2016 -EndTime 9/27/2016 | ?{$_.ActionName -match "UserPolicy"}`. Pretty simple to use and get relevant data out of the logging.

What does it cover? This is a very high level list of what is covered as not all actions that you may think flow under these high level actions are logged, but a great portion of them are.

```
Administration.Security
Administration.Farm.BackupRestore
Administration.Farm.Server
Administration.Farm.ConfigurationDatabase
Administration.SiteCollection
Administration.Quota
Administration.Feature
Administration.WebApplication
Administration.ContentDatabase
Administration.ServiceApplication
Administration.FormTemplate
```

And the verbs associated with one or more of the above actions.

```
Add Clear Close Copy Enter Exit Find Format Get Hide Join Lock Move New Open Optimize
Pop Push Redo Remove Rename Reset Resize Search Select Set Show Skip Split Step Switch Undo
Unlock Watch Connect Disconnect Read Receive Send Write Backup Checkpoint Compare Compress Convert ConvertFrom ConvertTo Dismount
Edit Expand Export Group Import Initialize Limit Merge Mount Out Publish Restore Save Sync Unpublish Update
Debug Measure Ping Repair Resolve Test Trace Approve Assert Complete Confirm Deny Disable Enable Install Invoke
Register Request Restart Resume Start Stop Submit Suspend Uninstall Unregister Wait Block Grant Protect Revoke Unblock
Unprotect Use
```

In the above example I provided, we can see `Administration.WebApplication` is the action with `New` as the verb, giving us `Administration.WebApplication.New` as the Administrative Action.

Hopefully this post helps with understanding what the Administrative Actions Logging is as well as how to use it. While Administrative Actions Logging doesn't exactly match up with my original request, I'm sure the SharePoint Product Group will get there!