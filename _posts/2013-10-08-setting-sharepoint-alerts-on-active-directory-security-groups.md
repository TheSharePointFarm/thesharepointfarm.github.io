---
layout: post
title: Setting SharePoint Alerts on Active Directory Security Groups
tags: [sp2013]
---

This post applies to SharePoint 2013 as of the August 2013 Cumulative Update.

If you have ever tried to set an alert on an email-enabled Active Directory Security Group (this will appear in Exchange as a "Mail Universal Security Group"), you may have found that SharePoint indicates that it cannot find an exact match, and just won't resolve the group.

This may be due to a possible bug in SubNew.aspx located in `C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\LAYOUTS\`.  The code for the People Picker is:

```xml
<wssawc:ClientPeoplePicker
Required="true"
ValidationEnabled="true"
id="clientPeoplePicker"
VisibleSuggestions="5"
Rows="3"
Width="100%"
runat="server"
SelectionSet="User,SecGroup"
/>
```

Notice the SelectionSet line.  This should indicate that it accepts both User and Security Groups.  However, the [ClientPeoplePicker](http://msdn.microsoft.com/en-us/library/sharepoint/microsoft.sharepoint.webcontrols.clientpeoplepicker.aspx) class contains no such property!  It does, however, contain another valid property, [PrincipalAccountType](http://msdn.microsoft.com/en-us/library/sharepoint/microsoft.sharepoint.webcontrols.clientpeoplepicker.principalaccounttype.aspx).  While I don't recommend this, as it will most likely be overwritten by new updates, and you must manually make the change, if you do edit SubNew.aspx so the ClientPeoplePicker instead uses the PrincipalAccountType property, mail-enabled Security Groups will now be resolvable within the New Alert dialog.

```xml
<wssawc:ClientPeoplePicker
Required="true"
ValidationEnabled="true"
id="clientPeoplePicker"
VisibleSuggestions="5"
Rows="3"
Width="100%"
runat="server"
PrincipalAccountType="User,SecGroup"
/>
```

One issue to note when this configuration completed is that all Security Groups will be visible through the dialog, however when clicking OK to save the Alert, it will validate that the object has an email address, and if not, throw a user-friendly exception.

Note that SharePoint 2010 does not suffer from this issue.  SharePoint 2010 uses the [PeopleEditor](http://msdn.microsoft.com/en-us/library/sharepoint/microsoft.sharepoint.webcontrols.peopleeditor.aspx) class which does have the [SelectionSet](http://msdn.microsoft.com/en-us/library/sharepoint/microsoft.sharepoint.webcontrols.peopleeditor.selectionset.aspx) property.