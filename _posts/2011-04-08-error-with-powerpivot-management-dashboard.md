---
layout: post
title: Error with PowerPivot Management Dashboard
tags: [sp2010, powerpivot]
---

7/22/2011: This issue is resolved in SQL Server 2008 R2 SP1 (applied to the PowerPivot instance on SharePoint).

I have an ongoing issue where the PowerPivot Management Dashboard in the Silverlight control just displays a Red X, and the Activity List is blank, no matter how many PowerPivot-enabled workbooks you interact with.  When viewing the Management Dashboard, the following error in the ULS log is present:

```text
03/24/2011 14:18:57.16 w3wp.exe (0x0DFC) 0x0598 SSAS Mid-Tier Service Request Processing 106 High Error loading history for workbook history bubble chart eb9fce20-5fa7-4395-b7c5-03a79648e8cf

03/24/2011 14:18:57.16 w3wp.exe (0x0DFC) 0x0598 SSAS Mid-Tier Service Request Processing 99 High EXCEPTION: System.NullReferenceException: Object reference not set to an instance of an object. at Microsoft.AnalysisServices.AdomdClient.AdomdDataReader.GetString(Int32 ordinal) at Microsoft.AnalysisServices.SharePoint.Integration.WorkbookHistoryDataProvider.GetHistory(WorkbookHistoryDataSet& historyDataSet) at Microsoft.AnalysisServices.SharePoint.Integration.WebServices.PowerPivotOperationsServiceImpl.GetWorkbookHistory(WorkbookHistoryDataSet& history) eb9fce20-5fa7-4395-b7c5-03a79648e8cf
```

In speaking with MS PSS, they let me know that this is an issue if you installed any SharePoint Cumulative Updates prior to installing PowerPivot.  If you install SharePoint RTM, then PowerPivot, then any updates, this error will not appear.  PSS is working with development and product managers to resolve the issue (namely being certain tables in the Usage database do not contain primary keys).