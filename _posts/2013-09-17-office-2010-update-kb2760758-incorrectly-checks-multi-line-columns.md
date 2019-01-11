---
layout: post
title: Office 2010 Update KB2760758 Incorrectly Checks Multi-Line Columns
tags: [sp2010,sp2013]
---

EDIT: 9/19 - this is a known issue and will be resolved in the Office 2010 client December 2013 updates.

EDIT2: 12/13 - the fix did not make it into the December 2013 update and the product group currently has no ETA on when this fix will arrive.

EDIT3: 1/17 - Hotfixes are available below, or you can wait until the February 2014 Cumulative Update.

If you have a custom content type with a multi-line site column added to it, and the client has Word 2010 SP1 or SP2 with [KB2760758](http://support.microsoft.com/kb/2760758) installed, attempting to save the document will yield an error:

![KB2760758-OfficeUpdate](/assets/images/2013/09/KB2760758-OfficeUpdate.png)


If you only enter a single line of text in the multi-line document property, the save will succeed.  Uninstallation of KB2760758 should also work.  To validate which version of MSO.DLL is installed, look at C:\Program Files (x86)\Common Files\Microsoft Shared\OFFICE14\MSO.DLL.  If the file version is 14.0.7106.5001, the issue is present.  Earlier versions, such as the SP2 MSO.DLL, 14.0.7015.1000, do not have this issue.

----

KB Article Number (s) : 2878218

Platform: [x64](http://hotfixv4.microsoft.com/Microsoft%20Office%202010/sp2/mso2010kb2878218fullfilex64glb/14.0.7113.5006/free/471983_intl_x64_zip.exe)

Platform: [i386](http://hotfixv4.microsoft.com/Microsoft%20Office%202010/sp2/mso2010kb2878218fullfilex86glb/14.0.7113.5006/free/471984_intl_i386_zip.exe)
