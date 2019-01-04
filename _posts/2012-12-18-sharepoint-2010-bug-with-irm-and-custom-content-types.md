---
layout: post
title: SharePoint 2010 bug with IRM and Custom Content Types
tags: [sp2010]
---

Alexey Grachev [ran across a bug](http://sharepoint.stackexchange.com/questions/54277/sslirm-is-it-supported-for-sharepoint-2010windows-7) with SharePoint 2010, IRM, and a Custom Content Type based off of the Document Content Type.  I've been able to validate the issue using Windows 8 x64, Office 2010 x86, and HTTP.

When using a Custom Content Type with custom properties, the user clicks New Document and the document opens in Word.  Once the properties are filled out in Word, the user saves the document back to the Document Library.

If the Document Library does not have IRM enabled on it, the save succeeds and the properties are filled in the appropriate fields in the Document Library.  If the Document Library does have IRM enabled, the save succeeds, but the fields do not have values.  When the user re-opens the document, the properties are not present.  It appears that the SharePoint/IRM combination strips out these custom properties when saving the content.

So far I've reproduced this on a farm running 14.0.6126.5000 (SharePoint 2010 SP1 with August 2012 Cumulative Update).