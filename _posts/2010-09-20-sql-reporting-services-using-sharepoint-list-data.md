---
layout: post
title: SQL Reporting Services – Using SharePoint List Data
tags: [sp2007, ssrs]
---

SQL Reporting Services allows you to leverage SharePoint data via XML query.  The process is fairly simple, although it is hard to find good details on how to do it.  The method below assumes some knowledge with Microsoft Visual Studio 2005 with Business Intelligence (or SQL Server Business Intelligence Development Studio).  You can get this by install the SQL 2005 tools.  The below should also work with the SQL 2008 development tools.

To start out with, you need to create a data source (Shared or not).  You will be create an XML data source with a connection string like:

`http://sitename/_vti_bin/lists.asmx`

You can use Windows Credentials or No Credentials.

Create a new dataset (on the Data tab).  Name it something logical and choose the data source you just created.  The command type is Text.

For the query itself, you will want [Stramit SharePoint Caml Viewer](https://archive.codeplex.com/?p=spcamlviewer).  This will make the process of getting the List GUID, View GUID, and Field names much, much easier.

After getting the above information from Stramit Caml Viewer, create a text query like this:

{% highlight xml %}
<Query>
<Method Namespace="http://schemas.microsoft.com/sharepoint/soap/" Name="GetListItems">
<Parameters>
<Parameter Name="listName"><DefaultValue>{GUID_OF_LIST}</DefaultValue></Parameter>
<Parameter Name="viewName"><DefaultValue>{GUID_OF_VIEW}</DefaultValue></Parameter>
<Parameter Name="rowLimit"><DefaultValue>9999999</DefaultValue></Parameter>
</Parameters>
</Method>
<SoapAction>http://schemas.microsoft.com/sharepoint/soap/GetListItems</SoapAction>
<ElementPath IgnoreNamespaces="true">GetListItemsResponse/GetListItemsResult/listitems/data/row 
{@ows_ID, @ows_Title}</ElementPath>
</Query>
{% endhighlight %}

The above query will pull the ID and Title field from the (un-) specified List and View.  You’ll note that the row limit is set very high, beyond what you’d likely find in a list.  If you set the row limit too low, your query will not gather all items from the List.  However, one thing you must also do is go to the View you’re using and set the Item Limit to the maximum allowed (2147483647, or a smaller number that will work for you).  If the Item Limit is too low, again, you will not see all results in your query.

Field names must begin with “@ows_FieldName” and separated by a comma.  Once you've built your query, hit the execute button (!) to see your query results.  You can then proceed to build your report!