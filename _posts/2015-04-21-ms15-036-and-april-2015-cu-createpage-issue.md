---
layout: post
title: MS15-036 and April 2015 CU CreatePage Issue
tags: [sp2013]
---

Update: The October 2015 Cumulative Update package resolves this issue.

MS15-036 ([KB2965219](https://support.microsoft.com/en-us/kb/2965219)) and the April 2015 CU include the following change:

> Assume that you type a page name, such as "Text with spaces," in the New item form in a site page library in SharePoint Server 2013 to create a new page. After you create the page, the automatically generated URL is inconsistent with the preview URL. For example, the preview URL may be displayed in a label as follows:
>
> Find it at : <%SitepagesUrl%>/Text with spaces.aspx

The behavior does not work as desired. If you put a page name with spaces, the "Find it at" field will generate %20 to replace the spaces:

![CreatePageError](/assets/images/2015/04/CreatePageError.png)

This is due to calling encodeUriComponent, which is encoding the spaces to %20.