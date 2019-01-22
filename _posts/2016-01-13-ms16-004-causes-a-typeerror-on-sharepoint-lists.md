---
layout: post
title: MS16-004 Causes a TypeError on SharePoint Lists
tags: [sp2013]
---

EDIT: Microsoft has indicated that [KB3114508](https://www.microsoft.com/en-us/download/details.aspx?id=50667), released on 1/12, precludes having to install the CU in order to resolve this issue.

If [MS16-004](https://support.microsoft.com/en-us/kb/3114503) is applied to a farm, users will see a TypeError when viewing SharePoint lists:

![MS16004List](/assets/images/2016/01/MS16004List.png)

This was noted on [SharePoint StackExchange](http://sharepoint.stackexchange.com/questions/167034/sharepoint-2013-jan-2016-updates) and the [TechNet forums](https://social.technet.microsoft.com/Forums/sharepoint/en-US/14788a0e-5acd-4d1f-8de4-a9250f149fe6/error-typeerror-unable-to-get-property-replace-of-undefined-or-null-reference). The resolution is to apply the January 2016 Cumulative Update for SharePoint.