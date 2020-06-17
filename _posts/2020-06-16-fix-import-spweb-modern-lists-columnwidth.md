---
layout: post
title: Fix Import-SPWeb Errors on Modern Lists and Libraries
tags: [sp2019]
---

One of the features of the Modern List and Library view in SharePoint Server 2019 is the custom column width -- these column width changes can be saved to the default All Items view or a new view for the List/Library. If a user makes the change and saves the view, if an administrator exports the Web or List/Library using `Export-SPWeb`, when performing an `Import-SPWeb` an error will be generated, similar to the below.

```powershell
Import-SPWeb : The element 'WebPart' in namespace 'urn:deployment-manifest-schema' has invalid child element
'ColumnWidth' in namespace 'urn:deployment-manifest-schema'. List of possible elements expected: 'Script, PagedRowset,
PagedClientCallbackRowset, PagedRecurrenceRowset, ViewFields, ViewData, Query, RowLimit, RowLimitExceeded, Toolbar,
Formats, Aggregations, List, MetaData, View, ViewStyle, ViewBody, ViewEmpty, ViewFooter, ViewHeader, ViewBidiHeader,
GroupByFooter, GroupByHeader, CalendarViewStyles, CalendarSettings, ListFormBody, Xsl, XslLink, JS, JSLink,
ParameterBindings, OpenApplicationExtension, Mobile, MobileItemLimit, Method, WebParts, InlineEdit, Joins,
ProjectedFields, SpotlightInfo, Visualization' in namespace 'urn:deployment-manifest-schema'.
At line:1 char:1
+ Import-SPWeb https://devsp02.cobaltatom.com/sites/teams2 -Path C:\exp ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidData: (Microsoft.Share...CmdletImportWeb:SPCmdletImportWeb) [Import-SPWeb], XmlSc
   hemaValidationException
    + FullyQualifiedErrorId : Microsoft.SharePoint.PowerShell.SPCmdletImportWeb
```

Based on this error, it is clear that the XML schema defined by SharePoint Server does not match the XML schema in the binary file containing the List in our saved export. Obviously, this is a bug, but one that is easy to work around until the SharePoint Product Group can implement a fix.

As always, what we're going to do is unsupported because we are modifying a file in the 16 hive. Make a copy of this file prior to making the modification. We are going to edit the file `%CommonProgramFiles%\microsoft shared\Web Server Extensions\16\TEMPLATE\XML\DeploymentManifest.xsd`. Open this file in Notepad or your favorite XML/text editor.

Find the line that reads `<!-- SPView definition -->`. Within that section is a line `<xs:element name="CustomFormatter" minOccurs="0" maxOccurs="1" />`. We are going to add the new line immediately below the `CustomFormatter` line item.

```xml
<xs:element name="ColumnWidth" minOccurs="0" maxOccurs="1" />
```

The section of the file will look like the below.

```xml
        <xs:element name="Visualization" minOccurs="0" maxOccurs="unbounded" />
        <xs:element name="CustomFormatter" minOccurs="0" maxOccurs="1" />
        <xs:element name="ColumnWidth" minOccurs="0" maxOccurs="1" />
      </xs:choice>
    </xs:sequence>
```

Notice how we are adding the line containing `ColumnWidth`. This will tell SharePoint to accept that element as part of the valid XML schema to import the List/Library view. With your modified file, overwrite `%CommonProgramFiles%\microsoft shared\Web Server Extensions\16\TEMPLATE\XML\DeploymentManifest.xsd`. Re-try your import, it should not require restarting any processes or even PowerShell.

If you still encounter the error after making this change, please let me know on Twitter.
