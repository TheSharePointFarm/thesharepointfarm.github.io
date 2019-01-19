---
layout: post
title: Announcing the SharePoint Patch Service
tags: [development,news]
---

Today I'm announcing the SharePoint Patch Service. This service allows you to quickly look up SharePoint patch information based on a SharePoint farm build number or knowledge base article. Todd Klindt has been storing this information on his website for a long time now, and I wanted to get it into database form. So, I've developed the SharePoint Patch API service.

[SharePoint Updates](https://sharepointupdates.com)

As of today, this service supports the following products:

* SharePoint Foundation 2010
* SharePoint Server 2010
* Project Server 2010
* Office Web Apps 2010
* FAST Search Server 2010
* SharePoint Foundation 2013
* SharePoint Server 2013
* Project Server 2013
* Office Web Apps 2013

Prior to the next Patch Tuesday, the remaining patches from security updates will be added for all products.

In addition to the service (which is Restful and emits JSON output), I've also developed a PowerShell cmdlet to access the service. The cmdlet is in it's first version, and will be undergoing updates to improve on functionality.

Any comments, questions, ideas , or if I've missed any patches or known community issues, please contact me via [Twitter](https://twitter.com/NaupliusTrevor).