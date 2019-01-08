---
layout: post
title: Word Documents with Ink Do Not Display Correctly
tags: [sp2010]
---

8/14/2013: This issue is resolved in the August 2013 Cumulative Update for SharePoint 2010.

If a Word document contains [Ink](http://office.microsoft.com/en-us/word-help/about-ink-features-in-office-HP001033204.aspx) in a SharePoint 2010 farm with the December 2012 Cumulative Update or later, the document will not display correctly when downloaded.  Word will open, but the document will not display, instead showing the background pane of Word.

![Inked](/assets/images/2013/07/Inked.png)

To work around this issue, you can run:

```powershell
$web = Get-SPWeb http://webUrl
$web.ParserEnabled = $false
$web.Update()
```

This will fix the issue for any newly uploaded documents, but will not resolve the issue for any documents previously uploaded.

The June 2013 Cumulative Update was supposed to resolve this issue, but in testing, it has not (this CU was also pulled). Microsoft believes a fix will be provided in the August or October 2013 Cumulative Update.