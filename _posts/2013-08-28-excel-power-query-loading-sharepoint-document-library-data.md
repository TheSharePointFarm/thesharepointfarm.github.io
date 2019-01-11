---
layout: post
title: Excel Power Query – Loading SharePoint Document Library Data
tags: [sp2010,sp2013]
---

With [Excel Power Query](http://office.microsoft.com/en-us/excel/download-microsoft-power-query-for-excel-FX104018616.aspx), you can query SharePoint List data quickly and easily, as well as a variety of other types of data sources.

![PowerQueryListData](/assets/images/2013/08/PowerQueryListData.png)

However, when we do this, only SharePoint Lists appear!  This is not helpful if, for instance, you have property promotion from an InfoPath form into a Document Library that you want to query.

![PowerQueryLists](/assets/images/2013/08/PowerQueryLists.png)

To work around this, we simply need to show the Formula Bar, which can be found in the Query Editor window under Settings.

![PowerQueryShowBar](/assets/images/2013/08/PowerQueryShowBar.png)

Next, edit the query to show all Contents of the SharePoint site.  Once we change `=SharePoint.Tables("http://url")` to `=SharePoint.Contents("http://url")`, click the Refresh button and all Document Libraries will appear!

![PowerQueryContents](/assets/images/2013/08/PowerQueryContents.png)

The other valid value for the SharePoint query type is `=SharePoint.Files("http://url")`.  With this type, all uploaded files on the SharePoint site will be displayed.

![PowerQueryFiles](/assets/images/2013/08/PowerQueryFiles.png)

There is a 4th query type, SharePoint.Count, which is ignored in the Power Query assemblies.