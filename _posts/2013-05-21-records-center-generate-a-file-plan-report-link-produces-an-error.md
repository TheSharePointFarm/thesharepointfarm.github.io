---
layout: post
title: Records Center “Generate a File Plan Report” link produces an error
tags: [sp2010]
---

Within a Records Center under Site Settings -> Manage Records Center, there is a link on the right hand side titled "Generate a file plan report".

![RecordsCenterGenerateFilePlan](/assets/images/2013/05/RecordsCenterGenerateFilePlan.png)

When clicking on this, I was seeing either the standard correlation Id error dialog or a yellow screen of death.  The following error appears in the ULS:

```text
05/21/2013 13:10:36.16 	w3wp.exe (0x0980)                       	0x1234	SharePoint Foundation         	Runtime                       	tkau	Unexpected	System.NullReferenceException: Object reference not set to an instance of an object.    at Microsoft.Office.RecordsManagement.Reporting.ApplicationPages.CustomizeReport.OnLoad(EventArgs e)     at System.Web.UI.Control.LoadRecursive()     at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint)	d5512f59-3bcc-4a67-be69-62530115a064
05/21/2013 13:10:36.23 	w3wp.exe (0x0980)                       	0x1234	SharePoint Foundation         	Monitoring                    	b4ly	Medium  	Leaving Monitored Scope (Request (GET:http://webapp1:80/sites/rc/_layouts/customizereport.aspx?Category=FilePlan&backtype=list&List=)). Execution Time=145.0229
```

You'll also notice that in the URL as well as the above ULS log, that the List parameter is blank.  There are two Javascript functions that get executed when clicking on the link, the first function calls the second function to retrieve the List Name based on a resource:

```csharp
<...>
	  function getFilePlanReportLink()
	  {ULSs5K:;
		  var listGuid = "<%= getRecordListID() %>";
		  var serverRelativeUrl = "<%= SPContext.Current.Web.ServerRelativeUrl %>";
		  if (serverRelativeUrl.length >= 1 && serverRelativeUrl.charAt(serverRelativeUrl.length - 1) != "/")
		  {
			  serverRelativeUrl += "/";
		  }
		  if(listGuid != "" || listGuid != null)
		  {
			  return STSNavigate(serverRelativeUrl + "_layouts/customizereport.aspx?Category=FilePlan&backtype=list&List=" + listGuid);
		  }
		  return STSNavigate(serverRelativeUrl + "_layouts/create.aspx");
</...>

<...>
		protected string getRecordListID()
		{
			string listGuid = string.Empty;
			SPWeb web = SPContext.Current.Web;
			string recordsUrlResourceString = "$Resources:dlccore,RecordsLib_ListFolder;";
			string recordsListUrlName = SPUtility.GetLocalizedString(recordsUrlResourceString, null , web.Language);
			try
			{
				SPList list = web.GetList(SPUrlUtility.CombineUrl(web.ServerRelativeUrl, recordsListUrlName));
				if(list != null)
					listGuid = list.ID.ToString();
			}
			catch { }
			return listGuid;
		}
</...>
```

The intent is to look for a Record Library with the internal name of "Records" and pass the library's Guid in this function.

To fix this error, on the Records Center site, simply create a new Record Library name "Records".

![RecordLibrary](/assets/images/2013/05/RecordLibrary.png)

Once the library is created, the Generate a File Plan Report link will work on the Manage Records Center page.