---
layout: post
title: So you want to enter a new Product Key… (This is unsupported)
tags: [sp2010]
---

So you feel like entering a new product key.  Say you used that MSDN key in production, and now want to enter the product key.  Well, according to Central Administration –> Upgrade and Migration –> Convert License Type, you can’t!  The Product Key field is greyed out.

Of course, there is a way around this, but…

**THIS IS UNSUPPORTED.  DON’T DO IT.**

Connect to your SQL Server and your Configuration Database.  After taking a backup of your Configuration Database, run the following query:

```sql
UPDATE Objects
SET Properties = Replace(Properties, 9223372036854775807, 92233720368547758)
WHERE ClassId = '4274DBC4-7D52-474F-961B-58D84F5C28FF'
```

Now, when you go to Convert License Type, you’ll notice that your product says it is a Trial, and it gives you the opportunity to enter a Product Key.  Once the new Product Key is entered, your license will now expire one nanosecond prior to 1/1/10000.

Again, this is completely unsupported.  In fact, the only supported way to change your product key is to rebuild your farm.  So don’t do the above.