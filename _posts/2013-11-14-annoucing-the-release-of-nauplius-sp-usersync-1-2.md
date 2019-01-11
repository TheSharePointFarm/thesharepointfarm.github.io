---
layout: post
title: Announcing the Release of Nauplius.SP.UserSync 1.2
tags: [development,news,sp2010,sp2013]
---

Nauplius.SP.UserSync is a timer job that synchronizes the User Information List with Active Directory.  It is targeted at SharePoint Foundation 2010 and 2013, where the User Profile Synchronization Services or AD Import is unavailable.  This timer job runs between midnight and 4 AM across all Web Applications and Site Collections to update user properties in the User Information List.  This solution is compatible with Classic and Claims-based (Windows Claims) Web Applications.

The single change between releases 1.1 and 1.2 is to synchronize all users, regardless if they’ve visited the Site Collection or not (e.g. sync even “inactive” users).

You can [download](https://foundationsync.codeplex.com/releases) the solution and find the [documentation](https://foundationsync.codeplex.com/documentation) at the [project site](https://foundationsync.codeplex.com/).