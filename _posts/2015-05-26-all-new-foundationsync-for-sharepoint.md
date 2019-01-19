---
layout: post
title: All New FoundationSync for SharePoint
tags: [development,news,sp2010,sp2013]
---

A new version of FoundationSync is available for SharePoint 2010 and 2013 Foundation! Both versions have identical features, so those still on Foundation 2010 won't miss out!

FoundationSync provides functionality to update each Site Collection's User Information List with data from Active Directory.

New in this release:

* ThumbnailPhoto import for user's pictures
* Exchange 2013 import for user's pictures (higher resolution pictures available)
* Extra logging capabilities, outputting to a report Document Library in Central Administration
* Settings are defined within a new persistent object. This allows upgrades without losing your settings, and no longer depends on the PropertyBag (good for performance)
* Allows you to delete users who do not exist with Active Directory or are disabled within Active Directory from SharePoint sites
* Restrict the Web Applications and/or Site Collections the timer job executes on

[Download](https://github.com/Nauplius/FoundationSync/releases/tag/2.6) the new release and please post any [issues](https://github.com/Nauplius/FoundationSync/issues) over in the GitHub repository.