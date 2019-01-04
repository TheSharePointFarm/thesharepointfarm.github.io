---
layout: post
title: Users with Commas in their DisplayName Cannot be Found by Gantt Charts and Datasheet Views
tags: [sp2010,sp2013]
---

This issue first appears with the SharePoint 2010 Project Task List (in the Gantt(V4) view).  When adding a user, either by using the People Picker or typing the Display Name in to the Assigned To field, the Gantt chart will report that the 'user does not exist or is not unique' upon synchronization of the row.

![GanttUserCannotBeFound](/assets/images/2013/03/GanttUserCannotBeFound.png)

This is expanded in SharePoint 2013 where all datasheet views that used the ActiveX control in SharePoint 2010 are now AJAX/JavaScript-enabled.

Microsoft has indicated that they're aware of this bug but it is a "won't fix" issue as it has a significant impact to the code base.  Their workarounds are as follows:

1) Change the display name for users in Active Directory from "LastName, FirstName" to "FirstName LastName" to eliminate the comma.

2) Use the user's alias instead of DisplayName when entering it into the row.  The Gantt chart will accept the value in this case, however this precludes use of the People Picker.

3) Where applicable, use the "New Item" option in the ribbon.