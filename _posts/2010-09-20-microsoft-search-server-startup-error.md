---
layout: post
title: Microsoft Search Server Startup Error
tags: [sp2007, mss]
---

MSS(E) generates the following error on startup:

```text
Event Type: Error
Event Source: Office Server Search
Event Category: Gatherer
Event ID: 10036
Date: 2/12/2009
Time: 4:30:12 PM
User: N/A
Computer: SEARCH

Description:
A database error occurred.
Source: Microsoft OLE DB Provider for SQL Server
Code: 2812 occurred 1 time(s)
Description: Could not find stored procedure 'dbo.Profile_GetAliasList'.
```

This is a known issue and can be safely ignored, according to Microsoft:

https://connect.microsoft.com/feedback/ViewFeedback.aspx?FeedbackID=309900&SiteID;=480