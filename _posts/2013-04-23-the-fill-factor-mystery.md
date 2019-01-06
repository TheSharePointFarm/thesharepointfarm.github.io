---
layout: post
title: 
tags: [sp2007,sp2010,sp2013]
---

The Microsoft recommendation for SharePoint database fill factor has changed over the years.  In SharePoint 2007, the [recommendation](http://technet.microsoft.com/en-us/library/cc262731(v=office.12).aspx) was 70.  However, a new stored procedure was introduced in the content and configuration database as of SharePoint 2007 Service Pack 2.  This stored procedure, named `proc_DefragmentIndicies`, set the fill factor to 80 when running SQL Server 2005.

SharePoint 2010 continued to leverage the same stored procedure by way of the Health Analyzer rule "[Databases used by SharePoint have fragmented indices](http://technet.microsoft.com/en-us/library/ff805067(v=office.14).aspx)".  This rule would run `proc_DefragmentIndicies`, and based on certain conditions, would reindex or rebuild the indices as well as set the fill factor to 80.

Now we introduce SharePoint 2013.  The same Health Analyzer rule exists, however `proc_DefragmentIndicies` has changed, and thanks to Vlad Catrinescu over at [SharePoint-Community.Net](http://sharepoint-community.net/) for pointing that out.  It no longer sets the fill factor!  In addition, store.sql (the SQL script used to create content databases) sets the fill factor to 95 for most indexes, and 100 for the remainder of the indexes; in SharePoint 2010, the fill factor was not set by default from this script.  A fill factor of 80, for content databases, is nowhere to be found.

Microsoft also introduced two new classes in the `Microsoft.SharePoint.dll` and `Microsoft.Office.Server.dll`.  These classes contain references to add indices, including one interesting property:

```csharp
public string FillFactorText
{
    get
    {
        switch (this.FillFactorSetting)
        {
            case FillFactor.Default:
                return "95";

            case FillFactor.NotFull:
                return "95";

            case FillFactor.Full:
                return "100";

            case FillFactor.NotFullLargeRow:
                return "80";
        }
        throw new InvalidOperationException("The property FillFactor has an llegal value or there is a missing case in FillFactorText.");
    }
}
```

It appears, and I have no confirmation on this, that a fill factor of 80 is, by default, no longer appropriate for SharePoint 2013.