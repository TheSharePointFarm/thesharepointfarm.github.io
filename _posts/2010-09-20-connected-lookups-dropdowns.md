---
layout: post
title: Connected Lookups (Dropdowns)
tags: [sp2007]
---

SharePoint does not have a built-in method of creating connected lookup fields (e.g. if I select a specific List Item, select another column only from that List Item).  [Connected Lookups](https://archive.codeplex.com/?p=cl) provides this ability.  You can have a parent with multiple children, and so far is the only working solution I’ve found that can have multiple children.

In this example, the Parent List has three columns:  Title, Child1, Child2.

![image3](/assets/images/2010/09/image3.png)

The Child List has four columns: Title, ParentDrop (mapping to Title), Child1 (mapping to Child1), and Child2 (mapping to Child2).  ParentDrop has a Parent Column of “(None)”, and the Broadcast flag checked.  Child1 has a Parent Column of “Title” (from our Parent List), and the Broadcast flag checked.  Child2 has a Parent Column of “Title”, and the Broadcast flag is unchecked.  Here is the example:

Step 1: Selecting a value for ParentDrop:

![image6](/assets/images/2010/09/image6.png)

Step 2: There is only one value to select for Child1 once we select our ParentDrop value:

![image9](/assets/images/2010/09/image9.png)

Step 3: Again, Child2 only has one value based on the value from Child1:

![image12](/assets/images/2010/09/image12.png)