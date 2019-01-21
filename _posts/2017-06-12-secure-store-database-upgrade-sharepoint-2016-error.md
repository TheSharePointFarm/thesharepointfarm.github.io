---
layout: post
title: Secure Store Database Upgrade to SharePoint 2016 Error
tags: [sp2016]
---

EDIT 11/14/2017: This is resolved in the November 2017 PU for SharePoint Server 2016

When upgrading the Secure Store database to SharePoint 2016, you may encounter an error if you have applied the May 2017 CU to SharePoint 2013. This CU increments the Secure Store database schema and prevents the upgrade to SharePoint 2016. The error message you'll see in ULS will be similar to this.

```
SharePoint Foundation Upgrade SecureStoreDatabaseSequence ajyxu ERROR VERSION LOG (GET): Upgrade object too new.  Current versions: (build version = 15.0.4923.1000, schema version = 15.0.3.0). Target versions: (build version = 16.0.4405.1000, schema version = 15.0.2.0).
```

SharePoint Server 2016 is looking for a schema version of '15.0.2.0' while the schema on the database was incremented to '15.0.3.0' via the May 2017 CU for SharePoint 2013. Microsoft is aware of the issue. Likely the fix will reside on the SharePoint 2016 side via Public Update.