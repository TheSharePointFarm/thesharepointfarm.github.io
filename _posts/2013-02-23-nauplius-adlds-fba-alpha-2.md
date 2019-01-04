---
layout: post
title: Nauplius.ADLDS.FBA Alpha 2
tags: [ad-lds,news,sp2010]
---

The second alpha of Nauplius.ADLDS.FBA for SharePoint 2010 brings a few improved features.  The STS Sync timer job is now tied to the Timer Service and should deploy globally to the farm.  There is improved logging as well.  In this solution, all issues surrounding setting the Membership and Role providers automatically should be resolved, as should setting the Custom Login Url automatically.

This solution assists in the provisioning of a forms based authentication configuration for Active Directory Lightweight Directory Services and Active Directory Application Mode.  Testing on multi-server farms is highly appreciated (to validate the SecurityTokenServices' web.config is properly handled), but do keep in mind that this is an alpha, so keep the testing to development and UAT farms.

Please post any issues found in the [solution's discussion forums](http://sharepointadlds.codeplex.com/discussions).

You can find installation and configuration details on the [solution's download page](http://sharepointadlds.codeplex.com/releases/view/102402).