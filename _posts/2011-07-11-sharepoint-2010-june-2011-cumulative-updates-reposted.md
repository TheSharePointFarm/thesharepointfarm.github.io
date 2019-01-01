---
layout: post
title: SharePoint 2010 June 2011 Cumulative Updates Reposted
tags: [sp2010]
---

Fixes for those with .NET 4.0 installed as well as provisioning/starting certain Service Applications.

**Known issue 1**
If you install an earlier build of this hotfix package on a SharePoint server that has .NET 4.0 installed, you cannot synchronize user profiles from AD and LDAP into the SharePoint User Profile Service application database. The user profile synchronization export process fails. Additionally, the System.PlatformNotSupportedException exception shows in Event Viewer.

Note This issue is fixed in build 14.0.6106.5002 of this hotfix package.

**Known issue 2**
After you install an earlier build of this hotfix package, the following services may fail at either runtime or the provision process if you run these services by using a user account other than the farm administrator account:

* Session State Service
* Secure Store Service
* Business Data Connectivity (BDC) Service

**Note** This issue is fixed in build 14.0.6106.5002 of this hotfix package.