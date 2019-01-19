---
layout: post
title: SharePoint 2016 Compliance Policy Internals
tags: [sp2016]
---

[This post is based on the SharePoint 2016 Preview — Functionality is not guaranteed to be supported, configured identically, or exist in RTM]

SharePoint 2016 brings a SharePoint Online feature called the Compliance Center, or a subset of the features, specifically the Document Deletion Policies. This allows for a centralized location to set document deletion based on a few parameters, such as when the document was created or last modified, and what the target Site Template is (e.g. you can target sites created with the Team Site template).

This post will cover the internals of the Compliance Policy feature, instead of the user-facing feature itself.

A site created from the Compliance Policy Processing template (POLICYCTR#0) may only be created by a Farm Administrator. This Site Template is not available through Self-Service Site Creation. When a Compliance Policy Processing site is created, the Web Application has a property of "PolicyCenterUrl" set to the Site.Url value. SharePoint will not allow for a second Compliance Center site to be created. It is possible to create a second one if the PolicyCenterUrl property is deleted.

At the heart of the Compliance Policy feature are five timer jobs:

Compliance Dar Task House Keeping
Compliance High Priority Policy Processing
Compliance Dar Processing
Compliance Policy Processing
Unified Policy OnPrem Sync Job

In the same order, here are the names and schedules of each job:

Name | Schedule
--- | ---
DarTaskHouseKeeping | daily between 02:00:00 and 02:30:00
HiPriPolicyProcessing | every 15 minutes between 0 and 59
DarProcessing | every 10 minutes between 0 and 59
PolicyProcessing | daily between 03:30:00 and 04:30:00
UnifiedPolicyOnPremSyncTimerJob | hourly between 0 and 59

For ULS logging, all of these log under the "Information Policy Management" category under the "Document Management Server" Product (or Set-SPLogLevel -TraceSeverity _traceSevLevel_ -Identity "Information Management. The timer jobs processing sites will ignore sites with the following Site Templates:

"POLICYCTR", "EDISC", "TBH", "TENANTADMIN", "OFFILE", "OFFILE", "CENTRALADMIN", "MYSITE", "SPSMSITEHOST"

HiPriPolicyProcessing has a maximum run time of 15 minutes, and runs against each Content Database in a Web Application. It executes the stored procedure, proc_GetWorkItemCountForType with an input value of "29A8C505-1DAA-44E9-8974-B7BA3996794B". This sproc looks for the GUID in dbo.ScheduledWorkItems.[Type] column. If no entries are found, the timer job exits. Otherwise, it executes the assigned policy when that policy is marked High, or a Category of 2.

DarTaskHouseKeeping will cleanup completed or failed tasks based on the expiration date of the task * -1. It then moves onto setting tasks to have a reoccurrence of None, Recurrent, or enqueue the task for execution.

DarProcessing has a maximum runtime of 10 minutes and runs against each Content Database in a Web Application. It enqueues and executes Dar tasks. Dar tasks are held within the List, Lists/DarTasks. It executes the assigned policy when that policy is marked Urgent, or a Category of 3.

PolicyProcessing is very similar to HiPriPolicyProcessing, except it has a runtime of 1440 minutes (24 hours) and only runs Dar Tasks with a priority of Medium, or a Category of 1.

UnifiedPolicyOnPremSyncTimerJob does not appear to do anything as during the initial execution, it validates if the SPWeb has a Property of "MsoTenantId". If not, it returns. MsoTenantId appears to be for hybrid DLP scenarios (and possibly SPO only).

Those are the basics! Hopefully the SharePoint PG will add additional DLP options in the future to cover other scenarios.