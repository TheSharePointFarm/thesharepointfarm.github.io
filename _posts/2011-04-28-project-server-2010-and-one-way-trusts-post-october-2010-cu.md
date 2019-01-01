---
layout: post
title: Project Server 2010 and One-Way Trusts – Post October 2010 CU
tags: [sp2010, ps2010]
---

With Project Server 2010, post Oct 2010 CU in a resource domain (where as your users are in a logon domain), issues arise while attempting to use Project Web Access' "Manage Users" to add new users.  If you attempt to add a new user from the logon domain, you receive the error:

The resource could not be saved due to the following reasons:

```text
The NT account specified is invalid. Check the spelling of the user name, verify that a valid domain name was included, and check that a duplicate domain was not used.
```

This is due to Project Server not having a record of the user in the Global Catalog.  Project Server relies on the Display Name as well as the Pre-Windows 2000 logon name being present in the Global Catalog.  In a one-way trust scenario, that Global Catalog entry is not available.

Workarounds include adding the user from the logon domain to the underlying SharePoint site Project Web Access is using (using Site Settings -> Site Permissions), or preferably, using Claims Based Authentication with the logon domain.