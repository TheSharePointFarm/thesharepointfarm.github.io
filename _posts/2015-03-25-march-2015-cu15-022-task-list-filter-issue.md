---
layout: post
title: March 2015 CU/MS15-022 Task List Filter Issue
tags: [sp2013]
---

After applying the March 2015 CU (and possibly MS15-022), it is no longer possible to Filter a View based on criteria in the Task list (e.g. Task Status). This appears to only impact new Task Lists created post-patch.

One workaround is to place the Task list on a Page. Edit the Web Part and check "Server Render" under Miscellaneous.

This does not appear to be a Lists.asmx Web Service issue as the filtered view correctly filters in 3rd party tools, such as Stramit Caml Viewer.

EDIT: This issue is fixed in the May 2015 Cumulative Update