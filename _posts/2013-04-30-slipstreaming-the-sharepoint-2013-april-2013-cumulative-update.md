---
layout: post
title: Slipstreaming the SharePoint 2013 April 2013 Cumulative Update
tags: [sp2013]
---

Okay, that was a misleading title.

As we know, with the SharePoint 2010 post-June 2012 Cumulative Updates, [you can no longer slipstream them into the base media](http://thesharepointfarm.com/2013/01/slipstreaming-the-august-2012-cumulative-update-or-higher-with-sharepoint-2010-service-pack-1/) due to the post-June 2012 Cumulative Updates requiring Service Pack 1.

It appears that the SharePoint 2013 April 2013 Cumulative update will also encounter this issue due to the requirement to have the March Public Update installed for all updates moving forward.

Certain components of the April 2013 CU, for example, let's take coreserver-x-none.  It updates the product 90150000-10F7-0000-1000-0000000FF1CE, otherwise known as the Microsoft Document Lifecycle Components.  The April 2013 CU requires that this particular product be at version 15.0.4481.1005 (March PU) prior to updating.  In order to update from 15.0.4481.1005, you obviously need to first install the March PU.  However, since the msp files have the same file names between the March PU and April CU, the April CU msp will overwrite the March PU and you'll end up with the April CU finding that the farm has a version number of ​​15.0.4420.1017 (RTM).  Since the product version number does not match, the particular product is not updated from RTM to the April CU.

We see the same thing between the SharePoint 2010 June 2012 and August 2012 CUs.  Going back to the original example, the Microsoft Document Lifecycle Components in the June 2012 CU requires version 14.0.4763.1000 (RTM) or 14.0.6029.1000 (SP1).  And in the August 2012 CU, it requires just version 14.0.6029.1000 (SP1).

Some products in the SharePoint 2013 April 2013 CU only require their version be at RTM.  In the end, you'll end up with a farm that has a mix of RTM and April CU which isn't a good situation to be in.