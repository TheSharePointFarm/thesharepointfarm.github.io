---
layout: post
title: Missing User Profile Synchronization Service
tags: [sp2010]
---

I ran into an issue where I was missing the User Profile Synchronization Service from one of my SharePoint servers in the farm.  I could see all other services (e.g. User Profile Service), but running a `Get-SPServiceInstance -Server SPServer` did not display the User Profile Synchronization Service, nor did Central Administration -> Services on Server.

To resolve this particular issue, just run:

`Install-SPService`