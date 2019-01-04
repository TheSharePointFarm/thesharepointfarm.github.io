---
layout: post
title: SharePoint Multiple Select Parameter Dropdown with SQL Server Reporting Services
tags: [sp2010,ssrs]
---

When leveraging a SharePoint List as an SSRS data source, one major problem is formatting of list items as they appear in reports.  This is fairly easy to [solve in code](http://thesharepointfarm.com/2010/09/formatting-sharepoint-data-in-sql.html). Another issue is leveraging Multiple Lines of Text columns as Report parameters.  This is actually fairly tricky and has limitations.

In order to leverage a Multiple Lines of Text column as a Report parameter, the first thing to keep in mind is that an Embedded Data Set is a requirement.  A Shared Data Set cannot be used with this solution.

First, add the following code to the Report.  As more values are added to the Multiple Lines of Text (e.g. more choices), this code will need to be manually adjusted.  Replace “MyField” with the internal field name.  The below example is valid for 4 choices in the Report parameter.

```vb
Public Function Query(ByVal s()) As String 
	Dim strQuery
	Dim intLength
	Dim strBlank(0) As String
	strBlank(0) = ""
	Dim strVal0,strVal1,strVal2,strVal3
	intLength = s.length()

	If intLength = 0 Then
		strVal0 = s(0)
	Else
		strVal0 = ""
	End If

	If intLength = 1 Then
		strVal1 = s(1)
	Else
		strVal1 = ""
	End If

	If intLength = 2 Then
		strVal2 = s(2)
	Else
		strVal2 = ""
	End If

	If intLength = 3 Then
		strVal3 = s(3)
	Else
		strVal3 = ""
	End If

	strQuery = " <Query><Where><Or><Or><Or><Eq><FieldRef Name=""MyField"" /> "
	strQuery = strQuery + "<Value Type=""Lookup"">"+strVal0+"</Value></Eq><Eq><FieldRef Name=""MyField"">/>"
	strQuery = strQuery + "<Value Type=""Lookup"">"+strVal1
	strQuery = strQuery + "</Value></Eq></Or><Eq><FieldRef Name=""MyField""/><Value Type=""Lookup"">"+strVal2+"</Value>"
	strQuery = strQuery + "</Eq></Or><Eq><FieldRef Name=""MyField""/><Value Type=""Lookup"">"+strVal3+"</Value></Eq></Or>"
	strQuery = strQuery + "</Where></Query></RSSSharePointList>"

	Return strQuery

End Function
```

Next, you’ll need to modify your Data Set Expression, removing a portion of the CAML query until it looks similar to the following:

```vb
="<RSSharePointList xmlns:xsi=""http://www.w3.org/2001/XMLSchema-instance"" xmlns:xsd=""http://www.w3.org/2001/XMLSchema"">"
   +"<ListName>MyList</ListName>"
	+"<ViewFields>"
	+"<FieldRef Name=""Title"" />"
	+"<FieldRef Name=""MyField"" />"
	+"</ViewFields>" + Code.Query(Parameters!MyField.Value)
```

This example should help you get started to using Multiple Lines of Text as a value for Report parameters.  This will allow you to leverage this field type in as a Multiple Select Report Parameter.