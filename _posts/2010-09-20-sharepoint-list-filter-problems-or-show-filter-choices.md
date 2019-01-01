---
layout: post
title: SharePoint List Filter Problems (or "Show Filter Choices")
tags: [sp2007]
---

When a SharePoint List gets to a certain size, around 500 items, some fields will no longer display a proper drop down for filter choices (e.g. single line of text columns).  Instead, it will display "Show Filter Choices".  Microsoft did this to prevent performance problems by showing >500 options to choose from.

If you are receiving a rendering error after selecting Show Filter Choices, be sure to install the June 2009 or newer CU.