---
layout: post
title: SharePoint Event Log Error 7076 - “Not enough storage is available to process this command”
tags: [sp2007]
---

Update: Microsoft has posted KB968484 on this issue, but with no real resolution.

Update 2: Microsoft has posted the April 2009 cumulative update (KB968857) for SharePoint (this fix is after SP2) which resolves this issue.

When you receive an Alert from SharePoint in Outlook 2007 (this post is as of SP1), you have the option of connecting to the List or Document Library in the Ribbon:

![image2](/assets/images/2010/09/image2.png)

This feature is broken, according to Microsoft Product Support Services.  While the List or Library will add to your SharePoint PST, and show up correctly, on a Send/Receive, you’ll get the following error message:

![image5](/assets/images/2010/09/image5.png)

Error `0x800401F3` means “Invalid Class String”, or `CO_E_CLASSSTRING`.

There is no fix for this error, but Microsoft did provide a work-around for me.  First, delete the connection to the List or Library in Outlook by going to Tools –> Account Settings –> SharePoint Lists.  Highlight the List or Library and click Remove.  Next, go to the List or Library in SharePoint, then go to Actions –> Connect to Outlook.

The problem as it was explained to me is that the GUID of the List or Library is not being passed to Outlook with the Connect to [List/Document Library], so Outlook has no idea what to query.

I was refunded for my case, and while the bug was filed with the product team, since there is a workaround, it was deemed low priority.  I was told there was a chance that the bug may be fixed in Service Pack 2 for Outlook.