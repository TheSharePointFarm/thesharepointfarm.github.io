---
layout: post
title: Updating an InfoPath-enabled List in Datasheet View causes Internet Explorer to Crash
tags: [sp2010]
---

Update: This is fixed in the [Office 2010 hotfix package (Stslist-x-none-msp) for October 2012](http://support.microsoft.com/kb/2760390) and the [Access 2013 hotfix package (Stslist-x-none-msp) for February 2013](http://support.microsoft.com/kb/2760341).

This bug has been validated on both Internet Explorer 9 and 10 using Office 2010 and Office 2013.  To replicate the issue, create a custom List that has the default Title field and a People or Group column that allows multiple selections.  Next, create a custom view based off of Datasheet View that exposes the Title and People or Group fields. Edit the List in InfoPath (just using the Customize Form option and publishing it back to SharePoint is good enough).

Finally, switch to the Datasheet View of the list.  Enter information into the Title field, but _skip_ the People or Group field, tabbing to the next line.  At this point, Internet Explorer will crash.

image[3]

I've opened a Connect bug.