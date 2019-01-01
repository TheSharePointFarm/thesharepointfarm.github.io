---
layout: post
title: 
tags: [sp2007, sp2010]
---

We ran into a situation where a Custom List had a Calendar view.  The Custom List also had a copy of NewForm.aspx called QuickAdd.aspx.  There was no customization done to QuickAdd.aspx.  The Supporting Files for the Custom List had been updated to set New Item form to QuickAdd.aspx.

In the Calendar view of this Custom List, clicking on any existing item within the Calendar lead to this QuickAdd.aspx?ID=n which in turn lead to NewForm.aspx?ID=n which did not display the item clicked on, but the default New Item form entry.  Obviously not desired behavior.

The solution, from Microsoft?  Simple!

Delete QuickAdd.aspx.

The self-healing nature of SharePoint allows it to correct the Supporting Files.  Once QuickAdd.aspx was deleted, the Custom List Calendar view correctly registered DispForm.aspx as the Display Form.

Note, for some reason, this did not affect any of the other existing views on the list.  Clicking on an existing item took you to DispForm.aspx?ID=n.